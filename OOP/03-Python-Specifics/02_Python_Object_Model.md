# 03 - Python 物件模型

## 本章目標 (Learning Objectives)
- 理解「Python 中一切皆物件」的深層含義
- 掌握 type() 和 isinstance() 的使用場景
- 學會使用 Magic Methods 自訂物件行為
- 理解 object-type-class 的三角關係
- 能夠設計符合 Python 風格的類別

## 為什麼需要這個? (Motivation)

### 問題場景

假設你正在設計一個電商系統的 Product 類別:

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

# 創建商品
iphone = Product("iPhone 15", 29900)

# 當你嘗試這些操作時...
print(iphone)  # <__main__.Product object at 0x...> - 不友善的輸出
print(len(iphone))  # TypeError - 為什麼不能用 len()?
print(iphone + iphone)  # TypeError - 為什麼不能相加?
```

**問題:**
- 物件的字串表示不友善
- 無法使用 Python 內建函數
- 物件行為不符合直覺

**解決方案:** 使用 Python 的 Magic Methods

```python
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def __str__(self):
        """友善的字串表示"""
        return f"{self.name} - ${self.price:,}"

    def __repr__(self):
        """開發者友善的表示"""
        return f"Product(name='{self.name}', price={self.price}, stock={self.stock})"

    def __len__(self):
        """支援 len() 函數"""
        return self.stock

    def __add__(self, other):
        """支援 + 運算符"""
        if isinstance(other, Product):
            return self.price + other.price
        return NotImplemented

# 現在這些操作都可以了!
iphone = Product("iPhone 15", 29900, 50)
print(iphone)           # iPhone 15 - $29,900
print(repr(iphone))     # Product(name='iPhone 15', price=29900, stock=50)
print(len(iphone))      # 50
```

## 核心概念 (Core Concepts)

### 1. Everything is an Object

在 Python 中,**一切皆物件**,包括:
- 數字、字串、列表等基本型別
- 函數
- 類別本身
- 模組

```python
# 數字是物件
x = 42
print(type(x))  # <class 'int'>
print(isinstance(x, object))  # True

# 函數是物件
def greet():
    return "Hello"

print(type(greet))  # <class 'function'>
print(isinstance(greet, object))  # True

# 類別也是物件
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
print(isinstance(MyClass, object))  # True

# 甚至 type 本身也是物件
print(type(type))  # <class 'type'>
print(isinstance(type, object))  # True
```

**這意味著:**
- 所有東西都有型別
- 所有東西都可以賦值給變數
- 所有東西都可以作為參數傳遞
- 所有東西都有屬性和方法

### 2. type() vs isinstance()

#### type() - 獲取物件的確切型別

```python
class Animal:
    pass

class Dog(Animal):
    pass

# type() 返回確切的類別
dog = Dog()
print(type(dog))  # <class '__main__.Dog'>
print(type(dog) == Dog)  # True
print(type(dog) == Animal)  # False - type() 不考慮繼承
```

**type() 的用途:**
- 檢查物件的確切型別
- 動態創建類別
- 元程式設計

#### isinstance() - 檢查物件是否是某類別的實例

```python
class Animal:
    pass

class Dog(Animal):
    pass

dog = Dog()
print(isinstance(dog, Dog))     # True
print(isinstance(dog, Animal))  # True - 考慮繼承!
print(isinstance(dog, object))  # True - 所有類別都繼承自 object
```

**isinstance() 的用途:**
- 型別檢查(推薦使用)
- 多型判斷
- 參數驗證

#### 實戰對比

```python
from abc import ABC

class PaymentMethod(ABC):
    pass

class CreditCard(PaymentMethod):
    pass

class PayPal(PaymentMethod):
    pass

def process_payment(payment):
    # ❌ 不推薦:使用 type()
    if type(payment) == CreditCard:
        print("處理信用卡支付")
    elif type(payment) == PayPal:
        print("處理 PayPal 支付")
    # 問題:無法處理子類別

def process_payment_better(payment):
    # ✅ 推薦:使用 isinstance()
    if isinstance(payment, PaymentMethod):
        print(f"處理 {payment.__class__.__name__} 支付")
    else:
        raise TypeError("不支援的支付方式")

