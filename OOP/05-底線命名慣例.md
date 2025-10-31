# 05 - 底線命名慣例

## 本章目標 (Learning Objectives)
- 理解 Python 中不同底線的含義
- 掌握 public, protected, private 的實作方式
- 學會使用名稱改寫(name mangling)
- 理解 dunder methods 的特殊性
- 在實際專案中正確使用命名慣例

## 為什麼需要這個? (Motivation)

### 問題場景

```python
class BankAccount:
    def __init__(self, balance):
        self.balance = balance  # 任何人都能直接修改!

account = BankAccount(10000)
account.balance = -999999  # 不安全!沒有驗證!
print(account.balance)  # -999999
```

**問題:**
- 沒有封裝
- 數據不安全
- 無法控制訪問

**解決方案:** 使用底線命名慣例

```python
class BankAccount:
    def __init__(self, balance):
        self.__balance = balance  # 私有屬性

    @property
    def balance(self):
        return self.__balance

    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount

    def withdraw(self, amount):
        if 0 < amount <= self.__balance:
            self.__balance -= amount
        else:
            raise ValueError("餘額不足")

account = BankAccount(10000)
# account.__balance = -999999  # AttributeError!
account.deposit(1000)  # 只能通過方法修改
print(account.balance)  # 11000
```

## 核心概念 (Core Concepts)

Python 使用底線作為命名慣例來表示不同的訪問級別:

### 1. 單個底線 `_`

#### 1.1 作為臨時變數

```python
# 不需要使用的值
for _ in range(5):
    print("Hello")

# 解包時忽略某些值
name, _, age = ("John", "Doe", 30)
print(name, age)  # John 30

# 在互動式解釋器中,_ 代表上一個表達式的結果
>>> 42
42
>>> _ + 8
50
```

#### 1.2 國際化函數慣例

```python
# 常見的國際化(i18n)慣例
from gettext import gettext as _

print(_("Hello"))  # 會被翻譯
```

### 2. 單個底線前綴 `_name`

**慣例:** Protected(受保護的)

**含義:** "內部使用,但子類別可以訪問"

```python
class Product:
    def __init__(self, name, price):
        self._name = name  # 受保護
        self._price = price

    def _calculate_discount(self):
        """內部方法,不應被外部調用"""
        return self._price * 0.1

class DiscountedProduct(Product):
    def get_discounted_price(self):
        # 子類別可以訪問 _name 和 _calculate_discount
        discount = self._calculate_discount()
        return self._price - discount

# 可以訪問,但不建議
p = Product("iPhone", 29900)
print(p._name)  # 可以訪問,但表示"請不要這樣做"
```

#### `from module import *` 的影響

```python
# module.py
def public_function():
    return "Public"

def _internal_function():
    return "Internal"

# main.py
from module import *

public_function()  # OK
_internal_function()  # NameError! 不會被導入
```

### 3. 雙底線前綴 `__name`

**實際效果:** Name Mangling(名稱改寫)

**含義:** "真正的私有,避免子類別覆蓋"

```python
class Product:
    def __init__(self, name, price):
        self.__price = price  # 私有屬性

    def get_price(self):
        return self.__price

    def __internal_method(self):  # 私有方法
        return "Internal"

# 名稱改寫
p = Product("iPhone", 29900)
# print(p.__price)  # AttributeError

# Python 實際上將 __price 改寫為 _Product__price
print(p._Product__price)  # 29900 (可以訪問,但不建議)

# 查看所有屬性
print([attr for attr in dir(p) if not attr.startswith('__')])
# ['_Product__internal_method', '_Product__price', 'get_price']
```

#### 名稱改寫的原理

```python
class Parent:
    def __init__(self):
        self.__private = "Parent private"
        self._protected = "Parent protected"

    def __private_method(self):
        return "Parent method"

class Child(Parent):
    def __init__(self):
        super().__init__()
        self.__private = "Child private"  # 不會覆蓋父類別的

    def __private_method(self):
        return "Child method"  # 不會覆蓋父類別的

c = Child()

# 兩個不同的 __private
print(c._Parent__private)  # Parent private
print(c._Child__private)   # Child private

# 兩個不同的方法
print(c._Parent__private_method())  # Parent method
print(c._Child__private_method())   # Child method
```

### 4. 單個底線後綴 `name_`

**用途:** 避免與 Python 關鍵字衝突

```python
# 避免關鍵字衝突
class MyClass:
    def __init__(self, class_):  # class 是關鍵字
        self.class_ = class_
        self.type_ = None
        self.from_ = None

# 常見用法
def create_dict(items, type_="ordered"):
    # type 是內建函數,使用 type_ 避免衝突
    pass
```

### 5. 雙底線前後綴 `__name__`

**Dunder Methods** (Double Underscore Methods)

**用途:** Python 的魔術方法和特殊屬性

