# 單一職責原則 (SRP) - 深度實戰指南

> "一個類別應該只有一個改變的理由" - Robert C. Martin

## 目錄
- [核心理念](#核心理念)
- [設計經驗](#設計經驗)
- [常見反模式](#常見反模式)
- [實戰技巧](#實戰技巧)
- [重構策略](#重構策略)
- [無招勝有招 - 內化心法](#無招勝有招---內化心法)

---

## 核心理念

### 什麼是「職責」?

職責不是指「方法數量」,而是指「改變的理由」:

```python
# ❌ 錯誤理解：認為一個類只能有一個方法
class User:
    def get_name(self):
        pass
    # 不能再有其他方法了？錯！

# ✅ 正確理解：相關的方法可以在同一類中
class User:
    def get_name(self):
        return self.name

    def get_email(self):
        return self.email

    def get_full_info(self):
        return f"{self.name} <{self.email}>"

    # 這些都是「用戶資料訪問」這個單一職責
```

### 如何判斷職責邊界?

**關鍵問題**: "這個類別會因為什麼原因而需要修改?"

```python
# ❌ 違反 SRP - 有多個改變的理由
class UserService:
    def __init__(self, db_connection):
        self.db = db_connection

    # 理由 1: 業務規則變更
    def register_user(self, username, password):
        # 驗證邏輯
        if len(password) < 8:
            raise ValueError("密碼太短")

        # 理由 2: 資料庫結構變更
        self.db.execute(
            "INSERT INTO users (username, password) VALUES (?, ?)",
            (username, password)
        )

        # 理由 3: 通知方式變更
        self._send_welcome_email(username)

    def _send_welcome_email(self, username):
        print(f"發送歡迎郵件給 {username}")
```

---

## 設計經驗

### 經驗 1: 按「變更的源頭」分離職責

不同的利益相關者(stakeholder)會導致不同的變更需求:

```python
# ❌ 混合了多個利益相關者的需求
class Employee:
    def __init__(self, name, salary, role):
        self.name = name
        self.salary = salary
        self.role = role

    # 財務部門關心
    def calculate_pay(self):
        return self.salary * 1.1

    # HR 部門關心
    def calculate_vacation_days(self):
        return 20 if self.role == "senior" else 15

    # IT 部門關心
    def save_to_database(self):
        print(f"Saving {self.name} to DB")

# ✅ 按利益相關者分離
class Employee:
    """核心員工數據 - 所有部門都關心"""
    def __init__(self, name, salary, role):
        self.name = name
        self.salary = salary
        self.role = role

class PayrollCalculator:
    """薪資計算 - 財務部門負責"""
    def calculate_pay(self, employee: Employee):
        return employee.salary * 1.1

class VacationPolicy:
    """假期政策 - HR 部門負責"""
    def calculate_vacation_days(self, employee: Employee):
        return 20 if employee.role == "senior" else 15

class EmployeeRepository:
    """資料持久化 - IT 部門負責"""
    def save(self, employee: Employee):
        print(f"Saving {employee.name} to DB")
```

**實戰心得**:
- 如果修改需求來自不同部門/角色,就該分離
- 分離後,各部門的需求變更不會相互影響

### 經驗 2: 內聚性檢查法

**高內聚**: 類別中的方法都使用類別中的大部分屬性

```python
# ✅ 高內聚 - 所有方法都使用大部分屬性
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

    def is_square(self):
        return self.width == self.height

# ❌ 低內聚 - 部分方法不相關
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.creation_date = None  # 這個屬性只被部分方法使用

    def area(self):
        return self.width * self.height

    def save_to_file(self, filename):  # 這個方法和幾何計算無關
        with open(filename, 'w') as f:
            f.write(f"{self.width},{self.height}")

    def log_creation(self):  # 這個方法和幾何計算無關
        print(f"Created at {self.creation_date}")
```

### 經驗 3: 提取直到痛苦消失

重構時的策略:「如果感到混亂,就繼續提取」

```python
# 階段 1: 原始混亂代碼
class OrderProcessor:
    def process(self, order_data):
        # 驗證
        if not order_data.get('items'):
            raise ValueError("No items")

        # 計算
        total = sum(item['price'] * item['qty'] for item in order_data['items'])

        # 折扣
        if order_data.get('coupon'):
            total *= 0.9

        # 稅
        total *= 1.1

        # 保存
        self.db.save(order_data)

        # 通知
        self.send_email(order_data['customer_email'])

# 階段 2: 第一次提取
class OrderProcessor:
    def process(self, order_data):
        self._validate(order_data)
        total = self._calculate_total(order_data)
        self._save_order(order_data, total)
        self._notify_customer(order_data)

    def _validate(self, order_data):
        if not order_data.get('items'):
            raise ValueError("No items")

    def _calculate_total(self, order_data):
        total = sum(item['price'] * item['qty'] for item in order_data['items'])
        if order_data.get('coupon'):
            total *= 0.9
        total *= 1.1
        return total

    # ... 其他方法

# 階段 3: 繼續提取直到清晰
class OrderValidator:
    def validate(self, order_data):
        if not order_data.get('items'):
            raise ValueError("No items")

class PriceCalculator:
    def calculate_subtotal(self, items):
        return sum(item['price'] * item['qty'] for item in items)

    def apply_discount(self, total, coupon):
        return total * 0.9 if coupon else total

    def apply_tax(self, total, tax_rate=0.1):
        return total * (1 + tax_rate)

class OrderRepository:
    def save(self, order_data, total):
        self.db.save({**order_data, 'total': total})

class NotificationService:
    def notify_order_placed(self, customer_email):
        self.send_email(customer_email)

class OrderProcessor:
    def __init__(self, validator, calculator, repository, notifier):
        self.validator = validator
        self.calculator = calculator
        self.repository = repository
        self.notifier = notifier

    def process(self, order_data):
        self.validator.validate(order_data)

        subtotal = self.calculator.calculate_subtotal(order_data['items'])
        subtotal = self.calculator.apply_discount(subtotal, order_data.get('coupon'))
        total = self.calculator.apply_tax(subtotal)

        self.repository.save(order_data, total)
        self.notifier.notify_order_placed(order_data['customer_email'])
```

---

## 常見反模式

### 反模式 1: God Object (上帝物件)

**症狀**: 一個類別什麼都做

```python
# ❌ God Object
class Application:
    def __init__(self):
        self.users = []
        self.products = []
        self.orders = []

    def register_user(self, username): ...
    def login_user(self, username, password): ...
    def add_product(self, name, price): ...
    def search_products(self, query): ...
    def create_order(self, user_id, product_ids): ...
    def calculate_order_total(self, order_id): ...
    def send_invoice(self, order_id): ...
    def generate_report(self): ...
    def backup_database(self): ...
    # ... 還有 30 個方法

# ✅ 重構: 按領域分離
class UserService: ...
class ProductService: ...
class OrderService: ...
class ReportService: ...
class BackupService: ...
```

### 反模式 2: Shotgun Surgery (散彈槍手術)

**症狀**: 一個小變更需要修改很多類別

```python
# ❌ 日期格式散落各處
class UserProfile:
    def display_join_date(self):
        return self.join_date.strftime("%Y-%m-%d")

class OrderHistory:
    def display_order_date(self):
        return self.order_date.strftime("%Y-%m-%d")

class Report:
    def format_report_date(self):
        return self.report_date.strftime("%Y-%m-%d")

# 如果要改成 "%d/%m/%Y" 格式,需要改 3 個地方!

# ✅ 集中職責
class DateFormatter:
    @staticmethod
    def format_date(date):
        return date.strftime("%Y-%m-%d")

class UserProfile:
    def display_join_date(self):
        return DateFormatter.format_date(self.join_date)

class OrderHistory:
    def display_order_date(self):
        return DateFormatter.format_date(self.order_date)
```

### 反模式 3: Feature Envy (特性依戀)

**症狀**: 一個方法過度使用另一個類別的數據

```python
# ❌ Feature Envy
class Invoice:
    def __init__(self, customer):
        self.customer = customer

    def calculate_discount(self):
        # 過度依戀 customer 的數據
        if self.customer.age > 65:
            return 0.1
        elif self.customer.membership_years > 5:
            return 0.05
        elif self.customer.total_purchases > 10000:
            return 0.08
        return 0

# ✅ 移動職責
class Customer:
    def get_discount_rate(self):
        if self.age > 65:
            return 0.1
        elif self.membership_years > 5:
            return 0.05
        elif self.total_purchases > 10000:
            return 0.08
        return 0

class Invoice:
    def __init__(self, customer):
        self.customer = customer

    def calculate_discount(self):
        return self.customer.get_discount_rate()
```

---

## 實戰技巧

### 技巧 1: 職責嗅探清單

在設計或審查類別時,問自己:

- [ ] 這個類別能用一句話描述嗎?
- [ ] 類別名稱中是否包含 "and"、"Manager"、"Handler"、"Processor"?
- [ ] 有多少個公開方法? (>7 個就要警惕)
- [ ] 修改這個類別的原因有幾個?
- [ ] 不同的方法是否使用不同的屬性集?
- [ ] 能否輕鬆為這個類別寫單元測試?

### 技巧 2: 類別職責矩陣

使用表格分析職責:

| 方法名 | 使用屬性 | 變更原因 | 誰關心 |
|--------|----------|----------|--------|
| calculate_pay | salary | 薪資政策變更 | 財務部 |
| save_to_db | id, name | 資料庫結構變更 | IT 部 |
| send_email | email | 郵件模板變更 | 市場部 |

如果「變更原因」或「誰關心」這兩列有不同的值,就該考慮拆分。

### 技巧 3: 提取的優先順序

1. **先提取「技術細節」**: 資料庫、文件 I/O、網路呼叫
2. **再提取「業務規則」**: 計算、驗證、策略
3. **最後提取「協調邏輯」**: 工作流、狀態機

```python
# 步驟 1: 提取技術細節
class UserRepository:
    def save(self, user): ...

# 步驟 2: 提取業務規則
class PasswordValidator:
    def validate(self, password): ...

# 步驟 3: 協調
class UserRegistrationService:
    def __init__(self, repository, validator):
        self.repository = repository
        self.validator = validator

    def register(self, username, password):
        self.validator.validate(password)  # 業務規則
        user = User(username, password)
        self.repository.save(user)  # 技術細節
```

---

## 重構策略

### 策略 1: Extract Class (提取類別)

**時機**: 當一個類別有兩組不同的方法和屬性

```python
# Before
class Person:
    def __init__(self, name, street, city, zipcode):
        self.name = name
        self.street = street
        self.city = city
        self.zipcode = zipcode

    def get_full_address(self):
        return f"{self.street}, {self.city} {self.zipcode}"

# After
class Address:
    def __init__(self, street, city, zipcode):
        self.street = street
        self.city = city
        self.zipcode = zipcode

    def get_full_address(self):
        return f"{self.street}, {self.city} {self.zipcode}"

class Person:
    def __init__(self, name, address: Address):
        self.name = name
        self.address = address
```

### 策略 2: Extract Method (提取方法)

**時機**: 當一個方法太長或有註釋說明某段代碼的功能

```python
# Before
def process_order(order):
    # 驗證訂單
    if not order.items:
        raise ValueError("Empty order")

    # 計算總價
    total = 0
    for item in order.items:
        total += item.price * item.quantity

    # 應用折扣
    if order.coupon:
        total *= 0.9

# After
def process_order(order):
    validate_order(order)
    total = calculate_total(order)
    total = apply_discount(total, order.coupon)
    return total

def validate_order(order):
    if not order.items:
        raise ValueError("Empty order")

def calculate_total(order):
    return sum(item.price * item.quantity for item in order.items)

def apply_discount(total, coupon):
    return total * 0.9 if coupon else total
```

### 策略 3: Replace Method with Method Object

**時機**: 當一個方法有太多局部變數難以提取

```python
# Before
class PriceCalculator:
    def calculate_price(self, order):
        base_price = order.quantity * order.item_price
        quantity_discount = max(0, order.quantity - 500) * order.item_price * 0.05
        shipping = min(base_price * 0.1, 100)
        # ... 還有很多局部變數和計算
        return base_price - quantity_discount + shipping

# After
class PriceCalculation:
    def __init__(self, order):
        self.order = order
        self.base_price = 0
        self.quantity_discount = 0
        self.shipping = 0

    def calculate(self):
        self.calculate_base_price()
        self.calculate_quantity_discount()
        self.calculate_shipping()
        return self.base_price - self.quantity_discount + self.shipping

    def calculate_base_price(self):
        self.base_price = self.order.quantity * self.order.item_price

    def calculate_quantity_discount(self):
        self.quantity_discount = max(0, self.order.quantity - 500) * self.order.item_price * 0.05

    def calculate_shipping(self):
        self.shipping = min(self.base_price * 0.1, 100)

class PriceCalculator:
    def calculate_price(self, order):
        return PriceCalculation(order).calculate()
```

---

## 無招勝有招 - 內化心法

當你完全掌握 SRP,不再需要刻意思考「是否符合 SRP」,而是:

### 心法 1: 直覺分離

**從**: "我需要檢查這個類別是否符合 SRP"
**到**: "這段代碼讓我感到不舒服,應該分離"

訓練方法:
1. 每次寫代碼時,留意「混亂感」
2. 當你需要註釋解釋一段代碼時,考慮提取
3. 當類別名稱難以命名時,可能職責不清

### 心法 2: 自然分層

**從**: "我需要按 SRP 分離這些類別"
**到**: "系統自然形成了這些層次"

分層模式:
```
Presentation Layer (UI)
    ↓
Application Layer (Use Cases)
    ↓
Domain Layer (Business Logic)
    ↓
Infrastructure Layer (技術細節)
```

每層自然有不同的職責,不用刻意思考。

### 心法 3: 小步重構

**從**: "這個類別太大了,需要一次性重構"
**到**: "每次只改善一點點"

實踐:
- 每次提交只做一個小重構
- 保持測試一直通過
- 逐步演進而非大爆炸式重構

### 心法 4: 命名驅動設計

**從**: "先寫代碼再想名字"
**到**: "名字決定了職責"

練習:
```python
# 如果你寫出這樣的名字,就知道職責不清
class UserManagerHandlerProcessorService:
    pass

# 好的名字自然清晰
class UserAuthenticator:
    pass

class UserProfileRepository:
    pass
```

### 心法 5: 測試即設計

**從**: "寫完代碼再寫測試"
**到**: "難測試的代碼就是設計不良"

如果測試難寫:
- 需要 mock 很多東西 → 職責太多
- 測試需要很多 setup → 依賴太複雜
- 不知道測試什麼 → 職責不清

### 終極心法: 忘記 SRP

當你不再想著「這符合 SRP 嗎」,而是自然地寫出單一職責的代碼時,你就真正掌握了 SRP。

**就像學武功**:
1. **有招**: 刻意練習每個招式 (學習 SRP 規則)
2. **熟招**: 招式連貫流暢 (能識別和重構違反 SRP 的代碼)
3. **無招**: 隨心所欲不逾矩 (自然寫出符合 SRP 的代碼)

---

## 實戰練習

### 練習 1: 識別職責

分析以下代碼,找出有幾個職責:

```python
class BlogPost:
    def __init__(self, title, content):
        self.title = title
        self.content = content
        self.created_at = datetime.now()

    def format_for_display(self):
        return f"<h1>{self.title}</h1><p>{self.content}</p>"

    def save_to_database(self):
        db.execute("INSERT INTO posts VALUES (?, ?)", (self.title, self.content))

    def send_notification(self):
        email.send(f"New post: {self.title}")
```

<details>
<summary>答案</summary>

有 4 個職責:
1. 資料模型 (title, content, created_at)
2. 顯示格式化 (format_for_display)
3. 資料持久化 (save_to_database)
4. 通知發送 (send_notification)

應該重構為 4 個類別:
- `BlogPost` (資料模型)
- `BlogPostFormatter` (顯示)
- `BlogPostRepository` (持久化)
- `NotificationService` (通知)
</details>

### 練習 2: 重構實戰

重構以下代碼:

```python
class Report:
    def generate(self, data):
        # 過濾資料
        filtered_data = [d for d in data if d['amount'] > 100]

        # 計算統計
        total = sum(d['amount'] for d in filtered_data)
        average = total / len(filtered_data) if filtered_data else 0

        # 格式化輸出
        output = f"Total: {total}\nAverage: {average}\n"
        output += "Details:\n"
        for d in filtered_data:
            output += f"- {d['name']}: {d['amount']}\n"

        # 儲存到文件
        with open('report.txt', 'w') as f:
            f.write(output)

        return output
```

<details>
<summary>參考答案</summary>

```python
class DataFilter:
    @staticmethod
    def filter_by_amount(data, min_amount):
        return [d for d in data if d['amount'] > min_amount]

class ReportCalculator:
    @staticmethod
    def calculate_total(data):
        return sum(d['amount'] for d in data)

    @staticmethod
    def calculate_average(data):
        return ReportCalculator.calculate_total(data) / len(data) if data else 0

class ReportFormatter:
    def format(self, data, total, average):
        output = f"Total: {total}\nAverage: {average}\n"
        output += "Details:\n"
        for d in data:
            output += f"- {d['name']}: {d['amount']}\n"
        return output

class ReportWriter:
    @staticmethod
    def write_to_file(content, filename):
        with open(filename, 'w') as f:
            f.write(content)

class ReportGenerator:
    def __init__(self):
        self.filter = DataFilter()
        self.calculator = ReportCalculator()
        self.formatter = ReportFormatter()
        self.writer = ReportWriter()

    def generate(self, data):
        filtered_data = self.filter.filter_by_amount(data, 100)
        total = self.calculator.calculate_total(filtered_data)
        average = self.calculator.calculate_average(filtered_data)
        output = self.formatter.format(filtered_data, total, average)
        self.writer.write_to_file(output, 'report.txt')
        return output
```
</details>

---

## 總結金句

1. **職責 ≠ 方法數量**,而是「改變的理由」
2. **高內聚,低耦合** - 相關的放一起,不相關的分開
3. **命名困難 = 職責不清** - 好名字是清晰職責的副產品
4. **小步重構** - 每次只改善一點點
5. **測試是設計的鏡子** - 難測試 = 設計不良
6. **忘記規則** - 當你不再刻意遵守時,你就真正掌握了

---

## 延伸閱讀

- Clean Code (Robert C. Martin) - Chapter 10: Classes
- Refactoring (Martin Fowler) - Extract Class, Extract Method
- Agile Software Development, Principles, Patterns, and Practices (Robert C. Martin)

---

**記住**: SRP 不是教條,而是幫助你寫出更好代碼的指導原則。當你內化了這個原則,你會發現自己自然而然地寫出單一職責的代碼,這就是「無招勝有招」的境界。
