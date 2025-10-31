# 04 - 命名空間與作用域

## 本章目標 (Learning Objectives)
- 理解 Python 的命名空間概念
- 掌握 LEGB 查找規則
- 學會正確使用 global 和 nonlocal
- 避免命名空間相關的常見錯誤
- 理解作用域在大型專案中的重要性

## 為什麼需要這個? (Motivation)

### 問題場景

```python
# 電商系統中的變數衝突問題
discount = 0.1  # 全局折扣

class Order:
    discount = 0.2  # 類別折扣

    def __init__(self):
        self.discount = 0.15  # 實例折扣

    def calculate_price(self, price):
        discount = 0.25  # 局部折扣
        # 問題:這裡的 discount 是哪個?
        return price * (1 - discount)

order = Order()
print(order.calculate_price(1000))  # 750 (使用局部 discount)
print(order.discount)  # 0.15 (實例 discount)
print(Order.discount)  # 0.2 (類別 discount)
print(discount)  # 0.1 (全局 discount)
```

**如果不理解命名空間:**
- 變數值不符合預期
- 難以追蹤變數來源
- 容易引入 bug

## 核心概念 (Core Concepts)

### 1. 什麼是命名空間?

**命名空間(Namespace)** 是從名稱到物件的映射,可以想像成一個字典。

```python
# 命名空間就像字典
namespace = {
    'name': 'iPhone',
    'price': 29900,
    'stock': 50
}
```

Python 有三種主要的命名空間:

#### 1.1 內建命名空間 (Built-in Namespace)

包含 Python 內建函數和異常:

```python
# 這些都在內建命名空間
print(len([1, 2, 3]))  # len 是內建函數
print(type(42))  # type 是內建函數
x = int("123")  # int 是內建函數

# 查看所有內建名稱
import builtins
print(dir(builtins))
```

#### 1.2 全局命名空間 (Global Namespace)

模組層級的名稱:

```python
# module.py
MODULE_NAME = "E-Commerce"  # 全局變數
TAX_RATE = 0.05

def calculate_tax(price):  # 全局函數
    return price * TAX_RATE

class Product:  # 全局類別
    pass
```

#### 1.3 局部命名空間 (Local Namespace)

函數或方法內部的名稱:

```python
def process_order(order_id):
    # 以下都是局部變數
    user = get_user()
    product = get_product(order_id)
    price = product.price

    def apply_discount():
        discount_rate = 0.1  # 內層函數的局部變數
        return price * (1 - discount_rate)

    return apply_discount()
```

### 2. LEGB 規則

Python 使用 **LEGB 規則**來查找變數:

```
L (Local)      → 局部作用域
E (Enclosing)  → 閉包作用域
G (Global)     → 全局作用域
B (Built-in)   → 內建作用域
```

**查找順序:** L → E → G → B

```python
# 示範 LEGB
x = "global"  # G: Global

def outer():
    x = "enclosing"  # E: Enclosing

    def inner():
        x = "local"  # L: Local
        print(x)

    inner()
    print(x)

outer()
print(x)

# 輸出:
# local    (L)
# enclosing (E)
# global   (G)
```

#### 完整的 LEGB 範例

```python
# B: Built-in
len([1, 2, 3])  # Python 內建的 len

# G: Global
len = "global len"  # 覆蓋內建的 len

def outer():
    # E: Enclosing
    len = "enclosing len"

    def inner():
        # L: Local
        len = "local len"
        print("Inner:", len)  # local len

    inner()
    print("Outer:", len)  # enclosing len

outer()
print("Global:", len)  # global len

# 恢復內建的 len
del len
print("Built-in:", len([1, 2, 3]))  # 3
```

### 3. global 關鍵字

在函數內修改全局變數需要使用 `global`:

```python
# 電商系統:全局庫存管理
total_stock = 1000

def sell_product(quantity):
    global total_stock  # 聲明使用全局變數

    if quantity <= total_stock:
        total_stock -= quantity
        print(f"售出 {quantity}, 剩餘 {total_stock}")
        return True
    else:
        print("庫存不足")
        return False

sell_product(100)  # 售出 100, 剩餘 900
sell_product(50)   # 售出 50, 剩餘 850
print(total_stock)  # 850
```

**不使用 global 的問題:**

```python
total_stock = 1000

def sell_product_wrong(quantity):
    # ❌ 這會創建局部變數,不會修改全局變數
    total_stock = total_stock - quantity  # UnboundLocalError!

# sell_product_wrong(100)  # Error!
```

### 4. nonlocal 關鍵字

在內層函數修改外層函數的變數需要使用 `nonlocal`:

