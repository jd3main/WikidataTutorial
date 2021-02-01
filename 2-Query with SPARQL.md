Wikidata 入門筆記 2 - Query with SPARQL
=============================

###### tags: `Wikidata`

{%hackmd pPCm34WnTpyby2g957L4Gg %}


## 前言

在這個單元裡，將會介紹一些 SPARQL 的語法

雖然內容很長，但你不需要一口氣讀完
就算只讀懂第一個範例，也可以玩很多有趣的 query 了
可以之後再根據需求回來找找有沒有好用的語法

### 參考連結
* [Building_a_query](https://www.wikidata.org/wiki/Wikidata:SPARQL_query_service/Building_a_query) (這邊有一些不錯的 step-by-step 教學)
* [SPARQL tutorial](https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial)
* [List of properties](https://www.wikidata.org/wiki/Wikidata:List_of_properties)

## 基本語法
### Example : House Cat with Picture
這是內建範例的其中一個拿來改的
用來查詢所有有圖片的家貓(Q146)

[\[Open in Query\]](https://query.wikidata.org/#%23Cats%2C%20with%20pictures%0A%23added%20before%202016-10%0A%0A%23defaultView%3AImageGrid%0ASELECT%20%3Fitem%20%3FitemLabel%20%3Fpic%0AWHERE%0A%7B%0A%20%20%3Fitem%20wdt%3AP31%20wd%3AQ146%20.%0A%20%20%3Fitem%20wdt%3AP18%20%3Fpic%20.%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22%20%7D%0A%7D)
```c=
#Cats, with pictures

SELECT ?item ?itemLabel ?pic
WHERE
{
  ?item wdt:P31 wd:Q146 .
  ?item wdt:P18 ?pic .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" }
}
```
|          |                 |
|----------|-----------------|
| line 1  | `#` 開頭是單行註解     |
| `SELECT` | 後面接要取出的東西 |
| `?item`, `?pic` | `?` 開頭表示是一個變數（跟php的`$`類似） |
| `?itemLabel` | 加上 Label 後綴，表示是 item 這個變數的 Label <br/> 只有在 Label Service 被開啟時會被解釋成 Label |
| `WHERE{...}`  | 裡面放條件，符合大括號內的所有條件的資料才會被取出|
| `wdt:`,`wd:`   | `wdt:` 是用來標示 Property 的前綴 <br/> `wd:` 是用來標示 Item 的前綴 <br/> 後面會詳細介紹
| line 6   | 「?item 的 P31 是 Q146 」 <br/> 如果一個項目沒有 P31 屬性或是 P31 的值不是 Q146 就會被排除掉 <br/> 注意一個敘述的結尾要加句號('.') |`wd:` 是用來標示 Item 的前綴 <br/> 在後面會詳細介紹 |
| line 7| 「?item 的 P18 是 ?pic」 <br/> 如果一個項目沒有 P18(圖片) 屬性則會被排除 <br/> 若有則該物件會被放到 item，其圖片會被放到 pic |
| line 8 | 這行會啟用 wikibase 的 label service <br/> 把有`Label`後綴的變數轉換成對應的 Label <br/> 語言選擇可以參考 [這張表格](https://phabricator.wikimedia.org/source/mediawiki/browse/master/languages/data/Names.php) |


### 變數
變數以 `?` 開頭
跟程式語言的變數一樣，名稱可以自己取
不需要事先宣告
`SELECT` 後面寫的也不算是宣告，而是指定最後要回傳哪些變數

### 前綴 (Prefix)
根據原本 RDF 的用法，Item, Property, Value 都是 [URI](https://zh.wikipedia.org/wiki/%E7%BB%9F%E4%B8%80%E8%B5%84%E6%BA%90%E6%A0%87%E5%BF%97%E7%AC%A6) 格式
一個 triple 可能寫成這樣
```javascript=
<http://www.wikidata.org/entity/Q30>  <http://www.wikidata.org/prop/direct/P36>  <http://www.wikidata.org/entity/Q61> .
```
而 Prefix 可以幫助我們縮短冗長的描述
```javascript=
wd:Q30  wdt:P36  wd:Q61 .
```
Wikidata 提供了一大堆前綴

| Prefix | URL                                          |
|--------|----------------------------------------------|
| wd:    | \<http://www.wikidata.org/entity/>           |
| wds:   | \<http://www.wikidata.org/entity/statement/> |
| wdv:   | \<http://www.wikidata.org/value/>            |
| wdt:   | \<http://www.wikidata.org/prop/direct/>      |
| wikibase: | \<http://wikiba.se/ontology#>             |
| p:     | \<http://www.wikidata.org/prop/>             |
| ps:    | \<http://www.wikidata.org/prop/statement/>   |
| pq:    | \<http://www.wikidata.org/prop/qualifier/>   |
| rdfs:  | \<http://www.w3.org/2000/01/rdf-schema#>     |
| bd:    | \<http://www.bigdata.com/rdf#>               |

> 完整的 [prefix list](https://en.wikibooks.org/wiki/SPARQL/Prefixes)

每個前綴有不同的意義
最常用的就是標示項目的 wd 和標示屬性的 wdt
用法就像前面的範例 code 一樣

### 條件的運作方式
WHERE{...} 裡的條件每句都要符合才會被取出。
可以想像成枚舉資料庫裡的所有東西，試著代進變數裡面
然後一句一句看是否符合，符合就取出，不符合就再試下一組
(當然內部實作不可能是這樣)

### Label Service

`[AUTO_LANGUAGE]` 會被自動轉換成當前 Query 頁面的語言
其他的則參考 [Mediawiki Language Code](https://phabricator.wikimedia.org/source/mediawiki/browse/master/languages/data/Names.php)

#### 中文的 Label

Mediawiki 有自己一套 language code
大致上跟 ISO 的一樣，但是中文的部分就很雜

zh 開頭的就有這些
| Language Code | Description |
|---------------|-------------|
| zh            | 中文
| **zh-hant**   | **繁體**
| zh-hans       | 簡體
| **zh-tw**     | **中文（台灣）**
| zh-cn         | 中文（中國）
| zh-hk         | 中文（香港）
| zh-mo         | 中文（澳門）
| zh-my         | 中文（馬來西亞）
| zh-sg         | 中文（新加坡）
| zh-classical  | 文言
| zh-min-nan    | 閩南語
| zh-yue        | 粵語

通常我會優先使用 zh-hant 和 zh-tw

#### 多個語言
你可以指定多個語言，並依照給定的優先順序來使用
例如： `zh-tw, zh-hant, en`
這表示使用 中文(台灣)

### Limit
在 `WHERE{ ... }` 後面 加上 `limit <number>` 就可以限制查詢的數量

例如要限制最多 100 條結果：
```
WHERE
{
    #...
    #...
} limit 100
```

### 選擇性的取值
有時候我們會希望一個屬性存在時就取出，沒有就算了
這時候可以用 `OPTIONAL{}` 把敘述包起來
例如貓咪的例子可以改成這樣
```javascript=
#Cats, with pictures

SELECT ?item ?itemLabel ?pic
WHERE
{
  ?item wdt:P31 wd:Q146 .
  OPTIONAL{ ?item wdt:P18 ?pic . }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" }
}
```

### Qualifier

#### 一般 Qualifier
有時候一個屬性有多個 Value
我們需要能透過判斷 qualifier 來篩選需要的東西
這時候就會用到 p,ps,pq
p = **p**roperty
ps = **p**roperty / **s**tatement
pq = **p**roperty / **q**ualifier

> 這裡有一篇[教學](https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial#Qualifiers) ，裡面的套色比教精美

#### Example - 蒙娜麗莎所用的材料
```javascript=
wd:Q12418 p:P186 ?statement1.       # Mona Lisa: material used: ?statement1
?statement1 ps:P186 wd:Q291034.     # value: poplar wood
?statement1 pq:P518 wd:Q861259.     # qualifier: applies to part: painting surface
```
第 1 行用 `p:` 取代 `wdt:` 
這時取出來的變數 `?statement1` 就不會是單一個 value
而是一個稱為 ***claim*** 或是 *statement node* 的結構
一個 claim 由一個 property、一個 value、和一些 qualifier 組成
(但不含references)
> 如果你很想知道這是什麼結構請參考 [DataModel](https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer)


第 2 行：「`?statement1` 的 value 就是 Q291034」
用 `ps:` 來取 value
`ps:` 後面也只能和前面 `p:` 一樣放 P186 了，不然就一定沒東西符合條件
因為經過第一行後 `?statement1` 只有可能有 P186 的 statement 的內容

第 3 行：「`?statement1` 有個 qualifier P518 ，其值為 Q291034」
用 `pq:` 來取 qualifier

上面的 code 只是示範語法，並不能 query 什麼
以下是稍微改了一下之後的幾種用法

* 查詢蒙娜麗莎的所有使用的材質 (這可以只用 wdt 做到)
```javascript=
SELECT ?mat ?matLabel
WHERE
{
  wd:Q12418 p:P186 ?statement.
  ?statement ps:P186 ?mat.

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" }
}
```
* 查詢哪些畫和蒙娜麗莎一樣，將某些材質用在了相同的地方
```javascript=
SELECT ?item ?itemLabel ?matLabel ?partLabel
WHERE
{
  wd:Q12418 p:P186 ?statement1.
  ?statement1 ps:P186 ?mat.
  ?statement1 pq:P518 ?part.
  
  ?item p:P186  ?statement2.
  ?statement2 ps:P186 ?mat.
  ?statement2 pq:P518 ?part.
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en" }
}
limit 100    # Maybe lots of result exist, so add a limit
```

## 更多語法
SPARQL 還有很多語法可以幫助簡化敘述
在排版得當的情況下也更容易閱讀其中的邏輯

#### 取出所有提到過的變數 ( * )
跟正規表達式一樣使用 `*`
```
SELECT * WHERE{ ... }
```
這樣 `{ }` 裡面提到的任何變數都會被放在查詢結果裡

#### 分號 ( ; )
用前面的貓咪例子來講
我們經常會有主詞(subject)相同的狀況
```javascript=
  ?item  wdt:P31  wd:Q146 .
  ?item  wdt:P18  ?pic .
```
這時候可以用 **`;`** 來簡化
```javascript=
  ?item  wdt:P31  wd:Q146;
         wdt:P18  ?pic .
```

> 小技巧：整段選起來按 ++shift++ + ++tab++ 有個很難用的自動對齊

#### 逗號 ( , )
有時候不只主詞，Property 也相同
例如想查詢有那些畫使用 油彩(Q296955) 和 金箔(Q929186) 做為材質(P186)
```javascript=
#defaultView:ImageGrid
SELECT ?item ?itemLabel ?pic
WHERE 
{
  ?item  wdt:P186  wd:Q296955.
  ?item  wdt:P186  wd:Q929186.
        
  OPTIONAL{ ?item  wdt:P18  ?pic. }
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```
[\[Open in Query\]](https://query.wikidata.org/#%23defaultView%3AImageGrid%0ASELECT%20%3Fitem%20%3FitemLabel%20%3Fpic%0AWHERE%20%0A%7B%0A%20%20%3Fitem%20%20wdt%3AP186%20%20wd%3AQ296955.%0A%20%20%3Fitem%20%20wdt%3AP186%20%20wd%3AQ929186.%0A%20%20%20%20%20%20%20%20%0A%20%20OPTIONAL%7B%20%3Fitem%20%20wdt%3AP18%20%20%3Fpic.%20%7D%0A%20%20%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22%5BAUTO_LANGUAGE%5D%2Cen%22.%20%7D%0A%7D)

可以簡寫成
```javascript=
#defaultView:ImageGrid
SELECT ?item ?itemLabel ?pic
WHERE 
{
  ?item  wdt:P186  wd:Q296955,wd:Q929186.
        
  OPTIONAL{ ?item  wdt:P18  ?pic. }
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
```

另一種排版方式：
```
  ?item  wdt:P186  wd:Q296955,
                   wd:Q929186.
```
#### 路徑 ( / )
```
Q1    P1  tmp1.
tmp1  P2  tmp2.
tmp2  P3  Q2.
```
可以簡化成
```
Q1  P1/P2/P3  Q2 .
```
#### 匿名變數 ( [ ] )
```
Q1  P1  T1.
T1  P2  Q2.
T1  P3  Q3.
```
可以簡化成
```
Q1  P1 [
         P2  Q2;
         P3  Q3.
       ]
```

#### 重複多次 ( * + )
`*` 可以表示一個路徑重複 **0** 次或多次
`+` 可以表示一個路徑重複 **1** 次或多次
例如
```javascript=
?item wdt:P31/wdt:P279* wd:Q735.
```
可以找到所有 藝術品(Q735) 或其子類別的 instance

> 通常 "is"，或是中文說「是...的一種」，會有幾種不同的意思
> 在 wikidata 裡面主要分成 [subclass of](https://www.wikidata.org/wiki/Property:P279) 和 [instance of](https://www.wikidata.org/wiki/Property:P31)
> subclass of 是子類別，例如恆星是天體的子類別
> instance of 是實例，例如太陽是恆星的實例

#### 或 ( | )

[原文](https://www.wikidata.org/wiki/Wikidata:SPARQL_tutorial#Property_paths)
```javascript=
SELECT ?descendant ?descendantLabel
WHERE
{
  ?descendant (wdt:P22|wdt:P25)+ wd:Q1339.
  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE]". }
}
```

這個例子是用來找 [約翰·塞巴斯蒂安·巴赫 ](https://www.wikidata.org/wiki/Q1339)的後代
藉由查詢 父母、父母的父母、父母的父母的父母 ...
只要其中一個是 [約翰·塞巴斯蒂安·巴赫](https://www.wikidata.org/wiki/Q1339)，那他就是巴赫的後代了
`(wdt:P22|wdt:P25)+` 就是一個以上的 P22 和 P25 的隨意組合


## DataModel & DataType
:::warning
施工中
:::
### Data Model
#### 連結
* [完整說明](https://www.mediawiki.org/w/index.php?title=Wikibase/DataModel)
* [簡要說明](https://www.mediawiki.org/wiki/Wikibase/DataModel/Primer)

### Data Type
#### 連結
* [說明文件](https://www.wikidata.org/wiki/Help:Data_type)
#### 列表
value 裡可以放各種不同類型的資料
這些類型我們稱之為 data type
目前有這些 data types
| data type | 簡述         | 範例 |
|-----------|-------------|---------|
| Commons media | 共享資源中的檔案 |Wikidata-logo.svg |
| Globe coordinate | 地球座標 | 23°46'N, 121°0'E |
| Item | wikidata 項目 | Q2 |
| Property | wikidata 屬性 | P31 |
| String | 字串 | [Panthera len](https://www.wikidata.org/wiki/Q140#P225) |
| Monolingual text | 多語言文字，即需要指定語言的字串 | [Roma (Italian)](https://www.wikidata.org/wiki/Q220#P1448) |
| Quantity | 純量或含單位的量值 | [73.477±0.001 yottagram](https://www.wikidata.org/wiki/Q405#P2067)  |
| Time | 時間 | [13 March 1902](https://www.wikidata.org/wiki/Q700797#P569) |
| URL | 網址 | [https://www.intel.com/content/www/us/en/homepage.html](https://www.wikidata.org/wiki/Q248#P856) |
| Mathematical expression | 數學式 | [O(n^2)](https://www.wikidata.org/wiki/Q486598#P3752) |
| External identifier | 外部id | [DoctorKoWJ](https://www.wikidata.org/wiki/Q18113714#P2013) |
| Geographic shape | 地理形狀 | [Data:Neighbourhoods/New York City.map](https://www.wikidata.org/wiki/Q11299#P3896) |
| Tabular data | 表格資料 |  |
| Lexemes | 語意屬性 |  |
| Forms | 語意屬性 |  |
| Senses | 語意屬性 |  |
#### 
以下介紹一些和 data type 有關連的東西

### Quantity 和單位轉換
在 Wikidata 裡單位轉換有點小麻煩
####
quantity 的組成大致長這樣：

$\qquad$ `amount [lower_bound] [upper_bound] unit`

amount 是主要的值

lower_bound 和 upper_bound 是可選的下界和上界
用來描述不確定範圍的值

unit 是一個網址，指向一個記載單位的 item（[這裡的其中一個](https://www.wikidata.org/wiki/Wikidata:Units)）
如果填入1，表示沒有單位（因次為1）

#### 範例
暫時放個範例和一些連結
這段範例是從 [這篇文章](https://en.wikibooks.org/wiki/SPARQL/WIKIDATA_Precision,_Units_and_Coordinates) 出來的
```javascript=
# Longest rivers in the USA
SELECT ?item ?itemLabel ?length ?unitLabel ?lowerbound ?upperbound ?precision ?length2 ?conversion ?length_in_m 
WHERE
{
  ?item          wdt:P31/wdt:P279*           wd:Q4022.    # rivers
  ?item          wdt:P17                     wd:Q30.      # country USA
  ?item          p:P2043                     ?stmnode.    # length
  ?stmnode       psv:P2043                   ?valuenode.
  ?valuenode     wikibase:quantityAmount     ?length.
  ?valuenode     wikibase:quantityUnit       ?unit.
  ?valuenode     wikibase:quantityLowerBound ?lowerbound.
  ?valuenode     wikibase:quantityUpperBound ?upperbound.
  BIND((?upperbound-?lowerbound)/2 AS ?precision).
  BIND(IF(?precision=0, ?length, (CONCAT(str(?length), "±", str(?precision)))) AS ?length2). 

  # conversion to SI unit
  ?unit          p:P2370                 ?unitstmnode.   # conversion to SI unit
  ?unitstmnode   psv:P2370               ?unitvaluenode. 
  ?unitvaluenode wikibase:quantityAmount ?conversion.
  ?unitvaluenode wikibase:quantityUnit   wd:Q11573.      # meter
  BIND(?length * ?conversion AS ?length_in_m).
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
} 
ORDER BY DESC(?length_in_m)
LIMIT 10
```

##### BIND
BIND 的基本格式： `BIND(expression AS ?variable)`
可以把 expression 的值指定給 ?variable
expression 中是可以進行 +,-,*,/ 運算的
##### IF
IF 的基本格式： `IF(condition,thenExpression,elseExpression)`
#### 延伸閱讀
https://www.wikidata.org/wiki/Wikidata:Units
https://www.mediawiki.org/w/index.php?title=Wikibase/DataModel#Quantities

## 
### 巨大範例
附上一個我實際用到的 Query

[Open in Query](https://query.wikidata.org/#SELECT%20%3Fitem%20%3FitemLabel%20%3FcreatorLabel%20%3Finception%20%3Fheight%20%3Fwidth%20%3FmaterialLabel%20%3Fimage%0AWHERE%20%7B%0A%20%20SERVICE%20wikibase%3Alabel%20%7B%20bd%3AserviceParam%20wikibase%3Alanguage%20%22zh-tw%2C%20zh-hant%2C%20zh%2C%20en%22.%20%7D%0A%0A%20%20%3Fitem%20%20wdt%3AP170%20%20wd%3AQ700797%2C%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%3Fcreator%3B%0A%20%20%20%20%20%20%20%20%20wdt%3AP18%20%20%20%3Fimage.%0A%0A%20%20OPTIONAL%20%7B%0A%20%20%20%20%3Fitem%20%20wdt%3AP186%20%20%3Fmaterial%3B%0A%20%20%20%20%20%20%20%20%20%20%20wdt%3AP571%20%20%3Finception.%0A%20%20%7D%0A%0A%20%20%23%20Height%0A%20%20%3Fitem%20%20%20%20p%3AP2048%2Fpsv%3AP2048%20%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityAmount%20%20%3Fh_value%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityUnit%20%20%20%20%3Fh_unit%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D.%0A%20%20%3Fh_unit%20%20p%3AP2370%2Fpsv%3AP2370%20%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityAmount%20%20%3Fh_conversion%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityUnit%20%20%20%20wd%3AQ11573%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D.%0A%20%20BIND%28%3Fh_value%20%2a%20%3Fh_conversion%20AS%20%3Fheight%29.%0A%0A%20%20%23%20Width%0A%20%20%3Fitem%20%20%20%20p%3AP2049%2Fpsv%3AP2049%20%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityAmount%20%20%3Fw_value%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityUnit%20%20%20%20%3Fw_unit%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D.%0A%20%20%3Fw_unit%20%20p%3AP2370%2Fpsv%3AP2370%20%20%5B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityAmount%20%20%3Fw_conversion%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20wikibase%3AquantityUnit%20%20%20%20wd%3AQ11573%3B%0A%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%5D.%0A%20%20BIND%28%3Fw_value%20%2a%20%3Fw_conversion%20AS%20%3Fwidth%29.%0A%7D)
```javascript=
SELECT ?item ?itemLabel ?creatorLabel ?inception ?height ?width ?materialLabel ?image
WHERE {
  SERVICE wikibase:label { bd:serviceParam wikibase:language "zh-tw, zh-hant, zh, en". }

  ?item  wdt:P170  wd:Q700797,
                   ?creator;
         wdt:P18   ?image.

  OPTIONAL {
    ?item  wdt:P186  ?material;
           wdt:P571  ?inception.
  }

  # Height
  ?item    p:P2048/psv:P2048  [
                                wikibase:quantityAmount  ?h_value;
                                wikibase:quantityUnit    ?h_unit;
                              ].
  ?h_unit  p:P2370/psv:P2370  [
                                wikibase:quantityAmount  ?h_conversion;
                                wikibase:quantityUnit    wd:Q11573;
                              ].
  BIND(?h_value * ?h_conversion AS ?height).

  # Width
  ?item    p:P2049/psv:P2049  [
                                wikibase:quantityAmount  ?w_value;
                                wikibase:quantityUnit    ?w_unit;
                              ].
  ?w_unit  p:P2370/psv:P2370  [
                                wikibase:quantityAmount  ?w_conversion;
                                wikibase:quantityUnit    wd:Q11573;
                              ].
  BIND(?w_value * ?w_conversion AS ?width).
}
```

| | |
|-|-|
| line 5 | 之所以要加上 `,?creator` 是因為我希望在指定作者的同時也要顯示作者名字 |

## 排序

以查詢政黨編號為例

* 依照字典序排序
```javascript=
SELECT ?item ?id
WHERE 
{
  ?item wdt:P31 wd:Q7278.
  ?item wdt:P5296 ?id.
}
ORDER BY ?id
```

* 轉成整數排序
```javascript=
SELECT ?item ?id
WHERE 
{
  ?item wdt:P31 wd:Q7278.
  ?item wdt:P5296 ?id.
}
ORDER BY xsd:integer(?id)
```