好的！FastAPI 是一个现代、高性能的 Python Web 框架，专为构建 API 设计，支持异步、自动生成 OpenAPI 文档（Swagger UI/Redoc），非常适合新手快速上手。以下是针对小白的 **FastAPI 常用功能+代码示例**，从基础到进阶逐步讲解。

### **一、环境准备**

首先需要安装 FastAPI 和 ASGI 服务器（推荐 `uvicorn`）：

bash

复制

```bash
pip install fastapi uvicorn
```

### **二、基础：第一个 FastAPI 应用**

#### 1. Hello World 示例

新建文件 `main.py`，写入以下代码：

python

运行

复制

```python
from fastapi import FastAPI

# 创建 FastAPI 应用实例
app = FastAPI()

# 定义根路由（GET 请求）
@app.get("/")  # 路由装饰器，指定路径和方法
def hello_world():
    return {"message": "Hello, FastAPI!"}  # 返回 JSON 数据（自动转成 JSON 响应）
```

#### 2. 运行服务

在终端执行：

bash

复制

```bash
uvicorn main:app --reload
```

- `main:app`：`main` 是文件名，`app` 是 FastAPI 实例名。
- `--reload`：开发模式，代码修改后自动重启服务。

#### 3. 访问测试

打开浏览器访问：

- **API 文档**：`http://127.0.0.1:8000/docs`（Swagger UI，交互式文档）
- **Redoc 文档**：`http://127.0.0.1:8000/redoc`（另一种风格的文档）
- **接口测试**：`http://127.0.0.1:8000/`（返回 `{"message":"Hello, FastAPI!"}`）

### **三、核心功能详解**

#### 1. 路由与 HTTP 方法

FastAPI 支持所有 HTTP 方法（`GET`、`POST`、`PUT`、`DELETE` 等），通过装饰器指定。

**示例：不同方法的路由**

python

运行

复制

```python
from fastapi import FastAPI

app = FastAPI()

# GET 请求（获取数据）
@app.get("/items/")  # 路径结尾加 / 表示这是一个目录（可选，但推荐）
def get_items():
    return {"items": ["苹果", "香蕉", "橘子"]}

# POST 请求（提交数据）
@app.post("/items/")  # 创建新 item
def create_item(name: str, price: float):
    return {"message": f"创建成功：{name}，价格：{price}"}

# PUT 请求（更新数据）
@app.put("/items/{item_id}")  # 路径参数 item_id
def update_item(item_id: int, new_name: str):
    return {"message": f"更新 item {item_id} 为 {new_name}"}

# DELETE 请求（删除数据）
@app.delete("/items/{item_id}")
def delete_item(item_id: int):
    return {"message": f"删除 item {item_id}"}
```

#### 2. 请求参数

FastAPI 支持多种参数类型，自动校验和解析。

##### （1）路径参数（Path Parameters）

通过 `{参数名}` 在路径中定义，自动提取并校验类型。

python

运行

复制

```python
from fastapi import FastAPI

app = FastAPI()

# 路径参数 item_id（自动转为整数，若传入非整数会报错）
@app.get("/items/{item_id}")
def get_item(item_id: int):
    return {"item_id": item_id, "name": "示例商品"}
```

- 访问 `http://127.0.0.1:8000/items/123` → 返回 `{"item_id":123,"name":"示例商品"}`
- 访问 `http://127.0.0.1:8000/items/abc` → 自动返回 422 错误（类型不匹配）

##### （2）查询参数（Query Parameters）

通过 `?key=value` 传递，支持默认值、列表等。

python

运行

复制

```python
from fastapi import FastAPI
from typing import List  # 导入列表类型

app = FastAPI()

# 查询参数：skip（跳过前n条）、limit（限制返回数量），带默认值
@app.get("/items/")
def list_items(skip: int = 0, limit: int = 10, search: str = None):
    # 模拟从数据库查询数据（这里用假数据）
    items = [{"id": i, "name": f"商品{i}"} for i in range(skip, skip+limit)]
    if search:
        items = [item for item in items if search in item["name"]]
    return items
```

- 访问 `http://127.0.0.1:8000/items/?skip=5&limit=3&search=商` → 返回过滤后的商品。

##### （3）请求体（Request Body）

用于接收复杂数据（如 JSON），用 Pydantic 模型定义结构（自动校验数据）。

python

运行

复制

