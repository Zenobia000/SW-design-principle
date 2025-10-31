# 01 - OOP 基礎概念

## 本章目標 (Learning Objectives)
- 理解物件導向程式設計(OOP)的核心思想
- 掌握 Class 和 Instance 的概念與差異
- 學會定義屬性(Attributes)和方法(Methods)
- 熟練使用 `__init__` 和 `self`
- 能夠設計簡單的物件導向系統

## 為什麼需要這個? (Motivation)

### 問題場景
想像你正在開發一個電商系統,需要管理數千個商品。如果使用傳統的程序式編程:

```python
# 傳統做法 - 使用字典和函數
product1_name = "iPhone 15"
product1_price = 29900
product1_stock = 50

product2_name = "MacBook Pro"
product2_price = 59900
product2_stock = 20

def calculate_discount(price, discount_rate):
    return price * (1 - discount_rate)

# 隨著商品增加,程式碼會變得非常混亂...
```

這種做法的問題:
- 數據和行為分離,難以維護
- 容易出現命名衝突
- 難以擴展和重用
- 沒有封裝性,數據容易被誤修改

### OOP 的解決方案
物件導向將「數據」和「操作數據的方法」封裝在一起,讓程式更貼近真實世界的思維:

```python
# OOP 做法 - 使用類別
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def calculate_discount(self, discount_rate):
        return self.price * (1 - discount_rate)

    def is_available(self):
        return self.stock > 0

# 創建商品實例
iphone = Product("iPhone 15", 29900, 50)
macbook = Product("MacBook Pro", 59900, 20)

print(iphone.calculate_discount(0.1))  # 26910.0
print(macbook.is_available())  # True
```

## 核心概念 (Core Concepts)

### 1. 什麼是 OOP?

**物件導向程式設計(Object-Oriented Programming, OOP)** 是一種程式設計範式,其核心概念圍繞著「物件」這一概念進行。

**白話文:** 物件就是一種設計稿,裡面記錄著各種描述物體特性的**屬性**和**方法**。

#### OOP 的三個核心要素:
1. **類別(Class)** - 設計稿、藍圖
2. **物件/實例(Object/Instance)** - 根據設計稿製造出來的實體
3. **封裝(Encapsulation)** - 將數據和方法包裝在一起

### 2. Class vs Instance

```
┌─────────────────────────────────────────┐
│          Class (類別)                    │
│         設計稿/藍圖                       │
│  ┌─────────────────────────────────┐   │
│  │  屬性定義:                        │   │
│  │  - name                          │   │
│  │  - price                         │   │
│  │  - stock                         │   │
│  │                                  │   │
│  │  方法定義:                        │   │
│  │  - calculate_discount()          │   │
│  │  - is_available()                │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
           │
           │ 實例化 (Instantiation)
           ▼
┌──────────────────────┐  ┌──────────────────────┐
│  Instance 1          │  │  Instance 2          │
│  實際商品            │  │  實際商品            │
│                      │  │                      │
│  name: "iPhone 15"   │  │  name: "MacBook Pro" │
│  price: 29900        │  │  price: 59900        │
│  stock: 50           │  │  stock: 20           │
└──────────────────────┘  └──────────────────────┘
```

**類別(Class):** 是物件的設計圖,定義了物件應該有什麼屬性和方法。

**實例(Instance):** 是根據類別創建的具體物件,每個實例都有自己的屬性值。

### 3. 屬性與方法

#### 實例屬性 (Instance Attributes)
每個實例獨有的數據,在 `__init__` 方法中定義:

```python
class Product:
    def __init__(self, name, price, stock):
        self.name = name      # 實例屬性
        self.price = price    # 實例屬性
        self.stock = stock    # 實例屬性
```

#### 類別屬性 (Class Attributes)
所有實例共享的數據:

```python
class Product:
    tax_rate = 0.05  # 類別屬性,所有商品共享

    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    def get_price_with_tax(self):
        return self.price * (1 + Product.tax_rate)

# 所有實例共享相同的 tax_rate
iphone = Product("iPhone 15", 29900, 50)
macbook = Product("MacBook Pro", 59900, 20)

print(iphone.tax_rate)   # 0.05
print(macbook.tax_rate)  # 0.05

# 修改類別屬性會影響所有實例
Product.tax_rate = 0.07
print(iphone.tax_rate)   # 0.07
print(macbook.tax_rate)  # 0.07
```

