# 10 - SOLID 原則

> ⭐⭐⭐ 這是連接 OOP 和 System Design 的關鍵章節!

## 本章目標 (Learning Objectives)
- 深入理解 SOLID 五大原則
- 學會識別違反 SOLID 原則的程式碼
- 掌握重構技巧使程式碼符合 SOLID
- 將 SOLID 原則應用於系統設計
- 為大型系統架構打下堅實基礎

## 為什麼需要這個? (Motivation)

### 問題場景:電商系統的演進

假設你正在開發一個電商系統,隨著業務增長,程式碼變得越來越難維護:

```python
# ❌ 違反多個 SOLID 原則的程式碼
class OrderService:
    def __init__(self):
        self.db = Database()

    def create_order(self, user_id, product_id, quantity):
        # 驗證用戶
        user = self.db.query(f"SELECT * FROM users WHERE id={user_id}")
        if not user:
            raise Exception("用戶不存在")

        # 驗證商品
        product = self.db.query(f"SELECT * FROM products WHERE id={product_id}")
        if not product or product['stock'] < quantity:
            raise Exception("商品庫存不足")

        # 計算價格
        price = product['price'] * quantity
        if user['is_vip']:
            price = price * 0.9  # VIP 9折

        # 扣庫存
        new_stock = product['stock'] - quantity
        self.db.execute(f"UPDATE products SET stock={new_stock} WHERE id={product_id}")

        # 創建訂單
        self.db.execute(f"INSERT INTO orders VALUES (...)")

        # 發送郵件
        self.send_email(user['email'], "訂單已創建")

        # 發送簡訊
        self.send_sms(user['phone'], "訂單已創建")

        # 記錄日誌
        with open('orders.log', 'a') as f:
            f.write(f"Order created: {user_id}, {product_id}\n")

    def send_email(self, email, message):
        # 發送郵件邏輯
        pass

    def send_sms(self, phone, message):
        # 發送簡訊邏輯
        pass
```

**這段程式碼的問題:**
1. 一個類別做太多事情(違反 SRP)
2. 新增支付方式需要修改現有程式碼(違反 OCP)
3. 直接依賴具體實作(違反 DIP)
4. 難以測試、難以維護、難以擴展

### SOLID 原則的解決方案

使用 SOLID 原則重構後:

```python
# ✅ 符合 SOLID 原則的程式碼
class OrderService:
    def __init__(self,
                 user_repo: UserRepository,
                 product_repo: ProductRepository,
                 order_repo: OrderRepository,
                 price_calculator: PriceCalculator,
                 notifier: Notifier):
        self.user_repo = user_repo
        self.product_repo = product_repo
        self.order_repo = order_repo
        self.price_calculator = price_calculator
        self.notifier = notifier

    def create_order(self, user_id, product_id, quantity):
        # 每個職責都委託給專門的類別
        user = self.user_repo.find_by_id(user_id)
        product = self.product_repo.find_by_id(product_id)

        price = self.price_calculator.calculate(user, product, quantity)
        order = Order(user_id, product_id, quantity, price)

        self.order_repo.save(order)
        self.notifier.notify(user, "訂單已創建")

        return order
```

現在程式碼:
- 易於理解
- 易於測試
- 易於擴展
- 易於維護

## 核心概念 (Core Concepts)

## 一、Single Responsibility Principle (SRP) - 單一職責原則

### 定義
**一個類別應該只有一個引起它變化的原因。**

換句話說:一個類別只做一件事,只有一個職責。

### 為什麼需要 SRP?

**問題:**
- 職責混雜導致程式碼耦合
- 修改一個功能可能影響其他功能
- 難以測試和重用

**好處:**
- 程式碼更清晰
- 更容易測試
- 更容易維護
- 更容易重用

### 實戰範例

#### ❌ 違反 SRP

```python
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save(self):
        """職責1:資料持久化"""
        db = Database()
        db.execute(f"INSERT INTO users VALUES ('{self.name}', '{self.email}')")

    def send_welcome_email(self):
        """職責2:發送郵件"""
        smtp = SMTP('smtp.gmail.com')
        smtp.send(self.email, "歡迎註冊")

    def generate_report(self):
        """職責3:生成報表"""
        return f"用戶報表: {self.name}, {self.email}"

    def validate_email(self):
        """職責4:驗證"""
        return '@' in self.email
```

**問題:** User 類別有4個職責,任何一個職責的變化都會導致這個類別需要修改。

