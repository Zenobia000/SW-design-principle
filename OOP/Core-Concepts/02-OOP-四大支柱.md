# 02 - OOP 四大支柱

## 本章目標 (Learning Objectives)
- 深入理解 OOP 的四大核心原則
- 掌握封裝(Encapsulation)的實作技巧
- 學會使用繼承(Inheritance)來重用程式碼
- 理解多型(Polymorphism)帶來的靈活性
- 運用抽象(Abstraction)設計清晰的介面

## 為什麼需要這個? (Motivation)

### 問題場景:電商系統的演進

假設我們的電商系統需要支援不同類型的商品:

```python
# 沒有 OOP 四大支柱的程式碼
def calculate_price_physical(base_price, weight, distance):
    shipping = weight * distance * 10
    return base_price + shipping

def calculate_price_digital(base_price):
    return base_price  # 數位商品無運費

def calculate_price_subscription(monthly_price, months):
    return monthly_price * months

# 每次新增商品類型都要寫新函數,難以維護
# 無法統一處理不同類型的商品
```

使用 OOP 四大支柱後:

```python
# 使用封裝、繼承、多型、抽象
class Product(ABC):  # 抽象基類
    def __init__(self, name, price):
        self._name = name          # 封裝
        self._price = price

    @abstractmethod
    def calculate_final_price(self):  # 抽象方法
        pass

class PhysicalProduct(Product):  # 繼承
    def calculate_final_price(self):  # 多型
        return self._price + self._calculate_shipping()

class DigitalProduct(Product):  # 繼承
    def calculate_final_price(self):  # 多型
        return self._price

# 統一處理所有商品
def process_order(products):
    total = sum(p.calculate_final_price() for p in products)
    return total
```

## 核心概念 (Core Concepts)

## 一、封裝 (Encapsulation)

### 定義
將數據(屬性)和操作數據的方法包裝在一起,並隱藏內部實作細節,只暴露必要的介面。

### 為什麼需要封裝?
1. **數據保護:** 防止外部直接修改內部狀態
2. **降低耦合:** 外部不需要知道內部實作
3. **易於維護:** 修改內部實作不影響外部使用
4. **提高安全性:** 可以添加驗證邏輯

### Python 中的封裝實作

#### 1. 使用 `@property` 裝飾器

```python
class Product:
    def __init__(self, name, price, stock):
        self._name = name      # 受保護屬性 (慣例)
        self._price = price
        self._stock = stock

    @property
    def price(self):
        """Getter: 讀取價格"""
        return self._price

    @price.setter
    def price(self, value):
        """Setter: 設定價格(含驗證)"""
        if value < 0:
            raise ValueError("價格不能為負數")
        self._price = value

    @property
    def stock(self):
        """只讀屬性(沒有 setter)"""
        return self._stock

    def restock(self, quantity):
        """透過方法修改庫存"""
        if quantity > 0:
            self._stock += quantity

# 使用範例
product = Product("iPhone", 29900, 50)

# 透過 property 存取
print(product.price)  # 29900

# 透過 setter 修改(會執行驗證)
product.price = 25000  # OK
# product.price = -100  # ValueError: 價格不能為負數

# 無法直接修改 stock (沒有 setter)
# product.stock = 100  # AttributeError

# 只能透過方法修改
product.restock(20)
```

#### 2. 私有屬性和方法

```python
class BankAccount:
    def __init__(self, account_number, initial_balance):
        self.__account_number = account_number  # 私有屬性 (名稱改寫)
        self.__balance = initial_balance

    @property
    def balance(self):
        """公開介面:查詢餘額"""
        return self.__balance

    def deposit(self, amount):
        """公開介面:存款"""
        if self.__validate_amount(amount):
            self.__balance += amount
            self.__log_transaction("存款", amount)
            return True
        return False

    def withdraw(self, amount):
        """公開介面:提款"""
        if self.__validate_amount(amount) and amount <= self.__balance:
            self.__balance -= amount
            self.__log_transaction("提款", amount)
            return True
        return False

    def __validate_amount(self, amount):
        """私有方法:驗證金額"""
        return amount > 0

    def __log_transaction(self, type, amount):
        """私有方法:記錄交易"""
        print(f"[交易記錄] {type}: ${amount}")

# 使用範例
account = BankAccount("1234567890", 10000)

# 可以使用公開介面
account.deposit(5000)    # [交易記錄] 存款: $5000
account.withdraw(2000)   # [交易記錄] 提款: $2000
print(account.balance)   # 13000

# 無法直接存取私有屬性
# print(account.__balance)  # AttributeError

# 私有方法也無法直接調用
# account.__validate_amount(100)  # AttributeError
```