#### 方法 (Methods)
定義在類別中的函數:

```python
class Product:
    def __init__(self, name, price, stock):
        self.name = name
        self.price = price
        self.stock = stock

    # 實例方法 - 操作實例數據
    def calculate_discount(self, discount_rate):
        return self.price * (1 - discount_rate)

    # 實例方法 - 修改實例狀態
    def sell(self, quantity):
        if quantity <= self.stock:
            self.stock -= quantity
            return True
        return False

    # 實例方法 - 查詢實例狀態
    def is_available(self):
        return self.stock > 0
```

### 4. `__init__` 與 `self`

#### `__init__` 方法
- 建構函數(Constructor),在創建實例時自動調用
- 用於初始化實例屬性
- 第一個參數必須是 `self`

```python
class Product:
    def __init__(self, name, price, stock=0):  # stock 有預設值
        """
        初始化商品

        Args:
            name: 商品名稱
            price: 商品價格
            stock: 庫存數量(預設為0)
        """
        self.name = name
        self.price = price
        self.stock = stock
        self.created_at = datetime.now()  # 自動記錄創建時間

# 創建實例時,__init__ 自動執行
product = Product("iPhone 15", 29900, 50)
```

#### `self` 參數
- 代表實例本身
- 透過 `self` 存取實例的屬性和方法
- Python 自動傳遞,不需要手動提供

```python
class Product:
    def __init__(self, name, price):
        self.name = name      # self.name 是實例屬性
        self.price = price

    def display_info(self):
        # 透過 self 存取實例屬性
        print(f"商品: {self.name}, 價格: ${self.price}")

    def apply_discount(self, rate):
        # 透過 self 修改實例屬性
        self.price = self.price * (1 - rate)
        # 透過 self 調用其他方法
        self.display_info()

product = Product("iPhone 15", 29900)
product.display_info()        # 商品: iPhone 15, 價格: $29900
product.apply_discount(0.1)   # 商品: iPhone 15, 價格: $26910.0
```

**為什麼需要 self?**
- 區分實例屬性和局部變數
- 允許實例之間獨立運作
- 實現物件的封裝性

## 實戰範例 (Hands-on Examples)

### 範例 1: 電商系統 - 商品管理

```python
from datetime import datetime

class Product:
    """電商商品類別"""

    # 類別屬性 - 所有商品共享
    tax_rate = 0.05
    total_products = 0

    def __init__(self, name, price, stock, category):
        # 實例屬性 - 每個商品獨有
        self.name = name
        self.price = price
        self.stock = stock
        self.category = category
        self.created_at = datetime.now()

        # 更新商品總數
        Product.total_products += 1

    def calculate_discount(self, discount_rate):
        """計算折扣價"""
        if not 0 <= discount_rate <= 1:
            raise ValueError("折扣率必須在 0-1 之間")
        return self.price * (1 - discount_rate)

    def get_price_with_tax(self):
        """計算含稅價格"""
        return self.price * (1 + self.tax_rate)

    def sell(self, quantity):
        """銷售商品"""
        if quantity > self.stock:
            print(f"庫存不足! 當前庫存: {self.stock}")
            return False

        self.stock -= quantity
        print(f"成功銷售 {quantity} 件 {self.name}")
        return True

    def restock(self, quantity):
        """補貨"""
        self.stock += quantity
        print(f"{self.name} 已補貨 {quantity} 件,當前庫存: {self.stock}")

    def is_available(self):
        """檢查是否有貨"""
        return self.stock > 0

    def display_info(self):
        """顯示商品資訊"""
        status = "有貨" if self.is_available() else "缺貨"
        print(f"""
        ========== 商品資訊 ==========
        名稱: {self.name}
        分類: {self.category}
        價格: ${self.price:,}
        含稅價: ${self.get_price_with_tax():,.2f}
        庫存: {self.stock} ({status})
        建立時間: {self.created_at.strftime('%Y-%m-%d %H:%M')}
        ============================
        """)

# 使用範例
if __name__ == "__main__":
    # 創建商品實例
    iphone = Product("iPhone 15 Pro", 29900, 50, "智慧型手機")
    macbook = Product("MacBook Pro 16\"", 79900, 20, "筆記型電腦")
    airpods = Product("AirPods Pro", 7490, 100, "耳機")

    # 顯示商品資訊
    iphone.display_info()

    # 計算折扣
    discount_price = iphone.calculate_discount(0.1)
    print(f"九折優惠價: ${discount_price:,.0f}")

    # 銷售商品
    iphone.sell(5)
    iphone.sell(60)  # 庫存不足

    # 補貨
    iphone.restock(30)

    # 查看總商品數
    print(f"\n系統中共有 {Product.total_products} 種商品")
```