#### ✅ 符合 SRP

```python
# 1. 用戶實體 - 職責:表示用戶數據
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

# 2. 資料持久化 - 職責:儲存用戶
class UserRepository:
    def __init__(self, db):
        self.db = db

    def save(self, user: User):
        self.db.execute(
            "INSERT INTO users VALUES (?, ?)",
            (user.name, user.email)
        )

    def find_by_email(self, email: str):
        result = self.db.query(
            "SELECT * FROM users WHERE email=?",
            (email,)
        )
        return User(result['name'], result['email']) if result else None

# 3. 郵件服務 - 職責:發送郵件
class EmailService:
    def __init__(self, smtp_config):
        self.smtp = SMTP(smtp_config)

    def send_welcome_email(self, user: User):
        self.smtp.send(
            to=user.email,
            subject="歡迎註冊",
            body=f"歡迎 {user.name}!"
        )

# 4. 報表生成器 - 職責:生成報表
class UserReportGenerator:
    def generate(self, user: User):
        return f"用戶報表: {user.name}, {user.email}"

# 5. 驗證器 - 職責:驗證數據
class EmailValidator:
    @staticmethod
    def validate(email: str):
        return '@' in email and '.' in email.split('@')[1]

# 使用
user = User("張三", "zhang@example.com")

# 每個職責由專門的類別處理
user_repo = UserRepository(db)
email_service = EmailService(smtp_config)
report_gen = UserReportGenerator()
validator = EmailValidator()

if validator.validate(user.email):
    user_repo.save(user)
    email_service.send_welcome_email(user)
    report = report_gen.generate(user)
```

### SRP 實戰:電商訂單系統

```python
from dataclasses import dataclass
from typing import List
from abc import ABC, abstractmethod

# ===== 實體(只負責數據表示) =====
@dataclass
class Order:
    id: str
    user_id: str
    items: List[dict]
    total_amount: float
    status: str = "pending"

# ===== 訂單驗證(職責:驗證訂單數據) =====
class OrderValidator:
    def validate(self, order: Order) -> bool:
        if not order.items:
            raise ValueError("訂單不能為空")
        if order.total_amount <= 0:
            raise ValueError("訂單金額必須大於0")
        return True

# ===== 價格計算(職責:計算價格) =====
class PriceCalculator:
    def calculate_total(self, items: List[dict]) -> float:
        return sum(item['price'] * item['quantity'] for item in items)

    def apply_discount(self, amount: float, discount_rate: float) -> float:
        return amount * (1 - discount_rate)

# ===== 庫存管理(職責:管理庫存) =====
class InventoryManager:
    def __init__(self, inventory_repo):
        self.inventory_repo = inventory_repo

    def check_availability(self, items: List[dict]) -> bool:
        for item in items:
            stock = self.inventory_repo.get_stock(item['product_id'])
            if stock < item['quantity']:
                return False
        return True

    def reserve(self, items: List[dict]):
        for item in items:
            self.inventory_repo.decrease_stock(
                item['product_id'],
                item['quantity']
            )

# ===== 訂單持久化(職責:儲存訂單) =====
class OrderRepository:
    def __init__(self, db):
        self.db = db

    def save(self, order: Order):
        self.db.execute("INSERT INTO orders ...")

    def find_by_id(self, order_id: str) -> Order:
        result = self.db.query("SELECT * FROM orders WHERE id=?", (order_id,))
        return self._map_to_order(result)

# ===== 通知服務(職責:發送通知) =====
class NotificationService:
    def __init__(self, email_service, sms_service):
        self.email_service = email_service
        self.sms_service = sms_service

    def notify_order_created(self, order: Order, user):
        self.email_service.send(user.email, f"訂單 {order.id} 已創建")
        self.sms_service.send(user.phone, f"訂單已創建")

# ===== 訂單服務(職責:協調訂單流程) =====
class OrderService:
    def __init__(self,
                 validator: OrderValidator,
                 price_calc: PriceCalculator,
                 inventory: InventoryManager,
                 order_repo: OrderRepository,
                 notifier: NotificationService):
        self.validator = validator
        self.price_calc = price_calc
        self.inventory = inventory
        self.order_repo = order_repo
        self.notifier = notifier

    def create_order(self, user_id: str, items: List[dict], user) -> Order:
        # 計算總價
        total = self.price_calc.calculate_total(items)

        # 創建訂單
        order = Order(
            id=self._generate_id(),
            user_id=user_id,
            items=items,
            total_amount=total
        )

        # 驗證
        self.validator.validate(order)

        # 檢查庫存
        if not self.inventory.check_availability(items):
            raise ValueError("庫存不足")

        # 預留庫存
        self.inventory.reserve(items)

        # 儲存訂單
        self.order_repo.save(order)

        # 發送通知
        self.notifier.notify_order_created(order, user)

        return order
```

