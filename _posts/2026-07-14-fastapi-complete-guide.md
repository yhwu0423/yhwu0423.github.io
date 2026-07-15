---
layout: post
title: "FastAPI 后端开发完全指南"
subtitle: "从零开始用 Python 构建现代 Web API"
date: 2026-07-14 10:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - 笔记
  - Python
  - 后端
  - FastAPI
---

> 这篇文章面向 Python 初学者，从安装到部署，手把手教你用 FastAPI 构建一个完整的后端应用。读完后你能独立写出带数据库、用户认证的 API 项目。

## 一、FastAPI 是什么？为什么选它？

### 1.1 一句话解释

FastAPI 是一个用 Python 写的 **Web 框架**，专门用来构建 **API 接口**（也就是后端给前端提供数据的"窗口"）。

### 1.2 为什么选 FastAPI 而不是 Django / Flask？

| 对比项 | FastAPI | Flask | Django |
| --- | --- | --- | --- |
| 学习难度 | 简单 | 简单 | 较高 |
| 性能 | 非常高（异步） | 一般 | 一般 |
| 自动文档 | 自带 Swagger + ReDoc | 需要插件 | 需要插件 |
| 类型检查 | 内置（Pydantic） | 无 | 无 |
| 适合场景 | API 服务、微服务 | 小型项目 | 全栈 Web 应用 |
| 异步支持 | 原生支持 | 需要额外配置 | 部分支持 |

**FastAPI 的核心优势**：
1. **快**：性能媲美 Node.js 和 Go，Python 框架中最快之一
2. **开发效率高**：写更少的代码，自动生成 API 文档
3. **类型安全**：利用 Python 类型提示，自动验证数据、减少 bug
4. **文档自动生成**：写完代码就自动有交互式 API 文档，不需要额外维护

---

## 二、环境准备

### 2.1 安装 Python

FastAPI 需要 **Python 3.7+**（建议 3.10 以上）。