**輸出:**
```
        ========== 商品資訊 ==========
        名稱: iPhone 15 Pro
        分類: 智慧型手機
        價格: $29,900
        含稅價: $31,395.00
        庫存: 50 (有貨)
        建立時間: 2025-10-31 14:30
        ============================

九折優惠價: $26,910
成功銷售 5 件 iPhone 15 Pro
庫存不足! 當前庫存: 45
iPhone 15 Pro 已補貨 30 件,當前庫存: 75

系統中共有 3 種商品
```

### 範例 2: 電商系統 - 購物車

```python
class ShoppingCart:
    """購物車類別"""

    def __init__(self, customer_name):
        self.customer_name = customer_name
        self.items = []  # 儲存商品列表
        self.created_at = datetime.now()

    def add_item(self, product, quantity=1):
        """加入商品到購物車"""
        if not product.is_available():
            print(f"{product.name} 目前缺貨")
            return False

        if quantity > product.stock:
            print(f"{product.name} 庫存不足,當前庫存: {product.stock}")
            return False

        self.items.append({
            'product': product,
            'quantity': quantity,
            'unit_price': product.price
        })
        print(f"已將 {quantity} 件 {product.name} 加入購物車")
        return True

    def remove_item(self, product_name):
        """從購物車移除商品"""
        for item in self.items:
            if item['product'].name == product_name:
                self.items.remove(item)
                print(f"已移除 {product_name}")
                return True
        print(f"購物車中沒有 {product_name}")
        return False

    def get_total(self):
        """計算總金額"""
        total = sum(item['unit_price'] * item['quantity'] for item in self.items)
        return total

    def get_total_with_tax(self):
        """計算含稅總金額"""
        total = self.get_total()
        # 假設所有商品使用相同稅率
        if self.items:
            tax_rate = self.items[0]['product'].tax_rate
            return total * (1 + tax_rate)
        return 0

    def display_cart(self):
        """顯示購物車內容"""
        print(f"\n{'='*50}")
        print(f"{self.customer_name} 的購物車")
        print(f"{'='*50}")

        if not self.items:
            print("購物車是空的")
            return

        for i, item in enumerate(self.items, 1):
            product = item['product']
            quantity = item['quantity']
            subtotal = item['unit_price'] * quantity
            print(f"{i}. {product.name}")
            print(f"   單價: ${item['unit_price']:,} x {quantity} = ${subtotal:,}")

        print(f"{'-'*50}")
        print(f"小計: ${self.get_total():,}")
        print(f"含稅總額: ${self.get_total_with_tax():,.2f}")
        print(f"{'='*50}\n")

    def checkout(self):
        """結帳"""
        if not self.items:
            print("購物車是空的,無法結帳")
            return False

        print("正在處理訂單...")

        # 扣除庫存
        for item in self.items:
            product = item['product']
            quantity = item['quantity']
            product.sell(quantity)

        total = self.get_total_with_tax()
        print(f"\n結帳成功! 總金額: ${total:,.2f}")
        print("感謝您的購買!")

        # 清空購物車
        self.items = []
        return True

# 使用範例
if __name__ == "__main__":
    # 創建商品
    iphone = Product("iPhone 15 Pro", 29900, 50, "智慧型手機")
    airpods = Product("AirPods Pro", 7490, 100, "耳機")

    # 創建購物車
    cart = ShoppingCart("張三")

    # 加入商品
    cart.add_item(iphone, 2)
    cart.add_item(airpods, 1)

    # 顯示購物車
    cart.display_cart()

    # 結帳
    cart.checkout()
```

## 常見陷阱 (Common Pitfalls)

### 陷阱 1: 忘記 self 參數

```python
# ❌ 錯誤
class Product:
    def __init__(name, price):  # 缺少 self
        self.name = name
        self.price = price

# 會報錯: __init__() takes 2 positional arguments but 3 were given

# ✅ 正確
class Product:
    def __init__(self, name, price):
        self.name = name
        self.price = price
```

### 陷阱 2: 混淆類別屬性和實例屬性