## 二、Open/Closed Principle (OCP) - 開放封閉原則

### 定義
**軟體實體應該對擴展開放,對修改封閉。**

換句話說:可以添加新功能,但不修改現有程式碼。

### 為什麼需要 OCP?

**問題:**
- 修改現有程式碼可能引入 bug
- 影響已測試的功能
- 增加維護成本

**好處:**
- 降低風險
- 提高穩定性
- 易於擴展

### 實戰範例

#### ❌ 違反 OCP

```python
class DiscountCalculator:
    def calculate(self, order, customer_type):
        if customer_type == "regular":
            return order.total * 1.0
        elif customer_type == "vip":
            return order.total * 0.9
        elif customer_type == "gold":
            return order.total * 0.8
        elif customer_type == "platinum":  # 新增類型需要修改這裡
            return order.total * 0.7
        # 每次新增客戶類型都要修改這個方法
```

**問題:** 每次新增客戶類型都需要修改 `calculate` 方法。

#### ✅ 符合 OCP

```python
from abc import ABC, abstractmethod

# 1. 定義抽象基類
class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, order):
        pass

# 2. 具體實作(對擴展開放)
class RegularDiscount(DiscountStrategy):
    def calculate(self, order):
        return order.total * 1.0

class VIPDiscount(DiscountStrategy):
    def calculate(self, order):
        return order.total * 0.9

class GoldDiscount(DiscountStrategy):
    def calculate(self, order):
        return order.total * 0.8

# 3. 新增折扣類型(不需要修改現有程式碼)
class PlatinumDiscount(DiscountStrategy):
    def calculate(self, order):
        return order.total * 0.7

class SeasonalDiscount(DiscountStrategy):
    def calculate(self, order):
        return order.total * 0.85

# 4. 使用策略模式
class Order:
    def __init__(self, total, discount_strategy: DiscountStrategy):
        self.total = total
        self.discount_strategy = discount_strategy

    def get_final_price(self):
        return self.discount_strategy.calculate(self)

# 使用
order1 = Order(1000, VIPDiscount())
order2 = Order(1000, PlatinumDiscount())

print(order1.get_final_price())  # 900
print(order2.get_final_price())  # 700
```

### OCP 實戰:支付系統