# 測試
cc = CreditCard()
pp = PayPal()

process_payment_better(cc)   # 處理 CreditCard 支付
process_payment_better(pp)   # 處理 PayPal 支付
```

### 3. Magic Methods (Dunder Methods)

Magic Methods 是 Python 中以雙下劃線開頭和結尾的特殊方法,用於定義物件如何與 Python 語法互動。

#### 3.1 物件表示

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __str__(self):
        """
        給用戶看的友善表示
        print() 和 str() 會調用
        """
        return f"{self.name}: ${self.price:,}"

    def __repr__(self):
        """
        給開發者看的詳細表示
        repr() 和互動式解釋器會調用
        應該能夠重新創建物件
        """
        return f"Product(name='{self.name}', price={self.price})"

# 使用
p = Product("iPhone", 29900)

print(str(p))   # iPhone: $29,900
print(repr(p))  # Product(name='iPhone', price=29900)

print(p)        # 調用 __str__
p               # 在互動式環境調用 __repr__
```

#### 3.2 算術運算符

```python
class Money:
    def __init__(self, amount, currency="USD"):
        self.amount = amount
        self.currency = currency

    def __str__(self):
        return f"${self.amount} {self.currency}"

    def __add__(self, other):
        """加法: money1 + money2"""
        if isinstance(other, Money):
            if self.currency != other.currency:
                raise ValueError("不同貨幣無法相加")
            return Money(self.amount + other.amount, self.currency)
        elif isinstance(other, (int, float)):
            return Money(self.amount + other, self.currency)
        return NotImplemented

    def __sub__(self, other):
        """減法: money1 - money2"""
        if isinstance(other, Money):
            if self.currency != other.currency:
                raise ValueError("不同貨幣無法相減")
            return Money(self.amount - other.amount, self.currency)
        elif isinstance(other, (int, float)):
            return Money(self.amount - other, self.currency)
        return NotImplemented

    def __mul__(self, factor):
        """乘法: money * number"""
        if isinstance(factor, (int, float)):
            return Money(self.amount * factor, self.currency)
        return NotImplemented

    def __truediv__(self, divisor):
        """除法: money / number"""
        if isinstance(divisor, (int, float)):
            if divisor == 0:
                raise ValueError("除數不能為零")
            return Money(self.amount / divisor, self.currency)
        return NotImplemented

# 使用範例
m1 = Money(1000)
m2 = Money(500)

print(m1 + m2)      # $1500 USD
print(m1 - m2)      # $500 USD
print(m1 * 2)       # $2000 USD
print(m1 / 4)       # $250.0 USD
print(m1 + 100)     # $1100 USD
```

#### 3.3 比較運算符

```python
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price

    def __eq__(self, other):
        """相等: product1 == product2"""
        if isinstance(other, Product):
            return self.price == other.price
        return False

    def __lt__(self, other):
        """小於: product1 < product2"""
        if isinstance(other, Product):
            return self.price < other.price
        return NotImplemented

    def __le__(self, other):
        """小於等於: product1 <= product2"""
        if isinstance(other, Product):
            return self.price <= other.price
        return NotImplemented

    def __gt__(self, other):
        """大於: product1 > product2"""
        if isinstance(other, Product):
            return self.price > other.price
        return NotImplemented

    def __ge__(self, other):
        """大於等於: product1 >= product2"""
        if isinstance(other, Product):
            return self.price >= other.price
        return NotImplemented

    def __str__(self):
        return f"{self.name}: ${self.price}"

# 使用範例
iphone = Product("iPhone 15", 29900)
samsung = Product("Samsung S24", 28900)
macbook = Product("MacBook Pro", 79900)

print(iphone == samsung)  # False
print(iphone > samsung)   # True
print(macbook > iphone)   # True

# 可以排序了!
products = [macbook, iphone, samsung]
sorted_products = sorted(products)  # 按價格排序
for p in sorted_products:
    print(p)
```

#### 3.4 容器相關

