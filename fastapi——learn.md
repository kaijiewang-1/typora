### 线程中的同步与异步：联系与区别

#### 1. 核心概念

- •**同步（Synchronous）**：指任务按顺序执行，前一个任务完成后才会执行下一个任务。调用方会**阻塞等待**被调用方完成操作，期间调用方无法执行其他任务。例如：主线程中直接执行耗时计算（如大循环），会导致主线程阻塞（界面卡顿）。
- •**异步（Asynchronous）**：指任务发起后，调用方**不阻塞等待**，而是继续执行后续逻辑。被调用方的操作在后台完成，完成后通过回调、事件或状态通知调用方。例如：主线程发起网络请求（异步），无需等待请求完成，而是通过 `onResponse`回调处理结果。

#### 2. 区别对比

| **维度**     | **同步**                               | **异步**                                 |
| :----------- | :------------------------------------- | :--------------------------------------- |
| **执行顺序** | 任务顺序执行，依赖前序任务完成         | 任务可并行或交错执行，无严格顺序依赖     |
| **阻塞特性** | 调用方会被阻塞，直到任务完成           | 调用方不阻塞，继续执行其他逻辑           |
| **线程开销** | 通常无需额外线程（单线程内顺序执行）   | 需配合多线程、消息队列或异步IO等机制     |
| **复杂度**   | 逻辑简单，易调试                       | 需处理回调、线程同步或竞态条件，复杂度高 |
| **典型场景** | 简单计算、必须顺序执行的操作（如事务） | 耗时操作（网络、IO）、实时响应（UI更新） |

#### 3. 联系

- •**目标一致**：都是为了解决任务执行效率问题，避免资源浪费。
- •**可结合使用**：异步任务完成后，结果可能需要同步到主线程（如通过 `Handler`或 `LiveData`更新UI）。
- •**依赖关系**：异步是同步的扩展，通过“非阻塞”方式提升并发能力。

在 Python 中，`Union` 和 `Optional` 是用于类型注解的重要工具，它们来自 `typing` 模块（Python 3.5+），用于指定变量或函数返回值的可能类型。以下是详细说明和用法示例：

------

### **1. `Union` 类型**

表示变量可以是**多种类型中的一种**。
语法：`Union[Type1, Type2, ...]`
Python 3.10+ 简写：`Type1 | Type2 | ...`

#### 示例：

python

运行

复制

```python
from typing import Union

# 定义变量可以是 int 或 str
value: Union[int, str] = 42
value = "hello"  # 合法

# 函数参数可以是 float 或 list
def process(data: Union[float, list]):
    if isinstance(data, float):
        print(f"Float: {data:.2f}")
    else:
        print(f"List length: {len(data)}")

process(3.14)      # Float: 3.14
process([1, 2, 3]) # List length: 3

# Python 3.10+ 简写
def process(data: float | list): ...
```

------

### **2. `Optional` 类型**

表示变量可以是**某种类型或 `None`**。
本质是 `Union[Type, None]` 的简写。
Python 3.10+ 简写：`Type | None`

#### 示例：

python

运行

复制

```python
from typing import Optional

# 表示返回值可能是 str 或 None
def find_user(id: int) -> Optional[str]:
    users = {1: "Alice", 2: "Bob"}
    return users.get(id)  # 找不到时返回 None

result = find_user(3)  # result 类型为 str | None
print(result)          # 输出 None

# 安全使用 Optional 值
if result is not None:
    print(f"User found: {result}")
else:
    print("User not found")

# Python 3.10+ 简写
def find_user(id: int) -> str | None: ...
```

------

### **关键区别**

|     类型      |           含义           | 等价形式 |
| :-----------: | :----------------------: | :------: |
| `Union[X, Y]` |  变量可以是 `X` 或 `Y`   |    `X    |
| `Optional[X]` | 变量可以是 `X` 或 `None` |    `X    |

------

### **最佳实践**

