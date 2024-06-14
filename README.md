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

### Extra Data Types
除了常見的 data types 可以用來對資料進行型態標註並轉型以外，FastAPI 也支援以下幾種：

1. `UUID`: 常用於資料庫，request 跟 response 中是 `str`
2. `datetime.datetime`: 表示時間，request 跟 response 中是符合國際標準ISO 8601 格式的 `str`，如: `2008-09-15T15:53:00+05:00`
3. `datetime.date`: 表示日期，request 跟 response 中是符合國際標準ISO 8601 格式的 `str`，如: `2008-09-15`
4. `datetime.time`: 表示時間，request 跟 response 中是符合國際標準ISO 8601 格式的 `str`，如: `15:53:00.123`
5. `datetime.timedelta`: 表示時間差，request 跟 response 中是 `float`
6. `frozenset`: 一般是 list 的形式，輸入時 FastAPI 會去掉重複的元素，輸出時轉型成 list
7. `bytes`: 二進位，request 跟 response 中是 `str`
8. `Decimal`: 代表快速且正確捨入的十進制浮點數，request 跟 response 中是 `float`

我們可以透過標注這類特殊型態來轉型 JSON 中的資料，而不是只能使用基本型態定義資料然後再自己轉型，並且這類資料再輸出的時候我們也不用自己轉換回基本型態，FastAPI 會自動幫我們轉型。

以下直接看範例：
```python
from datetime import datetime, time, timedelta
from typing import Annotated
from uuid import UUID

from fastapi import Body, FastAPI

app = FastAPI()


@app.put("/models/{model_id}")
async def read_items(
    model_id: UUID,
    create_datetime: Annotated[datetime, Body()],
    start_datetime: Annotated[datetime, Body()],
    process_after: Annotated[timedelta, Body()],
    repeat_at: Annotated[time | None, Body()] = None,
    creators: Annotated[frozenset, Body()],
    model: Annotated[bytes, Body()],
    price:  Annotated[Decimal, Body()],
):
    start_process = start_datetime + process_after
    duration = end_datetime - start_process
    return {
        "model_id": model_id,
        "start_datetime": start_datetime,
        "create_datetime": create_datetime,
        "process_after": process_after,
        "repeat_at": repeat_at,
        "start_process": start_process,
        "duration": duration,
        "creators": creators,
        "model": model.decode('utf-8'),
        "price": float(price)
    }
```

### Cookie

Day4 的 Annotated 有提到 `Cookie()`，他的用法跟 `Path()`、`Query()` 一樣，但他獲取的資料不太一樣，HTTP 中 Headers 中有 Cookie 來存放相關的 Cookies（即使多次發送和接收仍保持不變的資料），我們用 amazon.com 為例，點進去點開開發者工具之後馬上會看到 Headers 裡面有 Cookie，而且是一堆，要將這些資料取出，FastAPI 提供 `Cookie()` 讓我們可以取出並使用。

![alt text](images/image.png)

```python
from typing import Annotated

from fastapi import Cookie, FastAPI

app = FastAPI()


@app.get("/items/")
async def read_items(ads_id: Annotated[str | None, Cookie()] = None):
    return {"ads_id": ads_id}
```

### Header
Cookie 是比較特別的存在，但 HTTP 的Header 中還有一堆資訊可以使用，FastAPI 提供 `Header()` 來讓我們讀取這些資料，透過設定同名的參數名稱來讀取該 Header

```python
@app.get("/items/")
async def read_items(user_agent: Annotated[str | None, Header()] = None):
    return {"User-Agent": user_agent}
```

但因為 `-` 在 Python 中是不能用來取變數名稱的，所以 FastAPI 會自動幫你把你的變數名稱的 `-` 視為 `_`，並且因為 HTTP 的 headers 無視大小寫，所以你的變數名稱不需要跟著調整。

如果想要關閉 `-` 視為 `_` 的轉換的話，可以使用 `convert_underscore` 這個參數。

```python
@app.get("/items/")
async def read_items(
    strange_header: Annotated[str | None, Header(convert_underscores=False)] = None,
):
    return {"strange_header": strange_header}
```

至於重複的 Headers，FastAPI 可以幫你把重複的都集合在一起成一個 list。

```python
@app.get("/items/")
async def read_items(x_token: Annotated[list[str] | None, Header()] = None):
    return {"X-Token values": x_token}
```