```python
class ShoppingCart:
    def __init__(self):
        self._items = []

    def __len__(self):
        """支援 len() 函數"""
        return len(self._items)

    def __getitem__(self, index):
        """支援索引: cart[0]"""
        return self._items[index]

    def __setitem__(self, index, value):
        """支援索引賦值: cart[0] = item"""
        self._items[index] = value

    def __delitem__(self, index):
        """支援刪除: del cart[0]"""
        del self._items[index]

    def __contains__(self, item):
        """支援 in 運算符: item in cart"""
        return item in self._items

    def __iter__(self):
        """支援迭代: for item in cart"""
        return iter(self._items)

    def add(self, item):
        """添加商品"""
        self._items.append(item)

# 使用範例
cart = ShoppingCart()
cart.add("iPhone")
cart.add("MacBook")
cart.add("AirPods")

print(len(cart))           # 3
print(cart[0])             # iPhone
print("iPhone" in cart)    # True

for item in cart:
    print(item)

# 修改和刪除
cart[0] = "iPhone 15 Pro"
del cart[2]
```

#### 3.5 可調用物件

```python
class Discounter:
    """折扣計算器 - 可以像函數一樣調用"""

    def __init__(self, discount_rate):
        self.discount_rate = discount_rate

    def __call__(self, price):
        """使物件可以像函數一樣調用"""
        return price * (1 - self.discount_rate)

# 使用範例
discount_10 = Discounter(0.1)  # 9折
discount_20 = Discounter(0.2)  # 8折

print(discount_10(1000))  # 900.0
print(discount_20(1000))  # 800.0

# 可以把物件當函數使用
def apply_discount(price, discount_func):
    return discount_func(price)

print(apply_discount(1000, discount_10))  # 900.0
```

### 4. object-type-class 三角關係

這是 Python 物件模型的核心!

```python
# object 是所有類別的基類
class MyClass:
    pass

print(isinstance(MyClass, object))  # False - MyClass 不是 object 的實例
print(issubclass(MyClass, object))  # True - MyClass 繼承自 object

# type 是所有類別的元類
print(type(MyClass))  # <class 'type'>
print(isinstance(MyClass, type))  # True - MyClass 是 type 的實例

# type 和 object 的特殊關係
print(type(object))  # <class 'type'> - object 是 type 的實例
print(isinstance(type, object))  # True - type 繼承自 object
print(issubclass(type, object))  # True

# 三角關係
print(type(type))  # <class 'type'> - type 是自己的實例!
```

**關係圖:**
```
        object (所有類別的基類)
          ↑
          │ 繼承
          │
        type (所有類別的元類)
          ↑
          │ 實例化
          │
       MyClass
          ↑
          │ 實例化
          │
       my_instance
```

**理解這個關係:**
1. `object` 是所有類別的最終基類
2. `type` 是創建類別的類別(元類)
3. 所有類別都是 `type` 的實例
4. `type` 本身也是 `object` 的子類
5. `type` 是自己的實例(特殊情況)

## 實戰範例 (Hands-on Examples)

### 完整範例:電商系統的商品類別