去 [python.org](https://www.python.org/downloads/) 下载安装。安装时**勾选 "Add Python to PATH"**。

验证安装：

```bash
python --version
# 输出类似：Python 3.11.5
```

### 2.2 创建项目目录

```bash
# 创建项目文件夹
mkdir my-fastapi-app
cd my-fastapi-app
```

### 2.3 创建虚拟环境（强烈推荐）

虚拟环境是什么？简单说就是给你的项目创建一个**独立的 Python 小房间**，里面装的库不会影响电脑上其他项目。

```bash
# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
venv\Scripts\activate
# macOS / Linux:
source venv/bin/activate

# 激活成功后，命令行前面会出现 (venv)
```

### 2.4 安装 FastAPI 和 Uvicorn

```bash
pip install fastapi uvicorn
```

- `fastapi`：框架本体
- `uvicorn`：ASGI 服务器，用来运行 FastAPI 应用（就像 FastAPI 的"发动机"）

---

## 三、第一个 FastAPI 应用：Hello World

### 3.1 创建 main.py

在项目目录下创建 `main.py`，写入以下代码：

```python
from fastapi import FastAPI

# 创建一个 FastAPI 应用实例
app = FastAPI()

# 定义一个 GET 接口，路径是 "/"
@app.get("/")
def read_root():
    return {"message": "Hello, World!"}
```

**逐行解释**：

- `from fastapi import FastAPI`：导入 FastAPI 类
- `app = FastAPI()`：创建应用实例，所有接口都挂在这个 `app` 上
- `@app.get("/")`：这是一个**装饰器**，意思是"当用户用 GET 方法访问 `/` 路径时，执行下面的函数"
- `return {"message": "Hello, World!"}`：返回一个 JSON 响应

### 3.2 运行应用

```bash
uvicorn main:app --reload
```

- `main`：文件名（main.py）
- `app`：代码里创建的 FastAPI 实例的变量名
- `--reload`：代码修改后自动重启（开发时用，生产环境不要加）

运行成功后，你会看到：

```
INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
```

### 3.3 访问你的 API

打开浏览器：

- **接口地址**：[http://127.0.0.1:8000](http://127.0.0.1:8000) → 看到 `{"message": "Hello, World!"}`
- **Swagger 文档**：[http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs) → 自动生成的交互式文档
- **ReDoc 文档**：[http://127.0.0.1:8000/redoc](http://127.0.0.1:8000/redoc) → 另一种风格的文档

**这就是 FastAPI 最爽的地方**：你只写了几行代码，它就自动帮你生成了完整的 API 文档，还能在文档页面直接测试接口！

---

## 四、路由与请求方法

### 4.1 什么是路由？

路由就是 **URL 路径 + 请求方法** 的组合。当用户访问某个路径时，路由决定执行哪个函数。

```python
from fastapi import FastAPI

app = FastAPI()

# GET 请求 —— 获取数据
@app.get("/users")
def get_users():
    return {"users": ["小明", "小红"]}

# POST 请求 —— 创建数据
@app.post("/users")
def create_user():
    return {"message": "用户创建成功"}

# PUT 请求 —— 更新数据
@app.put("/users/{user_id}")
def update_user(user_id: int):
    return {"message": f"用户 {user_id} 已更新"}

# DELETE 请求 —— 删除数据
@app.delete("/users/{user_id}")
def delete_user(user_id: int):
    return {"message": f"用户 {user_id} 已删除"}
```

### 4.2 路径参数

路径参数就是 URL 中的**动态部分**，用花括号 `{}` 表示：

```python
@app.get("/users/{user_id}")
def get_user(user_id: int):
    return {"user_id": user_id, "name": f"用户{user_id}"}
```

- 访问 `/users/1` → `user_id` 就是 `1`
- 访问 `/users/42` → `user_id` 就是 `42`
- 访问 `/users/abc` → **自动返回错误**！因为你声明了 `user_id: int`，FastAPI 会自动验证类型

**这就是类型提示的威力**：你只要写 `: int`，FastAPI 就自动帮你做类型检查和错误提示，不需要手动写验证逻辑。

### 4.3 查询参数

查询参数是 URL 中 `?` 后面的部分，比如 `/users?page=1&size=10`：

```python
@app.get("/users")
def get_users(page: int = 1, size: int = 10):
    return {
        "page": page,
        "size": size,
        "message": f"返回第 {page} 页，每页 {size} 条"
    }
```

- `page: int = 1`：查询参数 `page`，类型是 int，默认值是 1
- 访问 `/users` → page=1, size=10（使用默认值）
- 访问 `/users?page=2` → page=2, size=10
- 访问 `/users?page=2&size=20` → page=2, size=20

**路径参数 vs 查询参数的区别**：

```python
# 路径参数：函数参数名出现在路径的 {} 中
@app.get("/users/{user_id}")       # user_id 是路径参数
def get_user(user_id: int): ...

# 查询参数：函数参数名没有出现在路径中
@app.get("/users")                  # page 和 size 是查询参数
def get_users(page: int = 1, size: int = 10): ...

# 混合使用
@app.get("/users/{user_id}/posts")  # user_id 是路径参数，page 是查询参数
def get_user_posts(user_id: int, page: int = 1): ...
```

---

## 五、请求体与数据模型（Pydantic）

### 5.1 什么是请求体？

当前端要给后端**发送数据**（比如注册时填的用户名、密码），这些数据通常放在 **请求体（Request Body）** 中，以 JSON 格式发送。

### 5.2 用 Pydantic 定义数据模型

Pydantic 是 FastAPI 的"亲兄弟"，用来定义数据的**结构和验证规则**。

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

# 定义一个数据模型
class User(BaseModel):
    name: str           # 必填，字符串类型
    age: int            # 必填，整数类型
    email: str          # 必填，字符串类型
    is_active: bool = True  # 可选，默认值为 True

@app.post("/users")
def create_user(user: User):
    return {
        "message": f"用户 {user.name} 创建成功",
        "user": user
    }
```

当前端发送这样的 JSON 请求体时：

```json
{
    "name": "小明",
    "age": 18,
    "email": "xiaoming@example.com"
}
```

FastAPI 会**自动**：
1. 把 JSON 解析成 `User` 对象
2. 验证每个字段的类型是否正确
3. 如果类型错误，返回清晰的错误信息
4. 在 Swagger 文档中生成对应的请求示例

### 5.3 数据验证

Pydantic 可以做更精细的验证：

```python
from pydantic import BaseModel, Field, field_validator

class User(BaseModel):
    name: str = Field(min_length=2, max_length=20, description="用户名")
    age: int = Field(ge=0, le=150, description="年龄")
    email: str = Field(description="邮箱地址")
    password: str = Field(min_length=6, description="密码，至少6位")

    @field_validator("email")
    @classmethod
    def validate_email(cls, v):
        if "@" not in v:
            raise ValueError("邮箱格式不正确，必须包含 @")
        return v
```

- `Field(min_length=2)`：最少 2 个字符
- `Field(ge=0, le=150)`：大于等于 0，小于等于 150
- `@field_validator`：自定义验证逻辑

如果验证不通过，FastAPI 会自动返回 **422 错误** 和详细的错误说明，不需要你手写任何验证代码。

### 5.4 响应模型

你也可以定义**返回数据**的模型，控制返回给前端的字段（比如不返回密码）：

```python
class UserCreate(BaseModel):
    name: str
    email: str
    password: str

class UserResponse(BaseModel):
    id: int
    name: str
    email: str
    # 注意：没有 password 字段，不会返回密码

@app.post("/users", response_model=UserResponse)
def create_user(user: UserCreate):
    # 假设保存到数据库后生成了 id
    return {"id": 1, "name": user.name, "email": user.email, "password": user.password}
    # 即使 return 中包含 password，因为 response_model 没有这个字段，
    # FastAPI 也会自动过滤掉，不返回给前端
```

---

## 六、完整 CRUD 实战：待办事项 API

现在我们把学到的知识串起来，做一个完整的"待办事项"增删改查 API。

### 6.1 完整代码

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI(
    title="待办事项 API",
    description="一个简单的待办事项管理接口",
    version="1.0.0"
)

# ========== 数据模型 ==========

class TodoCreate(BaseModel):
    title: str
    description: str = ""

class TodoUpdate(BaseModel):
    title: str = None
    description: str = None
    completed: bool = None

class TodoResponse(BaseModel):
    id: int
    title: str
    description: str
    completed: bool

# ========== 模拟数据库（用列表代替） ==========

todos = []
next_id = 1

# ========== API 接口 ==========

@app.get("/todos", response_model=list[TodoResponse])
def get_all_todos():
    """获取所有待办事项"""
    return todos

@app.get("/todos/{todo_id}", response_model=TodoResponse)
def get_todo(todo_id: int):
    """根据 ID 获取单个待办事项"""
    for todo in todos:
        if todo["id"] == todo_id:
            return todo
    raise HTTPException(status_code=404, detail="待办事项不存在")

@app.post("/todos", response_model=TodoResponse, status_code=201)
def create_todo(todo: TodoCreate):
    """创建一个新的待办事项"""
    global next_id
    new_todo = {
        "id": next_id,
        "title": todo.title,
        "description": todo.description,
        "completed": False
    }
    todos.append(new_todo)
    next_id += 1
    return new_todo

@app.put("/todos/{todo_id}", response_model=TodoResponse)
def update_todo(todo_id: int, todo: TodoUpdate):
    """更新待办事项"""
    for existing_todo in todos:
        if existing_todo["id"] == todo_id:
            if todo.title is not None:
                existing_todo["title"] = todo.title
            if todo.description is not None:
                existing_todo["description"] = todo.description
            if todo.completed is not None:
                existing_todo["completed"] = todo.completed
            return existing_todo
    raise HTTPException(status_code=404, detail="待办事项不存在")

@app.delete("/todos/{todo_id}")
def delete_todo(todo_id: int):
    """删除待办事项"""
    for i, todo in enumerate(todos):
        if todo["id"] == todo_id:
            todos.pop(i)
            return {"message": f"待办事项 {todo_id} 已删除"}
    raise HTTPException(status_code=404, detail="待办事项不存在")
```

### 6.2 运行并测试

```bash
uvicorn main:app --reload
```

然后打开 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)，你会看到所有接口都列出来了，可以直接在页面上点击 **"Try it out"** 按钮测试每个接口。

### 6.3 代码解析

**`HTTPException`**：当出现错误时，用它返回 HTTP 错误响应。

```python
raise HTTPException(status_code=404, detail="待办事项不存在")
# 返回：{"detail": "待办事项不存在"}，HTTP 状态码 404
```

**`status_code=201`**：创建资源成功时，按 HTTP 规范应该返回 201 而不是 200。

**`response_model=list[TodoResponse]`**：告诉 FastAPI 返回值是一个 `TodoResponse` 列表，自动做文档和验证。

**函数下面的三引号字符串**（`"""获取所有待办事项"""`）：会显示在 Swagger 文档中作为接口描述。

---

## 七、连接真正的数据库

前面我们用 Python 列表"假装"数据库，现在来连接真正的数据库。

### 7.1 选择方案

我们使用：
- **SQLite**：轻量级数据库，不需要安装服务器，一个文件就是一个数据库（适合学习和小项目）
- **SQLAlchemy**：Python 最流行的 ORM（Object-Relational Mapping），让你用 Python 代码操作数据库，而不需要手写 SQL

```bash
pip install sqlalchemy
```

### 7.2 项目结构

把代码拆分成多个文件，结构更清晰：

```
my-fastapi-app/
├── main.py          # 主入口
├── database.py      # 数据库连接配置
├── models.py        # 数据库模型（表结构）
└── schemas.py       # Pydantic 数据模型（请求/响应格式）
```

### 7.3 database.py —— 数据库连接

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, DeclarativeBase

# 数据库地址。SQLite 的格式是 sqlite:///文件名
# 如果用 MySQL：mysql+pymysql://用户名:密码@主机:端口/数据库名
DATABASE_URL = "sqlite:///./todos.db"

# 创建数据库引擎
engine = create_engine(
    DATABASE_URL,
    connect_args={"check_same_thread": False}  # SQLite 专用配置
)

# 创建会话工厂
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

# 所有数据库模型的基类
class Base(DeclarativeBase):
    pass
```

**逐行解释**：
- `create_engine`：创建数据库连接引擎，是所有数据库操作的基础
- `SessionLocal`：每次操作数据库需要一个"会话"，这个工厂用来创建会话
- `Base`：所有数据库表对应的类都要继承它

### 7.4 models.py —— 数据库表结构

```python
from sqlalchemy import Column, Integer, String, Boolean
from database import Base

class Todo(Base):
    __tablename__ = "todos"  # 数据库中的表名

    id = Column(Integer, primary_key=True, index=True, autoincrement=True)
    title = Column(String(100), nullable=False)
    description = Column(String(500), default="")
    completed = Column(Boolean, default=False)
```

这段代码的意思是：在数据库中创建一张叫 `todos` 的表，有 4 个字段：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| id | 整数 | 主键，自动递增 |
| title | 字符串(100) | 标题，不能为空 |
| description | 字符串(500) | 描述，默认为空 |
| completed | 布尔值 | 是否完成，默认 False |

### 7.5 schemas.py —— 请求/响应模型

```python
from pydantic import BaseModel

class TodoCreate(BaseModel):
    title: str
    description: str = ""

class TodoUpdate(BaseModel):
    title: str | None = None
    description: str | None = None
    completed: bool | None = None

class TodoResponse(BaseModel):
    id: int
    title: str
    description: str
    completed: bool

    model_config = {"from_attributes": True}
```

`model_config = {"from_attributes": True}` 告诉 Pydantic：可以从 SQLAlchemy 的模型对象中读取数据（因为 SQLAlchemy 返回的是对象，不是字典）。

### 7.6 main.py —— 主文件

```python
from fastapi import FastAPI, HTTPException, Depends
from sqlalchemy.orm import Session

from database import SessionLocal, engine, Base
from models import Todo
from schemas import TodoCreate, TodoUpdate, TodoResponse

# 创建数据库表（如果不存在的话）
Base.metadata.create_all(bind=engine)

app = FastAPI(title="待办事项 API（数据库版）")

# 依赖注入：获取数据库会话
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/todos", response_model=list[TodoResponse])
def get_all_todos(db: Session = Depends(get_db)):
    return db.query(Todo).all()

@app.get("/todos/{todo_id}", response_model=TodoResponse)
def get_todo(todo_id: int, db: Session = Depends(get_db)):
    todo = db.query(Todo).filter(Todo.id == todo_id).first()
    if not todo:
        raise HTTPException(status_code=404, detail="待办事项不存在")
    return todo

@app.post("/todos", response_model=TodoResponse, status_code=201)
def create_todo(todo: TodoCreate, db: Session = Depends(get_db)):
    new_todo = Todo(title=todo.title, description=todo.description)
    db.add(new_todo)
    db.commit()
    db.refresh(new_todo)
    return new_todo

@app.put("/todos/{todo_id}", response_model=TodoResponse)
def update_todo(todo_id: int, todo: TodoUpdate, db: Session = Depends(get_db)):
    db_todo = db.query(Todo).filter(Todo.id == todo_id).first()
    if not db_todo:
        raise HTTPException(status_code=404, detail="待办事项不存在")
    if todo.title is not None:
        db_todo.title = todo.title
    if todo.description is not None:
        db_todo.description = todo.description
    if todo.completed is not None:
        db_todo.completed = todo.completed
    db.commit()
    db.refresh(db_todo)
    return db_todo

@app.delete("/todos/{todo_id}")
def delete_todo(todo_id: int, db: Session = Depends(get_db)):
    todo = db.query(Todo).filter(Todo.id == todo_id).first()
    if not todo:
        raise HTTPException(status_code=404, detail="待办事项不存在")
    db.delete(todo)
    db.commit()
    return {"message": f"待办事项 {todo_id} 已删除"}
```

### 7.7 理解依赖注入（Depends）

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

这个函数做了三件事：
1. 创建一个数据库会话 `db`
2. 通过 `yield` 把它"借"给接口函数用
3. 接口函数执行完后，`finally` 自动关闭会话

在接口函数中通过 `db: Session = Depends(get_db)` 声明"我需要一个数据库会话"，FastAPI 就会自动调用 `get_db()` 并把结果传给你。

**这就是"依赖注入"**：你不自己创建依赖的东西，而是声明"我需要什么"，框架帮你准备好。好处是代码更干净，也方便测试时替换成假的数据库。

### 7.8 数据库操作速查

```python
# 查询所有
db.query(Todo).all()

# 条件查询
db.query(Todo).filter(Todo.completed == True).all()

# 查询第一条
db.query(Todo).filter(Todo.id == 1).first()

# 新增
new_todo = Todo(title="学FastAPI")
db.add(new_todo)
db.commit()         # 提交到数据库
db.refresh(new_todo)  # 刷新，获取数据库生成的 id

# 修改（先查出来，再改属性，再 commit）
todo = db.query(Todo).filter(Todo.id == 1).first()
todo.title = "新标题"
db.commit()

# 删除
db.delete(todo)
db.commit()
```

---

## 八、异步编程（async / await）

### 8.1 为什么需要异步？

想象你在咖啡店点了咖啡。**同步**做法是你站在柜台前干等，直到咖啡做好；**异步**做法是你拿个号码牌先去找座位坐下，做好了再叫你。

在后端中，很多操作是"等待"型的：等数据库返回、等外部 API 响应、等文件读取完。异步让你在等待的时候可以去处理别的请求，大大提高并发性能。

### 8.2 FastAPI 中的异步

FastAPI 原生支持异步，只需要把 `def` 改成 `async def`：

```python
# 同步写法（也完全没问题）
@app.get("/sync")
def sync_endpoint():
    return {"message": "同步接口"}

# 异步写法
@app.get("/async")
async def async_endpoint():
    return {"message": "异步接口"}
```

### 8.3 什么时候该用 async？

| 场景 | 用哪个 | 原因 |
| --- | --- | --- |
| 调用异步数据库驱动（如 asyncpg） | `async def` | 数据库操作是 IO 密集型 |
| 调用外部 API（如 httpx.AsyncClient） | `async def` | 网络请求需要等待 |
| 使用同步数据库（如 SQLAlchemy 默认模式） | `def` | 同步操作放在 `async def` 中反而会阻塞 |
| 简单的计算或返回固定数据 | 都行 | 差别不大 |

**新手建议**：如果你用的库不支持异步（比如普通的 SQLAlchemy），就用普通的 `def`。FastAPI 会自动把它放到线程池中运行，不会阻塞其他请求。不要为了"看起来高级"而强行 async。

---

## 九、中间件与 CORS

### 9.1 什么是中间件？

中间件是在**每个请求到达接口之前**和**响应返回之后**都会执行的代码。就像机场的安检——不管你去哪个登机口，都得先过安检。

```python
import time
from fastapi import FastAPI, Request

app = FastAPI()

@app.middleware("http")
async def add_process_time(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)  # 执行实际的接口
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    print(f"{request.method} {request.url.path} - 耗时: {process_time:.4f}s")
    return response
```

这个中间件记录每个请求的处理时间，方便排查性能问题。

### 9.2 CORS（跨域资源共享）

**什么是跨域问题？** 当你的前端运行在 `http://localhost:3000`，后端在 `http://localhost:8000`，浏览器会阻止前端访问后端——这是浏览器的安全策略。

**解决方案**：在后端配置 CORS，告诉浏览器"这些来源的请求是允许的"。

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000"],  # 允许的前端地址
    allow_credentials=True,
    allow_methods=["*"],      # 允许所有 HTTP 方法
    allow_headers=["*"],      # 允许所有请求头
)
```

**开发阶段**可以设成 `allow_origins=["*"]`（允许所有来源），但**上线时一定要改成具体的域名**，否则有安全风险。

---

## 十、用户认证（JWT Token）

### 10.1 认证流程

```
用户 → 发送用户名+密码 → 后端验证 → 返回 JWT Token
用户 → 之后每次请求都带上 Token → 后端验证 Token → 返回数据
```

### 10.2 安装依赖

```bash
pip install python-jose[cryptography] passlib[bcrypt]
```

- `python-jose`：生成和验证 JWT Token
- `passlib`：密码加密（不能明文存密码！）

### 10.3 完整认证代码

```python
from datetime import datetime, timedelta
from fastapi import FastAPI, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

app = FastAPI()

# ========== 配置 ==========

SECRET_KEY = "your-secret-key-change-this-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# 密码加密工具
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

# OAuth2 方案（告诉 FastAPI 从哪里获取 Token）
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

# ========== 数据模型 ==========

class UserCreate(BaseModel):
    username: str
    password: str

class Token(BaseModel):
    access_token: str
    token_type: str

# ========== 模拟数据库 ==========

fake_users_db = {}

# ========== 工具函数 ==========

def hash_password(password: str) -> str:
    """加密密码"""
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    """验证密码"""
    return pwd_context.verify(plain_password, hashed_password)

def create_access_token(data: dict) -> str:
    """生成 JWT Token"""
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)

