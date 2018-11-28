Wikidata 入門筆記 A - 編輯範例
==============================

###### tags: `Wikidata`

:::warning
(2018/11/28) 這是複製出來提供預覽的版本
:::

目前 Item 主要有幾種來源
>透過了解來源可以進一步了解編寫邏輯

* 各式維基計畫條目自動生成
    * 幾乎所有的維基計畫條目生成後就會自動生成對應的維基數據頁
     ![](https://upload.wikimedia.org/wikipedia/commons/3/33/Wikidata%E6%95%99%E5%AD%B8%E7%AF%84%E4%BE%8B-3.png)
     上圖是維基文庫，大部分的維基計畫在左手邊的工具一欄皆有【維基數據項目】按鈕，可連結至該條目的維基數據項目
     ![](https://upload.wikimedia.org/wikipedia/commons/2/2e/Wikidata%E6%95%99%E5%AD%B8%E7%AF%84%E4%BE%8B-4.png)
     上圖可以看到史記的維基數據項目，維基數據與大部分的維基計畫條目都會自動產生連結，在對應的維基數據項目可以完整看到連結的列表，下方的其他網站通常是連結到該項目對應的Wikimedia Commons分類
     ![](https://upload.wikimedia.org/wikipedia/commons/4/4a/Wikidata%E6%95%99%E5%AD%B8%E7%AF%84%E4%BE%8B-5.png)
     自動生成的範圍有多大，就連維基百科消歧義頁面都會自動生成對應的維基數據項目，上圖範例中的[維基數據頁面](https://www.wikidata.org/wiki/Q13178067)，內容多由機器人自動添加。
        * 主要例外: [Wikimedia Commons](https://commons.wikimedia.org/wiki/Main_Page) 的檔案主要作為 Wikidata 引用的其中一種素材，互動的方式會顯示在 Wikimedia Commons 檔案頁面；也因為 Wikimedia Commons 的檔案是作為素材與 Wikidata 互動所以並不是所有的檔案都會被應用在 Wikidata 中，事實上被應用的檔案比例並不高 
        ![](https://upload.wikimedia.org/wikipedia/commons/1/1e/Wikidata%E6%95%99%E5%AD%B8%E7%AF%84%E4%BE%8B-1.png)
        上圖為維基共享資源與wikidata的互動方式
        ![](https://upload.wikimedia.org/wikipedia/commons/6/60/Wikidata%E6%95%99%E5%AD%B8%E7%AF%84%E4%BE%8B-2.png)
        上圖為維基共享資源如何出現在wikidata中實例
        * 部分例外: 維基文庫


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