```python
def create_counter():
    count = 0  # 外層函數的變數

    def increment():
        nonlocal count  # 聲明使用外層的 count
        count += 1
        return count

    def decrement():
        nonlocal count
        count -= 1
        return count

    def get_count():
        return count  # 讀取不需要 nonlocal

    return increment, decrement, get_count

# 使用閉包
inc, dec, get = create_counter()
print(inc())  # 1
print(inc())  # 2
print(dec())  # 1
print(get())  # 1
```

### 5. 類別命名空間

類別也有自己的命名空間:

```python
class Product:
    # 類別命名空間
    category = "Electronics"  # 類別變數
    count = 0

    def __init__(self, name, price):
        # 實例命名空間
        self.name = name  # 實例變數
        self.price = price
        Product.count += 1  # 訪問類別變數

    def display(self):
        # 方法內的局部命名空間
        tax = 0.05  # 局部變數
        final_price = self.price * (1 + tax)
        print(f"{self.name}: ${final_price}")

# 不同的命名空間
print(Product.category)  # 類別命名空間
p = Product("iPhone", 29900)
print(p.name)  # 實例命名空間
```

## 實戰範例 (Hands-on Examples)

### 範例 1: 電商系統配置管理

```python
# config.py - 全局配置
class Config:
    # 全局配置命名空間
    DEBUG = True
    DATABASE_URL = "postgresql://localhost/ecommerce"
    TAX_RATE = 0.05

    @classmethod
    def update_tax_rate(cls, rate):
        """修改類別變數"""
        cls.TAX_RATE = rate

# order_service.py
class OrderService:
    def __init__(self):
        self.tax_rate = Config.TAX_RATE  # 實例變數

    def calculate_total(self, subtotal):
        # 局部作用域
        tax = subtotal * self.tax_rate
        total = subtotal + tax
        return total

# 使用
service = OrderService()
print(service.calculate_total(1000))  # 1050

# 更新全局配置
Config.update_tax_rate(0.07)

# 新的 service 會使用新稅率
new_service = OrderService()
print(new_service.calculate_total(1000))  # 1070

# 舊的 service 仍使用舊稅率(因為已經複製到實例)
print(service.calculate_total(1000))  # 1050
```

### 範例 2: 閉包實現折扣計算器

```python
def create_discount_calculator(base_discount):
    """
    創建折扣計算器
    使用閉包保持 base_discount 狀態
    """
    total_saved = 0  # 閉包變數

    def apply_discount(price, extra_discount=0):
        nonlocal total_saved  # 修改閉包變數

        # 計算最終折扣
        final_discount = base_discount + extra_discount
        discount_amount = price * final_discount
        final_price = price - discount_amount

        # 累積節省金額
        total_saved += discount_amount

        return {
            'original': price,
            'discount': final_discount,
            'saved': discount_amount,
            'final': final_price,
            'total_saved': total_saved
        }

    def get_total_saved():
        return total_saved

    def reset():
        nonlocal total_saved
        total_saved = 0

    return apply_discount, get_total_saved, reset

# VIP 折扣計算器
vip_discount, get_saved, reset = create_discount_calculator(0.1)

# 使用
result1 = vip_discount(1000)
print(f"購買價格: ${result1['original']}")
print(f"折扣: {result1['discount']*100}%")
print(f"節省: ${result1['saved']}")
print(f"實付: ${result1['final']}")
print(f"累計節省: ${result1['total_saved']}")
print()

# 額外折扣
result2 = vip_discount(500, extra_discount=0.05)
print(f"實付: ${result2['final']}")
print(f"累計節省: ${result2['total_saved']}")
```

### 範例 3: 避免命名空間污染

```python
# ❌ 不好的做法
from math import *  # 導入所有名稱到全局命名空間

pi = 3.14  # 覆蓋了 math.pi
sin = "sine"  # 覆蓋了 math.sin

# ✅ 好的做法
import math  # 導入模組

# 使用命名空間
MY_PI = 3.14  # 明確的命名
my_sin = "sine"

print(math.pi)  # 3.141592653589793
print(MY_PI)  # 3.14
```

### 範例 4: 動態作用域查看

```python
def show_scope_info():
    """展示當前作用域的所有變數"""
    local_var = "I'm local"

    print("=== 局部命名空間 ===")
    print(locals())

    print("\n=== 全局命名空間(部分) ===")
    global_vars = {k: v for k, v in globals().items()
                   if not k.startswith('__')}
    for name, value in list(global_vars.items())[:5]:
        print(f"{name}: {type(value)}")

global_var = "I'm global"
show_scope_info()
```

## 常見陷阱 (Common Pitfalls)

### 陷阱 1: 在賦值前引用變數

```python
x = 10

def func():
    print(x)  # UnboundLocalError!
    x = 20  # Python 看到這行,認為 x 是局部變數

# func()  # Error!

# ✅ 正確做法
def func_correct():
    global x
    print(x)  # 10
    x = 20
```