1. **避免滥用 `Any`**：优先用 `Union`/`Optional` 替代模糊的 `Any` 类型。
2. **类型守卫**：使用 `isinstance()` 或模式匹配安全处理联合类型。
3. **Python 3.10+**：推荐使用 `|` 语法更简洁。
4. **`Optional` 的替代**：直接写 `Type | None` 更直观。

------

### **复杂示例**

python

运行

复制

```python
from typing import Union, Optional

# 联合类型 + Optional
def parse_input(input: Union[str, bytes, None]) -> Optional[int]:
    if input is None:
        return None
    if isinstance(input, str):
        return int(input) if input.isdigit() else None
    if isinstance(input, bytes):
        return int(input.decode()) if input.isdigit() else None

result = parse_input(b"42")  # 类型: int | None
```

在 FastAPI 中，参数验证主要依赖 **Pydantic 模型**和 **内置的参数声明类**（如 `Query`、`Path`、`Body`），结合类型提示实现自动验证。以下针对 **Query（查询参数）**、**Path（路径参数）** 和 **Field（请求体字段）** 三类常见参数，详细介绍其验证方法及示例：

### **一、Query 参数（URL 查询参数）**

Query 参数是 URL 中 `?` 后的键值对（如 `/items?skip=0&limit=10`），用于过滤、分页等轻量级交互。FastAPI 通过 `Query` 类声明并验证这类参数。

#### **核心方法：`Query` 类**

`Query` 用于显式声明查询参数的元数据（如默认值、是否必填、验证规则、描述等），支持 Pydantic 的验证器（如 `gt`、`le`、`min_length` 等）。

##### **基础用法**

python

运行

复制

```python
from fastapi import FastAPI, Query
from pydantic import Field

app = FastAPI()

# 声明 query 参数：搜索关键字（字符串，可选，默认空）
@app.get("/items/")
async def read_items(
    q: str = Query(None, description="搜索关键字，可选"),  # None 表示可选
):
    results = {"items": [{"item_id": "Foo"}, {"item_id": "Bar"}]}
    if q:
        results.update({"filter_keyword": q})
    return results
```

##### **验证规则（数值/字符串限制）**

通过 Pydantic 的 `Field` 或 `Query` 内置参数设置验证：

python

运行

复制

```python
from fastapi import Query
from pydantic import Field

@app.get("/items/")
async def read_items(
    # 数值范围验证：skip 必须 ≥0（gt=-1 等价于 ≥0）
    skip: int = Query(..., gt=-1, description="跳过的记录数（≥0）"),
    # 字符串长度验证：limit 必须在 1-100 之间
    limit: int = Query(10, le=100, ge=1, description="返回的记录数（1-100）"),
    # 正则表达式验证：q 必须是字母数字组合（可选）
    q: str = Query(None, regex="^[a-zA-Z0-9]+$", description="仅允许字母数字")
):
    return {
        "skip": skip,
        "limit": limit,
        "q": q
    }
```

- `...` 表示必填（否则默认可选）。
- `gt`（大于）、`ge`（大于等于）、`le`（小于等于）、`lt`（小于）用于数值范围。
- `regex` 用于字符串正则匹配。

### **二、Path 参数（URL 路径参数）**

Path 参数是 URL 路径中的动态部分（如 `/users/{user_id}`），用于标识唯一资源（如用户、商品）。FastAPI 通过 `Path` 类声明并验证这类参数。

#### **核心方法：`Path` 类**

`Path` 类似 `Query`，但专门用于路径参数，支持相同的验证规则（数值范围、字符串格式等）。

##### **基础用法**

python

运行

复制

```python
from fastapi import FastAPI, Path

app = FastAPI()

# 声明路径参数：用户 ID（整数，必填）
@app.get("/users/{user_id}")
async def read_user(
    user_id: int = Path(..., description="用户唯一 ID")
):
    return {"user_id": user_id}
```

##### **验证规则**

python

运行

复制