### 封裝的實戰應用

```python
class ShoppingCart:
    """購物車 - 展示完整的封裝"""

    def __init__(self, customer_id):
        self.__customer_id = customer_id
        self.__items = []  # 私有屬性
        self.__discount_rate = 0

    @property
    def item_count(self):
        """只讀:商品數量"""
        return len(self.__items)

    @property
    def subtotal(self):
        """只讀:小計"""
        return sum(item['price'] * item['quantity'] for item in self.__items)

    @property
    def total(self):
        """只讀:總計(含折扣)"""
        return self.subtotal * (1 - self.__discount_rate)

    def add_item(self, product_id, name, price, quantity=1):
        """公開介面:加入商品"""
        if not self.__validate_product(price, quantity):
            return False

        item = {
            'product_id': product_id,
            'name': name,
            'price': price,
            'quantity': quantity
        }
        self.__items.append(item)
        return True

    def apply_discount(self, code):
        """公開介面:使用折扣碼"""
        discount = self.__validate_discount_code(code)
        if discount:
            self.__discount_rate = discount
            return True
        return False

    def __validate_product(self, price, quantity):
        """私有方法:驗證商品"""
        return price > 0 and quantity > 0

    def __validate_discount_code(self, code):
        """私有方法:驗證折扣碼"""
        discount_codes = {
            'SAVE10': 0.1,
            'SAVE20': 0.2,
            'VIP30': 0.3
        }
        return discount_codes.get(code, None)

    def get_items(self):
        """公開介面:獲取商品列表(返回副本,保護內部數據)"""
        return self.__items.copy()
```

## 二、繼承 (Inheritance)

### 定義
子類別繼承父類別的屬性和方法,實現程式碼重用和擴展。

### 為什麼需要繼承?
1. **程式碼重用:** 避免重複撰寫相同的程式碼
2. **層次結構:** 建立類別之間的關係
3. **擴展性:** 在不修改原有程式碼的情況下添加新功能
4. **多型基礎:** 為多型提供基礎

### 基本繼承語法

```python
class Product:
    """父類別:商品基類"""

    def __init__(self, product_id, name, price):
        self.product_id = product_id
        self.name = name
        self.price = price

    def get_info(self):
        return f"{self.name} - ${self.price}"

    def calculate_tax(self):
        return self.price * 0.05

class PhysicalProduct(Product):
    """子類別:實體商品"""

    def __init__(self, product_id, name, price, weight, dimensions):
        # 調用父類別的 __init__
        super().__init__(product_id, name, price)
        # 添加子類別特有的屬性
        self.weight = weight
        self.dimensions = dimensions

    def calculate_shipping(self, distance):
        """子類別特有的方法"""
        return self.weight * distance * 10

    def get_info(self):
        """覆寫父類別的方法"""
        base_info = super().get_info()
        return f"{base_info} (重量: {self.weight}kg)"

class DigitalProduct(Product):
    """子類別:數位商品"""

    def __init__(self, product_id, name, price, file_size, download_link):
        super().__init__(product_id, name, price)
        self.file_size = file_size
        self.download_link = download_link

    def calculate_tax(self):
        """覆寫:數位商品免稅"""
        return 0

    def get_download_info(self):
        """子類別特有的方法"""
        return f"檔案大小: {self.file_size}MB, 下載連結: {self.download_link}"

# 使用範例
physical = PhysicalProduct("P001", "MacBook Pro", 79900, 2.0, "35x24x1.6cm")
digital = DigitalProduct("D001", "Photoshop", 9800, 2500, "https://adobe.com/download")

print(physical.get_info())  # MacBook Pro - $79900 (重量: 2.0kg)
print(digital.get_info())   # Photoshop - $9800

print(physical.calculate_tax())  # 3995.0
print(digital.calculate_tax())   # 0

print(physical.calculate_shipping(10))  # 200.0
print(digital.get_download_info())  # 檔案大小: 2500MB, 下載連結: https://adobe.com/download
```

### 繼承的實戰應用:從火影忍者學繼承

