# 如何编写与维护一个 Python 项目

这是一份软件工程指南，而非工具链教程。核心问题是：

- 如何组织代码，使其易读、易改、易扩展？
- 如何让项目在需求不断变化时，仍然保持健康？

---

## 目录

1. [核心原则](#1-核心原则)
   - 1.1 依赖规则
   - 1.2 SOLID 在 Python 中的体现
   - 1.3 边界与上下文
   - 1.4 事务边界与 Unit of Work
2. [代码组织](#2-代码组织)
3. [可测试性设计](#3-可测试性设计)
4. [持续演进与重构](#4-持续演进与重构)
5. [错误处理与边界防护](#5-错误处理与边界防护)
   - 5.1 异常 vs 返回值
   - 5.2 边界校验
   - 5.3 可观测性（Observability）
6. [配置与环境管理](#6-配置与环境管理)
7. [工具链：让项目对 AI 编程友好](#7-工具链让项目对-ai-编程友好)
   - 7.1 依赖管理（uv）
   - 7.2 代码风格（PEP 8 + ruff）
   - 7.3 类型标注 + 类型检查
   - 7.4 让错误尽早暴露
   - 7.5 Docstring：让 AI 读懂意图
   - 7.6 CI 最小闭环

---

## 1. 核心原则

### 1.1 依赖规则

**核心思想**：业务逻辑不依赖框架、数据库、外部服务。依赖方向永远指向内层。

想象你的代码是一个洋葱：

```
外层（易变）          内层（稳定）
───────────────────────────────────
Web框架/CLI  →  应用服务  →  领域模型
数据库适配器 →  仓储接口  →  业务规则
外部API客户端 →  网关接口  →  核心实体
```

**为什么这样做**：外层（框架、数据库）经常变，内层（业务规则）相对稳定。如果业务逻辑依赖了具体的数据库或框架，换一个就要改业务代码——这是灾难。

**反面例子**：业务逻辑直接依赖 SQLAlchemy

```python
# 不好：业务逻辑和数据库耦合
class OrderService:
    def create_order(self, user_id: int, items: list[dict]) -> Order:
        # 业务逻辑里直接写 SQLAlchemy
        user = db.session.query(User).filter_by(id=user_id).first()
        if not user:
            raise ValueError("用户不存在")
        
        order = Order(user_id=user_id, status="pending")
        db.session.add(order)
        
        for item in items:
            # 又是数据库操作混在业务逻辑里
            product = db.session.query(Product).filter_by(id=item["product_id"]).first()
            if product.stock < item["quantity"]:
                raise ValueError(f"库存不足: {product.name}")
            # ...
        
        db.session.commit()
        return order
```

**正面例子**：业务逻辑依赖抽象接口

```python
# 好：定义抽象接口（Protocol）
# 注意：涉及 I/O 的接口应该是异步的（async）
from typing import Protocol

class UserRepository(Protocol):
    async def get_by_id(self, user_id: int) -> User | None: ...

class ProductRepository(Protocol):
    async def get_by_id(self, product_id: int) -> Product | None: ...
    async def update(self, product: Product) -> None: ...

class OrderRepository(Protocol):
    async def add(self, order: Order) -> None: ...


# 业务逻辑只依赖接口，不知道数据库的存在
class OrderService:
    def __init__(
        self,
        user_repo: UserRepository,
        product_repo: ProductRepository,
        order_repo: OrderRepository,
        uow: UnitOfWork,  # 事务由 UoW 统一管理
    ):
        self._user_repo = user_repo
        self._product_repo = product_repo
        self._order_repo = order_repo
        self._uow = uow

    async def create_order(self, user_id: int, items: list[dict]) -> Order:
        user = await self._user_repo.get_by_id(user_id)
        if not user:
            raise ValueError("用户不存在")

        order = Order(user_id=user_id, status="pending")

        for item in items:
            product = await self._product_repo.get_by_id(item["product_id"])
            if product.stock < item["quantity"]:
                raise ValueError(f"库存不足: {product.name}")
            order.add_item(product, item["quantity"])
            product.reduce_stock(item["quantity"])
            await self._product_repo.update(product)  # 此时不 commit

        await self._order_repo.add(order)  # 此时不 commit
        await self._uow.commit()  # 统一提交，保证原子性
        return order
```

现在你可以：
- 测试时传入假的 Repository（不需要真数据库）
- 换数据库只需要写一个新的 Repository 实现
- 业务逻辑完全不用改

> **为什么要用 async**：在 2024+ 的 Python Web 开发（尤其是 FastAPI 生态）中，`async/await` 是标配。如果仓储层和应用服务层设计成同步，后期为了性能迁移到异步的成本极高（因为 async 具有传染性）。**纯业务逻辑**（如 `Order.add_item()`）保持同步即可，因为是纯计算；**涉及 I/O 的接口**（Repository、外部服务）应默认采用异步定义。

---

### 1.2 SOLID 在 Python 中的体现

SOLID 是五个设计原则的首字母缩写。不需要死记硬背，理解核心意图即可。

#### S - 单一职责原则（Single Responsibility）

**一句话**：一个类/模块只做一件事，只有一个修改的理由。

```python
# 不好：一个类干了太多事
class UserManager:
    def create_user(self, data: dict) -> User: ...
    def send_welcome_email(self, user: User) -> None: ...  # 发邮件不该在这里
    def generate_report(self, users: list[User]) -> str: ...  # 生成报告也不该在这里


# 好：拆开
class UserService:
    def create_user(self, data: dict) -> User: ...

class EmailService:
    def send_welcome_email(self, user: User) -> None: ...

class UserReportGenerator:
    def generate(self, users: list[User]) -> str: ...
```

#### O - 开闭原则（Open/Closed）

**一句话**：对扩展开放，对修改关闭。加新功能时，尽量加新代码，而不是改老代码。

```python
# 不好：每加一种通知方式就要改这个函数
def notify_user(user: User, method: str, message: str) -> None:
    if method == "email":
        send_email(user.email, message)
    elif method == "sms":
        send_sms(user.phone, message)
    elif method == "push":  # 新加的，又要改这个函数
        send_push(user.device_id, message)


# 好：定义接口，新增通知方式只需加新类
from typing import Protocol

class Notifier(Protocol):
    def notify(self, user: User, message: str) -> None: ...

class EmailNotifier:
    def notify(self, user: User, message: str) -> None:
        send_email(user.email, message)

class SmsNotifier:
    def notify(self, user: User, message: str) -> None:
        send_sms(user.phone, message)

# 加新通知方式：写一个新类就行，不用改已有代码
class PushNotifier:
    def notify(self, user: User, message: str) -> None:
        send_push(user.device_id, message)
```

#### L - 里氏替换原则（Liskov Substitution）

**一句话**：子类必须能替换父类使用，不能破坏父类的契约。

```python
# 不好：Square 破坏了 Rectangle 的契约
class Rectangle:
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height

    def area(self) -> int:
        return self.width * self.height

class Square(Rectangle):
    def __init__(self, side: int):
        super().__init__(side, side)
    
    # 问题：设置宽度时，高度也要变，这破坏了 Rectangle 的行为预期
    @property
    def width(self) -> int:
        return self._side
    
    @width.setter
    def width(self, value: int) -> None:
        self._side = value  # 同时改了宽和高


# 好：用组合代替继承，或者重新设计抽象
from typing import Protocol

class Shape(Protocol):
    def area(self) -> int: ...

class Rectangle:
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height
    
    def area(self) -> int:
        return self.width * self.height

class Square:
    def __init__(self, side: int):
        self.side = side
    
    def area(self) -> int:
        return self.side * self.side
```

#### I - 接口隔离原则（Interface Segregation）

**一句话**：不要强迫调用方依赖它不需要的方法。接口要小而专。

```python
# 不好：一个大接口，强迫实现者实现不需要的方法
from typing import Protocol

class Worker(Protocol):
    def work(self) -> None: ...
    def eat(self) -> None: ...
    def sleep(self) -> None: ...

class Robot:  # 机器人不需要 eat 和 sleep，但被迫实现
    def work(self) -> None:
        print("Working...")
    
    def eat(self) -> None:
        pass  # 空实现，代码坏味道
    
    def sleep(self) -> None:
        pass


# 好：拆成小接口
class Workable(Protocol):
    def work(self) -> None: ...

class Eatable(Protocol):
    def eat(self) -> None: ...

class Robot:  # 只实现需要的接口
    def work(self) -> None:
        print("Working...")

class Human:  # 实现多个接口
    def work(self) -> None:
        print("Working...")
    
    def eat(self) -> None:
        print("Eating...")
```

#### D - 依赖倒置原则（Dependency Inversion）

**一句话**：高层模块不依赖低层模块，两者都依赖抽象。

这是前面"依赖规则"的具体体现，用 Python 的 `Protocol` 或 `ABC` 定义抽象接口：

```python
from typing import Protocol

# 抽象（接口）
class PaymentGateway(Protocol):
    def charge(self, amount: int, card_token: str) -> bool: ...

# 低层模块：具体实现
class StripeGateway:
    def charge(self, amount: int, card_token: str) -> bool:
        # Stripe API 调用
        return True

class PayPalGateway:
    def charge(self, amount: int, card_token: str) -> bool:
        # PayPal API 调用
        return True

# 高层模块：依赖抽象，不依赖具体实现
class PaymentService:
    def __init__(self, gateway: PaymentGateway):
        self._gateway = gateway

    def process_payment(self, amount: int, card_token: str) -> bool:
        return self._gateway.charge(amount, card_token)
```

---

### 1.3 边界与上下文

**核心思想**：把系统按业务能力划分成多个"限界上下文"（Bounded Context），每个上下文内部高内聚，上下文之间低耦合。

**什么时候拆模块**：
- 当一个概念在不同场景下含义不同时（比如"用户"在订单系统和内容系统里关注点不同）
- 当一块代码的修改频率和原因与其他代码不同时
- 当团队边界出现时（不同团队负责不同模块）

**什么时候拆服务**：
- 当模块需要独立部署、独立扩展时
- 当模块有不同的可用性要求时
- 当模块使用不同技术栈更合适时

**实际建议**：先从单体开始，用模块划分好边界。等真正需要时（性能、团队、部署）再拆成服务。过早拆服务是常见的过度设计。

```
my_project/
├── src/
│   └── my_project/
│       ├── order/          # 订单上下文
│       │   ├── domain/     # 订单领域模型
│       │   ├── application/# 订单应用服务
│       │   └── ...
│       ├── inventory/      # 库存上下文
│       │   ├── domain/
│       │   ├── application/
│       │   └── ...
│       └── shared/         # 跨上下文共享的东西（尽量少）
```

上下文之间的通信：
- 同步调用：通过明确定义的接口
- 异步消息：事件驱动，解耦更彻底

```python
# 订单上下文发布事件
class OrderCreatedEvent:
    order_id: str
    user_id: str
    total_amount: int

# 库存上下文订阅事件并处理
class InventoryEventHandler:
    async def handle_order_created(self, event: OrderCreatedEvent) -> None:
        # 扣减库存
        ...
```

---

### 1.4 事务边界与 Unit of Work

**核心问题**：如果一个用例需要操作多个 Repository（如：扣库存 + 创建订单），每个 Repo 内部自动 commit 会导致数据不一致（扣了库存但订单创建失败）。

**解决方案**：引入 **Unit of Work (UoW)** 模式，让 Service 层控制事务边界，而不是 Repository 层。

```python
from typing import Protocol
from contextlib import asynccontextmanager

# Unit of Work 接口
class UnitOfWork(Protocol):
    async def commit(self) -> None: ...
    async def rollback(self) -> None: ...


# SQLAlchemy 实现
class SqlAlchemyUnitOfWork:
    def __init__(self, session: AsyncSession):
        self._session = session

    async def commit(self) -> None:
        await self._session.commit()

    async def rollback(self) -> None:
        await self._session.rollback()


# 也可以用上下文管理器模式
class UnitOfWorkContext(Protocol):
    product_repo: ProductRepository
    order_repo: OrderRepository
    
    async def __aenter__(self) -> "UnitOfWorkContext": ...
    async def __aexit__(self, *args) -> None: ...
    async def commit(self) -> None: ...


# 使用上下文管理器风格
class OrderService:
    def __init__(self, uow_factory: Callable[[], UnitOfWorkContext]):
        self._uow_factory = uow_factory

    async def create_order(self, user_id: int, items: list[dict]) -> Order:
        async with self._uow_factory() as uow:
            # 所有操作在同一个事务中
            product = await uow.product_repo.get(items[0]["product_id"])
            order = Order(user_id=user_id)
            
            product.reduce_stock(items[0]["quantity"])
            await uow.product_repo.update(product)  # 不 commit
            await uow.order_repo.add(order)         # 不 commit
            
            await uow.commit()  # 一起提交，要么全成功，要么全回滚
            return order
```

**关键原则**：
- Repository 只负责数据访问，**不调用 commit**
- Service 层通过 UoW 控制事务边界
- 一个用例 = 一个事务

---

## 2. 代码组织

### 2.1 分层结构

推荐四层结构，从内到外：

| 层 | 职责 | 依赖规则 |
|---|---|---|
| **domain** | 业务实体、值对象、业务规则 | 不依赖任何外层 |
| **application** | 用例/应用服务，编排业务流程 | 只依赖 domain |
| **infrastructure** | 技术实现：数据库、外部API、消息队列 | 依赖 domain（实现其接口） |
| **interface** | 入口：HTTP API、CLI、消息消费者 | 依赖 application |

目录结构示例：

```
src/
└── my_project/
    ├── __init__.py
    ├── domain/                 # 领域层
    │   ├── __init__.py
    │   ├── entities.py         # 实体
    │   ├── value_objects.py    # 值对象
    │   ├── repositories.py     # 仓储接口（Protocol）
    │   └── services.py         # 领域服务
    │
    ├── application/            # 应用层
    │   ├── __init__.py
    │   ├── use_cases.py        # 用例/应用服务
    │   └── dto.py              # 数据传输对象
    │
    ├── infrastructure/         # 基础设施层
    │   ├── __init__.py
    │   ├── database/
    │   │   ├── models.py       # ORM 模型
    │   │   └── repositories.py # 仓储实现
    │   ├── external/
    │   │   └── payment.py      # 外部服务客户端
    │   └── config.py           # 配置加载
    │
    └── interface/              # 接口层
        ├── __init__.py
        ├── api/
        │   ├── routes.py       # HTTP 路由
        │   └── schemas.py      # 请求/响应 Schema
        └── cli/
            └── commands.py     # CLI 命令
```

### 2.2 模块划分

两种方式，各有优缺点：

**按技术层切分**（上面的例子）：
- 优点：结构清晰，适合小项目
- 缺点：一个功能的代码散落在多个目录，改一个功能要跳多个地方

**按业务能力切分**：
- 优点：一个功能的代码在一起，高内聚
- 缺点：需要更多的设计思考，适合中大型项目

```
src/
└── my_project/
    ├── order/              # 订单模块
    │   ├── domain/
    │   ├── application/
    │   ├── infrastructure/
    │   └── interface/
    │
    ├── user/               # 用户模块
    │   ├── domain/
    │   ├── application/
    │   ├── infrastructure/
    │   └── interface/
    │
    └── shared/             # 共享代码（尽量少）
        ├── domain/
        └── infrastructure/
```

**实际建议**：
- 小项目（< 5000 行）：按技术层切分
- 中大项目：按业务能力切分，每个模块内部再按层组织

### 2.3 依赖注入

依赖注入的核心是：**不要在类内部创建依赖，而是从外部传入**。

```python
# 不好：内部创建依赖
class OrderService:
    def __init__(self):
        self._repo = SqlAlchemyOrderRepository()  # 写死了具体实现
        self._payment = StripePayment()           # 写死了具体实现


# 好：从外部注入依赖
class OrderService:
    def __init__(
        self,
        repo: OrderRepository,      # 接口类型
        payment: PaymentGateway,    # 接口类型
    ):
        self._repo = repo
        self._payment = payment
```

**Python 中的依赖注入实现**：

**推荐方式：Composition Root（组合根）模式**

在 `main.py` 或 `dependencies.py` 中**手动实例化和组装**所有依赖。这是最清晰、最利于 IDE 跳转、也最利于 AI 理解（上下文在同一文件）的方式。

```python
# src/my_project/dependencies.py
# 组合根：所有依赖在这里组装，一目了然

from my_project.infrastructure.database import create_async_session
from my_project.infrastructure.repositories import (
    SqlAlchemyOrderRepository,
    SqlAlchemyProductRepository,
)
from my_project.infrastructure.payment import StripePayment
from my_project.application.services import OrderService
from my_project.config import settings


def create_order_service(session: AsyncSession) -> OrderService:
    """
    组装 OrderService 的所有依赖。
    
    这种显式组装的好处：
    1. 依赖关系一目了然，IDE 可直接跳转
    2. 测试时可以传入不同的实现
    3. AI 编程工具能准确理解依赖图
    """
    order_repo = SqlAlchemyOrderRepository(session)
    product_repo = SqlAlchemyProductRepository(session)
    payment = StripePayment(api_key=settings.stripe_api_key)
    uow = SqlAlchemyUnitOfWork(session)
    
    return OrderService(
        order_repo=order_repo,
        product_repo=product_repo,
        payment=payment,
        uow=uow,
    )
```

> **为什么不推荐 `dependency-injector` 等 DI 容器**：这类库在 Python 社区中有"过度设计"的嫌疑，特别是对于中小型项目。它们引入了大量样板代码和间接层，反而让代码更难理解。手动组装虽然看起来"原始"，但在 Python 中是最清晰的做法。

**FastAPI 中的依赖注入**：

FastAPI 原生 `Depends` 配合 `Annotated` 已经足够强大，不需要额外的 DI 框架：

```python
from typing import Annotated
from collections.abc import AsyncIterator
from fastapi import Depends, FastAPI

app = FastAPI()


async def get_db_session() -> AsyncIterator[AsyncSession]:
    async with async_session_maker() as session:
        yield session


async def get_order_service(
    session: Annotated[AsyncSession, Depends(get_db_session)],
) -> OrderService:
    return create_order_service(session)  # 复用组合根的工厂函数


# 使用 Annotated 简化依赖声明
OrderServiceDep = Annotated[OrderService, Depends(get_order_service)]


@app.post("/orders")
async def create_order(
    data: CreateOrderRequest,
    service: OrderServiceDep,
):
    return await service.create_order(data)
```

**关键原则**：
- 优先使用手动组装（Composition Root）
- 依赖关系应该显式、可追踪
- 框架自带的 DI（如 FastAPI Depends）足够用，不需要额外引入 DI 库

---

## 3. 可测试性设计

### 3.1 测试金字塔

```
        /\
       /  \      端到端测试（少）
      /----\     - 测试整个系统
     /      \    - 慢、脆弱、成本高
    /--------\   
   /          \  集成测试（中）
  /            \ - 测试模块间协作
 /--------------\- 需要真实依赖（数据库等）
/                \
/------------------\ 单元测试（多）
                     - 测试单个函数/类
                     - 快、稳定、成本低
```

**投入比例建议**：单元测试 70%，集成测试 20%，端到端测试 10%

### 3.2 让代码可测试

**原则 1：用纯函数处理逻辑**

纯函数：相同输入永远得到相同输出，没有副作用。

```python
# 好测试：纯函数
def calculate_discount(price: int, user_level: str) -> int:
    """计算折扣金额"""
    rates = {"bronze": 0.05, "silver": 0.10, "gold": 0.15}
    rate = rates.get(user_level, 0)
    return int(price * rate)

# 测试很简单
def test_calculate_discount():
    assert calculate_discount(1000, "gold") == 150
    assert calculate_discount(1000, "unknown") == 0
```

**原则 2：隔离副作用（I/O）**

把副作用（数据库、网络、文件）推到边缘，核心逻辑保持纯净。

```python
# 不好：业务逻辑和 I/O 混在一起
def process_order(order_id: str) -> None:
    order = db.query(Order).get(order_id)      # I/O
    if order.total > 1000:
        order.discount = order.total * 0.1      # 业务逻辑
    order.status = "processed"                  # 业务逻辑
    db.commit()                                 # I/O
    send_email(order.user.email, "订单已处理")  # I/O


# 好：分离业务逻辑和 I/O
# 纯业务逻辑（好测试）
def calculate_order_discount(total: int) -> int:
    if total > 1000:
        return int(total * 0.1)
    return 0

# I/O 在外层编排
class ProcessOrderUseCase:
    def __init__(self, repo: OrderRepository, notifier: Notifier):
        self._repo = repo
        self._notifier = notifier

    async def execute(self, order_id: str) -> None:
        order = await self._repo.get(order_id)
        order.discount = calculate_order_discount(order.total)  # 调用纯函数
        order.status = "processed"
        await self._repo.save(order)
        await self._notifier.notify(order.user, "订单已处理")
```

**原则 3：依赖注入，方便替换**

测试时可以注入假的实现：

```python
# 测试用的假实现
class FakeOrderRepository:
    def __init__(self):
        self._orders: dict[str, Order] = {}

    async def get(self, order_id: str) -> Order:
        return self._orders[order_id]

    async def save(self, order: Order) -> None:
        self._orders[order.id] = order

class FakeNotifier:
    def __init__(self):
        self.notifications: list[tuple] = []

    async def notify(self, user: User, message: str) -> None:
        self.notifications.append((user, message))


# 测试
async def test_process_order():
    repo = FakeOrderRepository()
    repo._orders["order-1"] = Order(id="order-1", total=2000, status="pending")
    
    notifier = FakeNotifier()
    
    use_case = ProcessOrderUseCase(repo=repo, notifier=notifier)
    await use_case.execute("order-1")
    
    order = await repo.get("order-1")
    assert order.status == "processed"
    assert order.discount == 200
    assert len(notifier.notifications) == 1
```

### 3.3 测试作为设计反馈

**如果代码难测，说明设计有问题**。常见信号：

| 测试难点 | 设计问题 | 解决方案 |
|---------|---------|---------|
| 需要大量 mock | 依赖太多 | 拆分职责，减少依赖 |
| 测试需要复杂的设置 | 初始化逻辑太复杂 | 简化构造函数，用工厂 |
| 测试结果不稳定 | 依赖外部状态 | 隔离副作用，用纯函数 |
| 私有方法想单独测 | 方法应该提取成独立单元 | 提取为独立函数或类 |

---

## 4. 持续演进与重构

### 4.1 重构的节奏

**什么时候重构**：
- 加新功能前：先把相关代码整理好，再加功能
- 修 bug 后：理解了问题，顺手改善结构
- Code review 时：发现可以改进的地方

**什么时候不重构**：
- 截止日期很紧，先完成功能
- 没有测试覆盖，改了容易出问题
- 代码即将被删除或重写

**安全重构的前提**：
1. 有测试覆盖（至少是你要改的部分）
2. 小步进行，每步都能运行测试
3. 不要同时重构和加功能

### 4.2 常见重构手法

**提取函数**：把一段代码提取成独立函数

```python
# 之前
def process_order(order: Order) -> None:
    # 验证
    if not order.items:
        raise ValueError("订单不能为空")
    if order.total <= 0:
        raise ValueError("金额必须大于0")
    
    # 计算折扣
    discount = 0
    if order.total > 1000:
        discount = order.total * 0.1
    # ...更多逻辑


# 之后：提取出独立函数
def validate_order(order: Order) -> None:
    if not order.items:
        raise ValueError("订单不能为空")
    if order.total <= 0:
        raise ValueError("金额必须大于0")

def calculate_discount(total: int) -> int:
    if total > 1000:
        return int(total * 0.1)
    return 0

def process_order(order: Order) -> None:
    validate_order(order)
    discount = calculate_discount(order.total)
    # ...
```

**提取类**：当一个类承担太多职责时

```python
# 之前：User 类做了太多事
class User:
    def __init__(self, name: str, email: str, street: str, city: str, zip_code: str):
        self.name = name
        self.email = email
        self.street = street
        self.city = city
        self.zip_code = zip_code
    
    def full_address(self) -> str:
        return f"{self.street}, {self.city} {self.zip_code}"


# 之后：提取 Address 类
from dataclasses import dataclass

@dataclass
class Address:
    street: str
    city: str
    zip_code: str
    
    def full(self) -> str:
        return f"{self.street}, {self.city} {self.zip_code}"

@dataclass
class User:
    name: str
    email: str
    address: Address
```

**引入参数对象**：当函数参数太多时

```python
# 之前：参数太多
def search_products(
    keyword: str,
    category: str,
    min_price: int,
    max_price: int,
    in_stock: bool,
    sort_by: str,
    page: int,
    page_size: int,
) -> list[Product]:
    ...


# 之后：用参数对象
@dataclass
class SearchCriteria:
    keyword: str = ""
    category: str = ""
    min_price: int = 0
    max_price: int = 999999
    in_stock: bool = False

@dataclass
class Pagination:
    page: int = 1
    page_size: int = 20
    sort_by: str = "relevance"

def search_products(
    criteria: SearchCriteria,
    pagination: Pagination,
) -> list[Product]:
    ...
```

**改名**：让名字准确表达意图

```python
# 不好
def process(d: dict) -> dict:
    ...

# 好
def transform_user_data_for_export(user_data: UserData) -> ExportFormat:
    ...
```

### 4.3 技术债管理

技术债：为了短期速度而做的妥协，需要后续偿还。

**如何识别**：
- 代码注释里的 `TODO`、`FIXME`、`HACK`
- 重复的代码
- 过长的函数/类
- 测试覆盖率低的模块
- 团队抱怨"这块代码很难改"的地方

**如何记录**：
- 在代码里用统一格式标记：`# TODO(name): 描述问题和改进方向`
- 在 issue tracker 里建立"技术债"标签
- 定期 review，评估优先级

**如何偿还**：
- 不要试图一次性还清
- 每个迭代分配固定比例时间（如 20%）处理技术债
- 优先处理：改动频繁区域、影响开发效率的债

---

## 5. 错误处理与边界防护

### 5.1 异常 vs 返回值

**核心原则：坚持使用自定义异常体系**

Python 的生态（ORM、框架、标准库）都是基于 Exception 的。**自定义异常体系**是 Python 中最地道（Idiomatic）的错误处理方式。

```python
# 定义业务异常体系
class AppError(Exception):
    """应用层错误基类"""
    pass

class NotFoundError(AppError):
    """资源不存在"""
    pass

class ValidationError(AppError):
    """验证失败"""
    pass

class InsufficientStockError(AppError):
    """库存不足"""
    def __init__(self, product_name: str, requested: int, available: int):
        self.product_name = product_name
        self.requested = requested
        self.available = available
        super().__init__(f"库存不足: {product_name} (需要 {requested}, 仅剩 {available})")


# 使用
async def get_user(user_id: int) -> User:
    user = await db.query(User).get(user_id)
    if user is None:
        raise NotFoundError(f"用户不存在: {user_id}")
    return user


async def create_order(self, product_id: int, quantity: int) -> Order:
    product = await self._product_repo.get(product_id)
    if product.stock < quantity:
        raise InsufficientStockError(product.name, quantity, product.stock)
    # ... 业务逻辑
```

**关于 Result 模式的务实建议**：

类似 Rust 的 `Result[T, E]` 模式虽然在函数式编程中很优雅，但在 Python 中**并非地道的写法**：

```python
# Result 模式示例（谨慎使用）
from dataclasses import dataclass
from typing import TypeVar, Generic

T = TypeVar("T")
E = TypeVar("E")

@dataclass
class Ok(Generic[T]):
    value: T

@dataclass
class Err(Generic[E]):
    error: E

Result = Ok[T] | Err[E]
```

**为什么在 Python 中慎用 Result 模式**：
1. **与生态不兼容**：SQLAlchemy、Pydantic、FastAPI 等主流库都使用异常
2. **增加认知负担**：代码中会充满 `match/case` 和拆包逻辑
3. **传染性**：一旦使用，调用链上所有函数都要处理 Result
4. **难以与第三方库统一**：第三方库抛异常，你返回 Result，风格混乱

**折中方案**（如果确实想用）：
- 仅在**纯验证函数**中返回 `T | None` 或 `bool`
- **系统错误和意外情况**坚持抛异常

```python
# 折中：简单验证用返回值
def is_valid_email(email: str) -> bool:
    return "@" in email and "." in email.split("@")[1]

def parse_positive_int(value: str) -> int | None:
    try:
        n = int(value)
        return n if n > 0 else None
    except ValueError:
        return None


# 业务错误坚持用异常
async def transfer_money(from_id: int, to_id: int, amount: int) -> None:
    if amount <= 0:
        raise ValidationError("转账金额必须大于0")
    
    from_account = await self._account_repo.get(from_id)
    if from_account.balance < amount:
        raise InsufficientBalanceError(from_account.id, amount, from_account.balance)
    
    # ... 执行转账
```

**实际建议**：
- **默认用异常**：除非团队有极强的函数式编程背景
- **基础设施层**（数据库、网络）：抛异常
- **应用层**：捕获异常，统一转换为 HTTP 响应（见 5.2）
- **纯验证**：可以返回 `bool` 或 `T | None`，保持简单

### 5.2 边界校验

**原则**：在系统边界（接口层）做校验，内部代码可以假设数据已经有效。

```python
# 接口层：严格校验
from pydantic import BaseModel, EmailStr, Field

class CreateUserRequest(BaseModel):
    name: str = Field(min_length=1, max_length=100)
    email: EmailStr
    age: int = Field(ge=0, le=150)


# 应用层：不用重复校验
class CreateUserUseCase:
    def execute(self, request: CreateUserRequest) -> User:
        # request 已经被 Pydantic 校验过了
        # 这里只关注业务逻辑
        return self._repo.create(
            name=request.name,
            email=request.email,
            age=request.age,
        )
```

> **提示**：Pydantic v2 默认允许隐式类型转换（如 `"18"` → `18`）。对于严格的边界校验，可启用 strict 模式：
> ```python
> model_config = ConfigDict(strict=True)
> ```
> 这能让类型错误在入口处立即暴露，而非在下游引发难以追踪的问题。

**统一错误响应**（FastAPI 示例）：

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

class AppError(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

class UserNotFoundError(AppError):
    def __init__(self, user_id: int):
        super().__init__(
            code="USER_NOT_FOUND",
            message=f"用户不存在: {user_id}",
            status_code=404,
        )

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"code": exc.code, "message": exc.message},
    )
```

### 5.3 可观测性（Observability）

代码"易维护"的一个关键维度是**生产环境出了问题能看懂**。没有结构化日志，只有 traceback，无法应对复杂系统的排查。

**核心原则**：使用 **结构化日志（Structured Logging）**，输出 JSON 格式，方便机器解析、日志平台聚合、以及 AI 分析。

**推荐库**：`structlog`（或 `loguru`）

```bash
uv add structlog
```

**基础配置**：

```python
# src/my_project/logging.py
import logging

import structlog

def setup_logging(json_format: bool = True) -> None:
    """
    配置结构化日志。
    
    Args:
        json_format: 生产环境用 True（JSON），本地开发可用 False（彩色输出）
    """
    processors = [
        structlog.contextvars.merge_contextvars,  # 支持上下文变量
        structlog.processors.add_log_level,
        structlog.processors.TimeStamper(fmt="iso"),
        structlog.processors.StackInfoRenderer(),
    ]
    
    if json_format:
        processors.append(structlog.processors.JSONRenderer())
    else:
        processors.append(structlog.dev.ConsoleRenderer())
    
    structlog.configure(
        processors=processors,
        wrapper_class=structlog.make_filtering_bound_logger(logging.INFO),
        context_class=dict,
        logger_factory=structlog.PrintLoggerFactory(),
        cache_logger_on_first_use=True,
    )
```

**在业务代码中使用**：

```python
import structlog

logger = structlog.get_logger()


class OrderService:
    async def create_order(self, user_id: int, items: list[dict]) -> Order:
        # 绑定请求级别的上下文（会自动附加到后续所有日志）
        log = logger.bind(user_id=user_id, item_count=len(items))
        
        log.info("order_creation_started")
        
        try:
            user = await self._user_repo.get_by_id(user_id)
            if not user:
                log.warning("user_not_found")
                raise NotFoundError(f"用户不存在: {user_id}")
            
            order = Order(user_id=user_id, status="pending")
            # ... 业务逻辑
            
            await self._order_repo.add(order)
            await self._uow.commit()
            
            log.info("order_created", order_id=order.id, total=order.total)
            return order
            
        except Exception as e:
            log.error("order_creation_failed", error=str(e), error_type=type(e).__name__)
            raise
```

**输出示例**（JSON 格式）：

```json
{"event": "order_creation_started", "user_id": 123, "item_count": 3, "level": "info", "timestamp": "2025-01-20T10:30:00Z"}
{"event": "order_created", "user_id": 123, "item_count": 3, "order_id": "ord_abc123", "total": 29900, "level": "info", "timestamp": "2025-01-20T10:30:01Z"}
```

**请求级别的上下文追踪**（FastAPI 中间件）：

```python
import uuid
from contextvars import ContextVar
import structlog

# 请求 ID 上下文
request_id_ctx: ContextVar[str] = ContextVar("request_id", default="")

@app.middleware("http")
async def logging_middleware(request: Request, call_next):
    request_id = request.headers.get("X-Request-ID", str(uuid.uuid4()))
    request_id_ctx.set(request_id)
    
    # 绑定到 structlog 上下文
    structlog.contextvars.clear_contextvars()
    structlog.contextvars.bind_contextvars(
        request_id=request_id,
        path=request.url.path,
        method=request.method,
    )
    
    logger.info("request_started")
    response = await call_next(request)
    logger.info("request_completed", status_code=response.status_code)
    
    return response
```

**日志规范**：

| 场景 | 日志级别 | 示例 |
|-----|---------|------|
| 业务流程关键节点 | INFO | `order_created`, `payment_processed` |
| 可恢复的问题 | WARNING | `retry_attempt`, `cache_miss` |
| 需要排查的错误 | ERROR | `payment_failed`, `external_api_error` |
| 调试信息 | DEBUG | `query_executed`, `cache_hit` |

**关键原则**：
- **事件命名用 snake_case**：`order_created` 而不是 "Order created"
- **携带结构化数据**：`order_id=123` 而不是 `f"order_id: {order_id}"`
- **避免敏感信息**：不要记录密码、token、完整信用卡号等
- **错误日志要带上下文**：记录 `error_type`、相关 ID，方便关联排查

---

## 6. 配置与环境管理

### 6.1 配置的分层

优先级从低到高：默认值 → 配置文件 → 环境变量 → 命令行参数

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    # 默认值
    app_name: str = "my-app"
    debug: bool = False
    
    # 必须配置（无默认值）
    database_url: str
    secret_key: str
    
    # 可选配置
    redis_url: str = "redis://localhost:6379"
    
    model_config = {
        "env_file": ".env",           # 从 .env 文件读取
        "env_file_encoding": "utf-8",
        "extra": "ignore",            # 忽略多余的环境变量
    }

# 使用
settings = Settings()
print(settings.database_url)
```

### 6.2 敏感信息隔离

**规则**：
1. 密钥、密码永远不提交到代码库
2. 用 `.env` 文件存储本地开发配置
3. 用 `.env.example` 作为模板（提交到代码库）
4. 生产环境用环境变量或密钥管理服务

```bash
# .env.example（提交到代码库）
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
SECRET_KEY=your-secret-key-here
STRIPE_API_KEY=sk_test_xxx

# .env（不提交，在 .gitignore 中）
DATABASE_URL=postgresql://dev:dev123@localhost:5432/myapp
SECRET_KEY=local-dev-secret-key
STRIPE_API_KEY=sk_test_actual_key
```

`.gitignore` 里加上：

```
.env
.env.local
.env.*.local
```

**不同环境的切换**：

```python
import os
from enum import Enum

class Environment(str, Enum):
    LOCAL = "local"
    STAGING = "staging"
    PRODUCTION = "production"

class Settings(BaseSettings):
    environment: Environment = Environment.LOCAL
    
    @property
    def is_production(self) -> bool:
        return self.environment == Environment.PRODUCTION
    
    @property
    def is_local(self) -> bool:
        return self.environment == Environment.LOCAL
```

---

## 7. 工具链：让项目对 AI 编程友好

核心思路：**让项目产生足够多的反馈信号**，AI 编程助手才能根据 lint 错误、类型错误、运行时报错自我修正。

### 7.1 依赖管理（uv）

uv 是现代 Python 项目的推荐工具，速度快、体验好。

**初始化项目**：

```bash
# 创建新项目
uv init my-project
cd my-project

# 或在已有目录初始化
uv init
```

生成的文件：
- `pyproject.toml`：项目配置和依赖声明
- `uv.lock`：锁定的精确版本（必须提交到代码库）
- `.python-version`：Python 版本

**管理依赖**：

```bash
# 添加生产依赖
uv add fastapi uvicorn pydantic-settings

# 添加开发依赖
uv add --dev pytest ruff pyright

# 添加到特定组
uv add --group test pytest pytest-cov
uv add --group lint ruff

# 移除依赖
uv remove requests
```

**运行命令**：

```bash
# 运行 Python 脚本（自动同步环境）
uv run python main.py

# 运行开发服务器
uv run uvicorn main:app --reload

# 运行测试
uv run pytest

# 运行 lint
uv run ruff check .
```

**锁定与同步**：

```bash
# 更新锁文件（解析最新版本）
uv lock

# 更新特定包
uv lock --upgrade-package requests

# 同步环境（安装锁定的版本）
uv sync

# CI/生产环境：严格使用锁定版本
uv sync --frozen
```

### 7.2 代码风格（PEP 8 + ruff）

遵守 PEP 8，使用 ruff 统一风格。

**配置** `pyproject.toml`：

```toml
[tool.ruff]
line-length = 88
target-version = "py312"

[tool.ruff.lint]
select = [
    "E",    # pycodestyle errors
    "W",    # pycodestyle warnings
    "F",    # pyflakes
    "I",    # isort
    "UP",   # pyupgrade
    "B",    # flake8-bugbear
    "SIM",  # flake8-simplify
]
ignore = [
    "E501",  # 行太长（让 formatter 处理）
]

[tool.ruff.format]
quote-style = "double"
```

**日常使用**：

```bash
# 检查代码问题
uv run ruff check .

# 自动修复
uv run ruff check . --fix

# 格式化代码
uv run ruff format .

# 检查格式（CI 用）
uv run ruff format . --check
```

### 7.3 类型标注 + 类型检查

**为什么类型标注对 AI 很重要**：
1. 提供上下文：AI 能从类型知道函数期望什么、返回什么
2. 错误尽早暴露：类型错误在编辑时就能发现，不用等运行
3. 自动补全更准确：IDE 和 AI 都能给出更好的建议

**基本类型标注**：

```python
from typing import Protocol
from collections.abc import Sequence

def greet(name: str) -> str:
    return f"Hello, {name}"

def process_items(items: Sequence[int]) -> list[int]:
    return [x * 2 for x in items]

class UserRepository(Protocol):
    def get(self, user_id: int) -> User | None: ...
    def save(self, user: User) -> None: ...
```

**配置 pyright**（推荐）：

在 `pyproject.toml` 中：

```toml
[tool.pyright]
pythonVersion = "3.12"
typeCheckingMode = "basic"  # 或 "standard"、"strict"
```

运行检查：

```bash
uv run pyright
```

**或配置 mypy**：

```toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_ignores = true
disallow_untyped_defs = true
```

运行检查：

```bash
uv run mypy .
```

### 7.4 让错误尽早暴露

**三层反馈循环**：

| 阶段 | 工具 | 反馈速度 |
|-----|------|---------|
| 编辑时 | ruff (lint) + pyright (类型) | 即时（编辑器集成） |
| 保存时 | 编辑器自动运行 | 1-2 秒 |
| 提交时 | CI（lint + type + test） | 1-5 分钟 |

**编辑器配置**（VS Code / Cursor）：

安装扩展：
- `charliermarsh.ruff`（Ruff 官方扩展）
- `ms-pyright.pyright`（或使用 Pylance）

配置 `.vscode/settings.json`：

```json
{
    "[python]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "charliermarsh.ruff",
        "editor.codeActionsOnSave": {
            "source.fixAll": "explicit",
            "source.organizeImports": "explicit"
        }
    },
    "python.analysis.typeCheckingMode": "basic"
}
```

**测试快速反馈**：

```bash
# 遇到第一个失败就停止
uv run pytest -x

# 只运行上次失败的测试
uv run pytest --lf

# 监视模式（需要 pytest-watch）
uv run ptw
```

### 7.5 Docstring：让 AI 读懂意图

AI 编程（Cursor/Copilot）不仅依赖类型，更依赖**语义上下文**。**Docstring 就是给 AI 的 Prompt**。

> **推荐风格**：Google Style（`Args:`, `Returns:`, `Example:`）。这种格式结构清晰，主流 AI 编程工具（Cursor/Copilot）解析效果最佳。

**原则：意图优先的文档**

不要写 "Adds a and b" 这种描述"做什么"的注释，要写"为什么"和"业务含义"。

```python
# 不好：描述了显而易见的事情
def add(a: int, b: int) -> int:
    """Adds a and b."""
    return a + b


# 好：描述业务意图，AI 理解后能正确维护
def calculate_total_with_tax(subtotal: int, tax_rate: float) -> int:
    """
    计算含税总价。
    
    根据 2024 年税务规定，税率应用于税前金额。
    金额单位为分（cents），避免浮点数精度问题。
    
    Args:
        subtotal: 税前金额（单位：分）
        tax_rate: 税率，如 0.13 表示 13%
    
    Returns:
        含税总价（单位：分），向下取整
    
    Example:
        >>> calculate_total_with_tax(10000, 0.13)
        11300
    """
    return int(subtotal * (1 + tax_rate))
```

**核心业务逻辑必须写 Docstring**：

```python
class OrderService:
    async def create_order(
        self,
        user_id: int,
        items: list[OrderItemInput],
        coupon_code: str | None = None,
    ) -> Order:
        """
        创建订单。
        
        业务流程：
        1. 验证用户存在且状态正常
        2. 验证库存充足（会预扣库存）
        3. 应用优惠券（如有）
        4. 生成订单号并持久化
        5. 发送订单创建事件（异步通知库存、营销等系统）
        
        注意：
        - 库存在此处预扣，支付成功后正式扣减
        - 优惠券在此处标记使用，支付失败后自动释放
        
        Raises:
            NotFoundError: 用户不存在
            InsufficientStockError: 库存不足
            InvalidCouponError: 优惠券无效或已过期
        """
        ...
```

**显式导入（Explicit Imports）**：

避免 `from module import *`，这不仅是代码风格问题，更是为了让 AI 准确分析依赖关系。

```python
# 不好：AI 无法知道 Order 从哪来
from models import *

def create_order() -> Order:
    ...


# 好：AI 能准确理解依赖
from my_project.domain.entities import Order, OrderItem
from my_project.domain.value_objects import Money, OrderStatus

def create_order() -> Order:
    ...
```

**关键原则**：
- **Docstring 写意图**：描述"为什么"和业务规则，而非"做什么"
- **显式导入**：让 AI 能追踪依赖关系
- **命名自解释**：`calculate_tax_for_2024_regulation()` 比 `calc_tax()` 好
- **Example 很有价值**：给 AI（和人类）展示预期行为

### 7.6 CI 最小闭环

GitHub Actions 示例（`.github/workflows/ci.yml`）：

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v4

      - name: Set up Python
        run: uv python install

      - name: Install dependencies
        run: uv sync --frozen

      - name: Lint
        run: uv run ruff check .

      - name: Format check
        run: uv run ruff format . --check

      - name: Type check
        run: uv run pyright

      - name: Test
        run: uv run pytest --cov
```

**关键点**：
- `uv sync --frozen`：确保 CI 用的是锁定版本，和本地一致
- 每个步骤单独运行：失败时容易定位问题
- 顺序：lint → format → type → test（快的先跑，快速失败）

---

## 总结

一个好的 Python 项目应该：

1. **结构清晰**：分层组织，依赖指向内层
2. **边界明确**：模块间通过接口通信，可独立变化
3. **异步优先**：I/O 接口（Repository、外部服务）默认 async，保持扩展性
4. **事务一致**：用 Unit of Work 管理事务边界，避免数据不一致
5. **可测试**：依赖注入（Composition Root），副作用隔离
6. **可观测**：结构化日志（structlog），生产环境可排查
7. **持续演进**：小步重构，测试护航
8. **反馈及时**：lint + type + test，错误尽早暴露
9. **AI 友好**：Docstring 写意图、显式导入、命名自解释

**关键决策速查**：

| 场景 | 推荐做法 |
|-----|---------|
| 错误处理 | 自定义异常体系（Pythonic），慎用 Result 模式 |
| 依赖注入 | 手动组装（Composition Root），FastAPI 用原生 Depends |
| 日志 | structlog 结构化日志，JSON 格式 |
| I/O 接口 | 默认 async def |
| 事务管理 | Service 层控制，Repository 不 commit |

工具是手段，原则是目的。选对工具，坚持原则，项目才能长期健康。