def get_current_user(token: str = Depends(oauth2_scheme)) -> str:
    """从 Token 中解析出当前用户"""
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise HTTPException(status_code=401, detail="无效的 Token")
        return username
    except JWTError:
        raise HTTPException(status_code=401, detail="无效的 Token")

# ========== API 接口 ==========

@app.post("/register")
def register(user: UserCreate):
    """用户注册"""
    if user.username in fake_users_db:
        raise HTTPException(status_code=400, detail="用户名已存在")
    fake_users_db[user.username] = hash_password(user.password)
    return {"message": f"用户 {user.username} 注册成功"}

@app.post("/login", response_model=Token)
def login(form_data: OAuth2PasswordRequestForm = Depends()):
    """用户登录，返回 Token"""
    # 验证用户名
    if form_data.username not in fake_users_db:
        raise HTTPException(status_code=401, detail="用户名或密码错误")
    # 验证密码
    if not verify_password(form_data.password, fake_users_db[form_data.username]):
        raise HTTPException(status_code=401, detail="用户名或密码错误")
    # 生成 Token
    access_token = create_access_token(data={"sub": form_data.username})
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/me")
def read_current_user(current_user: str = Depends(get_current_user)):
    """获取当前登录用户信息（需要 Token）"""
    return {"username": current_user, "message": "这是受保护的接口"}