```python
class Ninja:
    """父類別:忍者"""

    def __init__(self, name, village, rank):
        self.name = name
        self.village = village
        self.rank = rank
        self.chakra = 100

    def introduce(self):
        return f"我是{self.village}的{self.rank} - {self.name}"

    def basic_jutsu(self):
        return f"{self.name}使用基礎忍術"

class Uchiha(Ninja):
    """子類別:宇智波一族"""

    def __init__(self, name, village, rank, sharingan_level):
        super().__init__(name, village, rank)
        self.sharingan_level = sharingan_level
        self.clan = "宇智波"

    def sharingan(self):
        """宇智波特有能力"""
        if self.sharingan_level >= 3:
            return f"{self.name}開啟萬花筒寫輪眼!"
        return f"{self.name}開啟{self.sharingan_level}勾玉寫輪眼"

    def amaterasu(self):
        """高級忍術"""
        if self.sharingan_level >= 3:
            self.chakra -= 50
            return f"{self.name}釋放天照!"
        return "需要萬花筒寫輪眼"

class Uzumaki(Ninja):
    """子類別:漩渦一族"""

    def __init__(self, name, village, rank, tailed_beast=None):
        super().__init__(name, village, rank)
        self.chakra = 500  # 漩渦一族查克拉量大
        self.tailed_beast = tailed_beast
        self.clan = "漩渦"

    def shadow_clone(self, count):
        """影分身術"""
        chakra_cost = count * 10
        if self.chakra >= chakra_cost:
            self.chakra -= chakra_cost
            return f"{self.name}創造了{count}個影分身"
        return "查克拉不足"

    def rasengan(self):
        """螺旋丸"""
        self.chakra -= 30
        return f"{self.name}使用螺旋丸!"

# 使用範例
sasuke = Uchiha("佐助", "木葉忍者村", "上忍", 3)
naruto = Uzumaki("鳴人", "木葉忍者村", "上忍", "九尾")

print(sasuke.introduce())  # 我是木葉忍者村的上忍 - 佐助
print(sasuke.sharingan())  # 佐助開啟萬花筒寫輪眼!
print(sasuke.amaterasu())  # 佐助釋放天照!

print(naruto.introduce())  # 我是木葉忍者村的上忍 - 鳴人
print(naruto.shadow_clone(10))  # 鳴人創造了10個影分身
print(naruto.rasengan())  # 鳴人使用螺旋丸!
```

### super() 的正確用法

```python
class Vehicle:
    """交通工具基類"""

    def __init__(self, brand, model, year):
        self.brand = brand
        self.model = model
        self.year = year
        print(f"Vehicle.__init__ called for {brand} {model}")

    def start(self):
        return f"{self.brand} {self.model} 啟動"

class Car(Vehicle):
    """汽車類"""

    def __init__(self, brand, model, year, doors):
        super().__init__(brand, model, year)  # 調用父類別的 __init__
        self.doors = doors
        print(f"Car.__init__ called, doors: {doors}")

    def start(self):
        parent_start = super().start()  # 調用父類別的 start
        return f"{parent_start},共{self.doors}門"

class ElectricCar(Car):
    """電動汽車類"""

    def __init__(self, brand, model, year, doors, battery_capacity):
        super().__init__(brand, model, year, doors)
        self.battery_capacity = battery_capacity
        print(f"ElectricCar.__init__ called, battery: {battery_capacity}kWh")

    def start(self):
        parent_start = super().start()
        return f"{parent_start},電池容量{self.battery_capacity}kWh"

# 使用範例
tesla = ElectricCar("Tesla", "Model 3", 2024, 4, 75)
# 輸出:
# Vehicle.__init__ called for Tesla Model 3
# Car.__init__ called, doors: 4
# ElectricCar.__init__ called, battery: 75kWh

print(tesla.start())
# Tesla Model 3 啟動,共4門,電池容量75kWh
```

## 三、多型 (Polymorphism)

### 定義
同一個介面可以有不同的實作,不同的類別可以用統一的方式處理。

### 為什麼需要多型?
1. **靈活性:** 可以用統一的方式處理不同類型的物件
2. **可擴展性:** 新增類別不需要修改現有程式碼
3. **降低耦合:** 呼叫者不需要知道物件的具體類型
4. **提高可維護性:** 程式碼更簡潔、易讀

### 多型的實作