```python
from abc import ABC, abstractmethod
from typing import Dict

# ===== 支付介面(穩定,不需要修改) =====
class PaymentMethod(ABC):
    @abstractmethod
    def pay(self, amount: float) -> Dict:
        """執行支付"""
        pass

    @abstractmethod
    def refund(self, transaction_id: str, amount: float) -> Dict:
        """執行退款"""
        pass

# ===== 具體支付實作(對擴展開放) =====
class CreditCardPayment(PaymentMethod):
    def __init__(self, card_number, cvv):
        self.card_number = card_number
        self.cvv = cvv

    def pay(self, amount: float):
        # 信用卡支付邏輯
        print(f"信用卡支付: ${amount}")
        return {"status": "success", "transaction_id": "CC123"}

    def refund(self, transaction_id: str, amount: float):
        print(f"信用卡退款: ${amount}")
        return {"status": "success"}

class PayPalPayment(PaymentMethod):
    def __init__(self, email):
        self.email = email

    def pay(self, amount: float):
        print(f"PayPal 支付: ${amount}")
        return {"status": "success", "transaction_id": "PP456"}

    def refund(self, transaction_id: str, amount: float):
        print(f"PayPal 退款: ${amount}")
        return {"status": "success"}

# ===== 新增支付方式(不需要修改現有程式碼) =====
class ApplePayPayment(PaymentMethod):
    def __init__(self, device_id):
        self.device_id = device_id

    def pay(self, amount: float):
        print(f"Apple Pay 支付: ${amount}")
        return {"status": "success", "transaction_id": "AP789"}

    def refund(self, transaction_id: str, amount: float):
        print(f"Apple Pay 退款: ${amount}")
        return {"status": "success"}

class CryptoPayment(PaymentMethod):
    def __init__(self, wallet_address):
        self.wallet_address = wallet_address

    def pay(self, amount: float):
        print(f"加密貨幣支付: ${amount}")
        return {"status": "success", "transaction_id": "BTC101"}

    def refund(self, transaction_id: str, amount: float):
        print(f"加密貨幣退款: ${amount}")
        return {"status": "success"}

# ===== 支付處理器(不需要修改) =====
class PaymentProcessor:
    def process_payment(self, payment_method: PaymentMethod, amount: float):
        """統一的支付處理流程"""
        result = payment_method.pay(amount)

        if result['status'] == 'success':
            self._log_transaction(result['transaction_id'], amount)
            self._send_receipt(amount)

        return result

    def process_refund(self, payment_method: PaymentMethod,
                      transaction_id: str, amount: float):
        """統一的退款處理流程"""
        result = payment_method.refund(transaction_id, amount)

        if result['status'] == 'success':
            self._log_refund(transaction_id, amount)

        return result

    def _log_transaction(self, transaction_id, amount):
        print(f"記錄交易: {transaction_id}, ${amount}")

    def _send_receipt(self, amount):
        print(f"發送收據: ${amount}")

    def _log_refund(self, transaction_id, amount):
        print(f"記錄退款: {transaction_id}, ${amount}")

# ===== 使用範例 =====
processor = PaymentProcessor()

# 使用不同的支付方式(不需要修改 PaymentProcessor)
payment_methods = [
    CreditCardPayment("1234-5678-9012-3456", "123"),
    PayPalPayment("user@example.com"),
    ApplePayPayment("iPhone-12345"),
    CryptoPayment("1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa")
]

for method in payment_methods:
    processor.process_payment(method, 1000)
    print()
```

## 三、Liskov Substitution Principle (LSP) - 里氏替換原則

### 定義
**子類別必須能夠替換其父類別,而不影響程式的正確性。**

### 為什麼需要 LSP?

**問題:**
- 子類別行為與父類別不一致
- 多型失效
- 程式邏輯錯誤

**好處:**
- 保證繼承的正確性
- 確保多型有效
- 提高程式可靠性

### 實戰範例

#### ❌ 違反 LSP

```python
class Bird:
    def fly(self):
        return "飛行中"

class Sparrow(Bird):
    def fly(self):
        return "麻雀飛行中"

class Penguin(Bird):
    def fly(self):
        raise Exception("企鵝不會飛!")  # 違反 LSP!

# 問題
def make_bird_fly(bird: Bird):
    return bird.fly()  # 期望所有 Bird 都能飛

sparrow = Sparrow()
penguin = Penguin()

print(make_bird_fly(sparrow))  # OK
print(make_bird_fly(penguin))  # Exception! 破壞了替換性
```

#### ✅ 符合 LSP

```python
from abc import ABC, abstractmethod

# 重新設計繼承結構
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class FlyingBird(Bird):
    def move(self):
        return self.fly()

    def fly(self):
        return "飛行中"

class FlightlessBird(Bird):
    def move(self):
        return self.walk()

    def walk(self):
        return "行走中"

class Sparrow(FlyingBird):
    def fly(self):
        return "麻雀飛行中"

class Penguin(FlightlessBird):
    def walk(self):
        return "企鵝走路中"

# 現在可以安全替換
def make_bird_move(bird: Bird):
    return bird.move()

sparrow = Sparrow()
penguin = Penguin()

print(make_bird_move(sparrow))  # 麻雀飛行中
print(make_bird_move(penguin))  # 企鵝走路中
```

### LSP 實戰:幾何形狀

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self._width = width
        self._height = height

    def area(self) -> float:
        return self._width * self._height

    @property
    def width(self):
        return self._width

    @width.setter
    def width(self, value):
        self._width = value

    @property
    def height(self):
        return self._height

    @height.setter
    def height(self, value):
        self._height = value

class Square(Shape):
    """不繼承 Rectangle,避免違反 LSP"""
    def __init__(self, side):
        self._side = side

    def area(self) -> float:
        return self._side ** 2

    @property
    def side(self):
        return self._side

    @side.setter
    def side(self, value):
        self._side = value

# 使用
def print_area(shape: Shape):
    print(f"面積: {shape.area()}")

rect = Rectangle(10, 5)
square = Square(10)