```python
from typing import List
from datetime import datetime

class Product:
    """
    商品類別 - 展示 Python 物件模型的完整應用
    """

    # 類別變數
    _id_counter = 0
    all_products: List['Product'] = []

    def __init__(self, name: str, price: float, stock: int):
        # 自動生成 ID
        Product._id_counter += 1
        self.id = Product._id_counter

        self.name = name
        self.price = price
        self.stock = stock
        self.created_at = datetime.now()

        # 加入所有商品列表
        Product.all_products.append(self)

    # ===== 物件表示 =====
    def __str__(self):
        """用戶友善的表示"""
        status = "有貨" if self.stock > 0 else "缺貨"
        return f"[{self.id}] {self.name} - ${self.price:,} ({status})"

    def __repr__(self):
        """開發者友善的表示"""
        return (f"Product(name='{self.name}', price={self.price}, "
                f"stock={self.stock})")

    # ===== 比較運算符 =====
    def __eq__(self, other):
        """判斷相等(基於價格)"""
        if isinstance(other, Product):
            return self.price == other.price
        return False

    def __lt__(self, other):
        """小於比較(基於價格)"""
        if isinstance(other, Product):
            return self.price < other.price
        return NotImplemented

    def __hash__(self):
        """使物件可以作為字典的鍵或加入集合"""
        return hash((self.id, self.name))

    # ===== 算術運算符 =====
    def __add__(self, other):
        """
        相加:合併兩個商品的價格
        product1 + product2
        """
        if isinstance(other, Product):
            return self.price + other.price
        elif isinstance(other, (int, float)):
            return self.price + other
        return NotImplemented

    def __mul__(self, quantity):
        """
        乘法:計算總價
        product * 3
        """
        if isinstance(quantity, int):
            return self.price * quantity
        return NotImplemented

    def __rmul__(self, quantity):
        """
        反向乘法:支援 3 * product
        """
        return self.__mul__(quantity)

    # ===== 容器行為 =====
    def __len__(self):
        """返回庫存數量"""
        return self.stock

    def __bool__(self):
        """
        布林值:有庫存時為 True
        if product: ...
        """
        return self.stock > 0

    # ===== 屬性訪問 =====
    def __getattr__(self, name):
        """
        當訪問不存在的屬性時調用
        """
        if name == 'is_available':
            return self.stock > 0
        raise AttributeError(f"'{type(self).__name__}' 沒有屬性 '{name}'")

    # ===== 類別方法 =====
    @classmethod
    def get_all_products(cls):
        """獲取所有商品"""
        return cls.all_products

    @classmethod
    def find_by_id(cls, product_id):
        """根據 ID 查找商品"""
        for product in cls.all_products:
            if product.id == product_id:
                return product
        return None

    @classmethod
    def get_expensive_products(cls, min_price):
        """獲取高價商品"""
        return [p for p in cls.all_products if p.price >= min_price]

    # ===== 靜態方法 =====
    @staticmethod
    def calculate_tax(price, tax_rate=0.05):
        """計算稅額"""
        return price * tax_rate

# ===== 使用範例 =====
if __name__ == "__main__":
    # 創建商品
    iphone = Product("iPhone 15 Pro", 29900, 50)
    macbook = Product("MacBook Pro", 79900, 20)
    airpods = Product("AirPods Pro", 7490, 100)

    # 物件表示
    print(iphone)  # [1] iPhone 15 Pro - $29,900 (有貨)
    print(repr(macbook))  # Product(name='MacBook Pro', price=79900, stock=20)

    # 比較
    print(iphone < macbook)  # True (價格比較)
    print(iphone == Product("Test", 29900, 10))  # True (價格相同)

    # 算術
    print(iphone + macbook)  # 109800 (價格相加)
    print(iphone * 2)  # 59800 (買兩個的總價)
    print(3 * iphone)  # 89700 (反向乘法)

    # 容器行為
    print(len(iphone))  # 50 (庫存)
    if iphone:
        print("iPhone 有貨")

    # 屬性訪問
    print(iphone.is_available)  # True (自動屬性)

    # 排序
    products = [macbook, iphone, airpods]
    sorted_products = sorted(products)
    print("\n按價格排序:")
    for p in sorted_products:
        print(p)

    # 類別方法
    print(f"\n總共有 {len(Product.get_all_products())} 種商品")
    expensive = Product.get_expensive_products(20000)
    print(f"高價商品: {[p.name for p in expensive]}")

    # 靜態方法
    tax = Product.calculate_tax(29900)
    print(f"\niPhone 稅額: ${tax:,.2f}")
```

## 常見陷阱 (Common Pitfalls)

### 陷阱 1: __str__ vs __repr__

```python
# ❌ 錯誤:兩者返回相同內容
class Product:
    def __str__(self):
        return f"{self.name}"

    def __repr__(self):
        return f"{self.name}"  # 不夠詳細

# ✅ 正確
class Product:
    def __str__(self):
        """給用戶看"""
        return f"{self.name} - ${self.price}"

    def __repr__(self):
        """給開發者看,應該能重建物件"""
        return f"Product(name='{self.name}', price={self.price})"
```

### 陷阱 2: 忘記返回 NotImplemented