```python
from abc import ABC, abstractmethod

class PaymentMethod(ABC):
    """支付方式基類"""

    @abstractmethod
    def pay(self, amount):
        """抽象方法:執行支付"""
        pass

    @abstractmethod
    def refund(self, amount):
        """抽象方法:執行退款"""
        pass

class CreditCard(PaymentMethod):
    """信用卡支付"""

    def __init__(self, card_number, cvv):
        self.card_number = card_number
        self.cvv = cvv

    def pay(self, amount):
        # 信用卡支付邏輯
        print(f"信用卡支付 ${amount}")
        print(f"卡號: {self.card_number[-4:].rjust(16, '*')}")
        return True

    def refund(self, amount):
        print(f"退款至信用卡 ${amount}")
        return True

class PayPal(PaymentMethod):
    """PayPal 支付"""

    def __init__(self, email):
        self.email = email

    def pay(self, amount):
        print(f"PayPal 支付 ${amount}")
        print(f"帳號: {self.email}")
        return True

    def refund(self, amount):
        print(f"退款至 PayPal 帳戶 ${amount}")
        return True

class ApplePay(PaymentMethod):
    """Apple Pay 支付"""

    def __init__(self, device_id):
        self.device_id = device_id

    def pay(self, amount):
        print(f"Apple Pay 支付 ${amount}")
        print(f"裝置 ID: {self.device_id}")
        return True

    def refund(self, amount):
        print(f"退款至 Apple Pay ${amount}")
        return True

# 多型的威力:統一處理不同的支付方式
def process_payment(payment_method: PaymentMethod, amount: float):
    """處理支付 - 不需要知道具體的支付方式"""
    print(f"\n開始處理支付...")
    if payment_method.pay(amount):
        print("支付成功!")
        return True
    print("支付失敗!")
    return False

def process_refund(payment_method: PaymentMethod, amount: float):
    """處理退款 - 不需要知道具體的支付方式"""
    print(f"\n開始處理退款...")
    if payment_method.refund(amount):
        print("退款成功!")
        return True
    print("退款失敗!")
    return False

# 使用範例
if __name__ == "__main__":
    # 創建不同的支付方式
    credit_card = CreditCard("1234567812345678", "123")
    paypal = PayPal("user@example.com")
    apple_pay = ApplePay("iPhone-12345")

    # 統一處理 - 多型的體現
    payment_methods = [credit_card, paypal, apple_pay]

    for method in payment_methods:
        process_payment(method, 1000)

    # 退款也是統一處理
    process_refund(credit_card, 500)
```

**輸出:**
```
開始處理支付...
信用卡支付 $1000
卡號: ************5678
支付成功!

開始處理支付...
PayPal 支付 $1000
帳號: user@example.com
支付成功!

開始處理支付...
Apple Pay 支付 $1000
裝置 ID: iPhone-12345
支付成功!

開始處理退款...
退款至信用卡 $500
退款成功!
```

### 多型的實戰應用

```python
class Animal(ABC):
    """動物基類"""

    def __init__(self, name):
        self.name = name

    @abstractmethod
    def make_sound(self):
        """發出聲音"""
        pass

    @abstractmethod
    def move(self):
        """移動方式"""
        pass

class Dog(Animal):
    def make_sound(self):
        return f"{self.name}: 汪汪汪!"

    def move(self):
        return f"{self.name}正在跑步"

class Cat(Animal):
    def make_sound(self):
        return f"{self.name}: 喵喵喵~"

    def move(self):
        return f"{self.name}正在優雅地走路"

class Bird(Animal):
    def make_sound(self):
        return f"{self.name}: 啾啾!"

    def move(self):
        return f"{self.name}正在飛翔"

# 多型應用:動物園管理
class Zoo:
    def __init__(self):
        self.animals = []

    def add_animal(self, animal: Animal):
        self.animals.append(animal)

    def morning_routine(self):
        """早晨例行公事 - 多型的應用"""
        print("=== 動物園的早晨 ===")
        for animal in self.animals:
            print(animal.make_sound())  # 不同動物有不同叫聲
            print(animal.move())        # 不同動物有不同移動方式
            print()

# 使用範例
zoo = Zoo()
zoo.add_animal(Dog("小黑"))
zoo.add_animal(Cat("小白"))
zoo.add_animal(Bird("小黃"))

zoo.morning_routine()
```

## 四、抽象 (Abstraction)