```python
@app.get("/items/{item_id}")
async def read_item(
    # 数值必须 > 0（gt=0）
    item_id: int = Path(..., gt=0, description="商品 ID（必须 > 0）"),
    # 字符串必须是 "new" 或 "old"（枚举验证）
    status: str = Path("new", enum=["new", "old"], description="商品状态（new/old）"),
    # 正则匹配：文件名必须以 .jpg 或 .png 结尾
    filename: str = Path(..., regex=r"^.+\.(jpg|png)$", description="文件名（jpg/png）")
):
    return {
        "item_id": item_id,
        "status": status,
        "filename": filename
    }
```

- `enum` 用于限制枚举值。
- `regex` 同样支持正则验证路径参数格式。

### **三、Field 参数（请求体字段）**

Field 通常指 **请求体（Request Body）** 中的字段（如 POST/PUT 提交的 JSON 数据），用于接收复杂结构化数据（如对象、列表）。FastAPI 支持通过 **Pydantic 模型**或直接使用 `Body`/`Field` 声明验证。

#### **方法 1：Pydantic 模型 + Field**

通过定义 Pydantic 模型类，用 `Field` 声明字段的验证规则，清晰且可复用。

python

运行

复制

```python
from pydantic import BaseModel, Field
from fastapi import FastAPI

app = FastAPI()

# 定义请求体模型
class ItemCreate(BaseModel):
    name: str = Field(..., min_length=3, max_length=50, description="商品名称（3-50字符）")
    price: float = Field(..., gt=0, description="价格（必须 > 0）")
    tags: list[str] = Field(default=[], description="标签列表（可选）")
    in_stock: bool = Field(True, description="是否库存充足（默认 True）")

# 使用模型接收请求体
@app.post("/items/")
async def create_item(item: ItemCreate):
    # FastAPI 自动验证 item 符合 ItemCreate 模型
    return {"message": "商品创建成功", "item": item}
```

#### **方法 2：直接使用 Body/Field**

对于简单场景，可直接在路径操作函数中声明参数，结合 `Body` 和 `Field` 验证。

python

运行

复制

```python
from fastapi import FastAPI, Body, Field
from pydantic import BaseModel

app = FastAPI()

@app.post("/users/")
async def create_user(
    # 直接声明请求体字段（等价于 Pydantic 模型）
    username: str = Body(..., Field(..., min_length=3, description="用户名（≥3字符）")),
    age: int = Body(..., Field(..., ge=18, description="年龄（≥18）")),
    email: str = Body(None, Field(None, regex=r"^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$", description="邮箱格式"))
):
    return {"username": username, "age": age, "email": email}
```

### **四、验证错误的统一处理**

当参数验证失败时，FastAPI 会自动返回 **422 Unprocessable Entity** 错误，并包含详细的错误信息（如哪个字段不符合规则、原因）。例如：

json

复制

```json
{
  "detail": [
    {
      "loc": [  // 错误位置（如查询参数在 "query"）
        "query",
        "limit"
      ],
      "msg": "ensure this value is greater than or equal to 1",  // 错误信息
      "type": "value_error.number.not_ge",  // 错误类型
      "ctx": {
        "limit_value": 1
      }
    }
  ]
}
```

### **总结对比**

|    参数类型     |        位置        |      核心验证工具       |            典型用途             |
| :-------------: | :----------------: | :---------------------: | :-----------------------------: |
|      Query      |     URL 问号后     |       `Query` 类        |      过滤、分页、排序参数       |
|      Path       |     URL 路径中     |        `Path` 类        | 资源标识（如用户 ID、商品 ID）  |
| Field（请求体） | 请求体（JSON/XML） | Pydantic 模型 + `Field` | 提交复杂数据（如创建/更新对象） |

通过结合 `Query`、`Path` 和 Pydantic 的 `Field`，FastAPI 实现了对各类参数的强类型验证，同时自动生成 OpenAPI 文档，兼顾开发效率与代码健壮性。