```python
# ❌ 可能的問題
class Product:
    stock = 0  # 類別屬性 - 所有實例共享!

    def __init__(self, name):
        self.name = name

p1 = Product("A")
p2 = Product("B")

p1.stock = 10  # 這會創建一個新的實例屬性,不會影響 p2
print(p2.stock)  # 0 (仍是類別屬性的值)

Product.stock = 20  # 修改類別屬性
print(p2.stock)  # 20 (因為 p2 沒有自己的 stock 實例屬性)
print(p1.stock)  # 10 (因為 p1 有自己的 stock 實例屬性)

# ✅ 正確做法
class Product:
    def __init__(self, name, stock):
        self.name = name
        self.stock = stock  # 實例屬性
```

### 陷阱 3: 在 `__init__` 外定義實例屬性

```python
# ⚠️ 不建議
class Product:
    def __init__(self, name):
        self.name = name

    def set_price(self, price):
        self.price = price  # 在 __init__ 外定義

p = Product("iPhone")
print(p.price)  # AttributeError: 'Product' object has no attribute 'price'

# ✅ 建議
class Product:
    def __init__(self, name, price=0):
        self.name = name
        self.price = price  # 在 __init__ 中定義,並給預設值
```

### 陷阱 4: 可變預設參數

```python
# ❌ 危險!
class ShoppingCart:
    def __init__(self, items=[]):  # 可變預設參數
        self.items = items

cart1 = ShoppingCart()
cart2 = ShoppingCart()

cart1.items.append("iPhone")
print(cart2.items)  # ['iPhone'] - 意外共享了!

# ✅ 正確
class ShoppingCart:
    def __init__(self, items=None):
        self.items = items if items is not None else []
```

## 與 System Design 的連結 (Connection to System Design)

### 1. 單一職責原則 (Single Responsibility Principle)
每個類別只負責一件事:
- `Product` 類別只負責商品相關邏輯
- `ShoppingCart` 類別只負責購物車邏輯
- 未來可以新增 `Order`、`Payment` 等類別

### 2. 模組化設計
OOP 的封裝特性讓系統更模組化:
```
電商系統架構
├── Product (商品模組)
├── ShoppingCart (購物車模組)
├── Order (訂單模組)
├── Payment (支付模組)
└── User (用戶模組)
```

### 3. 可擴展性
透過 OOP,我們可以輕鬆擴展系統:
```python
# 未來可以新增不同類型的商品
class DigitalProduct(Product):  # 數位商品
    pass

class PhysicalProduct(Product):  # 實體商品
    pass
```

### 4. 數據封裝與安全性
在大型系統中,封裝可以:
- 保護數據不被直接修改
- 提供統一的訪問介面
- 便於添加驗證邏輯

## 練習題 (Exercises)

### 基礎練習

**練習 1:** 創建一個 `User` 類別
- 屬性: username, email, created_at
- 方法: update_email(), display_info()

**練習 2:** 創建一個 `BankAccount` 類別
- 屬性: account_number, balance, owner
- 方法: deposit(), withdraw(), get_balance()

### 進階練習

**練習 3:** 擴展電商系統
在現有的 `Product` 和 `ShoppingCart` 基礎上,新增:
- `Order` 類別:管理訂單狀態
- `Customer` 類別:管理客戶資訊
- 實現完整的購物流程

**練習 4:** 設計一個圖書館管理系統
- `Book` 類別:書籍資訊
- `Member` 類別:會員資訊
- `Library` 類別:管理借還書

### 挑戰練習

**練習 5:** 設計一個簡單的社交媒體系統
需求:
- 用戶可以發布貼文
- 用戶可以追蹤其他用戶
- 用戶可以對貼文按讚和評論

提示:需要設計 `User`、`Post`、`Comment` 等類別

---

## 總結

本章學習了:
- ✅ OOP 的核心概念與優勢
- ✅ Class 和 Instance 的區別
- ✅ 屬性和方法的定義與使用
- ✅ `__init__` 和 `self` 的作用
- ✅ 實戰:設計電商系統的基本元件

**下一章預告:** 我們將深入學習 OOP 的四大支柱:封裝、繼承、多型、抽象,並了解如何運用這些特性設計更強大的系統。

---

**學習建議:**
1. 動手實作所有範例程式碼
2. 完成至少 3 個練習題
3. 嘗試設計自己的類別來解決實際問題
4. 思考如何將學到的概念應用到實際專案中