```

### 10.4 测试流程

1. 注册：`POST /register`，发送 `{"username": "admin", "password": "123456"}`
2. 登录：`POST /login`，发送表单 `username=admin&password=123456`，得到 Token
3. 访问受保护接口：`GET /me`，请求头带上 `Authorization: Bearer <你的Token>`

在 Swagger 文档（`/docs`）中，右上角有个"Authorize"按钮，输入 Token 后就可以直接测试受保护的接口。

---

## 十一、文件上传

FastAPI 处理文件上传也很简单：

```bash
pip install python-multipart
```

```python
from fastapi import FastAPI, File, UploadFile

app = FastAPI()

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    # file.filename  - 原始文件名
    # file.content_type - 文件类型（如 image/png）
    # await file.read() - 读取文件内容

    contents = await file.read()
    file_size = len(contents)

    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "file_size": f"{file_size / 1024:.1f} KB"
    }

# 上传多个文件
@app.post("/upload-multiple")
async def upload_files(files: list[UploadFile] = File(...)):
    return {
        "filenames": [f.filename for f in files],
        "count": len(files)
    }
```

**实际项目中**，你通常会把文件保存到磁盘或上传到云存储（如阿里云 OSS、AWS S3）：

```python
import os

@app.post("/upload")
async def upload_file(file: UploadFile = File(...)):
    upload_dir = "uploads"
    os.makedirs(upload_dir, exist_ok=True)

    file_path = os.path.join(upload_dir, file.filename)
    with open(file_path, "wb") as f:
        content = await file.read()
        f.write(content)

    return {"filename": file.filename, "saved_to": file_path}