### 定義
隱藏複雜的實作細節,只暴露必要的介面和功能。

### 為什麼需要抽象?
1. **簡化複雜性:** 使用者不需要了解內部運作
2. **統一介面:** 定義標準,確保一致性
3. **強制實作:** 確保子類別實作必要的方法
4. **提高可維護性:** 修改實作不影響介面

### 抽象類別的實作

```python
from abc import ABC, abstractmethod

class DatabaseConnection(ABC):
    """資料庫連接抽象類別"""

    def __init__(self, host, port, database):
        self.host = host
        self.port = port
        self.database = database
        self.connection = None

    @abstractmethod
    def connect(self):
        """建立連接"""
        pass

    @abstractmethod
    def disconnect(self):
        """斷開連接"""
        pass

    @abstractmethod
    def execute_query(self, query):
        """執行查詢"""
        pass

    @abstractmethod
    def execute_update(self, query):
        """執行更新"""
        pass

    def get_connection_info(self):
        """具體方法:獲取連接資訊"""
        return f"{self.host}:{self.port}/{self.database}"

class MySQLConnection(DatabaseConnection):
    """MySQL 實作"""

    def connect(self):
        print(f"連接到 MySQL: {self.get_connection_info()}")
        self.connection = f"MySQL Connection to {self.database}"
        return True

    def disconnect(self):
        print(f"斷開 MySQL 連接: {self.database}")
        self.connection = None
        return True

    def execute_query(self, query):
        if not self.connection:
            raise Exception("尚未建立連接")
        print(f"MySQL 查詢: {query}")
        return ["result1", "result2"]

    def execute_update(self, query):
        if not self.connection:
            raise Exception("尚未建立連接")
        print(f"MySQL 更新: {query}")
        return True

class PostgreSQLConnection(DatabaseConnection):
    """PostgreSQL 實作"""

    def connect(self):
        print(f"連接到 PostgreSQL: {self.get_connection_info()}")
        self.connection = f"PostgreSQL Connection to {self.database}"
        return True

    def disconnect(self):
        print(f"斷開 PostgreSQL 連接: {self.database}")
        self.connection = None
        return True

    def execute_query(self, query):
        if not self.connection:
            raise Exception("尚未建立連接")
        print(f"PostgreSQL 查詢: {query}")
        return ["data1", "data2"]

    def execute_update(self, query):
        if not self.connection:
            raise Exception("尚未建立連接")
        print(f"PostgreSQL 更新: {query}")
        return True

# 使用抽象類別
class DatabaseManager:
    """資料庫管理器 - 依賴抽象而非具體實作"""

    def __init__(self, db_connection: DatabaseConnection):
        self.db = db_connection

    def initialize(self):
        """初始化資料庫"""
        self.db.connect()
        print("資料庫初始化完成")

    def get_users(self):
        """獲取用戶列表"""
        return self.db.execute_query("SELECT * FROM users")

    def update_user(self, user_id, data):
        """更新用戶資料"""
        query = f"UPDATE users SET data='{data}' WHERE id={user_id}"
        return self.db.execute_update(query)

    def cleanup(self):
        """清理資源"""
        self.db.disconnect()

# 使用範例
if __name__ == "__main__":
    # 可以輕鬆切換不同的資料庫實作
    mysql = MySQLConnection("localhost", 3306, "myapp")
    postgres = PostgreSQLConnection("localhost", 5432, "myapp")

    # 使用 MySQL
    manager = DatabaseManager(mysql)
    manager.initialize()
    users = manager.get_users()
    manager.cleanup()

    print("\n" + "="*50 + "\n")

    # 切換到 PostgreSQL - 程式碼不需要改變
    manager = DatabaseManager(postgres)
    manager.initialize()
    users = manager.get_users()
    manager.cleanup()
```

## 實戰範例 (Hands-on Examples)

### 完整範例:電商系統的四大支柱實戰

