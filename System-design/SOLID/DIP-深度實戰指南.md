# 依賴倒置原則 (DIP) - 深度實戰指南

> "高層模組不應該依賴低層模組,兩者都應該依賴抽象" - Robert C. Martin

## 目錄
- [核心理念](#核心理念)
- [設計經驗](#設計經驗)
- [依賴注入模式](#依賴注入模式)
- [架構層面的 DIP](#架構層面的-dip)
- [實戰技巧](#實戰技巧)
- [無招勝有招 - 依賴流動的藝術](#無招勝有招---依賴流動的藝術)

---

## 核心理念

### 什麼是「依賴倒置」?

傳統依賴: 高層 → 低層 (具體實現)
倒置後: 高層 → 抽象 ← 低層

```python
# ❌ 傳統依賴:高層依賴低層
class MySQLDatabase:
    def save(self, data):
        print(f"Saving to MySQL: {data}")

class UserService:  # 高層
    def __init__(self):
        self.db = MySQLDatabase()  # 直接依賴低層實現

    def create_user(self, user):
        self.db.save(user)

# 問題:
# 1. UserService 緊密耦合 MySQLDatabase
# 2. 無法切換資料庫
# 3. 無法測試 (無法 mock 資料庫)

# ✅ 依賴倒置:都依賴抽象
class Database(ABC):  # 抽象
    @abstractmethod
    def save(self, data):
        pass

class MySQLDatabase(Database):  # 低層
    def save(self, data):
        print(f"Saving to MySQL: {data}")

class UserService:  # 高層
    def __init__(self, database: Database):  # 依賴抽象
        self.database = database

    def create_user(self, user):
        self.database.save(user)

# 優點:
# 1. UserService 不知道具體資料庫
# 2. 可以隨時切換資料庫
db = MySQLDatabase()
service = UserService(db)

# 3. 可以輕鬆測試
class MockDatabase(Database):
    def save(self, data):
        print(f"Mock save: {data}")

test_service = UserService(MockDatabase())
```

### DIP 的兩個關鍵原則

1. **高層模組不應該依賴低層模組,兩者都應該依賴抽象**
2. **抽象不應該依賴細節,細節應該依賴抽象**

```python
# 原則 1: 兩者都依賴抽象
class OrderProcessor:  # 高層
    def __init__(self, payment: PaymentGateway, notification: Notifier):
        self.payment = payment  # 依賴抽象
        self.notification = notification

class StripePayment(PaymentGateway):  # 低層,也依賴抽象
    pass

# 原則 2: 細節依賴抽象
class PaymentGateway(ABC):  # 抽象
    @abstractmethod
    def charge(self, amount):
        pass
    # ✓ 不包含任何技術細節

class StripePayment(PaymentGateway):  # 細節
    def charge(self, amount):
        # Stripe API 的具體實現
        stripe.Charge.create(amount=amount)
```

---

## 設計經驗

### 經驗 1: 識別高層與低層

**高層**: 業務邏輯,策略,用例
**低層**: 技術細節,I/O,框架,資料庫

```python
# 層次分析
# 高層:業務邏輯
class OrderService:
    def place_order(self, order):
        # 業務規則:驗證、計算、處理
        pass

# 中層:協調
class OrderRepository:
    def save(self, order):
        pass

# 低層:技術實現
class MySQLConnection:
    def execute(self, sql):
        pass

# ✅ 正確的依賴方向(向上依賴抽象)
class OrderService:
    def __init__(self, repository: OrderRepository):  # 依賴抽象
        self.repository = repository

class OrderRepositoryImpl(OrderRepository):
    def __init__(self, db: DatabaseConnection):  # 依賴抽象
        self.db = db

class MySQLConnection(DatabaseConnection):
    pass
```

### 經驗 2: 抽象的穩定性

**穩定的抽象,不穩定的實現**

```python
# ✓ 穩定的抽象:很少改變
class Logger(ABC):
    @abstractmethod
    def log(self, message: str, level: str):
        pass

# ✗ 不穩定的實現:經常改變
class FileLogger(Logger):
    def __init__(self, filename):  # 可能改成支援輪轉
        self.filename = filename

    def log(self, message, level):
        # 實現可能改變:格式、存儲方式等
        pass

class CloudLogger(Logger):
    def __init__(self, api_key, endpoint):  # 新的實現
        self.api_key = api_key
        self.endpoint = endpoint

    def log(self, message, level):
        # 完全不同的實現
        pass

# 高層代碼永遠依賴穩定的 Logger 抽象
class Application:
    def __init__(self, logger: Logger):
        self.logger = logger

    def run(self):
        self.logger.log("App started", "INFO")
```

### 經驗 3: 抽象屬於高層

**關鍵洞察**: 抽象應該放在高層模組中,而不是低層

```python
# ❌ 錯誤:抽象在低層模組
# infrastructure/database.py
class DatabaseInterface(ABC):
    @abstractmethod
    def execute_sql(self, sql):  # 洩漏了實現細節(SQL)
        pass

# application/service.py
from infrastructure.database import DatabaseInterface

class UserService:
    def __init__(self, db: DatabaseInterface):  # 高層依賴低層的抽象
        self.db = db

# ✅ 正確:抽象在高層模組
# application/repository.py
class UserRepository(ABC):  # 高層定義需求
    @abstractmethod
    def find_by_id(self, user_id: int) -> User:
        pass

    @abstractmethod
    def save(self, user: User):
        pass

# application/service.py
class UserService:
    def __init__(self, repository: UserRepository):  # 依賴自己的抽象
        self.repository = repository

# infrastructure/mysql_repository.py
from application.repository import UserRepository

class MySQLUserRepository(UserRepository):  # 低層實現高層的抽象
    def find_by_id(self, user_id: int) -> User:
        # MySQL 實現
        pass
```

---

## 依賴注入模式

### 模式 1: 構造器注入 (Constructor Injection)

**最推薦**: 依賴明確,強制性,易於測試

```python
class OrderService:
    def __init__(
        self,
        repository: OrderRepository,
        payment: PaymentGateway,
        notifier: Notifier
    ):
        # 依賴在構造時注入
        self.repository = repository
        self.payment = payment
        self.notifier = notifier

    def place_order(self, order):
        self.repository.save(order)
        self.payment.charge(order.total)
        self.notifier.send(order.customer.email)

# 使用
service = OrderService(
    MySQLOrderRepository(),
    StripePayment(),
    EmailNotifier()
)
```

### 模式 2: 方法注入 (Method Injection)

**適用**: 依賴不是每次都需要,或者依賴會變化

```python
class ReportGenerator:
    def generate(self, data, formatter: ReportFormatter):
        # 依賴作為方法參數注入
        formatted = formatter.format(data)
        return formatted

# 使用:每次可以用不同的 formatter
generator = ReportGenerator()
pdf_report = generator.generate(data, PDFFormatter())
excel_report = generator.generate(data, ExcelFormatter())
```

### 模式 3: 屬性注入 (Property Injection)

**較少用**: 依賴可選,或需要延遲設置

```python
class Service:
    def __init__(self):
        self._logger = None

    @property
    def logger(self):
        return self._logger or DefaultLogger()

    @logger.setter
    def logger(self, logger: Logger):
        self._logger = logger

# 使用
service = Service()
service.logger = CustomLogger()  # 可選設置
```

### 模式 4: DI 容器 (簡單實現)

```python
class DIContainer:
    def __init__(self):
        self._services = {}

    def register(self, interface, implementation):
        self._services[interface] = implementation

    def resolve(self, interface):
        implementation = self._services.get(interface)
        if implementation is None:
            raise ValueError(f"No implementation for {interface}")

        # 檢查是否是類別(需要實例化)
        if isinstance(implementation, type):
            # 獲取構造器參數
            import inspect
            sig = inspect.signature(implementation.__init__)
            dependencies = {}

            for param_name, param in sig.parameters.items():
                if param_name == 'self':
                    continue
                # 遞迴解析依賴
                param_type = param.annotation
                dependencies[param_name] = self.resolve(param_type)

            return implementation(**dependencies)
        else:
            return implementation

# 使用
container = DIContainer()

# 註冊
container.register(Database, MySQLDatabase)
container.register(Logger, FileLogger)
container.register(UserRepository, UserRepositoryImpl)

# 解析(自動注入依賴)
user_repo = container.resolve(UserRepository)
```

---

## 架構層面的 DIP

### 六邊形架構 (Hexagonal Architecture)

DIP 的架構體現:

```
┌─────────────────────────────────────┐
│         Application Core            │
│      (Business Logic)               │
│                                     │
│  ┌─────────────────────────┐       │
│  │   Domain Entities       │       │
│  └─────────────────────────┘       │
│  ┌─────────────────────────┐       │
│  │   Use Cases/Services    │       │
│  └─────────────────────────┘       │
│  ┌─────────────────────────┐       │
│  │   Ports (Interfaces)    │       │
│  └─────────────────────────┘       │
└──────────────┬──────────────────────┘
               │
       ┌───────┴───────┐
       │               │
   Adapters         Adapters
  (Implementations)
```

```python
# Core: 領域層(高層)
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

# Core: Port(抽象,定義在高層)
class UserRepository(ABC):
    @abstractmethod
    def save(self, user: User):
        pass

    @abstractmethod
    def find_by_email(self, email: str) -> Optional[User]:
        pass

# Core: 用例(高層業務邏輯)
class RegisterUserUseCase:
    def __init__(self, repository: UserRepository):
        self.repository = repository

    def execute(self, name, email):
        existing = self.repository.find_by_email(email)
        if existing:
            raise ValueError("Email already exists")

        user = User(name, email)
        self.repository.save(user)
        return user

# Infrastructure: Adapter(低層實現)
class MySQLUserRepository(UserRepository):
    def __init__(self, db_connection):
        self.db = db_connection

    def save(self, user: User):
        self.db.execute(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            (user.name, user.email)
        )

    def find_by_email(self, email: str) -> Optional[User]:
        result = self.db.query(
            "SELECT * FROM users WHERE email = ?",
            (email,)
        )
        if result:
            return User(result['name'], result['email'])
        return None

# Presentation: 另一個 Adapter
class WebController:
    def __init__(self, use_case: RegisterUserUseCase):
        self.use_case = use_case

    def register(self, request):
        try:
            user = self.use_case.execute(
                request.data['name'],
                request.data['email']
            )
            return {"status": "success", "user": user}
        except ValueError as e:
            return {"status": "error", "message": str(e)}

# 組裝(在程序入口)
db = MySQLConnection()
repository = MySQLUserRepository(db)
use_case = RegisterUserUseCase(repository)
controller = WebController(use_case)
```

### 清晰架構 (Clean Architecture)

```python
# 依賴方向:外層 → 內層

# 層 1: Entities (最內層,最穩定)
class Order:
    def __init__(self, items, customer):
        self.items = items
        self.customer = customer

    def total(self):
        return sum(item.price for item in self.items)

# 層 2: Use Cases
class PlaceOrderUseCase:
    def __init__(self, order_repo: OrderRepository, payment: PaymentGateway):
        self.order_repo = order_repo
        self.payment = payment

    def execute(self, order_data):
        order = Order(order_data['items'], order_data['customer'])

        # 業務規則
        if order.total() < 0:
            raise ValueError("Invalid order")

        # 呼叫 port
        self.order_repo.save(order)
        self.payment.charge(order.total())

        return order

# 層 3: Interface Adapters
class OrderController:
    def __init__(self, use_case: PlaceOrderUseCase):
        self.use_case = use_case

    def create_order(self, request):
        order_data = self._parse_request(request)
        order = self.use_case.execute(order_data)
        return self._format_response(order)

# 層 4: Frameworks & Drivers (最外層,最不穩定)
class FlaskApp:
    def __init__(self, controller: OrderController):
        self.controller = controller
        self.app = Flask(__name__)

        @self.app.route('/orders', methods=['POST'])
        def create_order():
            return self.controller.create_order(request)
```

---

## 實戰技巧

### 技巧 1: 接口隔離與 DIP 配合

```python
# ✅ 小接口 + DIP = 靈活性
class Readable(ABC):
    @abstractmethod
    def read(self):
        pass

class Writable(ABC):
    @abstractmethod
    def write(self, data):
        pass

# 服務只依賴需要的接口
class DataProcessor:
    def __init__(self, source: Readable, sink: Writable):
        self.source = source
        self.sink = sink

    def process(self):
        data = self.source.read()
        processed = self._transform(data)
        self.sink.write(processed)

# 實現可以實現一個或多個接口
class File(Readable, Writable):
    def read(self):
        return "file data"

    def write(self, data):
        print(f"Writing: {data}")

class APIClient(Readable):
    def read(self):
        return "api data"

# 靈活組合
processor1 = DataProcessor(File(), File())
processor2 = DataProcessor(APIClient(), File())
```

### 技巧 2: 工廠模式配合 DIP

```python
# 抽象工廠
class DatabaseFactory(ABC):
    @abstractmethod
    def create_connection(self) -> Database:
        pass

class MySQLFactory(DatabaseFactory):
    def create_connection(self):
        return MySQLDatabase()

class PostgreSQLFactory(DatabaseFactory):
    def create_connection(self):
        return PostgreSQLDatabase()

# 服務依賴抽象工廠
class Application:
    def __init__(self, db_factory: DatabaseFactory):
        self.db = db_factory.create_connection()

# 配置決定使用哪個工廠
if config.database == 'mysql':
    factory = MySQLFactory()
else:
    factory = PostgreSQLFactory()

app = Application(factory)
```

### 技巧 3: 配置外部化

```python
# config.py
class Config:
    DATABASE_TYPE = 'mysql'
    DATABASE_HOST = 'localhost'
    LOG_LEVEL = 'INFO'

# factory.py
class ServiceFactory:
    @staticmethod
    def create_database(config: Config) -> Database:
        if config.DATABASE_TYPE == 'mysql':
            return MySQLDatabase(config.DATABASE_HOST)
        elif config.DATABASE_TYPE == 'postgres':
            return PostgreSQLDatabase(config.DATABASE_HOST)
        else:
            raise ValueError(f"Unknown database: {config.DATABASE_TYPE}")

    @staticmethod
    def create_logger(config: Config) -> Logger:
        if config.LOG_LEVEL == 'DEBUG':
            return DebugLogger()
        else:
            return ProductionLogger()

# main.py
def main():
    config = Config()

    # 創建依賴
    db = ServiceFactory.create_database(config)
    logger = ServiceFactory.create_logger(config)

    # 注入依賴
    service = UserService(db, logger)

    # 運行應用
    app = Application(service)
    app.run()
```

### 技巧 4: 測試中的 DIP

```python
# 生產代碼
class UserService:
    def __init__(self, repository: UserRepository, mailer: Mailer):
        self.repository = repository
        self.mailer = mailer

    def register(self, email):
        user = User(email)
        self.repository.save(user)
        self.mailer.send_welcome(email)

# 測試代碼
class InMemoryUserRepository(UserRepository):
    def __init__(self):
        self.users = []

    def save(self, user):
        self.users.append(user)

class SpyMailer(Mailer):
    def __init__(self):
        self.sent_emails = []

    def send_welcome(self, email):
        self.sent_emails.append(email)

# 測試
def test_register_user():
    repo = InMemoryUserRepository()
    mailer = SpyMailer()
    service = UserService(repo, mailer)

    service.register("test@example.com")

    assert len(repo.users) == 1
    assert repo.users[0].email == "test@example.com"
    assert "test@example.com" in mailer.sent_emails
```

---

## 無招勝有招 - 依賴流動的藝術

### 心法 1: 依賴方向的直覺

**從**: "這個類需要資料庫,所以 import MySQLDatabase"
**到**: "這個類需要持久化能力,所以依賴 Repository 抽象"

訓練:每次寫 `import` 時,問自己:
- 我導入的是高層還是低層?
- 這個依賴穩定嗎?
- 能用抽象代替嗎?

```python
# ❌ 直接依賴
from infrastructure.mysql_db import MySQLDatabase

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()

# ✅ 依賴抽象
from domain.repository import UserRepository

class UserService:
    def __init__(self, repository: UserRepository):
        self.repository = repository
```

### 心法 2: 抽象的粒度感

**好的抽象**: 不太大(包含不需要的方法),不太小(太多接口)

```python
# ❌ 太大
class DataAccess(ABC):
    @abstractmethod
    def query(self, sql): pass

    @abstractmethod
    def execute(self, sql): pass

    @abstractmethod
    def begin_transaction(self): pass

    @abstractmethod
    def commit(self): pass

    @abstractmethod
    def rollback(self): pass

    @abstractmethod
    def backup(self): pass

    @abstractmethod
    def restore(self): pass
    # ... 20 個方法

# ❌ 太小
class Findable(ABC):
    @abstractmethod
    def find(self, id): pass

class Saveable(ABC):
    @abstractmethod
    def save(self, entity): pass

class Deleteable(ABC):
    @abstractmethod
    def delete(self, id): pass
# ... 需要實現 10 個接口

# ✅ 恰到好處
class Repository(ABC):
    @abstractmethod
    def find(self, id): pass

    @abstractmethod
    def save(self, entity): pass

    @abstractmethod
    def delete(self, id): pass

    @abstractmethod
    def find_all(self): pass
```

### 心法 3: 讓編譯器/類型系統幫你

使用類型提示,讓依賴關係明確:

```python
# ✅ 類型明確,依賴清晰
class OrderService:
    def __init__(
        self,
        repository: OrderRepository,  # 清楚:依賴 OrderRepository
        payment: PaymentGateway,       # 清楚:依賴 PaymentGateway
        logger: Logger                 # 清楚:依賴 Logger
    ):
        self.repository = repository
        self.payment = payment
        self.logger = logger

# IDE 會檢查:
service = OrderService(
    MySQLOrderRepository(),  # ✓ 符合 OrderRepository
    StripePayment(),         # ✓ 符合 PaymentGateway
    "not a logger"           # ✗ IDE 警告!
)
```

### 心法 4: 依賴圖的心智模型

在腦海中維護一個依賴圖:

```
Application Layer (穩定)
    ↓ 依賴
Domain Layer (最穩定)
    ↑ 依賴
Infrastructure Layer (不穩定)
```

**規則**: 箭頭只能向上(向穩定方向)

### 心法 5: 忘記 DIP

當你達到「無招」境界:
- 自然地先定義接口再實現
- 自然地將抽象放在高層
- 自然地通過構造器注入依賴
- 不再需要想「這符合 DIP 嗎」

**標誌**:
- 你的測試很容易寫
- 你的模組很容易替換
- 你的代碼很容易理解
- 依賴方向自然正確

這就是**依賴流動的藝術** - 依賴自然地朝向穩定,朝向抽象,朝向業務核心。

---

## 實戰練習

### 練習 1: 識別依賴方向

```python
# infrastructure/email_service.py
class EmailService:
    def send(self, to, subject, body):
        # 發送郵件
        pass

# application/user_service.py
from infrastructure.email_service import EmailService

class UserService:
    def __init__(self):
        self.email = EmailService()

    def register(self, user):
        # 註冊用戶
        self.email.send(user.email, "Welcome", "...")

# 問題:依賴方向對嗎?如何改進?
```

<details>
<summary>答案</summary>

依賴方向錯誤!高層(application)依賴了低層(infrastructure)。

改進:
```python
# application/mailer.py (抽象在高層)
class Mailer(ABC):
    @abstractmethod
    def send(self, to, subject, body):
        pass

# application/user_service.py
class UserService:
    def __init__(self, mailer: Mailer):
        self.mailer = mailer

    def register(self, user):
        self.mailer.send(user.email, "Welcome", "...")

# infrastructure/email_service.py (低層實現高層抽象)
from application.mailer import Mailer

class EmailService(Mailer):
    def send(self, to, subject, body):
        # 實現
        pass
```
</details>

### 練習 2: 重構緊耦合代碼

```python
class OrderProcessor:
    def __init__(self):
        self.db = MySQLDatabase()
        self.payment = StripeAPI()
        self.email = SMTPClient()

    def process(self, order):
        self.db.save(order)
        self.payment.charge(order.total)
        self.email.send(order.customer.email, "Order placed")
```

<details>
<summary>參考答案</summary>

```python
# 定義抽象(在業務層)
class OrderRepository(ABC):
    @abstractmethod
    def save(self, order): pass

class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount): pass

class Notifier(ABC):
    @abstractmethod
    def notify(self, recipient, message): pass

# 重構服務
class OrderProcessor:
    def __init__(
        self,
        repository: OrderRepository,
        payment: PaymentGateway,
        notifier: Notifier
    ):
        self.repository = repository
        self.payment = payment
        self.notifier = notifier

    def process(self, order):
        self.repository.save(order)
        self.payment.charge(order.total)
        self.notifier.notify(order.customer.email, "Order placed")

# 實現抽象
class MySQLOrderRepository(OrderRepository):
    def save(self, order):
        # MySQL 實現
        pass

class StripePayment(PaymentGateway):
    def charge(self, amount):
        # Stripe 實現
        pass

class EmailNotifier(Notifier):
    def notify(self, recipient, message):
        # Email 實現
        pass

# 組裝
processor = OrderProcessor(
    MySQLOrderRepository(),
    StripePayment(),
    EmailNotifier()
)
```
</details>

---

## 總結金句

1. **高層定義抽象,低層實現細節** - 抽象屬於高層
2. **依賴指向穩定** - 不穩定的依賴穩定的
3. **構造器注入** - 讓依賴顯式且強制
4. **小接口,大靈活性** - ISP + DIP
5. **架構體現 DIP** - 六邊形架構、清晰架構
6. **測試即設計驗證** - 易測試的設計符合 DIP

---

## DIP 與其他原則的關係

- **與 OCP**: DIP 是實現 OCP 的關鍵手段
- **與 LSP**: DIP 依賴抽象,LSP 確保子類可替換
- **與 ISP**: 小接口使 DIP 更靈活
- **與 SRP**: 單一職責使依賴更清晰

---

## 延伸閱讀

- Clean Architecture (Robert C. Martin)
- Dependency Injection in .NET (Mark Seemann)
- Growing Object-Oriented Software, Guided by Tests (Steve Freeman, Nat Pryce)

---

**記住**: DIP 的本質是**控制依賴的方向**。當你的依賴自然地流向抽象、流向穩定、流向業務核心時,你就達到了「無招勝有招」的境界 - 不再需要刻意思考依賴方向,因為它已經成為你的本能。