```python
from fastapi import FastAPI
from pydantic import BaseModel  # 导入 Pydantic 基类

app = FastAPI()

# 定义 Pydantic 模型（描述请求体的结构）
class Item(BaseModel):
    name: str  # 必填字段（字符串）
    price: float  # 必填字段（浮点数）
    description: str = None  # 可选字段（默认 None）
    tags: List[str] = []  # 可选列表字段（默认空列表）

# POST 请求接收 Item 类型的请求体
@app.post("/items/")
def create_item(item: Item):
    return item  # 直接返回接收到的数据（自动转 JSON）
```

- 用 Postman 或 Swagger UI 发送 POST 请求，请求体选 JSON：

  json

  复制

  ```json
  {
    "name": "手机",
    "price": 3999.99,
    "description": "新款智能手机",
    "tags": ["电子", "通讯"]
  }
  ```

- FastAPI 会自动校验数据（如 `price` 不是数字会报错），并转换为 `Item` 对象。

#### 3. 响应模型（Response Model）

控制接口返回的数据结构，避免暴露敏感字段。

**示例：过滤返回字段**

python

运行

复制

```python
from fastapi import FastAPI
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

# 数据库中的完整模型（假设包含敏感字段）
class UserInDB(BaseModel):
    username: str
    password: str  # 敏感字段，不想返回给前端
    email: str
    full_name: Optional[str] = None

# 对外返回的简化模型（隐藏 password）
class UserOut(BaseModel):
    username: str
    email: str
    full_name: Optional[str] = None

    class Config:
        orm_mode = True  # 支持从 ORM 对象转换（后续数据库部分会用到）

# 接口返回 UserOut 类型的数据（自动过滤掉 password）
@app.get("/users/{username}")
def get_user(username: str):
    # 模拟从数据库查询到 UserInDB 对象
    user_db = UserInDB(
        username=username,
        password="secret123",  # 敏感数据
        email=f"{username}@example.com",
        full_name="示例用户"
    )
    # 将 UserInDB 转换为 UserOut（自动排除 password）
    return user_db
```

- 访问

   

  ```
  http://127.0.0.1:8000/users/zhangsan
  ```

   

  → 返回：

  json

  复制

  ```json
  {
    "username": "zhangsan",
    "email": "zhangsan@example.com",
    "full_name": "示例用户"
  }
  ```

#### 4. 数据库集成（以 MySQL 为例）

FastAPI 通常搭配 ORM（对象关系映射）库操作数据库，推荐使用 **SQLAlchemy**（通用）或 **Tortoise ORM**（异步）。

##### （1）使用 SQLAlchemy

**步骤 1：安装依赖**

bash

复制

```bash
pip install sqlalchemy pymysql  # pymysql 是 MySQL 驱动
```

**步骤 2：配置数据库连接和模型**
新建 `database.py`：

python

运行

复制

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

# 数据库连接 URL（格式：mysql+pymysql://用户名:密码@主机:端口/数据库名）
DATABASE_URL = "mysql+pymysql://root:你的密码@localhost:3306/mydb"

# 创建引擎（连接数据库）
engine = create_engine(DATABASE_URL)

# 声明基类（用于定义模型）
Base = declarative_base()

# 定义用户模型（对应数据库表 user）
class User(Base):
    __tablename__ = "user"  # 表名
    id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(50), nullable=False)
    age = Column(Integer)
    email = Column(String(100), unique=True)

# 创建表（如果不存在）
Base.metadata.create_all(bind=engine)

# 创建会话类（用于操作数据库）
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

**步骤 3：在 FastAPI 中使用数据库**
修改 `main.py`：

python

运行

复制

```python
from fastapi import FastAPI, Depends
from database import SessionLocal, User
from pydantic import BaseModel

app = FastAPI()

# 依赖项：获取数据库会话（每次请求创建新会话，请求结束后关闭）
def get_db():
    db = SessionLocal()
    try:
        yield db  # 返回会话给接口使用
    finally:
        db.close()  # 请求结束后关闭会话

# Pydantic 模型（用于请求体和响应）
class UserCreate(BaseModel):
    name: str
    age: int
    email: str

class UserOut(BaseModel):
    id: int
    name: str
    age: int
    email: str

    class Config:
        orm_mode = True  # 允许从 ORM 对象转换

# 创建用户（POST 请求）
@app.post("/users/", response_model=UserOut)
def create_user(user: UserCreate, db=Depends(get_db)):
    # 将 Pydantic 模型转为 ORM 对象
    db_user = User(name=user.name, age=user.age, email=user.email)
    db.add(db_user)  # 添加到会话
    db.commit()  # 提交事务
    db.refresh(db_user)  # 刷新获取自增 ID
    return db_user  # 返回 ORM 对象（自动转 JSON）

# 获取所有用户（GET 请求）
@app.get("/users/", response_model=list[UserOut])
def get_users(db=Depends(get_db)):
    users = db.query(User).all()  # 查询所有用户
    return users
```

