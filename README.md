# 30 days learning FastAPI from zero
無聊開始的 30 天重新學習 FastAPI，這份筆記主要紀錄自己看到什麼有趣跟之前沒注意到的，簡單來說是寫給自己看的，可能不會描述的那麼清楚，閱讀時請搭配[官方文件](https://fastapi.tiangolo.com/learn/)一起看。

## Day1

### 環境
使用 [devcontainer](https://code.visualstudio.com/docs/devcontainers/containers) 作為程式碼開發環境，用 [poetry](https://python-poetry.org/) 進行套件管理，程式碼風格使用 [ruff](https://github.com/astral-sh/ruff)。

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

>Concurrency is about dealing with lots of things at once.
>
>Parallelism is about doing lots of things at once.
>
>Not the same, but related.
>
>Concurrency is about structure, parallelism is about execution.
>
>Concurrency provides a way to structure a solution to solve a problem that may (but not necessarily) be parallelizable.
>
>   — Rob Pike


簡單來說，concurrent 是透過排程，來讓多個工作看似同時進行，但實際上可能不是同時執行，舉例來說，老闆可能會給你 5 個任務要你完成，而你會將工作排優先順序，然後執行，如果一個工作卡住了，那你就先做另一個，但所有工作都有所進展，沒有哪個完全停擺。
而 parallel 則是指這些工作真的都在同時運行，繼續剛剛的例子，老闆把五個工作交給五個人，大家都拿到一個工作，各司其職，不分心的專心處理，全部工作都同時間進行著。

我喜歡他的一句總結


> Modern versions of Python have support for "asynchronous code" using something called "coroutines", with async and await syntax.

一語道破這些關鍵字的概念為何。

一些有趣的知識：
- 在 FastAPI 中，`async def` 會被非同步執行，而 `def` 會交由外部的 threadpool 執行
- `async def` 不是使用原生的 `asyncio` 而是使用 `AnyIO`。

## Day2

FastAPI 提供有趣的指令來運行服務，運行後會看到有趣的畫面說明服務背後做了哪些事，看起來是賞心悅目。
- fastapi dev: 測試用
- fastapi run: 部署用

不過這邊就不多說明。

回頭來看程式碼，相當簡單，創建一個 web app 物件，然後為這個 web app 添加 root 的 route，並定義觸發該 route 的 operation 是 get，接著回傳一個 dict。

這邊有趣的是你回傳的資料結構 FastAPI 會自行幫你轉換成 JSON 回傳，非常貼心，其餘的跟其他 Python 網頁框架差不多，所以應該很好理解。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/")
async def root():
    return {"message": "Hello World"}
```

接著來看怎麼使用 path parameters，FastAPI 也提供的用法也很簡單，我們直接看 code。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/items/{item_id}")
async def root(item_id: int):
    return {"message": "Hello World"}
```

透過在 route 上面定義對應的變數名稱，以及在函數簽章定義相同的變數名稱，使得使用者輸入 URL 的時候可以正確被解析，如：`127.0.0.1/items/toy123`，這時 `toy123` 會被解析為我們的 path parameter。

有趣的是，你在函數簽章進行 type annotation 是有意義的，FastAPI 會幫你在解析出 path parameter 後嘗試進行 data validation，看資料符不符合你定義的資料型態，如果不行的話就會出錯。

route 定義的順序是有關係的，匹配的順序是從上到下，直接看 code。

```python
@app.get("/users/me")
def read_me():
    return {"user": "Cool Boy"}

@app.get("/users/{user_id}")
def read_user(user_id: int):
    return {"user": user_id}
```

今天如果使用 `/users/me` 是可以正常使用的，因為會先匹配到上面的程式碼，但反過來的話，你永遠都只會匹配到 `/users/{user_id}`，導致你永遠不能成為一個酷男孩了。

## Day3

### Predifined Values

在接收 path parameter 的時候，除了像 ID 這種我們輸入較為隨機的情況，也有輸入的可能性我們已知的情況，對於這種固定選項的輸入，我們可以使用 `Enum`

```python
from enum import Enum

from fastapi import FastAPI

class CustomerName(Enum):
    allen = "allen"
    sunny = "sunny"
    eric = "eric"


app = FastAPI()


@app.get("/customers/{customer_name}")
async def get_model(customer_name: CustomerName):
    if customer_name is CustomerName.allen:
        return {"customer_name": customer_name, "message": "Welcome our Cool Boy!"}

    if model_name.value == "sunny":
        return {"customer_name": customer_name, "message": "Sunny is the best!"}

    return {"customer_name": customer_name, "message": "Welcome!"}
```

看程式碼可以得知繼承 `Enum` 之後我們可以直接透過使用屬性來匹配，也可以呼叫 `.value()` 來得知值為何。

但回傳的時候，我們不需要使用屬性或 `.value()` 來取得值，可以直接回傳，FastAPI 會自動幫你處理，相當方便。

這邊比較有趣的是，`customer_name is CustomerName.allen` 這一段，這代表資料傳進來並被轉換成 `Enum` 的值之後，他的記憶體空間將會直接指向 `Enum` 的屬性，而不是複製一份值，這個就要說到 Python 的內部機制稱為「String Interning」。

### String Interning

這是一種記憶體最佳化策略，Python 對於靜態的字串，也就是 Literal String 的部分，Python 高機率會將這個字串「駐留」，當未來再次遇到相同的字串的時候，會使用過去已建立的物件，而不是建立一個新的。

```python
print("a" is "a") # True
```

但對於動態生成的字串，像是字串串接、切片，Python 則較低機率會將它駐留。

```python
a = "a" + "b"
print(a is "ab") # True

b = "Cool" + "Boy"
print(b is "Cool Boy") # False

c = b[:4]
print(c is "Cool") # False
```

但有趣的是，若是 dict 的 key，Python 會自動將該字串駐留

```python
d = {"Cool Boy": 1, "Best Girl": 2}
e = "Cool Boy"

print(e is "Cool Boy")
```

當然，你也可以手動將字串駐留

```python
import sys

a = "hello"
d = sys.intern("".join(["he", "llo"]))
print(a is d)  # True
```

使用駐留對於處理大量的重複字串可以帶來顯著的效率提升，寫程式的時候不妨試試。

### Path parameters containing paths

當今天我們的 route 想要直接接收檔案的路徑的時候，雖然可以用之前學的寫法，但 OpenAPI 目前沒有支援，所以會導致我們不能善用 API documentation 進行測試跟定義，因此為了兼容 OpenAPI，我們要稍微調整一下寫法。

```python
from fastapi import FastAPI

app = FastAPI()


@app.get("/files/{file_path:path}")
async def read_file(file_path: str):
    return {"file_path": file_path}
```

透過對 file_path 這個 path parameter 進行型態標注，並標注其為 `path`，讓 FastAPI 知道我們將預期接受到完整的路徑（例：`files/a/b/c/README.md`）。

## Day4

### Query Parameters

除了接受參數以外，我們還可以接受篩選的條件

```
http://127.0.0.1:8000/items/?skip=0&limit=10
```

這邊的篩選條件就是 `skip=0` 跟 `limit=10`，至於怎麼獲取這些資料，有幾種寫法，但我喜歡比較 explicitly 的作法，可以這麼寫

```python
from typing import Annotated
from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/{item_id}")
async def read_user_item(
    item_id: str, 
    skip: Annotated[int, Query()] = 0,
    limit: Annotated[int | None, Query()] = None
):
    item = {"item_id": item_id, "needy": needy, "skip": skip, "limit": limit}
    return item

```

除了定義為 path parameter 以外的都會被視為 query paramenter，對於 query paramenter 可以透過 Annotated 去明確定義這個參數是什麼類型，第一個描述的是 data type，第二個描述的是種類。

### Request Body

當使用者傳遞給我們的 Request 包含資料，這些資料就是 Request Body，通常會出現在 `POST`、`DELETE`、`PUT` 這些 path operation，而不會出現在 `GET`。

對於使用者傳遞進來的資料我們當然不可能照單全收，而且理論上資料格式是 server-side 定義的，對於這類資料，我們可以用 data class 去描述它。

```python
from typing import Annotated

from fastapi import FastAPI, Query
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


app = FastAPI()


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, q: Annotated[str | None, Query()] = None):
    result = {"item_id": item_id, **item.dict()}
    if q:
        result.update({"q": q})
    return result
```

當資料傳遞進來時，FastAPI 會幫我們將資料嘗試用 data class 去轉換並實例化它，我們後續的操作可以直接操作這個 data class 相當方便。

這邊要留意 path parameter、query parameter、request body 的混用，如果變數名稱有定義在 route 上，那就是 path parameter，如果資料是 pydantic model，那就會視為 request body，其餘的資料則視為 query parameter，但如果你都有使用 `Annotated` 去明確的描述，那理論上你不知道這一點也還好。

### Annotated

官方文件是這麼說明的

> Special typing form to add context-specific metadata to an annotation.

總之可以為 type 添加 metadata，在 FastAPI 中，這些 metadata 除了描述之外，還可以用來驗證，以下面的例子來說，我們可以限制資料的長度

```python
from typing import Annotated

from fastapi import FastAPI, Query

app = FastAPI()


@app.get("/items/")
async def read_items(q: Annotated[str | None, Query(max_length=50)] = None):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"q": q})
    return results

```

除此之外還有很多可以設定的，長度、數值大小、title、description、examples 等等，這邊就不一一舉例。

也不只有 `Query()` 可以用，還有 `Path()`、`Body()`、`Cookie()` 可以使用，相當方便，個人建議撰寫 API 時都記得使用 `Annotated` 把所有東西都定義的明確一點。

### Multiple Parameters

前面有提到使用一個 data class 去描述傳入的資料，但其實也不限於一個，可以使用多個，假設傳入的資料有三個 key，兩個 key 都對應一個 JSON，其中一個是簡單的資料，如下：

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    },
    "importance": 5
}
```

我們可以用兩個 data class 去描述，一個 data class 去描述一個 key 所對應的資料，剩下的則直接用一個參數去接，記得要使用 `Annotated`，不然會被視為 query parameter。

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None


class User(BaseModel):
    username: str
    full_name: str | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User, importance: Annotated[int, Body()]):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

這邊有一個有趣的用法，如果你的資料是

```json
{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```

那可能會這麼設計

```python
class ItemInner(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

class Item(BaseModel):
    item: ItemInner
    
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

看起來有夠冗，但透過 `embed` 這個參數，我們可以簡化

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item, "user": user}
    return results
```

這些我們就可以把“嵌入”的資料抓出來用 data class 定義，比上面的解法好看很多。

### Fields

前面對於比較大範圍資料的定義，FastAPI 提供很多很有用的工具，而對於小範圍資料的描述，Pydantic 則提供了 `Field()` 這個函數給我們使用。

```python
from typing import Annotated

from fastapi import Body, FastAPI
from pydantic import BaseModel, Field

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = Field(
        default=None, title="The description of the item", max_length=300
    )
    price: float = Field(gt=0, description="The price must be greater than zero")
    tax: float | None = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Annotated[Item, Body(embed=True)]):
    results = {"item_id": item_id, "item": item}
    return results
