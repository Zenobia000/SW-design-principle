# 開放封閉原則 (OCP) - 深度實戰指南

> "軟體實體應該對擴展開放,對修改封閉" - Bertrand Meyer

## 目錄
- [核心理念](#核心理念)
- [設計經驗](#設計經驗)
- [擴展策略](#擴展策略)
- [常見陷阱](#常見陷阱)
- [實戰模式](#實戰模式)
- [無招勝有招 - 預見變化](#無招勝有招---預見變化)

---

## 核心理念

### 什麼是「開放」與「封閉」?

- **對擴展開放**: 當需求變更時,可以通過添加新代碼來擴展功能
- **對修改封閉**: 已有的代碼不需要修改,保持穩定

```python
# ❌ 違反 OCP - 每次新增支付方式都要修改
class PaymentProcessor:
    def process(self, payment_type, amount):
        if payment_type == "credit_card":
            print(f"Processing credit card payment: ${amount}")
        elif payment_type == "paypal":
            print(f"Processing PayPal payment: ${amount}")
        elif payment_type == "crypto":  # 新增需求:要修改這個類別
            print(f"Processing crypto payment: ${amount}")
        # 每次新增都要改這裡!

# ✅ 符合 OCP - 通過擴展新增功能
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        pass

class CreditCardPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing credit card payment: ${amount}")

class PayPalPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing PayPal payment: ${amount}")

# 新增支付方式:只需添加新類別,不修改現有代碼
class CryptoPayment(PaymentMethod):
    def process(self, amount):
        print(f"Processing crypto payment: ${amount}")

class PaymentProcessor:
    def process(self, payment_method: PaymentMethod, amount):
        payment_method.process(amount)
```

### OCP 的本質: 抽象

OCP 的關鍵在於**找到正確的抽象點**:

```python
# 問自己: "什麼會變? 什麼不變?"

# 會變: 具體的支付方式 (信用卡、PayPal、加密貨幣...)
# 不變: 都需要「處理支付」這個動作

# 所以抽象出「支付方法」接口
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount):
        pass
```

---

## 設計經驗

### 經驗 1: 不要過早抽象

**錯誤做法**: 一開始就建立複雜的抽象體系

```python
# ❌ 過早抽象 - 只有一種支付方式時就建立抽象
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount): pass

    @abstractmethod
    def refund(self, amount): pass

    @abstractmethod
    def validate(self): pass

    @abstractmethod
    def get_fee(self): pass

class CreditCardPayment(PaymentMethod):
    # 實作很多可能永遠不會變化的方法
    pass

# ✅ 正確做法 - Rule of Three: 重複三次再抽象
# 第一次: 直接實現
def process_credit_card(amount):
    print(f"Processing credit card: ${amount}")

# 第二次: 發現類似模式,但先忍住
def process_paypal(amount):
    print(f"Processing PayPal: ${amount}")

# 第三次: 明確模式,開始抽象
class PaymentMethod(ABC):
    @abstractmethod
    def process(self, amount): pass
```

### 經驗 2: 識別變化軸

系統可能有多個變化維度,需要識別主要變化軸:

```python
# 變化軸分析: 報表系統
# 軸 1: 報表類型 (銷售報表、庫存報表、財務報表)
# 軸 2: 輸出格式 (PDF、Excel、HTML)

# ❌ 錯誤: 混合多個變化軸
class SalesPDFReport: pass
class SalesExcelReport: pass
class SalesHTMLReport: pass
class InventoryPDFReport: pass
class InventoryExcelReport: pass
# ... 組合爆炸!

# ✅ 正確: 分離變化軸 (Strategy Pattern + Template Method)
class Report(ABC):
    def __init__(self, formatter):
        self.formatter = formatter

    @abstractmethod
    def collect_data(self):
        pass

    def generate(self):
        data = self.collect_data()
        return self.formatter.format(data)

class SalesReport(Report):
    def collect_data(self):
        return {"total_sales": 10000}

class InventoryReport(Report):
    def collect_data(self):
        return {"total_items": 500}

# 格式化是另一個變化軸
class PDFFormatter:
    def format(self, data):
        return f"PDF: {data}"

class ExcelFormatter:
    def format(self, data):
        return f"Excel: {data}"

# 使用: 兩個軸獨立變化
sales_pdf = SalesReport(PDFFormatter())
sales_excel = SalesReport(ExcelFormatter())
inventory_pdf = InventoryReport(PDFFormatter())
```

### 經驗 3: 穩定的依賴於抽象

**依賴穩定性原則**: 依賴應該指向更穩定的方向

```python
# 穩定性分析:
# - 抽象接口: 非常穩定 (很少改變)
# - 業務邏輯: 中等穩定
# - 技術細節: 不穩定 (經常改變)

# ✅ 正確的依賴方向
class OrderService:  # 業務邏輯
    def __init__(self, repository: OrderRepository):  # 依賴抽象
        self.repository = repository

class OrderRepository(ABC):  # 抽象 (穩定)
    @abstractmethod
    def save(self, order): pass

class MySQLOrderRepository(OrderRepository):  # 技術細節 (不穩定)
    def save(self, order):
        # MySQL 特定實現
        pass

class MongoOrderRepository(OrderRepository):  # 另一個技術細節
    def save(self, order):
        # MongoDB 特定實現
        pass

# 當資料庫從 MySQL 切換到 MongoDB:
# - OrderService 不需要改變 ✓
# - OrderRepository 接口不需要改變 ✓
# - 只需要添加 MongoOrderRepository 並修改配置
```

---

## 擴展策略

### 策略 1: 策略模式 (Strategy Pattern)

**適用**: 演算法或行為的變化

```python
# 場景: 訂單折扣策略
class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, order): pass

class NoDiscount(DiscountStrategy):
    def calculate(self, order):
        return 0

class PercentageDiscount(DiscountStrategy):
    def __init__(self, percentage):
        self.percentage = percentage

    def calculate(self, order):
        return order.total * self.percentage

class BulkDiscount(DiscountStrategy):
    def calculate(self, order):
        if order.quantity > 100:
            return order.total * 0.1
        return 0

# 新增季節性折扣:不需要修改任何現有代碼
class SeasonalDiscount(DiscountStrategy):
    def __init__(self, season_multiplier):
        self.multiplier = season_multiplier

    def calculate(self, order):
        base_discount = order.total * 0.05
        return base_discount * self.multiplier

class Order:
    def __init__(self, total, quantity, discount_strategy):
        self.total = total
        self.quantity = quantity
        self.discount_strategy = discount_strategy

    def get_final_price(self):
        discount = self.discount_strategy.calculate(self)
        return self.total - discount
```

### 策略 2: 模板方法模式 (Template Method)

**適用**: 流程固定,步驟實現變化

```python
# 場景: 資料導入流程
class DataImporter(ABC):
    def import_data(self, source):
        # 模板方法:定義流程
        raw_data = self.extract(source)
        validated_data = self.validate(raw_data)
        transformed_data = self.transform(validated_data)
        self.load(transformed_data)

    @abstractmethod
    def extract(self, source):
        pass

    @abstractmethod
    def validate(self, data):
        pass

    @abstractmethod
    def transform(self, data):
        pass

    @abstractmethod
    def load(self, data):
        pass

# CSV 導入
class CSVImporter(DataImporter):
    def extract(self, source):
        return csv.reader(source)

    def validate(self, data):
        return [row for row in data if len(row) > 0]

    def transform(self, data):
        return [{'name': row[0], 'value': row[1]} for row in data]

    def load(self, data):
        db.bulk_insert(data)

# 新增 JSON 導入:流程不變,只改實現
class JSONImporter(DataImporter):
    def extract(self, source):
        return json.load(source)

    def validate(self, data):
        return [item for item in data if 'name' in item]

    def transform(self, data):
        return [{'name': item['name'].upper(), 'value': item.get('value', 0)} for item in data]

    def load(self, data):
        db.bulk_insert(data)
```

### 策略 3: 裝飾器模式 (Decorator Pattern)

**適用**: 動態添加功能

```python
# 場景: 通知系統
class Notifier(ABC):
    @abstractmethod
    def send(self, message): pass

class EmailNotifier(Notifier):
    def send(self, message):
        print(f"Sending email: {message}")

# 裝飾器:添加額外功能
class NotifierDecorator(Notifier):
    def __init__(self, notifier: Notifier):
        self.notifier = notifier

    def send(self, message):
        self.notifier.send(message)

class SMSDecorator(NotifierDecorator):
    def send(self, message):
        super().send(message)
        print(f"Sending SMS: {message}")

class SlackDecorator(NotifierDecorator):
    def send(self, message):
        super().send(message)
        print(f"Sending Slack: {message}")

# 使用:動態組合功能,不修改原始類別
notifier = EmailNotifier()
notifier = SMSDecorator(notifier)
notifier = SlackDecorator(notifier)
notifier.send("Hello!")
# 輸出:
# Sending email: Hello!
# Sending SMS: Hello!
# Sending Slack: Hello!
```

### 策略 4: 插件架構 (Plugin Architecture)

**適用**: 需要第三方擴展

```python
# 場景: 可擴展的文本編輯器
class Plugin(ABC):
    @abstractmethod
    def execute(self, context): pass

class Editor:
    def __init__(self):
        self.plugins = []

    def register_plugin(self, plugin: Plugin):
        self.plugins.append(plugin)

    def process(self, text):
        context = {'text': text, 'metadata': {}}

        for plugin in self.plugins:
            plugin.execute(context)

        return context['text']

# 核心系統不需要知道插件的具體實現
class SpellCheckPlugin(Plugin):
    def execute(self, context):
        # 拼字檢查邏輯
        context['metadata']['spell_checked'] = True

class SyntaxHighlightPlugin(Plugin):
    def execute(self, context):
        # 語法高亮邏輯
        context['text'] = f"<highlighted>{context['text']}</highlighted>"

# 第三方可以添加新插件,不修改 Editor
class CustomFormatterPlugin(Plugin):
    def execute(self, context):
        context['text'] = context['text'].upper()

# 使用
editor = Editor()
editor.register_plugin(SpellCheckPlugin())
editor.register_plugin(SyntaxHighlightPlugin())
editor.register_plugin(CustomFormatterPlugin())  # 擴展
```

---

## 常見陷阱

### 陷阱 1: 為了 OCP 而 OCP

```python
# ❌ 過度設計
class StringFormatter(ABC):
    @abstractmethod
    def format(self, s): pass

class UpperCaseFormatter(StringFormatter):
    def format(self, s):
        return s.upper()

class LowerCaseFormatter(StringFormatter):
    def format(self, s):
        return s.lower()

# 這種簡單的操作不需要 OCP!
# ✅ 簡單直接
class StringUtils:
    @staticmethod
    def to_upper(s):
        return s.upper()

    @staticmethod
    def to_lower(s):
        return s.lower()
```

**判斷標準**:
- 這個功能真的會頻繁變化嗎?
- 變化的模式是否可預測?
- 抽象的成本是否值得?

### 陷阱 2: 抽象洩漏 (Leaky Abstraction)

```python
# ❌ 抽象洩漏:抽象暴露了實現細節
class Database(ABC):
    @abstractmethod
    def execute_sql(self, sql): pass  # SQL 是關聯式資料庫特有的!

class MySQLDatabase(Database):
    def execute_sql(self, sql):
        # MySQL 實現
        pass

class MongoDatabase(Database):  # MongoDB 不用 SQL!
    def execute_sql(self, sql):
        # 尷尬:需要將 SQL 轉換成 MongoDB 查詢
        pass

# ✅ 更好的抽象:隱藏實現細節
class Database(ABC):
    @abstractmethod
    def find(self, criteria): pass

    @abstractmethod
    def save(self, entity): pass

    @abstractmethod
    def delete(self, entity_id): pass

class MySQLDatabase(Database):
    def find(self, criteria):
        sql = self._build_sql_query(criteria)
        return self._execute(sql)

class MongoDatabase(Database):
    def find(self, criteria):
        return self.collection.find(criteria)
```

### 陷阱 3: 預測錯誤的變化點

```python
# ❌ 為不會變化的地方建立抽象
class MathOperations(ABC):
    @abstractmethod
    def add(self, a, b): pass

    @abstractmethod
    def subtract(self, a, b): pass

# 數學運算不太可能變化!除非你在做特殊領域
# 這是浪費時間的抽象

# ✅ 簡單實現
class Calculator:
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b
```

---

## 實戰模式

### 模式 1: 配置驅動的可擴展性

```python
# 使用配置而非硬編碼來實現 OCP
class ValidationRule(ABC):
    @abstractmethod
    def validate(self, value): pass

class MinLengthRule(ValidationRule):
    def __init__(self, min_length):
        self.min_length = min_length

    def validate(self, value):
        return len(value) >= self.min_length

class RegexRule(ValidationRule):
    def __init__(self, pattern):
        self.pattern = re.compile(pattern)

    def validate(self, value):
        return bool(self.pattern.match(value))

class Validator:
    def __init__(self):
        self.rules = []

    def add_rule(self, rule: ValidationRule):
        self.rules.append(rule)

    def validate(self, value):
        return all(rule.validate(value) for rule in self.rules)

# 通過配置擴展,不修改代碼
validation_config = {
    'username': [
        MinLengthRule(3),
        RegexRule(r'^[a-zA-Z0-9_]+$')
    ],
    'email': [
        RegexRule(r'^[\w\.-]+@[\w\.-]+\.\w+$')
    ]
}

def create_validator(field_name):
    validator = Validator()
    for rule in validation_config[field_name]:
        validator.add_rule(rule)
    return validator
```

### 模式 2: 表驅動方法 (Table-Driven Method)

```python
# 使用資料結構代替條件邏輯
# ❌ 硬編碼的條件
class ShippingCalculator:
    def calculate(self, weight, destination):
        if destination == "domestic":
            if weight < 1:
                return 5
            elif weight < 5:
                return 10
            else:
                return 20
        elif destination == "international":
            if weight < 1:
                return 15
            elif weight < 5:
                return 30
            else:
                return 60

# ✅ 表驅動:新增規則不需要修改代碼
SHIPPING_RATES = {
    'domestic': [
        (1, 5),
        (5, 10),
        (float('inf'), 20)
    ],
    'international': [
        (1, 15),
        (5, 30),
        (float('inf'), 60)
    ]
}

class ShippingCalculator:
    def __init__(self, rates=SHIPPING_RATES):
        self.rates = rates

    def calculate(self, weight, destination):
        for max_weight, rate in self.rates[destination]:
            if weight < max_weight:
                return rate
        raise ValueError("Invalid weight or destination")

# 新增規則:只需修改配置
SHIPPING_RATES['express'] = [
    (1, 25),
    (5, 50),
    (float('inf'), 100)
]
```

### 模式 3: 事件驅動架構

```python
# 通過事件系統實現鬆耦合的擴展
class Event:
    def __init__(self, name, data):
        self.name = name
        self.data = data

class EventBus:
    def __init__(self):
        self.handlers = {}

    def subscribe(self, event_name, handler):
        if event_name not in self.handlers:
            self.handlers[event_name] = []
        self.handlers[event_name].append(handler)

    def publish(self, event: Event):
        if event.name in self.handlers:
            for handler in self.handlers[event.name]:
                handler(event.data)

# 核心業務邏輯
class OrderService:
    def __init__(self, event_bus: EventBus):
        self.event_bus = event_bus

    def place_order(self, order):
        # 處理訂單
        print(f"Order placed: {order}")

        # 發布事件
        self.event_bus.publish(Event('order_placed', order))

# 擴展:添加新的事件處理器,不修改 OrderService
class EmailNotificationHandler:
    def handle(self, order):
        print(f"Sending email for order: {order}")

class InventoryUpdateHandler:
    def handle(self, order):
        print(f"Updating inventory for order: {order}")

class LoyaltyPointsHandler:
    def handle(self, order):
        print(f"Adding loyalty points for order: {order}")

# 配置:組裝系統
event_bus = EventBus()
event_bus.subscribe('order_placed', EmailNotificationHandler().handle)
event_bus.subscribe('order_placed', InventoryUpdateHandler().handle)
event_bus.subscribe('order_placed', LoyaltyPointsHandler().handle)  # 新增

order_service = OrderService(event_bus)
order_service.place_order({'id': 123, 'total': 100})
```

---

## 無招勝有招 - 預見變化

### 心法 1: 識別變化模式

**從**: "這段代碼以後可能會變"
**到**: "這種變化會以什麼模式發生"

變化模式分類:
1. **列舉式變化**: 添加新的選項 → 策略模式
2. **流程變化**: 步驟改變但結構固定 → 模板方法
3. **組合式變化**: 功能可以疊加 → 裝飾器模式
4. **未知擴展**: 第三方擴展 → 插件架構

### 心法 2: 痛苦驅動設計

**不要預測變化,讓痛苦告訴你**:

```python
# 第一次實現:簡單直接
def calculate_shipping(weight, destination):
    if destination == "domestic":
        return weight * 5
    elif destination == "international":
        return weight * 15

# 第二次修改:添加快遞
def calculate_shipping(weight, destination, speed):
    if destination == "domestic":
        if speed == "express":
            return weight * 10
        else:
            return weight * 5
    # ... 開始感到痛苦

# 第三次修改:添加保險
# 這時候你會感到「這樣下去不行」
# 現在是重構的時機!

class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, weight): pass

class DomesticStandard(ShippingStrategy):
    def calculate(self, weight):
        return weight * 5
# ...
```

### 心法 3: 抽象的層次

不同層次需要不同程度的抽象:

```
應用層: 高抽象 (很容易變化)
    ↓ 依賴
領域層: 中抽象 (業務規則穩定,實現可能變)
    ↓ 依賴
基礎設施層: 低抽象 (技術細節經常變化)
```

```python
# 應用層:高度抽象
class OrderUseCase:
    def __init__(self, repository, payment_gateway):
        self.repository = repository
        self.payment_gateway = payment_gateway

    def execute(self, order_data):
        order = Order.create(order_data)
        self.payment_gateway.charge(order.total)
        self.repository.save(order)

# 領域層:業務邏輯,中等抽象
class Order:
    def __init__(self, items, customer):
        self.items = items
        self.customer = customer

    def calculate_total(self):
        return sum(item.price for item in self.items)

# 基礎設施層:技術實現,必須抽象
class PaymentGateway(ABC):  # 必須是抽象,因為會頻繁切換
    @abstractmethod
    def charge(self, amount): pass
```

### 心法 4: YAGNI vs 可擴展性的平衡

**YAGNI** (You Aren't Gonna Need It): 不要實現你不需要的功能

平衡之道:
```python
# ❌ 過度設計
class Logger(ABC):
    @abstractmethod
    def log(self, level, message, context, tags, metadata): pass

class FileLogger(Logger): ...
class DatabaseLogger(Logger): ...
class CloudLogger(Logger): ...
# 但你現在只需要簡單的 print!

# ❌ 完全不考慮擴展
class OrderService:
    def process(self, order):
        print(f"Saving to MySQL: {order}")  # 硬編碼
        print(f"Sending email to: {order.email}")  # 硬編碼

# ✅ 平衡:在已知變化點抽象,其他保持簡單
class OrderService:
    def __init__(self, repository):  # 資料庫可能變:抽象
        self.repository = repository

    def process(self, order):
        self.repository.save(order)
        self._send_email(order)  # Email 暫時不會變:直接實現

    def _send_email(self, order):
        print(f"Sending email to: {order.email}")
```

判斷標準:
- 有明確變化歷史 → 抽象
- 業務明確說會變 → 抽象
- 技術選型不確定 → 抽象
- 其他情況 → 先簡單實現

### 心法 5: 重構觸發點

何時開始應用 OCP?

**觸發信號**:
1. **第二次修改同一段代碼時** → "Rule of Three"
2. **條件語句超過 3 個分支** → 考慮策略模式
3. **複製貼上代碼後修改** → 考慮模板方法
4. **註釋說"以後可能需要..."** → 可能該抽象了(但小心過度設計)

```python
# 觸發點示例
# 第一次
def process_payment(amount, method):
    if method == "credit_card":
        # 處理信用卡
        pass

# 第二次:添加 PayPal
def process_payment(amount, method):
    if method == "credit_card":
        # 處理信用卡
        pass
    elif method == "paypal":
        # 處理 PayPal
        pass

# 第三次:是時候重構了!
# 不要等到有 10 個 elif
```

### 終極心法: 遞延決策

**最好的抽象是在你理解問題之後做出的**

1. 第一次:直接實現,先讓它工作
2. 第二次:注意到模式,但還不重構
3. 第三次:模式清晰,開始抽象
4. 持續演進:隨著理解加深,改進抽象

這就是「無招勝有招」:
- 不是一開始就建立完美抽象
- 而是讓系統引導你找到正確抽象
- 當你理解了變化的本質,正確的抽象自然浮現

---

## 實戰練習

### 練習 1: 識別變化軸

分析以下代碼的變化點:

```python
class ReportGenerator:
    def generate(self, data, format, destination):
        if format == "pdf":
            content = self._generate_pdf(data)
        elif format == "excel":
            content = self._generate_excel(data)

        if destination == "email":
            self._send_email(content)
        elif destination == "file":
            self._save_file(content)
```

<details>
<summary>答案</summary>

兩個獨立的變化軸:
1. **格式變化**: PDF, Excel (可能添加 CSV, HTML)
2. **目標變化**: Email, File (可能添加 Cloud, API)

重構策略:
```python
# 分離兩個軸
class ReportFormatter(ABC):
    @abstractmethod
    def format(self, data): pass

class ReportDestination(ABC):
    @abstractmethod
    def deliver(self, content): pass

class ReportGenerator:
    def generate(self, data, formatter: ReportFormatter, destination: ReportDestination):
        content = formatter.format(data)
        destination.deliver(content)
```
</details>

### 練習 2: 重構違反 OCP 的代碼

```python
class DiscountCalculator:
    def calculate(self, customer_type, amount):
        if customer_type == "regular":
            return amount * 0.05
        elif customer_type == "premium":
            return amount * 0.10
        elif customer_type == "vip":
            if amount > 1000:
                return amount * 0.20
            else:
                return amount * 0.15
```

<details>
<summary>參考答案</summary>

```python
class DiscountStrategy(ABC):
    @abstractmethod
    def calculate(self, amount): pass

class RegularDiscount(DiscountStrategy):
    def calculate(self, amount):
        return amount * 0.05

class PremiumDiscount(DiscountStrategy):
    def calculate(self, amount):
        return amount * 0.10

class VIPDiscount(DiscountStrategy):
    def calculate(self, amount):
        if amount > 1000:
            return amount * 0.20
        else:
            return amount * 0.15

class Customer:
    def __init__(self, name, discount_strategy: DiscountStrategy):
        self.name = name
        self.discount_strategy = discount_strategy

    def get_discount(self, amount):
        return self.discount_strategy.calculate(amount)

# 添加新的客戶類型:只需添加新策略
class CorporateDiscount(DiscountStrategy):
    def calculate(self, amount):
        return amount * 0.25
```
</details>

---

## 總結金句

1. **對擴展開放,對修改封閉** - 新增代碼而非修改舊代碼
2. **不要過早抽象** - Rule of Three: 重複三次再抽象
3. **識別變化軸** - 不同的變化方向需要不同的抽象
4. **穩定依賴抽象** - 讓不穩定的依賴於穩定的
5. **痛苦驅動設計** - 讓真實的變更需求引導你的抽象
6. **遞延決策** - 在理解問題後再建立抽象

---

## OCP 與其他原則的關係

- **與 SRP**: SRP 是 OCP 的基礎 - 單一職責的類別更容易擴展
- **與 LSP**: LSP 確保擴展不會破壞既有行為
- **與 DIP**: DIP 提供了實現 OCP 的手段 - 通過依賴抽象
- **與 ISP**: ISP 幫助 OCP - 小接口更容易擴展

---

## 延伸閱讀

- Design Patterns (Gang of Four) - Strategy, Template Method, Decorator
- Clean Architecture (Robert C. Martin) - Plugin Architecture
- Refactoring to Patterns (Joshua Kerievsky)

---

**記住**: OCP 不是要求你預見所有可能的變化,而是當變化發生時,你能夠通過擴展而非修改來應對。最好的抽象是在你經歷過變化之後建立的,這就是「無招勝有招」 - 讓經驗引導設計,而非教條。