```python
class Product:
    def __init__(self, name):  # 建構子
        self.name = name

    def __str__(self):  # 字串表示
        return f"Product: {self.name}"

    def __len__(self):  # len() 函數
        return len(self.name)

    def __add__(self, other):  # + 運算符
        return Product(self.name + other.name)

# 特殊屬性
print(Product.__name__)  # 'Product'
print(Product.__module__)  # '__main__'
print(Product.__doc__)  # 文件字串
```

## 實戰範例 (Hands-on Examples)

### 範例 1: 銀行帳戶系統

```python
class BankAccount:
    """銀行帳戶 - 展示完整的命名慣例"""

    # 類別變數(公開)
    bank_name = "Python Bank"

    # 私有類別變數
    __interest_rate = 0.02

    def __init__(self, account_number, initial_balance):
        # 公開屬性
        self.account_number = account_number

        # 受保護屬性(子類別可訪問)
        self._balance = initial_balance

        # 私有屬性(真正的私有)
        self.__transactions = []
        self.__pin = self.__generate_pin()

    # 公開方法
    def deposit(self, amount):
        """存款"""
        if self._validate_amount(amount):
            self._balance += amount
            self.__log_transaction("deposit", amount)
            return True
        return False

    def withdraw(self, amount):
        """提款"""
        if self._validate_amount(amount) and amount <= self._balance:
            self._balance -= amount
            self.__log_transaction("withdraw", amount)
            return True
        return False

    def get_balance(self):
        """查詢餘額"""
        return self._balance

    # 受保護方法(子類別可以覆蓋)
    def _validate_amount(self, amount):
        """驗證金額"""
        return amount > 0

    def _calculate_interest(self):
        """計算利息"""
        return self._balance * self.__class__.__interest_rate

    # 私有方法(僅內部使用)
    def __log_transaction(self, type, amount):
        """記錄交易"""
        from datetime import datetime
        self.__transactions.append({
            'type': type,
            'amount': amount,
            'timestamp': datetime.now()
        })

    def __generate_pin(self):
        """生成PIN碼"""
        import random
        return str(random.randint(1000, 9999))

    # 使用 property 暴露只讀屬性
    @property
    def transaction_count(self):
        """交易次數(只讀)"""
        return len(self.__transactions)

    # Dunder methods
    def __str__(self):
        return f"Account {self.account_number}: ${self._balance}"

    def __repr__(self):
        return f"BankAccount('{self.account_number}', {self._balance})"

# 使用
account = BankAccount("123456", 10000)
account.deposit(5000)
account.withdraw(2000)

print(account)  # Account 123456: $13000
print(f"交易次數: {account.transaction_count}")  # 2

# 可以訪問受保護屬性(但不建議)
print(account._balance)  # 13000

# 無法直接訪問私有屬性
# print(account.__transactions)  # AttributeError
```

### 範例 2: 產品類別層次

```python
class Product:
    """產品基類"""

    def __init__(self, product_id, name, price):
        # 公開
        self.product_id = product_id

        # 受保護(子類別可訪問)
        self._name = name
        self._price = price
        self._discount = 0

        # 私有(避免子類別意外覆蓋)
        self.__creation_time = datetime.now()

    # 公開方法
    @property
    def name(self):
        return self._name

    @property
    def price(self):
        return self._price

    @price.setter
    def price(self, value):
        if self._validate_price(value):
            self._price = value

    def get_final_price(self):
        """獲取最終價格"""
        return self._calculate_final_price()

    # 受保護方法(子類別可以覆蓋)
    def _validate_price(self, price):
        """驗證價格"""
        return price > 0

    def _calculate_final_price(self):
        """計算最終價格"""
        return self._price * (1 - self._discount)

    # 私有方法
    def __log_price_change(self, old_price, new_price):
        """記錄價格變化"""
        print(f"價格從 ${old_price} 變為 ${new_price}")

class DigitalProduct(Product):
    """數位產品 - 覆蓋受保護方法"""

    def __init__(self, product_id, name, price, file_size):
        super().__init__(product_id, name, price)
        self.file_size = file_size

    def _calculate_final_price(self):
        """覆蓋:數位產品不含運費"""
        return super()._calculate_final_price()

class PhysicalProduct(Product):
    """實體產品"""

    def __init__(self, product_id, name, price, weight):
        super().__init__(product_id, name, price)
        self._weight = weight  # 受保護

    def _calculate_final_price(self):
        """覆蓋:實體產品含運費"""
        base_price = super()._calculate_final_price()
        shipping = self._calculate_shipping()
        return base_price + shipping

    def _calculate_shipping(self):
        """計算運費"""
        return self._weight * 10

# 使用
digital = DigitalProduct("D001", "Software", 1000, 500)
physical = PhysicalProduct("P001", "Book", 500, 2.0)

print(digital.get_final_price())   # 1000
print(physical.get_final_price())  # 520 (500 + 20運費)
```

### 範例 3: 單例模式