### Response Model
前面使用 data type 都是輸入資料的轉型以及驗證，但對於輸出的資料也可以進行轉型跟驗證，直接看範例

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: list[str] = []


@app.post("/items/", response_model=Item)
async def create_item(item: Item) -> Any:
    return item

```

重複的欄位可以透過繼承來避免重複設定

```python
class BaseUser(BaseModel):
    username: str
    email: EmailStr
    full_name: str | None = None


class UserIn(BaseUser):
    password: str
```

有預設值並且沒有被設定的欄位可以忽視

```python
class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float = 10.5
    tags: list[str] = []

@app.get("/items/{item_id}", response_model=Item, response_model_exclude_unset=True)
async def read_item(item_id: str):
    return items[item_id]
```

就是想要忽略某些欄位可以使用 `response_model_exclude`，就是想要回傳某些欄位可以使用 `response_model_include`。

### Status Code

直接 hard code status code 最怕打錯以及忘記 status code 是什麼然後到處查，比較好的做法是使用 FastAPI 已經預先定義好的狀態碼。

```python
from fastapi import FastAPI, status

app = FastAPI()


@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
    return {"name": name}
```

### Form data

前面有說到使用 JSON 傳遞資料，但要傳遞 key-value 的資料，我們也可以使用 `Form()`，不過每個 key 都必須定義一個相對應的參數去接收，而無法像 JSON 那樣直接用一個 Pydantic model 來接收整包資料。看以下範例：

```python
@app.post("/login/")
async def login(username: Annotated[str, Form()], password: Annotated[str, Form()]):
    return {"username": username}
```

`Form()` 的使用方式就跟 `Query()`、`Path()` 等等都一樣。

Form 跟 JSON 兩者是不能混用的，這跟 HTTP 的規範有關，Form 使用的 `media type` 是 `application/x-www-form-urlencoded` 或 `multipart/form-data`(有 files，因為 file 需要拆分成多個部分傳遞)，而 JSON 是使用 `application/json`。

還有就是 `multipart/form-data` 的 file 會以二進位的方式傳遞，而不是純文本資料。

## Day6
前面有提到 file 會以 form-data 形式傳遞，雖然都是接受 file，但 FastAPI 提供兩種方式來讀取上傳的 file，首先是 `File()`，我們一樣用 `Annotated` 來標注資料，data type 設定為 `bytes`，用這種方式得到的資料會被完整儲存在記憶體裡面，適合小資料。

```python
@app.post("/files/")
async def create_file(file: Annotated[bytes, File()]):
    return {"file_size": len(file)}
```

另一種方式是 `UploadFile`，這種方式有更多額外的好處
1. 不需要 default value（就是前面看到 Annotated 後面放置的 `File()`）
2. 不會全部放置到記憶體，而是排存(spool)於記憶體，有一個上限，每次接受到上限之後會先將資料寫入到 Disk 之後再繼續接收資料
3. 有 metadata 提供額外的資訊
4. 有 file-like（提供 `read()`、`write()` 這類 file 常見的操作）的 `async` 介面
5. 背後有一個實際的 `SpooledTemporaryFile`，代表其他套件如果需要 file-like 物件的，你也可以直接使用無縫接軌

`UploadFile` 的特性讓他相當適合處理大資料（圖片、影片、BLOB 等等），除了不會把記憶體塞爆，也不會頻繁的（如果你的 buffer 很大）呼叫 I/O 寫入，相當有效率。

```python
@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile):
    return {"filename": file.filename}