```

---

## 十二、错误处理

### 12.1 HTTPException

```python
from fastapi import HTTPException

@app.get("/items/{item_id}")
def read_item(item_id: int):
    if item_id < 0:
        raise HTTPException(
            status_code=400,
            detail="item_id 不能为负数"
        )
    if item_id > 100:
        raise HTTPException(
            status_code=404,
            detail="找不到该项目"
        )
    return {"item_id": item_id}
```

### 12.2 自定义异常处理器

当你想统一处理所有异常（比如统一返回格式）：

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

class AppException(Exception):
    def __init__(self, code: int, message: str):
        self.code = code
        self.message = message

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.code,
        content={
            "success": False,
            "code": exc.code,
            "message": exc.message,
            "data": None
        }
    )

@app.get("/test-error")
def test_error():
    raise AppException(code=400, message="这是一个自定义错误")
```

这样所有接口的错误响应都会有统一的格式，前端处理起来更方便。

---

## 十三、项目结构最佳实践

当项目变大时，所有代码放在一个文件里会很乱。推荐的项目结构：

```
my-fastapi-app/
├── app/
│   ├── __init__.py
│   ├── main.py              # 应用入口
│   ├── config.py             # 配置（数据库地址、密钥等）
│   ├── database.py           # 数据库连接
│   ├── models/               # 数据库模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── todo.py
│   ├── schemas/              # Pydantic 模型
│   │   ├── __init__.py
│   │   ├── user.py
│   │   └── todo.py
│   ├── routers/              # 路由（按功能分文件）
│   │   ├── __init__.py
│   │   ├── users.py
│   │   └── todos.py
│   ├── services/             # 业务逻辑
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── todo_service.py
│   └── utils/                # 工具函数
│       ├── __init__.py
│       └── auth.py
├── requirements.txt          # 依赖列表
├── .env                      # 环境变量
└── README.md
```