```python
class DatabaseConnection:
    """資料庫連接 - 單例模式"""

    __instance = None  # 私有類別變數

    def __new__(cls):
        """確保只有一個實例"""
        if cls.__instance is None:
            cls.__instance = super().__new__(cls)
            cls.__instance.__initialized = False
        return cls.__instance

    def __init__(self):
        """只初始化一次"""
        if self.__initialized:
            return

        self.__connection = self.__create_connection()
        self.__initialized = True

    def __create_connection(self):
        """私有:創建連接"""
        print("創建資料庫連接")
        return "connection_object"

    def query(self, sql):
        """公開:執行查詢"""
        return f"執行: {sql}"

# 測試單例
db1 = DatabaseConnection()  # 創建資料庫連接
db2 = DatabaseConnection()  # 不會再次創建
print(db1 is db2)  # True - 同一個實例
```

## 常見陷阱 (Common Pitfalls)

### 陷阱 1: 過度使用私有屬性

```python
# ❌ 過度私有化
class Product:
    def __init__(self, name, price):
        self.__name = name  # 太私有了
        self.__price = price

    def get_name(self):
        return self.__name

# 難以測試和調試

# ✅ 適度使用
class Product:
    def __init__(self, name, price):
        self._name = name  # 受保護即可
        self._price = price

    @property
    def name(self):
        return self._name
```

### 陷阱 2: 誤用名稱改寫

```python
# ❌ 意外的名稱改寫
class MyClass:
    def __init__(self):
        self.__x = 1

    def __method(self):  # 變成 _MyClass__method
        pass

# 子類別無法訪問
class Child(MyClass):
    def __init__(self):
        super().__init__()
        # print(self.__x)  # AttributeError!

# ✅ 使用單底線
class MyClass:
    def __init__(self):
        self._x = 1  # 子類別可以訪問

    def _method(self):
        pass
```

### 陷阱 3: 忘記底線不是真正的封裝

```python
class Secret:
    def __init__(self):
        self.__secret = "password"

s = Secret()

# ❌ 以為無法訪問
# print(s.__secret)  # AttributeError

# ✅ 實際上可以通過名稱改寫訪問
print(s._Secret__secret)  # password

# Python 的封裝是"君子協議"
```

## 與 System Design 的連結 (Connection to System Design)

### 1. API 設計

```python
class UserAPI:
    """用戶 API - 清晰的公開介面"""

    def __init__(self, db):
        self._db = db  # 內部依賴

    # 公開 API
    def get_user(self, user_id):
        """公開方法"""
        user_data = self._fetch_from_db(user_id)
        return self._transform_user(user_data)

    def create_user(self, user_data):
        """公開方法"""
        validated = self._validate_user(user_data)
        return self._save_to_db(validated)

    # 內部方法(不是 API 的一部分)
    def _fetch_from_db(self, user_id):
        return self._db.query(f"SELECT * FROM users WHERE id={user_id}")

    def _validate_user(self, data):
        # 驗證邏輯
        return data

    def _transform_user(self, data):
        # 轉換邏輯
        return data

    def _save_to_db(self, data):
        return self._db.insert("users", data)
```

### 2. 框架設計

```python
class Framework:
    """框架基類"""

    def run(self):
        """公開:框架入口"""
        self._setup()
        self._process()
        self._teardown()

    # 受保護:子類別可以覆蓋
    def _setup(self):
        """子類別可以覆蓋"""
        pass

    def _process(self):
        """子類別必須實作"""
        raise NotImplementedError

    def _teardown(self):
        """子類別可以覆蓋"""
        pass

    # 私有:框架內部使用
    def __log(self, message):
        """內部日誌"""
        print(f"[Framework] {message}")
```

## 練習題 (Exercises)

### 基礎練習

**練習 1:** 設計一個安全的 User 類別
- 使用適當的命名慣例
- password 應該是私有的
- email 應該是受保護的
- username 應該是公開的

**練習 2:** 重構以下程式碼
```python
# 找出命名問題並修復
class MyClass:
    def __init__(self):
        self.internal_value = 100
        self.secret_key = "abc123"

    def helper_function(self):
        return self.internal_value * 2
```

### 進階練習

**練習 3:** 實作一個配置類別
- 使用單例模式
- 配置值應該是受保護的
- 提供公開的 getter/setter
- 內部使用私有方法驗證

**練習 4:** 設計一個可擴展的支付處理器
- 定義清晰的公開API
- 內部方法使用受保護
- 敏感操作使用私有方法

## 總結

本章學習了 Python 的命名慣例:

✅ **單個底線 `_`**
- 臨時變數
- 國際化慣例

✅ **單底線前綴 `_name`**
- Protected(受保護)
- 子類別可訪問
- import * 不會導入

✅ **雙底線前綴 `__name`**
- Name Mangling
- 避免子類別意外覆蓋
- 不是真正的私有

✅ **單底線後綴 `name_`**
- 避免關鍵字衝突

✅ **雙底線前後綴 `__name__`**
- Dunder methods
- Python 特殊方法

**重要原則:**
- Python 的封裝是"君子協議"
- 優先使用 `_name` 而不是 `__name`
- 清晰的命名比嚴格的私有更重要
- 不要過度封裝

**下一章預告:** 繼承進階 - 深入學習 super(), MRO, 多重繼承等進階主題。