```

那問題就來了，我幹嘛使用 File? UploadFile 不是更好嗎? 我認為使用哪個主要考慮“資料大小”跟“資料操作”，資料小用 File 直接內存讀取然後進行即時的資料操作，輕鬆又簡單，資料大的話就必須使用 UploadFile 分批寫入，要對資料進行操作就必須等資料完整儲存之後再讀取進行操作。

前面有提到 metadata，就是描述資料的資料，而這些資料都被記錄在 `UploadFile` 這個物件的屬性底下
1. `filename`: 上傳資料的原始名稱(`str`)(e.g. `image.jpg`)
2. `content-type`: 上傳資料的內容格式(`str`)(e.g. `image/jpeg`)，這個資訊會定義在 HTTP 的 Header 中
3. `file`: 一個 `SpooledTemporaryFile`

接著說明前面提到 `async` 的介面，這些行為都是基於 Python 原生的 `SpooledTemporaryFile` 的行為，但變成非同步版本的，有以下：
1. `read(size)`: 從 file 當中讀取 `size`(`int`) 的 bytes/characters
2. `write(data)`: 將 data(`str` or `bytes`) 寫入到 file 中
3. `seek(offset)`: 前往 file 中的 byte position
4. `close()`: 關閉 file（清除並關閉這個串流）

有人可能會對 read size 跟 seek offset 感到疑惑，下面舉幾個實際的場景:
- `seek`: 這在重複讀取資料很有效，因為當你 `read` 資料之後，如果有完整讀取，指標會跑到文件的最尾端，這時候我們可以 `seek(0)` 將指標移動回最開始的位置，然後呼叫 `read` 再次讀取。
- `read`: 如果你只想要接受特定類型的資料，可以不用在完整接收使用者的資料之後再判斷是否合法，可以透過讀取資料的前幾個字節(Magic Number)來查看是否符合，如：`read(8)`。[Magic Number](https://en.wikipedia.org/wiki/Magic_number_(programming)#In_files) 是透過在檔案開頭添加數個 bytes 來代表自身的檔案類型。

來看一個 `async` 的使用方式

```python
contents = await myfile.read()
```

非常簡單，就是加上 `await`，代表這邊需要“等一下”，此時這個任務會由 FastAPI 交由背景的 threadpool 去執行。

而一般的使用方式為，我們不能直接使用 `myfile`，因為他的方法是非同步的，我們要用其底層的 `SpooledTemporaryFile`，所以是 `myfile.file`

```python
contents = myfile.file.read()
```

直得留意的是 file 也是可以為空的，而且也可以添加額外的 metadata 就跟前面的 `Path()`、`Query()` 一樣

```python
@app.post("/files/")
async def create_file(file: Annotated[Union[bytes, None], File()] = None):
    if not file:
        return {"message": "No file sent"}
    else:
        return {"file_size": len(file)}
```

多個上傳資料也是可以的，將型態標註為 list 即可，metadata 使用方式則一樣

```python
@app.post("/uploadfiles/")
async def create_upload_files(files: list[UploadFile], File(description="Multiple files as UploadFile")):
    return {"filenames": [file.filename for file in files]}
```

同時接受 Form 跟 File 也是可行的（畢竟 Form 跟 File 是同一個 content-type），File 也可以多份，不過是不同參數名稱去對應

```python
@app.post("/files/")
async def create_file(
    file: Annotated[bytes, File()],
    fileb: Annotated[UploadFile, File()],
    token: Annotated[str, Form()],
):
    return {
        "file_size": len(file),
        "token": token,
        "fileb_content_type": fileb.content_type,
    }
```

而一個參數接受一個文件、一個參數接收多個文件兩者最大的差異在“如何對待這些資料”
- 一對多: 我可以假設這些資料的樣式應該一樣，所以會用同一個方式對待他們，通常用於批量處理相同資料
- 一對一: 這些資料的樣式應該不一樣，我會根據不同文件用不同方式處理，通常用於資料格式不同時

## Day7
在 FastAPI 中要回傳錯誤，就是 Status Code 404、500 這類錯誤，我們要使用 `HTTPException` 來回傳，並在其中定義錯誤的 Status Code 跟 detail，detail 的部分除了 str 之外也是可以使用 JSON 可解析的內容，相當方便！

```python
from fastapi import FastAPI, HTTPException

app = FastAPI()

items = {"foo": "The Foo Wrestlers"}


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```

這邊可以發現我們不是使用 return 去傳遞錯誤，還是透過 raise 去觸發，更有錯誤的感覺，我自己滿喜歡這個方式的，不過這個設計當然也有其他的考量，後面會提到。

除了 status code 跟 detail 以外，我們還可以添加屬於我們自己的 headers

```python
@app.get("/items-header/{item_id}")
async def read_item_header(item_id: str):
    if item_id not in items:
        raise HTTPException(
            status_code=404,
            detail="Item not found",
            headers={"X-Error": "There goes my error"},
        )
    return {"item": items[item_id]}