### 13.1 使用 APIRouter 拆分路由

**`app/routers/todos.py`**：

```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

router = APIRouter(
    prefix="/todos",     # 路径前缀，这个文件里的路由都以 /todos 开头
    tags=["待办事项"]     # Swagger 文档中的分组名
)

@router.get("/")
def get_all_todos():
    return {"todos": []}

@router.post("/")
def create_todo():
    return {"message": "创建成功"}
```

**`app/main.py`**：

```python
from fastapi import FastAPI
from app.routers import todos, users

app = FastAPI()

# 注册路由
app.include_router(todos.router)
app.include_router(users.router)
```

这样不同功能的接口放在不同文件中，代码组织更清晰，多人协作也不会冲突。

---

## 十四、部署上线

### 14.1 准备 requirements.txt

```bash
pip freeze > requirements.txt
```

这个文件记录了项目用到的所有依赖包，别人拿到你的代码后执行 `pip install -r requirements.txt` 就能安装全部依赖。

### 14.2 使用 Gunicorn + Uvicorn（Linux 服务器）

开发时用 `uvicorn main:app --reload` 就够了，但生产环境需要更稳定的方案：

```bash
pip install gunicorn

# 启动（4 个 worker 进程）
gunicorn app.main:app -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:8000
```

- `-w 4`：4 个工作进程，充分利用多核 CPU
- `-k uvicorn.workers.UvicornWorker`：使用 Uvicorn 的异步 Worker
- `-b 0.0.0.0:8000`：监听所有网络接口的 8000 端口