```python
# ❌ 錯誤
class Product:
    def __add__(self, other):
        if isinstance(other, Product):
            return self.price + other.price
        return None  # 應該返回 NotImplemented

# ✅ 正確
class Product:
    def __add__(self, other):
        if isinstance(other, Product):
            return self.price + other.price
        return NotImplemented  # Python 會嘗試反向操作
```

### 陷阱 3: __eq__ 和 __hash__ 不一致

```python
# ❌ 錯誤:定義了 __eq__ 但沒定義 __hash__
class Product:
    def __eq__(self, other):
        return self.id == other.id
    # 物件變成 unhashable,無法加入集合

# ✅ 正確
class Product:
    def __eq__(self, other):
        if isinstance(other, Product):
            return self.id == other.id
        return False

    def __hash__(self):
        return hash(self.id)  # 基於相同的屬性
```

### 陷阱 4: 濫用 Magic Methods

```python
# ❌ 不好:語義不清
class Product:
    def __add__(self, other):
        """加法表示...合併商品??"""
        return Product(
            self.name + other.name,
            self.price + other.price,
            self.stock + other.stock
        )

# ✅ 更好:使用明確的方法
class Product:
    def merge_with(self, other):
        """明確的方法名"""
        return Product(
            f"{self.name} & {self.name}",
            self.price + other.price,
            self.stock + other.stock
        )
```

## 與 System Design 的連結 (Connection to System Design)

### 1. API 設計

Magic Methods 讓 API 更 Pythonic:

```python
# 設計類似字典的 API
class Configuration:
    def __getitem__(self, key):
        return self.settings[key]

    def __setitem__(self, key, value):
        self.settings[key] = value

# 使用起來很自然
config = Configuration()
config['database_url'] = 'postgresql://...'
print(config['database_url'])
```

### 2. ORM 設計

```python
# 類似 Django ORM 的設計
class QuerySet:
    def __len__(self):
        return self.count()

    def __iter__(self):
        return iter(self._fetch())

    def __getitem__(self, index):
        return self._fetch()[index]

# 使用
users = User.objects.filter(age__gt=18)
print(len(users))  # 調用資料庫 COUNT
for user in users:  # 調用資料庫 SELECT
    print(user)
```

### 3. 運算符重載用於 DSL

```python
# 查詢建構器
class Query:
    def __gt__(self, value):
        return f"field > {value}"

    def __lt__(self, value):
        return f"field < {value}"

# 使用
age = Query()
condition = age > 18  # 生成 SQL 條件
```

## 練習題 (Exercises)

### 基礎練習

**練習 1:** 設計一個 Vector 類別
- 支援加法、減法
- 支援點積(用 * 運算符)
- 實作 __str__ 和 __repr__

**練習 2:** 設計一個 Temperature 類別
- 支援攝氏和華氏轉換
- 支援比較運算
- 支援算術運算

### 進階練習

**練習 3:** 設計一個類似列表的容器
- 實作所有容器 Magic Methods
- 支援切片操作
- 支援 += 運算符

**練習 4:** 設計一個可調用的快取裝飾器
- 使用 __call__
- 自動快取函數結果
- 支援過期時間

### 挑戰練習

**練習 5:** 設計一個 ORM Query Builder
要求:
- 支援鏈式調用
- 使用運算符重載構建查詢條件
- 惰性求值

提示:參考 Django ORM 或 SQLAlchemy

---

## 總結

本章學習了 Python 物件模型的核心概念:

✅ **Everything is an Object**
- Python 中一切皆物件
- 所有東西都有型別和行為

✅ **type() vs isinstance()**
- type() 獲取確切型別
- isinstance() 考慮繼承(推薦)

✅ **Magic Methods**
- 自訂物件與 Python 語法的互動
- 讓自訂類別更 Pythonic
- 豐富的內建協議

✅ **object-type-class 三角關係**
- 理解 Python 的元物件系統
- 為元程式設計打基礎

**下一章預告:** 我們將深入探討命名空間與作用域,理解 LEGB 規則,學會正確使用 global 和 nonlocal。

---

**學習建議:**
1. 不要過度使用 Magic Methods
2. 只在語義清晰時使用運算符重載
3. 總是實作 __repr__
4. __eq__ 和 __hash__ 要一致
5. 參考標準庫的實作方式