```

可能會有人疑惑為什麼要用 “X-” 開頭，這是一種慣例，通常自定義的 headers 大家會用 "X-" 開頭，但這種慣例已經被建議不要這麼做了([出處](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers))，取而代之，不要使用前綴且使用有意義的文字去表示會比較好。

> Custom proprietary headers can be added using the 'X-' prefix, but this convention was deprecated in June 2012, because of the inconveniences it caused when non-standard fields became standard in RFC 6648 - Deprecating the "X-" Prefix and Similar Constructs in Application Protocols

除了使用 FastAPI 提供的 `HTTPException`，我們還可以自定義例外，這邊比較複雜，稍微說明一下。
在我們的 handler 中我們可以直接“拋出”我們自定義的例外，這個例外 FastAPI 是不知道要怎麼解析的，所以我們要告訴 FastAPI 這個例外要怎麼處理，因此要使用 `exception_handler` 去註冊這個例外後續的處理動作，至於回傳的內容，因為我們是自行處理，所以可以更有彈性的使用 `JSONResponse` 去回傳一般的 response，當然，你在自定義的行為裡面不想要用 `JSONResponse` 繼續使用 `HTTPException` 也可以，並沒有限制說自定義例外不能使用 `HTTPException` 。

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse


class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


app = FastAPI()


@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )


@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```

自定義的 Exception 在處理 I18N 的時候特別好用，對於後端來說就是針對特定的問題 raise 特定的 Exception，然後將這些 Exception 都註冊到 FastAPI 上，當觸發時給予特定的參數 --> 觸發特定的例外 --> 回傳特定的訊息，就是那麼流暢。

## Day8
今天來看一下怎麼把 API 文件寫得更好，讓客戶變得更加滿意！ 首先是 StatusCode，雖然我們可以直接使用 404、204 等等，但這樣其實不太好，第一是你有可能腦霧打錯，第二是如果你忘記 404 在做什麼，你還要上網去查，但 FastAPI 很貼心有提供 StatusCode 的常數給我們使用

```python
from fastapi import FastAPI, status
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
    return item
```

做到善用 StatusCode 還不夠，下一步使用者會要求說你們可不可以把 API 分一下類，這樣我們比較容易使用，當然，使用者是上帝，我們立刻來處理。對於這種需求，我們可以使用 `tag` 將相關連的標注在同一個類別。

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
    return item


@app.get("/items/", tags=["items"])
async def read_items():
    return [{"name": "Foo", "price": 42}]


@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]
```

對於外部需求我們做到的，但對於內部需求，同事會問說可不可以將有哪些類別整理一下列出來，不然要歸類時找不到，當然，當個好同事很重要，我們也可以透過 `Enum` 來整理。

```python
from enum import Enum

from fastapi import FastAPI

app = FastAPI()


class Tags(Enum):
    items = "items"
    users = "users"


@app.get("/items/", tags=[Tags.items])
async def get_items():
    return ["Portal gun", "Plumbus"]


@app.get("/users/", tags=[Tags.users])
async def read_users():
    return ["Rick", "Morty"]
```

恭喜，到這邊你將 API 分門別類整理乾淨了，但使用者依然不滿意，他們想要知道更多 API 的資訊，好吧，那我們就加上 `summary` 跟 `description` 吧，FastAPI 也都想到了，都有提供對應的功能給我們使用。

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
async def create_item(item: Item):
    return item
```

使用者是滿意了，但又換同事不滿意了，他們說 `description` 這樣寫不好看，如果文字很多的話格式會跑掉，但又不想要將文字另外儲存成變數或讀取外部文件來處理大量的描述，還好，我們不用自行處理這個問題，FastAPI 有將 docstring 轉換成  `description` 的功能，而且還支援 markdown！

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()


class Item(BaseModel):
    name: str
    description: str | None = None
    price: float
    tax: float | None = None
    tags: set[str] = set()


@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    response_description="The created item",
)
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
    return item
```

到這邊是終於告一個段落，使用者繼續變多，工程師也持續開發，但隨著時間過去，我們遇到一個問題， API 要停止支援了！ 天底下沒有不散的宴席，這很正常，有些過時的 API 勢必要淘汰，但怎麼跟使用者說呢？ 總不能每個人都發公告吧？ FastAPI 再次貼心的解決這個問題，它有提供 `deprecated` 這個功能

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "johndoe"}]


@app.get("/elements/", tags=["items"], deprecated=True)
async def read_elements():
    return [{"item_id": "Foo"}]
```

透過這個功能，使用者會在 OpenAPI 上看到這個 API 變成灰色的，意味著這個 API 已經不建議使用、不支援了，透過這個方式，我們持續的跟使用者保持良好的互動，大家繼續快樂的開發～～
