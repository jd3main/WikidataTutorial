Wikidata 入門筆記 3 - API與工具
==============================

###### tags: `Wikidata`

:::warning
(2018/11/28) 這是複製出來提供預覽的版本
:::

* [目錄](https://hackmd.io/4zBqrwb9SEGDf7e_jUXa-g)
* [Wikidata 入門筆記 1 -  基本介紹](https://hackmd.io/beKtq8AURSq0a1jpk-ktXw)
* [Wikidata 入門筆記 2 -  Query with SPARQL](https://hackmd.io/FvtDFlNYSbSMEOEvJGr0yA)
* **Wikidata 入門筆記 3 -  API與工具**

## 工具
* [Wikidata:Tools](https://www.wikidata.org/wiki/Wikidata:Tools)  // 這裡有很多工具可以玩


## API 概觀
### 端口
MediaWiki 提供了海量的 api
所有 WIkimedia 的網站都以 `/w/api.php` 做為 API 端口

例如：
| 網站     | API 端口 |
|--------|------------------------------------|
| 中文維基 | https://zh.wikipedia.org/w/api.php |
| 英文維基 | https://en.wikipedia.org/w/api.php |
| 維基數據 | https://www.wikidata.org/w/api.php |

如果沒有傳送任何參數（例如直接用上面的 URL）
會看到 help 頁面

### 
Wikidata 仰賴 Wikibase（一個 MediaWiki 的 extension）
比對一下 Wikipedia 和 Wikidata 的 api 頁面
可以發現 Wikidata 的頁面多出了一些以 `wb` 為前綴的 api
這些就是 **W**iki**b**ase 的 api

> 想查閱中文說明可以看中文維基的 api 頁面
> 但是 wb 因為只有 wikidata 有全部啟用，
> 而 wikidata 只有英文的說明文件所以沒辦法

## 實作
### 工具安裝 - openssl

以 debian/ubuntu 為例
輸入 `sudo apt install openssl` 來安裝 openssl

> 以下我是在 WSL Ubuntu 16.04 上實驗

### 不想用 terminal？
你也可以用瀏覽器直接 GET
以下的範例只要把 Host 和 Path 接起來，貼到瀏覽器的網址列就行了
不過這樣你可能沒辦法指定 Accept 的格式

### SPARQL Query (1) - 透過 Query Service
Query 的時候你有兩種選擇
一種是用 `/w/api.php` 提供的 API
一種是用 Query Service 提供的服務
以下先介紹 Query Service 的部分

#### 
準備好要 Query 的 SPARQL code
```javascript=
SELECT ?item ?itemLabel 
WHERE 
{
  ?item wdt:P31 wd:Q146.
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
}
LIMIT 5
```
[↗ Open in Query](https://query.wikidata.org/#SELECT%20%3Fitem%20%3FitemLabel%20%0AWHERE%20%0A%7B%0A%20%20%3Fitem%20wdt%3AP31%20wd%3AQ146.%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22en%22.%20%7D%0A%7D%0ALIMIT%205)

為了方便看整個結果，Limit 調得比較小

#### 
在 Query Service 裡打開 code
![](https://i.imgur.com/z9jjl0k.png)

`/sparql?query=` 後面這段是 [Encode](https://zh.wikipedia.org/wiki/%E7%99%BE%E5%88%86%E5%8F%B7%E7%BC%96%E7%A0%81) 過的 query code
可以看到他是使用 GET 進行 query 的

> 用 POST 也行，暫不介紹

#### 
接下來要寫一個 GET 請求
HTTP GET 的請求內容大概會長這樣

```
GET <path> HTTP/1.1
Host: query.wikidata.org
Accept: application/sparql-results+json
...
```
雖然我們這裡是用 https，但是 http header 的部分還是一樣的
加密是底下 ([TLS](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)) 做的事情

第一行就是請求方法、路徑、HTTP 版本
> 版本部分通常是用 HTTP/1.1 ， HTTP/2 的支援度還不高


從第二行開始是各種欄位
格式大致長這樣：`<欄位>: <值1>,<值2>`
**Host** 是必須欄位，填入伺服器的域名
**Accept** 表示 Client 端能解讀什麼格式的內容，供 server 端參考

目前我們只會用到這兩個欄位

> 參考資料：[Wikipedia-HTTP頭欄位](https://zh.wikipedia.org/zh-tw/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5)

#### 
接著來製作 header 

```-
GET /sparql?query=SELECT%20%3Fitem%20%3FitemLabel%20%0AWHERE%20%0A%7B%0A%20%20%3Fitem%20wdt%3AP31%20wd%3AQ146.%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22en%22.%20%7D%0A%7D%0ALIMIT%2010 HTTP/1.1
Host: query.wikidata.org
Accept: application/sparql-results+json
```
基本上就是把剛才那段網址中，網域後的部分整個放到路徑
Host 這裡只需要網域，不要填 `https://` 或是後面（斜線後）的路徑
Accept 部分目前有幾種支援的格式
可以參考 [這裡](https://www.mediawiki.org/wiki/Wikidata_Query_Service/User_Manual#Supported_formats)


| Format | HTTP Header | Query parameter | Description |
|--------|-------------|-----------------|-------------|
| XML | Accept: application/sparql-results+xml | format=xml | XML result format, is returned by default. As specified in [https://www.w3.org/TR/rdf-sparql-XMLres/](https://www.w3.org/TR/rdf-sparql-XMLres/) |
| JSON | Accept: application/sparql-results+json | format=json | JSON result format, as in: [https://www.w3.org/TR/sparql11-results-json/](https://www.w3.org/TR/sparql11-results-json/) |
| TSV | Accept: text/tab-separated-values |  | As specified in [https://www.w3.org/TR/sparql11-results-csv-tsv/](https://www.w3.org/TR/sparql11-results-csv-tsv/) |
| CSV | Accept: text/csv |  | As specified in [https://www.w3.org/TR/sparql11-results-csv-tsv/](https://www.w3.org/TR/sparql11-results-csv-tsv/) |
| Binary RDF | Accept: application/x-binary-rdf-results-table |  |

#### 
最後可以來 Query 了
在的 terminal 裡輸入
```bash
openssl s_client -connect query.wikidata.org:443
```
> `:443` 是 https 用的 port

接著成功連線後輸入 header

成功的話可以看到 Reply header 和內容

json 的內容會長像這樣：
```json
{
  "head" : {
    "vars" : [ "item", "itemLabel" ]
  },
  "results" : {
    "bindings" : [ {
      "item" : {
        "type" : "uri",
        "value" : "http://www.wikidata.org/entity/Q27745011"
      },
      "itemLabel" : {
        "xml:lang" : "en",
        "type" : "literal",
        "value" : "Marble"
      }
    }, {
      "item" : {
        "type" : "uri",
        "value" : "http://www.wikidata.org/entity/Q28114532"
      },
      "itemLabel" : {
        "xml:lang" : "en",
        "type" : "literal",
        "value" : "Nala"
      }
    },
    ... (略)
    ]
  }
}
```

CSV 的內容：
```
item,itemLabel
http://www.wikidata.org/entity/Q378619,CC
http://www.wikidata.org/entity/Q498787,Muezza
http://www.wikidata.org/entity/Q677525,Orangey
http://www.wikidata.org/entity/Q851190,Mrs. Chippy
http://www.wikidata.org/entity/Q1050083,Catmando
```

### SPARQL Query (2) - 透過 api.php

// TODO

### Search Entities
例如想用關鍵字搜尋項目
[wbsearchentities](https://www.wikidata.org/w/api.php?action=help&modules=wbsearchentities) 這個 api 可以幫我們

同樣的先用 openssl 連上 www.wikidata.org:443
```-
openssl s_client -connect www.wikidata.org:443
```

傳送請求：
```-
GET /w/api.php?action=wbsearchentities&search=abc&language=en&format=json HTTP/1.1
Host: www.wikidata.org
```

這次格式的指定變成用 GET 的參數傳了
在那張 [海量 API](https://www.wikidata.org/w/api.php) 表格底下可以找到 format 支援的內容
![](https://i.imgur.com/0bUxHSb.png)


### 

<style>
.part{
    margin-left: 26px;
}
h1.part{
    text-align:center;
}
h2.part{
    margin-left: 0px;
    border: solid;
    border-left: none;
    border-right: none;
    border-top: 3px #900 solid;
    border-bottom: 1px #396 solid;
    color: black;
    text-align:center;
    background-color: #F6F6F6;
    padding: 6px;
}
h3.part{
    margin-left: 0px;
    border-top: solid 1.5px #069;
    border-bottom: solid 1px #396;
    padding: 5px 5px;
}
h4.part{
    margin-left: 15px;
    border: dotted #396;
    border-width: 1px 0px 0px 0px;
}
h5.part{
    margin-left: 20px;
}
h6.part{
    margin-left: 23px;
}
</style>