### 14.3 Docker 部署（推荐）

创建 `Dockerfile`：

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

构建并运行：

```bash
docker build -t my-fastapi-app .
docker run -d -p 8000:8000 my-fastapi-app
```

### 14.4 使用 Nginx 反向代理

生产环境通常在 FastAPI 前面加一层 Nginx：

```nginx
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

Nginx 负责：接收外部请求、HTTPS 证书、静态文件服务、负载均衡等。

---

## 十五、实用技巧与常见问题

### 15.1 环境变量管理

不要把密码、密钥等敏感信息写在代码里！用 `.env` 文件 + `python-dotenv`：

```bash
pip install python-dotenv
```

**`.env` 文件**：

```
DATABASE_URL=sqlite:///./todos.db
SECRET_KEY=my-super-secret-key
DEBUG=true
```

**`config.py`**：

```python
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite:///./app.db")
SECRET_KEY = os.getenv("SECRET_KEY", "default-secret")
DEBUG = os.getenv("DEBUG", "false").lower() == "true"
```

记得把 `.env` 加到 `.gitignore` 中，不要上传到 GitHub。

### 15.2 自动生成 API 文档的高级配置

```python
app = FastAPI(
    title="我的 API 项目",
    description="这是一个用 FastAPI 构建的后端服务",
    version="1.0.0",
    docs_url="/docs",       # Swagger 文档地址，设为 None 可禁用
    redoc_url="/redoc",     # ReDoc 文档地址
)
```

### 15.3 常用状态码速查

| 状态码 | 含义 | 使用场景 |
| --- | --- | --- |
| 200 | OK | 请求成功（默认） |
| 201 | Created | 资源创建成功 |
| 204 | No Content | 删除成功，无返回内容 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未登录/Token 无效 |
| 403 | Forbidden | 没有权限 |
| 404 | Not Found | 资源不存在 |
| 422 | Unprocessable Entity | 数据验证失败（FastAPI 自动返回） |
| 500 | Internal Server Error | 服务器内部错误 |

### 15.4 推荐学习资源

- **官方文档**：[fastapi.tiangolo.com](https://fastapi.tiangolo.com/) —— 写得非常好，有中文版
- **源码仓库**：[github.com/tiangolo/fastapi](https://github.com/tiangolo/fastapi)
- **Pydantic 文档**：[docs.pydantic.dev](https://docs.pydantic.dev/)
- **SQLAlchemy 文档**：[docs.sqlalchemy.org](https://docs.sqlalchemy.org/)

---

## 十六、完整依赖汇总

```bash
# 核心
pip install fastapi uvicorn