print_area(rect)    # 面積: 50
print_area(square)  # 面積: 100
```

## 四、Interface Segregation Principle (ISP) - 介面隔離原則

### 定義
**客戶端不應該依賴它不需要的介面。**

換句話說:將大介面拆分成小介面。

### 實戰範例

#### ❌ 違反 ISP

```python
class Worker(ABC):
    @abstractmethod
    def work(self):
        pass

    @abstractmethod
    def eat(self):
        pass

    @abstractmethod
    def sleep(self):
        pass

class Human(Worker):
    def work(self):
        print("人類工作")

    def eat(self):
        print("人類吃飯")

    def sleep(self):
        print("人類睡覺")

class Robot(Worker):
    def work(self):
        print("機器人工作")

    def eat(self):
        pass  # 機器人不需要吃飯!

    def sleep(self):
        pass  # 機器人不需要睡覺!
```

#### ✅ 符合 ISP

```python
from abc import ABC, abstractmethod

# 拆分成小介面
class Workable(ABC):
    @abstractmethod
    def work(self):
        pass

class Eatable(ABC):
    @abstractmethod
    def eat(self):
        pass

class Sleepable(ABC):
    @abstractmethod
    def sleep(self):
        pass

# 各自實作需要的介面
class Human(Workable, Eatable, Sleepable):
    def work(self):
        print("人類工作")

    def eat(self):
        print("人類吃飯")

    def sleep(self):
        print("人類睡覺")

class Robot(Workable):
    def work(self):
        print("機器人工作")
```

## 五、Dependency Inversion Principle (DIP) - 依賴反轉原則

### 定義
**高層模組不應該依賴低層模組,兩者都應該依賴抽象。**

### 實戰範例

#### ❌ 違反 DIP

```python
class MySQLDatabase:
    def connect(self):
        print("連接 MySQL")

    def query(self, sql):
        print(f"執行 SQL: {sql}")

class UserService:
    def __init__(self):
        self.db = MySQLDatabase()  # 直接依賴具體實作

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")

# 問題:如果要切換到 PostgreSQL,必須修改 UserService
```

#### ✅ 符合 DIP

```python
from abc import ABC, abstractmethod

# 1. 定義抽象
class Database(ABC):
    @abstractmethod
    def connect(self):
        pass

    @abstractmethod
    def query(self, sql):
        pass

# 2. 具體實作
class MySQLDatabase(Database):
    def connect(self):
        print("連接 MySQL")

    def query(self, sql):
        print(f"MySQL: {sql}")
        return []

class PostgreSQLDatabase(Database):
    def connect(self):
        print("連接 PostgreSQL")

    def query(self, sql):
        print(f"PostgreSQL: {sql}")
        return []

# 3. 高層模組依賴抽象
class UserService:
    def __init__(self, database: Database):  # 依賴注入
        self.db = database

    def get_user(self, user_id):
        return self.db.query(f"SELECT * FROM users WHERE id={user_id}")

# 使用
mysql = MySQLDatabase()
postgres = PostgreSQLDatabase()

service1 = UserService(mysql)      # 使用 MySQL
service2 = UserService(postgres)   # 切換到 PostgreSQL,不需修改 UserService
```

## 實戰範例:完整的電商系統

整合所有 SOLID 原則的完整範例請參考:
- [System-design/SOLID/](../../System-design/SOLID/)

## 與 System Design 的連結

SOLID 原則是設計大型系統的基石:

1. **SRP** → 微服務設計(每個服務單一職責)
2. **OCP** → 插件架構、可擴展系統
3. **LSP** → API 版本兼容性
4. **ISP** → API 設計、服務拆分
5. **DIP** → 依賴注入、鬆耦合架構

## 練習題

### 基礎練習
1. 識別違反 SRP 的程式碼並重構
2. 使用策略模式實作符合 OCP 的折扣系統
3. 設計符合 LSP 的繼承結構

### 進階練習
4. 重構電商系統使其符合所有 SOLID 原則
5. 設計一個可擴展的日誌系統
6. 設計一個支援多種資料庫的 ORM

## 總結

SOLID 原則是寫出高品質程式碼的關鍵,也是通向 System Design 的橋樑。

**下一章預告:** 設計模式入門 - 學習常用的設計模式來解決常見問題。

**延伸閱讀:**
- [SOLID 深度實戰指南](../../System-design/SOLID/03-深度指南/)
- [SOLID 綜合實戰案例](../../System-design/SOLID/04-綜合案例/)