```python
from abc import ABC, abstractmethod
from datetime import datetime
from typing import List

# ===== 抽象層 =====
class Product(ABC):
    """商品抽象基類"""

    def __init__(self, product_id: str, name: str, price: float):
        self._product_id = product_id  # 封裝
        self._name = name
        self._price = price

    @property
    def product_id(self):
        return self._product_id

    @property
    def name(self):
        return self._name

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value: float):
        if value < 0:
            raise ValueError("價格不能為負數")
        self._price = value

    @abstractmethod  # 抽象
    def calculate_final_price(self) -> float:
        """計算最終價格 - 子類別必須實作"""
        pass

    @abstractmethod
    def get_delivery_info(self) -> str:
        """獲取配送資訊 - 子類別必須實作"""
        pass

# ===== 繼承層 =====
class PhysicalProduct(Product):
    """實體商品 - 繼承 Product"""

    def __init__(self, product_id: str, name: str, price: float,
                 weight: float, shipping_fee: float):
        super().__init__(product_id, name, price)
        self._weight = weight
        self._shipping_fee = shipping_fee

    def calculate_final_price(self) -> float:
        """多型:實體商品需加運費"""
        return self._price + self._shipping_fee

    def get_delivery_info(self) -> str:
        """多型:實體商品有配送時間"""
        return f"預計3-5個工作天送達,重量:{self._weight}kg"

class DigitalProduct(Product):
    """數位商品 - 繼承 Product"""

    def __init__(self, product_id: str, name: str, price: float,
                 file_size: int, download_link: str):
        super().__init__(product_id, name, price)
        self._file_size = file_size
        self._download_link = download_link

    def calculate_final_price(self) -> float:
        """多型:數位商品無運費"""
        return self._price

    def get_delivery_info(self) -> str:
        """多型:數位商品即時下載"""
        return f"立即下載,檔案大小:{self._file_size}MB"

class SubscriptionProduct(Product):
    """訂閱商品 - 繼承 Product"""

    def __init__(self, product_id: str, name: str, monthly_price: float,
                 billing_cycle: int):
        super().__init__(product_id, name, monthly_price)
        self._billing_cycle = billing_cycle

    def calculate_final_price(self) -> float:
        """多型:訂閱商品按週期計費"""
        return self._price * self._billing_cycle

    def get_delivery_info(self) -> str:
        """多型:訂閱商品無配送"""
        return f"訂閱{self._billing_cycle}個月,立即開通"

# ===== 購物車 =====
class ShoppingCart:
    """購物車 - 展示封裝和多型"""

    def __init__(self, customer_id: str):
        self.__customer_id = customer_id  # 封裝:私有屬性
        self.__items: List[Product] = []  # 封裝:私有屬性

    @property
    def item_count(self):
        """封裝:只讀屬性"""
        return len(self.__items)

    def add_product(self, product: Product):
        """多型:接受任何 Product 子類別"""
        self.__items.append(product)
        print(f"已加入: {product.name}")

    def calculate_total(self) -> float:
        """多型:統一調用不同商品的計算方法"""
        total = sum(item.calculate_final_price() for item in self.__items)
        return total

    def display_cart(self):
        """顯示購物車 - 多型應用"""
        print("\n" + "="*60)
        print(f"購物車詳情 (客戶 ID: {self.__customer_id})")
        print("="*60)

        for i, item in enumerate(self.__items, 1):
            print(f"\n{i}. {item.name}")
            print(f"   原價: ${item.price:,.2f}")
            print(f"   最終價格: ${item.calculate_final_price():,.2f}")
            print(f"   配送資訊: {item.get_delivery_info()}")

        print(f"\n{'-'*60}")
        print(f"總計: ${self.calculate_total():,.2f}")
        print("="*60 + "\n")

# ===== 使用範例 =====
if __name__ == "__main__":
    # 創建不同類型的商品(繼承)
    laptop = PhysicalProduct(
        "P001",
        "MacBook Pro 16\"",
        79900,
        2.0,
        150
    )

    software = DigitalProduct(
        "D001",
        "Adobe Creative Cloud",
        1680,
        5000,
        "https://adobe.com/download"
    )

    spotify = SubscriptionProduct(
        "S001",
        "Spotify Premium",
        149,
        12
    )

    # 創建購物車
    cart = ShoppingCart("CUST001")

    # 多型:用統一的方式加入不同類型的商品
    cart.add_product(laptop)
    cart.add_product(software)
    cart.add_product(spotify)

    # 顯示購物車(多型的體現)
    cart.display_cart()
```

