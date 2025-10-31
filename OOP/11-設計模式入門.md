# 11 - 設計模式入門

## 本章目標
- 理解常用設計模式的目的和應用場景
- 掌握 Singleton, Factory, Strategy, Observer 模式
- 學會在實際專案中應用設計模式
- 為 System Design 打基礎

## 為什麼需要設計模式?

設計模式是軟體設計中常見問題的可重用解決方案。

**好處:**
- 提供標準化的解決方案
- 提高程式碼可讀性
- 加速開發流程
- 便於團隊溝通

## 核心設計模式

### 1. Singleton Pattern (單例模式)

**目的:** 確保一個類別只有一個實例

```python
class DatabaseConnection:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialize()
        return cls._instance
    
    def _initialize(self):
        self.connection = "Database Connected"
    
    def query(self, sql):
        return f"Executing: {sql}"

# 使用
db1 = DatabaseConnection()
db2 = DatabaseConnection()
print(db1 is db2)  # True - 同一個實例
```

**應用場景:**
- 資料庫連接
- 配置管理器
- 日誌記錄器
- 快取管理

### 2. Factory Pattern (工廠模式)

**目的:** 將物件創建邏輯封裝起來

```python
from abc import ABC, abstractmethod

class Product(ABC):
    @abstractmethod
    def operation(self):
        pass

class ConcreteProductA(Product):
    def operation(self):
        return "Product A"

class ConcreteProductB(Product):
    def operation(self):
        return "Product B"

class ProductFactory:
    @staticmethod
    def create_product(product_type):
        if product_type == "A":
            return ConcreteProductA()
        elif product_type == "B":
            return ConcreteProductB()
        else:
            raise ValueError("Unknown product type")

# 使用
factory = ProductFactory()
product = factory.create_product("A")
print(product.operation())  # Product A
```

**應用場景:**
- 物件創建複雜時
- 需要根據條件創建不同物件
- 隱藏創建細節

### 3. Strategy Pattern (策略模式)

**目的:** 定義一系列算法,讓它們可以互相替換

```python
from abc import ABC, abstractmethod

class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount):
        pass

class NoDiscount(DiscountStrategy):
    def calculate(self, amount):
        return amount

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage):
        self.percentage = percentage
    
    def calculate(self, amount):
        return amount * (1 - self.percentage)

class FixedDiscount(DiscountStrategy):
    def __init__(self, discount):
        self.discount = discount
    
    def calculate(self, amount):
        return max(0, amount - self.discount)

class Order:
    def __init__(self, amount, discount_strategy: DiscountStrategy):
        self.amount = amount
        self.discount_strategy = discount_strategy
    
    def get_final_price(self):
        return self.discount_strategy.calculate(self.amount)

# 使用
order1 = Order(1000, NoDiscount())
order2 = Order(1000, PercentageDiscount(0.1))
order3 = Order(1000, FixedDiscount(100))

print(order1.get_final_price())  # 1000
print(order2.get_final_price())  # 900
print(order3.get_final_price())  # 900
```

**應用場景:**
- 支付方式選擇
- 折扣計算
- 排序算法選擇
- 驗證策略

### 4. Observer Pattern (觀察者模式)

**目的:** 定義物件間的一對多依賴關係

```python
class Subject:
    def __init__(self):
        self._observers = []
    
    def attach(self, observer):
        self._observers.append(observer)
    
    def detach(self, observer):
        self._observers.remove(observer)
    
    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer(ABC):
    @abstractmethod
    def update(self, message):
        pass

class EmailObserver(Observer):
    def update(self, message):
        print(f"Email: {message}")

class SMSObserver(Observer):
    def update(self, message):
        print(f"SMS: {message}")

# 使用
subject = Subject()
subject.attach(EmailObserver())
subject.attach(SMSObserver())

subject.notify("New order created")
# Email: New order created
# SMS: New order created
```

**應用場景:**
- 事件系統
- 通知系統
- MVC 架構
- 發布-訂閱系統

## 電商系統實戰

整合多個設計模式的完整範例:

```python
# 使用 Singleton 管理配置
class Config:
    _instance = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

# 使用 Factory 創建產品
class ProductFactory:
    @staticmethod
    def create(product_type, **kwargs):
        if product_type == "digital":
            return DigitalProduct(**kwargs)
        elif product_type == "physical":
            return PhysicalProduct(**kwargs)

# 使用 Strategy 計算價格
class PriceCalculator:
    def __init__(self, strategy):
        self.strategy = strategy
    
    def calculate(self, product):
        return self.strategy.calculate(product)

# 使用 Observer 發送通知
class OrderNotifier(Subject):
    def create_order(self, order):
        self.notify(f"Order {order.id} created")
```

## 與 System Design 的連結

設計模式在系統設計中的應用:

1. **Singleton** → 共享資源管理
2. **Factory** → 微服務創建
3. **Strategy** → 可插拔的業務邏輯
4. **Observer** → 事件驅動架構

## 練習題

1. 實作一個使用 Factory 模式的支付處理器
2. 使用 Strategy 模式實作不同的運費計算方式
3. 使用 Observer 模式實作庫存預警系統

## 總結

設計模式是連接 OOP 和 System Design 的重要橋樑。掌握常用設計模式能幫助你:
- 寫出更優雅的程式碼
- 設計可擴展的系統
- 更好地理解開源框架
- 為學習 System Design 打基礎

**延伸閱讀:**
- [Design Patterns: Elements of Reusable OO Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [Refactoring Guru](https://refactoring.guru/design-patterns)

**下一章:** Library-Framework-API 設計