```

寫法基本跟 `Path()`、`Query()` 這些 FastAPI 提供的差不多，相當直覺。

### Extra JSON Schema data in Pydantic models

前面已經寫了很多，程式碼方面是定義的清清楚楚，工程師相當快樂，也把 OpenAPI 的網址傳給別人要別人試試看，結果使用者根本不知道怎麼使用，因為沒有範例，我們來看怎麼添加範例。

先來看 pydantic model 怎麼添加，我們透過設定 `model_config` 這個屬性，並設定其 `json_schema_extra` 屬性中的 `examples` 來達成。

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

    model_config = {
        "json_schema_extra": {
            "examples": [
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ]
        }
    }
```

針對 pydantic model 的屬性我們也可以設定

```python
class Item(BaseModel):
    name: str = Field(examples=["Foo"])
    description: str | None = Field(default=None, examples=["A very nice Item"])
    price: float = Field(examples=[35.4])
    tax: float | None = Field(default=None, examples=[3.2])

```

FastAPI 提供的 `Body()`、`Path()`、`Query()` 等等也都有提供。

```python
@app.put("/items/{item_id}")
async def update_item(
    item_id: int,
    item: Annotated[
        Item,
        Body(
            examples=[
                {
                    "name": "Foo",
                    "description": "A very nice Item",
                    "price": 35.4,
                    "tax": 3.2,
                }
            ],
        ),
    ],
):
```

不過如果你要設定多個 examples，寫法必須做出一點調整來迎合 OpenAPI，openapi_examples 中的 key 是什麼不太重要，你看得懂就好，重點是裡面必須有三個 key，`summary`、`description`、`value`。

```python
@app.put("/items/{item_id}")
async def update_item(
    *,
    item_id: int,
    item: Annotated[
        Item,
        Body(
            openapi_examples={
                "normal": {
                    "summary": "A normal example",
                    "description": "A **normal** item works correctly.",
                    "value": {
                        "name": "Foo",
                        "description": "A very nice Item",
                        "price": 35.4,
                        "tax": 3.2,
                    },
                },
                "converted": {
                    "summary": "An example with converted data",
                    "description": "FastAPI can convert price `strings` to actual `numbers` automatically",
                    "value": {
                        "name": "Bar",
                        "price": "35.4",
                    },
                },
                "invalid": {
                    "summary": "Invalid data is rejected with an error",
                    "value": {
                        "name": "Baz",
                        "price": "thirty five point four",
                    },
                },
            },
        ),
    ],
):
...
```

## Day5
https://fastapi.tiangolo.com/tutorial/extra-data-types/
continue