**测试**：

- 发送 POST 请求到 `/users/`，创建用户 → 返回带 ID 的用户数据。
- 发送 GET 请求到 `/users/` → 返回所有用户列表。

#### 5. 依赖注入（Dependency Injection）

FastAPI 的依赖注入系统可以复用代码（如权限校验、数据库连接）。

**示例：统一校验用户是否登录**

python

运行

复制

```python
from fastapi import FastAPI, Depends, HTTPException
from typing import Annotated  # 用于类型提示（Python 3.10+）

app = FastAPI()

# 模拟一个“当前用户”的依赖项
async def get_current_user(token: Annotated[str, "模拟的认证令牌"]):
    if token != "fake-token":  # 假设有效令牌是 "fake-token"
        raise HTTPException(status_code=401, detail="无效的令牌")
    return {"username": "zhangsan"}  # 返回当前用户信息

# 需要登录才能访问的接口
@app.get("/profile")
def get_profile(user: Annotated[dict, Depends(get_current_user)]):
    return {"message": f"欢迎 {user['username']} 查看个人资料！"}
```

- 访问 `http://127.0.0.1:8000/profile?token=fake-token` → 返回欢迎信息。
- 访问 `http://127.0.0.1:8000/profile?token=wrong` → 返回 401 错误（未授权）。

#### 6. 异步路由（Async Routes）

FastAPI 支持异步函数（`async def`），适合高并发场景（如调用外部 API、数据库异步操作）。

**示例：异步获取数据**

python

运行

复制

```python
from fastapi import FastAPI
import asyncio

app = FastAPI()

# 异步路由
@app.get("/async-example")
async def async_example():
    # 模拟异步操作（如调用外部 API）
    await asyncio.sleep(1)  # 等待 1 秒（不会阻塞其他请求）
    return {"message": "这是异步接口的响应！"}
```

#### 7. 文件上传

FastAPI 支持上传单个或多个文件。

**示例：上传单个文件**

python

运行

复制

```python
from fastapi import FastAPI, File, UploadFile
from typing import UploadFile

app = FastAPI()

@app.post("/upload-file/")
async def upload_file(file: UploadFile = File(...)):  # File(...) 表示必传
    # 保存文件到本地（示例路径）
    with open(f"uploads/{file.filename}", "wb") as f:
        content = await file.read()  # 异步读取文件内容
        f.write(content)
    return {"filename": file.filename, "size": len(content)}
```

- 用 Postman 测试：选择 `form-data`，添加一个 key 为 `file` 的文件字段。

#### 8. 错误处理

自定义错误响应，或捕获异常。

**示例：捕获 404 错误**

python

运行

复制

```python
from fastapi import FastAPI, HTTPException
from fastapi.exceptions import RequestValidationError
from starlette.responses import JSONResponse

app = FastAPI()

# 自定义 404 错误响应
@app.exception_handler(404)
async def not_found_exception_handler(request, exc):
    return JSONResponse(
        status_code=404,
        content={"message": "请求的资源不存在"}
    )

# 接口返回 404（模拟）
@app.get("/not-found")
def not_found():
    raise HTTPException(status_code=404, detail="页面不存在")
```

### **四、学习资源与下一步**

#### 1. 官方文档（必看）

FastAPI 官方文档非常详细，包含交互式示例：
https://fastapi.tiangolo.com/

#### 2. 实践项目建议

- 构建一个简单的博客 API（增删改查文章）。
- 结合 JWT 实现用户认证（使用 `python-jose` 或 `passlib` 库）。
- 集成前端（如 Vue/React）调用 FastAPI 接口。

#### 3. 常用扩展库

- `fastapi-sqlalchemy`：简化 SQLAlchemy 集成。
- `python-jose`：JWT 认证。
- `python-multipart`：文件上传（已内置，无需额外安装）。
- `typer`：构建 CLI 工具（FastAPI 作者的另一款库）。

通过以上内容，你已经掌握了 FastAPI 的核心功能！从基础路由到数据库集成，再到实用功能（文件上传、错误处理），足够应对大部分 API 开发场景。动手写代码、调试接口，结合官方文档，很快就能熟练掌握～