Wikidata 入門筆記 3 - API與工具
==============================

###### tags: `Wikidata`

{%hackmd pPCm34WnTpyby2g957L4Gg %}

## 關於工具
[Wikidata:Tools](https://www.wikidata.org/wiki/Wikidata:Tools) 有很多工具可以玩，不過大多年久失修

開發者可以在 GitHub 找到一些不錯的套件，例如 [這個CLI工具](https://github.com/maxlath/wikibase-cli)

我們也正在開發 [Golang 寫成的 client](https://github.com/jd3main/gowd) 以及能將 SPARQL Query 圖像化的網頁服務，如果你有興趣參與，歡迎聯絡 [Wikidata Taiwan](https://www.facebook.com/WikidataTW/?ref=br_rs) 社群

## 先備知識
因為 API 偏向開發者需要知道的
所以以下假設你知道：
* API 是什麼意思
* HTTP GET 和 POST 是什麼
* 如何在網址列傳 GET 參數

了解以下的東西可能有助於理解接下來介紹的 API，但非必要
* Wikibase Data Model [^[1]^](https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer)[^[2]^](https://www.mediawiki.org/w/index.php?title=Wikibase/DataModel)
* Wikibase Data Type [^[1]^](https://www.wikidata.org/wiki/Help:Data_type)

## 概觀
參考連結：[Wikidata:Data access](https://www.wikidata.org/wiki/Wikidata:Data_access)
####
以下我們將介紹四種方法
* Special:EntityData
* Wikibase API
* SPARQL query
* 完整 dump

#### Special:EntityData
透過 special page 提供的介面取得格式化的資料
例如 [李梅樹(Q700797)](https://www.wikidata.org/wiki/Q700797) 的 [json檔](https://www.wikidata.org/wiki/Special:EntityData/Q700797.json)

#### Wikibase API
Wikibase 是支撐 wikidata 的套件
他提供了許多 API 來存取資料

#### SPARQL query
Query service 除了前面提過的 GUI 介面以外
也提供了 API 端口

#### 完整 dump
將整個 wikidata 資料庫載下來

## Linked Data interface
你可以透過特定 URL 直接下載各種格式的資料
wikidata 提供了這個 [namespace](https://www.wikidata.org/wiki/Help:Namespaces) 作為端口：

$\quad$ https://www.wikidata.org/wiki/Special:EntityData

> [完整說明](https://www.mediawiki.org/wiki/Wikibase/EntityData)

#### HTTP GET
GET 參數：
|            | 範例  | 內容                              |
|------------|------|-----------------------------------|
| **id**     | Q2   | P,Q,L ... 等為開頭的編號            |
| **format** | json | html, json, rdf, nt, ttl, n3, php |
| **revision** |  | oldid |

例如 地球(Q2) 的 json 檔：
* https://www.wikidata.org/wiki/Special:EntityData?id=Q2&format=json

instance of (P31) 的 turtle 檔：
* https://www.wikidata.org/wiki/Special:EntityData?id=P31&format=ttl

revision 是指定版本號用的
你可以在左側的「永久連結(Permanent Link)」或是編輯紀錄裡找到指定版本的永久連結
並在網址列找到 oldid 的值

例如 學生計算機年會(Q19851407) 最早版本的永久連結是
$\qquad$ https://www.wikidata.org/w/index.php?title=Q19851407&oldid=214218139
其中 oldid 為 214218139
使用起來大概長這樣：
* https://www.wikidata.org/wiki/Special:EntityData?id=Q19851407&format=ttl&oldid=214218139

#### 再簡單一點
Q 或 L 開頭的 entity 有更簡短的作法
只要加上`/id.format` 就行了
以 [李梅樹(Q700797)](https://www.wikidata.org/wiki/Q700797) 為例：
$\qquad$ https://www.wikidata.org/wiki/Special:EntityData/Q70797.json

或是用：https://www.wikidata.org/entity/Q70797.json
它會重新導向到 Special:EntityData 那個連結

## Wikibase API

### MediaWiki API
MediaWiki 提供了海量的 API
所有 Wikimedia 的網站都以 `/w/api.php` 做為 API 端口

例如：
| 網站     | API 端口 |
|--------|------------------------------------|
| 中文維基 | https://zh.wikipedia.org/w/api.php |
| 英文維基 | https://en.wikipedia.org/w/api.php |
| 維基數據 | https://www.wikidata.org/w/api.php |

如果沒有傳送任何參數（例如直接用上面的 URL）
會看到 help 頁面

### Wikibase
Wikidata 仰賴 Wikibase（一個 MediaWiki 的 extension）
比對一下 Wikipedia 和 Wikidata 的 api 頁面
可以發現 Wikidata 的頁面多出了一些以 `wb` 為前綴的 api
這些就是 **W**iki**b**ase 的 api

> 如果想看中文說明文件
> 在 wikidata 裡調整為中文介面後再瀏覽 api 說明文件就會看到

### API 使用範例
以下以兩個實用 API 作為範例

#### wbgetentities
這個 API 能取得一個或多個 entity 的資料

* 取得 Q42 的資料：
$\qquad$ [api.php?action=wbgetentities&ids=Q42](https://www.wikidata.org/w/api.php?action=wbgetentities&ids=Q42)
* 取得 Q42 和 P17 的資料：
$\qquad$ [api.php?action=wbgetentities&ids=Q42|P17](https://www.wikidata.org/w/api.php?action=wbgetentities&ids=Q42)
* 取得 Q42 和 P17 的資料（json格式）：
$\qquad$ [api.php?action=wbgetentities&ids=Q42|P17&format=json](https://www.wikidata.org/w/api.php?action=wbgetentities&ids=Q42|P17&format=json)
#### wbsearchentities
這個 API 能用文字搜尋 entity

## 實作 - query
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

## 實作 - Search Entities

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


## 完整 dump
[Wikidata:Database download](https://www.wikidata.org/wiki/Wikidata:Database_download)