# 数据库
pip install sqlalchemy

# 认证
pip install python-jose[cryptography] passlib[bcrypt]

# 文件上传
pip install python-multipart

# 环境变量
pip install python-dotenv

# 生产部署
pip install gunicorn

# 一次性安装全部
pip install fastapi uvicorn sqlalchemy python-jose[cryptography] passlib[bcrypt] python-multipart python-dotenv
```

---

## 总结

回顾一下我们学了什么：

| 章节 | 你学会了 |
| --- | --- |
| 基础路由 | GET/POST/PUT/DELETE、路径参数、查询参数 |
| 数据模型 | Pydantic 定义请求体、自动验证、响应模型 |
| CRUD 实战 | 完整的增删改查 API |
| 数据库 | SQLAlchemy 连接数据库、ORM 操作 |
| 异步编程 | async/await 的使用场景 |
| 中间件与 CORS | 请求拦截、跨域配置 |
| 用户认证 | JWT Token 注册登录、接口保护 |
| 文件上传 | 单文件、多文件上传 |
| 错误处理 | HTTPException、自定义异常 |
| 项目结构 | APIRouter 拆分、分层架构 |
| 部署 | Uvicorn、Gunicorn、Docker、Nginx |

FastAPI 的学习曲线并不陡峭，但它的功能足够覆盖大多数后端开发场景。建议你跟着本文动手把代码敲一遍，然后尝试在待办事项的基础上加功能（比如用户系统、分类标签、搜索），在实践中加深理解。

有什么不清楚的地方，查 [官方文档](https://fastapi.tiangolo.com/) 是最靠谱的。祝你学得愉快！
