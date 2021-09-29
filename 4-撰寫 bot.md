---
breaks: true  # False means NOT to render line breaks as hard line breaks.
---

Wikidata 入門筆記 4 - 撰寫 bot
==============================

###### tags: `Wikidata`

{%hackmd pPCm34WnTpyby2g957L4Gg %}

:::info
[(點我用書本模式瀏覽)](https://hackmd.io/@jd3main/H1TpO43CX/)
:::

:::warning
施工中
:::


## 概述
這篇會簡單介紹如何用 Python 撰寫 Wikidata bot。

以下介紹兩個套件：**[Pywikibot](https://github.com/wikimedia/pywikibot)** 和 **[WikidataIntegrator](https://github.com/SuLab/WikidataIntegrator)**

#### Pywikibot
Pywikibot 是官方提供的套件，功能相當完整，可以滿足大多數的需求，尤其當你需要和其他維基網站（例如維基百科）互動時，這會是你最好的選擇。

##### 優點：
 * 官方支援
 * 方便製作跨不同維基網站的 bot
 * Lazy 取得 entity 的資料

##### 缺點：
 * 目前不支援 Lexical Data
 * 教學文件新舊混雜
 * GitHub 上只是鏡像，主要 repo 在 Gerrit，開發者要貢獻的學習成本比較高

#### WikidataIntegrator
和 Pywikibot 不同，WikidataIntegrator 並不是對於 MediaWiki 的完整包裝，而是強調整合透過 Query Service 提供結構化的查詢能力。

##### 優點：
* 不用親自寫 SPARQL 就能做基本的結構化查詢

##### 缺點：
 * 沒有支援完整的 MediaWiki API
 * 部分功能沒有跟上新版本的 API
 * 缺乏教學文件
 * 不利於編輯已存在的 entity
     * 資料結構包裝得不好用
     * 容易不小心打亂已經存在的 claims

## pywikibot

> 本文撰寫時的版本：6.6.1

### 實用資源
* [Tutorial](https://www.wikidata.org/wiki/Wikidata:Pywikibot_-_Python_3_Tutorial)
* [API Reference](https://doc.wikimedia.org/pywikibot/master/api_ref/index.html)



### 安裝
```
pip install pywikibot
```

### 登入

例如正在撰寫名叫 `nice_bot` 的機器人
資料夾可能長這樣

```
nice_bot/
   |--  __init__.py
   |--  user-config.py
```

帳號必須在 `user-config.py` 裡給定（很遺憾的似乎不能從 API 直接傳）

`user-config.py` 的內容如下：
```python=
family = 'wikidata'
mylang = 'wikidata'
usernames['wikidata']['test'] = 'nice_bot'
```



如果要登入 www.wikidata.org 則改成：
```python=3
usernames['wikidata']['wikidata'] = 'nice_bot'
```


在 `__init__.py` 裡可以這樣寫：

```python=
from pywikibot.data.api import LoginManager

psw = input("password: ")

site = pywikibot.Site("test", "wikidata")

login_manager = LoginManager(password=psw, site=site)
success = login_manager.login()
print(f"success: {success}")
```

要注意的是 `LoginManager` 必須使用 `pywikibot.data.api` 底下的，`pywikibot.login.LoginManager` 是沒辦法直接使用的。

> 雖然 `LoginManager` 可以傳 user 參數
> 依然需要在 `user-config.py` 當中設定 `usernames` 才能使用


### 疑難排解

#### `badtoken: Invalid CSRF token`
有可能是 bot 運行太久導致 token 過期
如果必須長時間執行，可以試試 ++[這個方法](https://phabricator.wikimedia.org/T261050#6421048)++


## WikidataIntegrator
> 本文撰寫時的版本：0.9.12

### 安裝
```
pip install wikidataintegrator
```

### 文件
在我撰寫這篇時，沒找到可直接在網站上瀏覽的文件，因此提供手動 build 文件的方式

#### 步驟
1. 更新 pip
    ```
    python -m pip install -U pip
    ```
 
2. 安裝 [Sphinx](https://www.sphinx-doc.org/en/master/)

    ```
    pip install Sphinx
    ```

3. 從 [GitHub](https://github.com/SuLab/WikidataIntegrator) 下載，或找到套件安裝的路徑

4. 找到 `docs/source/conf.py`，在裡面加入這幾行

    ```python=
    def skip(app, what, name, obj, would_skip, options):
        if name == "__init__":
            return False
        return would_skip

    def setup(app):
        app.connect("autodoc-skip-member", skip)
    ```
    
    這是為了讓 `__init__()` 也被納入文件

5. 在 `docs` 資料夾開啟 termina，輸入：
    ```
    make html
    ```
    
    > 如果還有顯示缺少套件，應該大多可以直接用 pip 安裝

6. 打開 `docs/build/html/index.html` 就可以瀏覽文件了

:::warning
文件中可能有不少壞掉的地方，通常是因為 docstring 中多出或缺少空白行，如有需要可以手動修正或是找更好的文件生成軟體來處理
:::