### 陷阱 2: 循環中的閉包

```python
# ❌ 錯誤:所有函數都引用同一個變數
functions = []
for i in range(3):
    functions.append(lambda: i)

print([f() for f in functions])  # [2, 2, 2] 而不是 [0, 1, 2]

# ✅ 正確:使用預設參數捕獲當前值
functions = []
for i in range(3):
    functions.append(lambda x=i: x)

print([f() for f in functions])  # [0, 1, 2]
```

### 陷阱 3: 可變預設參數

```python
# ❌ 危險!
def add_item(item, items=[]):
    items.append(item)
    return items

list1 = add_item("A")
list2 = add_item("B")
print(list1)  # ['A', 'B'] - 意外共享!
print(list2)  # ['A', 'B']

# ✅ 正確
def add_item_correct(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items

list1 = add_item_correct("A")
list2 = add_item_correct("B")
print(list1)  # ['A']
print(list2)  # ['B']
```

### 陷阱 4: 過度使用 global

```python
# ❌ 不好:過度使用 global
counter = 0

def increment():
    global counter
    counter += 1

def decrement():
    global counter
    counter -= 1

# ✅ 更好:使用類別封裝狀態
class Counter:
    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1

    def decrement(self):
        self.count -= 1

counter = Counter()
counter.increment()
counter.decrement()
```

## 與 System Design 的連結 (Connection to System Design)

### 1. 配置管理

```python
# 環境配置的命名空間管理
class Config:
    """全局配置 - 類似微服務的配置中心"""

    # 開發環境
    class Development:
        DEBUG = True
        DATABASE_URL = "postgresql://localhost/dev_db"

    # 生產環境
    class Production:
        DEBUG = False
        DATABASE_URL = "postgresql://prod-server/prod_db"

    # 當前環境
    current = Development

# 服務使用配置
class UserService:
    def __init__(self):
        self.db_url = Config.current.DATABASE_URL
```

### 2. 依賴注入

```python
# 使用閉包實現簡單的依賴注入
def create_service(db_connection, cache_client):
    """工廠函數 - 注入依賴"""

    def get_user(user_id):
        # 閉包捕獲 db_connection 和 cache_client
        cached = cache_client.get(f"user:{user_id}")
        if cached:
            return cached

        user = db_connection.query(f"SELECT * FROM users WHERE id={user_id}")
        cache_client.set(f"user:{user_id}", user)
        return user

    return get_user

# 使用
db = DatabaseConnection()
cache = RedisClient()
get_user = create_service(db, cache)
```

### 3. 避免全局狀態

在大型系統中,應該避免過度使用全局狀態:

```python
# ❌ 不好:全局狀態
current_user = None

def login(user):
    global current_user
    current_user = user

# ✅ 更好:請求上下文
class RequestContext:
    def __init__(self):
        self.user = None
        self.session = None

    def set_user(self, user):
        self.user = user

# 每個請求有自己的上下文
def handle_request(request):
    context = RequestContext()
    context.set_user(authenticate(request))
    return process_request(context)
```

## 練習題 (Exercises)

### 基礎練習

**練習 1:** 實作一個計數器
- 使用閉包
- 支援 increment, decrement, reset
- 不能使用類別

**練習 2:** 修復作用域錯誤
```python
# 找出並修復以下程式碼的問題
total = 0

def add(x):
    total = total + x
    return total

result = add(10)
```

### 進階練習

**練習 3:** 實作配置管理器
- 支援多環境配置(dev, staging, prod)
- 使用命名空間隔離
- 提供切換環境的功能

**練習 4:** 實作裝飾器工廠
- 使用閉包
- 支援參數化
- 正確處理作用域

### 挑戰練習

**練習 5:** 實作簡單的依賴注入容器
要求:
- 使用閉包管理依賴
- 支援單例和瞬時模式
- 避免命名衝突

## 總結

本章學習了:

✅ **命名空間概念**
- 內建、全局、局部命名空間
- 類別和實例命名空間

✅ **LEGB 規則**
- Python 的變數查找順序
- 理解作用域層次

✅ **global 和 nonlocal**
- 何時使用
- 如何正確使用

✅ **常見陷阱**
- 避免命名空間污染
- 正確使用閉包
- 避免可變預設參數

**下一章預告:** 底線命名慣例 - 學習 Python 的命名規範,理解 public, protected, private 的實作。

---

**學習建議:**
1. 理解 LEGB 規則是關鍵
2. 避免過度使用 global
3. 優先使用類別而不是全局變數
4. 了解閉包的強大之處
5. 在大型專案中特別注意命名空間管理