**輸出:**
```
已加入: MacBook Pro 16"
已加入: Adobe Creative Cloud
已加入: Spotify Premium

============================================================
購物車詳情 (客戶 ID: CUST001)
============================================================

1. MacBook Pro 16"
   原價: $79,900.00
   最終價格: $80,050.00
   配送資訊: 預計3-5個工作天送達,重量:2.0kg

2. Adobe Creative Cloud
   原價: $1,680.00
   最終價格: $1,680.00
   配送資訊: 立即下載,檔案大小:5000MB

3. Spotify Premium
   原價: $149.00
   最終價格: $1,788.00
   配送資訊: 訂閱12個月,立即開通

------------------------------------------------------------
總計: $83,518.00
============================================================
```

## 常見陷阱 (Common Pitfalls)

### 陷阱 1: 過度封裝

```python
# ❌ 過度封裝
class Person:
    def __init__(self, name):
        self.__name = name

    def get_name(self):
        return self.__name

    def set_name(self, name):
        self.__name = name

# ✅ 適度封裝
class Person:
    def __init__(self, name):
        self._name = name

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if not value:
            raise ValueError("名稱不能為空")
        self._name = value
```

### 陷阱 2: 繼承層次過深

```python
# ❌ 過深的繼承
class A:
    pass

class B(A):
    pass

class C(B):
    pass

class D(C):
    pass

class E(D):  # 太深了,難以維護
    pass

# ✅ 使用組合代替繼承
class Component:
    def operation(self):
        pass

class ComplexObject:
    def __init__(self):
        self.component = Component()  # 組合
```

### 陷阱 3: 違反抽象原則

```python
# ❌ 抽象類別中有具體實作邏輯
class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        print("動物發出聲音")  # 不應該有具體實作
        return "sound"

# ✅ 抽象方法應該只定義介面
class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        """子類別必須實作此方法"""
        pass
```

## 與 System Design 的連結 (Connection to System Design)

### 1. SOLID 原則的預告
OOP 四大支柱是 SOLID 原則的基礎:
- **封裝** → Single Responsibility Principle
- **抽象** → Dependency Inversion Principle
- **繼承** → Liskov Substitution Principle
- **多型** → Open/Closed Principle

### 2. 微服務架構
OOP 的思想可以應用到微服務:
```
每個微服務 = 一個類別
- 封裝:內部實作細節不暴露
- 抽象:定義清晰的 API 介面
- 多型:相同介面,不同實作
```

### 3. 系統可擴展性
透過 OOP 四大支柱,我們可以:
- 輕鬆添加新功能(繼承)
- 替換實作而不影響其他部分(多型)
- 保護核心邏輯(封裝)
- 定義清晰的系統邊界(抽象)

## 練習題 (Exercises)

### 基礎練習

**練習 1:** 實作一個銀行帳戶系統
- 使用封裝保護餘額
- 實作存款、提款、查詢方法
- 添加交易驗證邏輯

**練習 2:** 設計形狀類別體系
- 抽象基類 Shape
- 子類別:Circle, Rectangle, Triangle
- 實作計算面積和周長的多型

### 進階練習

**練習 3:** 擴展電商系統
在現有基礎上添加:
- 不同的優惠策略(策略模式)
- 會員等級系統
- 訂單狀態管理

**練習 4:** 實作一個簡單的 ORM
- 定義 Model 抽象基類
- 實作 save(), delete(), find() 方法
- 支援不同的資料庫後端

### 挑戰練習

**練習 5:** 設計一個遊戲角色系統
需求:
- 不同職業(戰士、法師、弓箭手)
- 技能系統
- 裝備系統
- 等級和經驗值系統

運用所有四大支柱來實作!

---

## 總結

本章學習了 OOP 的四大支柱:

✅ **封裝(Encapsulation)**
- 保護數據安全
- 提供清晰的介面
- 使用 `@property` 和私有屬性

✅ **繼承(Inheritance)**
- 重用程式碼
- 建立類別層次
- 正確使用 `super()`

✅ **多型(Polymorphism)**
- 統一介面,不同實作
- 提高系統靈活性
- 降低耦合度

✅ **抽象(Abstraction)**
- 隱藏實作細節
- 定義標準介面
- 使用抽象基類(ABC)

**下一章預告:** 我們將深入探討 Python 的物件模型,理解 `type`、`object` 和 `class` 的關係,以及強大的 Magic Methods。

---

**學習建議:**
1. 四大支柱是相輔相成的,要綜合運用
2. 不要過度設計,根據實際需求選擇合適的特性
3. 多做練習,從實戰中理解概念
4. 思考如何將這些概念應用到大型系統設計
