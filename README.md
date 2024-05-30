devcontainer
pipx
poetry
ruff
AnyIO
dependencies injection

# 30 days learning FastAPI from scrach
無聊開始的 30 天重新學習 FastAPI

## Day1
### 環境
使用 devcontainer 作為程式碼開發環境，用 poetry 進行套件管理，程式碼風格使用 ruff。

### 簡介
現代、好用、速度快，基於 Statlatte 這個 ASGI 網頁框架來作為其非同步的基底，以及 pydantic 用於資料驗證，還支援 OpenAPI 的整合，總之就是讚。

### WSGI
一般我們寫的 Django、Flask 是 Web Application，在其之上會需要 Web Server 來幫忙處理 HTTP 的請求跟傳輸、靜態資源提供等等的需求，因為 Web Application、Web Server 的選擇可以有很多樣，若沒有定義一個標準介面，那就會使得 Web Application 跟 Web Server 很耦合、很難變動，所以出現了 WSGI（Web Server Gateway Interface），去定義了 Web Application 跟 Web Server 之間的介面，雙邊透過實現這個界面來串接起來，未來就算我想要換工具，反正大家只要都符合 WSGI，我就能想換就換，非常快樂。

### ASGI
Asynchronous Server Gateway Interface，簡單來說是 WSGI 的進化版，支持非同步的場景，雙邊透過實現這個介面，讓你的服務可以非同步的運行。

### Type
簡單介紹，這邊沒有太多我不知道的新知識，除了 Annotated 我自己比較少用，但覺得未來可以多加使用

### Concurrency and async/await
說明“非同步”是什麼，簡單來說就是你的程式跑到要等待很久的地方的時候，透過非同步可以讓你的 process 不要傻傻的在那邊等待，而是繼續做別的事情，等到晚一點的時候再回頭去檢查結果。
等待很久的行為基本上都是 I/O，不是你自己程式運行，而是要等待別人完成的，像是：透過網路傳輸資料（等待網路）、讀寫文件（等待 OS）、操作資料庫（等待資料庫）。
一般我們寫的程式都是 sequential 的，也就是一行一行跑，稱為「同步」。
而上述那種跑到一半可以去做別的事情的，跳著執行的，稱為「非同步」。
接著說明 concurrent 跟 parallel，很多人會搞混的地方，我喜歡從 Rod Pile 說過的話來幫助我理解
```
Concurrency is about dealing with lots of things at once.

Parallelism is about doing lots of things at once.

Not the same, but related.

Concurrency is about structure, parallelism is about execution.

Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.

 — Rob Pike
```
簡單來說，concurrent 是透過排程，來讓多個工作看似同時進行，但實際上可能不是同時執行，舉例來說，老闆可能會給你 5 個任務要你完成，而你會將工作排優先順序，然後執行，如果一個工作卡住了，那你就先做另一個，但所有工作都有所進展，沒有哪個完全停擺。
而 parallel 則是指這些工作真的都在同時運行，繼續剛剛的例子，老闆把五個工作交給五個人，大家都拿到一個工作，各司其職，不分心的專心處理，全部工作都同時間進行著。

我喜歡他的一句總結
```
Modern versions of Python have support for "asynchronous code" using something called "coroutines", with async and await syntax.
```
一語道破這些關鍵字的概念為何。

在 FastAPI 中，async def 會被非同步執行，而 def 會交由外部的 threadpool 執行，而 async def 不是使用原生的 asyncio 而是使用 AnyIO。

## Day2
continue