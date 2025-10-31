# 里氏替換原則 (LSP) - 深度實戰指南

> "子類型必須能夠替換其基類型" - Barbara Liskov

## 目錄
- [核心理念](#核心理念)
- [設計經驗](#設計經驗)
- [違反 LSP 的信號](#違反-lsp-的信號)
- [繼承的正確使用](#繼承的正確使用)
- [重構技巧](#重構技巧)
- [無招勝有招 - is-a 的真諦](#無招勝有招---is-a-的真諦)

---

## 核心理念

### 什麼是「可替換性」?

LSP 的核心:客戶端使用基類的地方,換成子類也應該正常工作,**且不需要知道這是子類**。

```python
# ✅ 符合 LSP
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class Sparrow(Bird):
    def move(self):
        return "Flying"

class Ostrich(Bird):
    def move(self):
        return "Running"

def make_bird_move(bird: Bird):
    print(bird.move())  # 不需要知道是什麼鳥

# 可以替換使用
make_bird_move(Sparrow())  # Flying
make_bird_move(Ostrich())  # Running

# ❌ 違反 LSP
class Bird:
    def fly(self):
        return "Flying"

class Sparrow(Bird):
    pass  # OK

class Ostrich(Bird):
    def fly(self):
        raise Exception("Ostriches can't fly!")  # 破壞了預期!

def make_bird_fly(bird: Bird):
    print(bird.fly())  # 期望所有鳥都能飛

make_bird_fly(Sparrow())  # OK
make_bird_fly(Ostrich())  # 💥 異常!違反了 LSP
```

### LSP 的契約 (Contract)

子類必須遵守基類的契約:

1. **前置條件不能加強**: 子類不能要求更嚴格的輸入
2. **後置條件不能減弱**: 子類必須保證至少和基類一樣的輸出
3. **不變條件必須保持**: 基類的不變性在子類中也要成立
4. **方法簽名要兼容**: 參數和返回值類型要兼容

```python
# ❌ 違反契約:加強了前置條件
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def set_width(self, width):
        self.width = width  # 接受任何值

class Square(Rectangle):
    def set_width(self, width):
        if width != self.height:  # ❌ 加強了前置條件!
            raise ValueError("Square width must equal height")
        self.width = width

# ✅ 正確:不加強前置條件
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side * self.side
```

---

## 設計經驗

### 經驗 1: 行為的一致性比類型繼承更重要

**錯誤思維**: "正方形是矩形,所以 Square 應該繼承 Rectangle"
**正確思維**: "正方形的行為與矩形不同,不應該繼承"

```python
# ❌ 經典的正方形-矩形問題
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def set_width(self, width):
        self.width = width

    def set_height(self, height):
        self.height = height

    def area(self):
        return self.width * self.height

class Square(Rectangle):
    def set_width(self, width):
        self.width = width
        self.height = width  # 違反了 LSP!

    def set_height(self, height):
        self.width = height
        self.height = height

# 問題:
def test_rectangle(rect: Rectangle):
    rect.set_width(5)
    rect.set_height(4)
    assert rect.area() == 20  # 對 Rectangle 成立

test_rectangle(Rectangle(0, 0))  # ✓ 通過
test_rectangle(Square(0))        # ✗ 失敗! area() = 16

# ✅ 正確設計:基於行為
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self._width = width
        self._height = height

    def resize(self, width, height):
        self._width = width
        self._height = height

    def area(self):
        return self._width * self._height

class Square(Shape):
    def __init__(self, side):
        self._side = side

    def resize(self, side):  # 不同的 API,更誠實
        self._side = side

    def area(self):
        return self._side * self._side
```

### 經驗 2: 異常也是契約的一部分

```python
# ❌ 子類拋出基類沒有的異常
class FileReader:
    def read(self, filename):
        # 文檔:可能拋出 FileNotFoundError
        with open(filename) as f:
            return f.read()

class NetworkFileReader(FileReader):
    def read(self, filename):
        # ❌ 拋出了基類沒有聲明的異常!
        response = requests.get(filename)
        if response.status_code == 500:
            raise ConnectionError("Server error")  # 新異常!
        return response.text

# 客戶端代碼
def process_file(reader: FileReader, filename):
    try:
        content = reader.read(filename)
        return content
    except FileNotFoundError:
        return "File not found"
    # 沒有處理 ConnectionError!

# ✅ 正確:子類異常應該是基類異常的子類型
class FileReader:
    def read(self, filename):
        # 可能拋出 IOError 或其子類
        pass

class LocalFileReader(FileReader):
    def read(self, filename):
        try:
            with open(filename) as f:
                return f.read()
        except OSError as e:
            raise IOError(f"Failed to read: {e}")

class NetworkFileReader(FileReader):
    def read(self, filename):
        try:
            response = requests.get(filename)
            response.raise_for_status()
            return response.text
        except requests.RequestException as e:
            raise IOError(f"Failed to read: {e}")  # 同樣是 IOError
```

### 經驗 3: 退化繼承 (Refused Bequest)

子類不應該繼承不需要的方法:

```python
# ❌ 退化繼承
class Stack:
    def __init__(self):
        self.items = []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        return self.items.pop()

    def get(self, index):  # Stack 需要按索引訪問
        return self.items[index]

class FixedStack(Stack):
    """固定大小的棧"""
    def __init__(self, max_size):
        super().__init__()
        self.max_size = max_size

    def push(self, item):
        if len(self.items) >= self.max_size:
            raise OverflowError("Stack full")
        super().push(item)

    def get(self, index):
        # ❌ 不應該暴露這個方法!
        raise NotImplementedError("FixedStack doesn't support indexing")

# ✅ 正確:只繼承需要的
class Stack(ABC):
    @abstractmethod
    def push(self, item):
        pass

    @abstractmethod
    def pop(self):
        pass

class IndexableStack(Stack):
    """可索引的棧"""
    def __init__(self):
        self.items = []

    def push(self, item):
        self.items.append(item)

    def pop(self):
        return self.items.pop()

    def get(self, index):
        return self.items[index]

class FixedStack(Stack):
    """固定大小的棧 - 不繼承 get"""
    def __init__(self, max_size):
        self.items = []
        self.max_size = max_size

    def push(self, item):
        if len(self.items) >= self.max_size:
            raise OverflowError("Stack full")
        self.items.append(item)

    def pop(self):
        return self.items.pop()
```

---

## 違反 LSP 的信號

### 信號 1: 類型檢查

如果你需要檢查子類的具體類型,就違反了 LSP:

```python
# ❌ 類型檢查 = 違反 LSP
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class FlyingBird(Bird):
    def move(self):
        return "Flying"

class Penguin(Bird):
    def move(self):
        return "Swimming"

def bird_show(bird: Bird):
    if isinstance(bird, Penguin):  # ❌ 需要檢查類型!
        print("This bird swims")
    else:
        print("This bird flies")

# ✅ 正確:多態處理
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

    @abstractmethod
    def get_movement_type(self):
        pass

class FlyingBird(Bird):
    def move(self):
        return "Flying"

    def get_movement_type(self):
        return "air"

class Penguin(Bird):
    def move(self):
        return "Swimming"

    def get_movement_type(self):
        return "water"

def bird_show(bird: Bird):
    print(f"This bird moves in {bird.get_movement_type()}")
```

### 信號 2: 空實現或拋出異常

```python
# ❌ 違反 LSP 的信號
class Vehicle(ABC):
    @abstractmethod
    def start_engine(self):
        pass

class Car(Vehicle):
    def start_engine(self):
        print("Engine started")

class Bicycle(Vehicle):
    def start_engine(self):
        raise NotImplementedError("Bicycle has no engine")  # ❌

# ✅ 重新設計抽象
class Vehicle(ABC):
    @abstractmethod
    def start_moving(self):
        pass

class MotorizedVehicle(Vehicle):
    def start_moving(self):
        self.start_engine()
        print("Engine started, moving")

    @abstractmethod
    def start_engine(self):
        pass

class ManualVehicle(Vehicle):
    def start_moving(self):
        print("Pedaling, moving")

class Car(MotorizedVehicle):
    def start_engine(self):
        print("Engine started")

class Bicycle(ManualVehicle):
    pass  # 不需要實現 start_engine
```

### 信號 3: 子類加強了前置條件

```python
# ❌ 加強前置條件
class UserService:
    def create_user(self, username, email):
        # 只要求 username 不為空
        if not username:
            raise ValueError("Username required")
        return User(username, email)

class AdminUserService(UserService):
    def create_user(self, username, email):
        # ❌ 加強了條件!
        if not username or len(username) < 8:
            raise ValueError("Admin username must be at least 8 chars")
        return AdminUser(username, email)

# 客戶端代碼
def register_user(service: UserService, username, email):
    return service.create_user(username, email)

register_user(UserService(), "Bob", "bob@example.com")  # ✓ OK
register_user(AdminUserService(), "Bob", "bob@example.com")  # ✗ 失敗!

# ✅ 正確:不加強前置條件
class UserService:
    def create_user(self, username, email, min_length=1):
        if not username or len(username) < min_length:
            raise ValueError(f"Username must be at least {min_length} chars")
        return User(username, email)

class AdminUserService(UserService):
    def create_user(self, username, email, min_length=8):
        # 通過默認參數而非強制要求
        return super().create_user(username, email, min_length)
```

### 信號 4: 子類減弱了後置條件

```python
# ❌ 減弱後置條件
class DataProvider:
    def get_data(self):
        # 保證:總是返回非空列表
        return [1, 2, 3]

class CachedDataProvider(DataProvider):
    def get_data(self):
        # ❌ 可能返回 None!
        if self.cache_empty():
            return None
        return self.cache

# 客戶端代碼
def process_data(provider: DataProvider):
    data = provider.get_data()
    return len(data)  # 期望總是有數據

process_data(DataProvider())  # ✓ OK: 3
process_data(CachedDataProvider())  # ✗ TypeError: object of type 'NoneType' has no len()

# ✅ 正確:保持後置條件
class CachedDataProvider(DataProvider):
    def get_data(self):
        if self.cache_empty():
            return []  # 返回空列表而非 None
        return self.cache
```

---

## 繼承的正確使用

### 原則 1: 優先組合而非繼承

```python
# ❌ 濫用繼承
class ArrayList(list):
    def add_all(self, items):
        for item in items:
            self.append(item)

# 問題:繼承了 list 的所有方法(insert, remove, clear...)
# 可能不是你想要的

# ✅ 使用組合
class ArrayList:
    def __init__(self):
        self._items = []  # 組合

    def add(self, item):
        self._items.append(item)

    def add_all(self, items):
        for item in items:
            self.add(item)

    def get(self, index):
        return self._items[index]

    # 只暴露需要的方法
```

### 原則 2: 繼承表達 is-a,組合表達 has-a

```python
# ✓ is-a: 員工是人
class Person:
    def __init__(self, name):
        self.name = name

class Employee(Person):  # Employee IS-A Person
    def __init__(self, name, employee_id):
        super().__init__(name)
        self.employee_id = employee_id

# ✓ has-a: 公司有員工
class Company:
    def __init__(self):
        self.employees = []  # Company HAS employees

    def hire(self, employee: Employee):
        self.employees.append(employee)

# ❌ 錯誤: 公司不是員工列表
class Company(list):
    pass
```

### 原則 3: 抽象類定義契約,具體類實現細節

```python
# ✅ 好的抽象層次
class PaymentGateway(ABC):
    """定義所有支付網關的契約"""

    @abstractmethod
    def charge(self, amount: Decimal) -> PaymentResult:
        """
        收費
        前置條件: amount > 0
        後置條件: 返回包含 success 和 transaction_id 的結果
        異常: 可能拋出 PaymentError
        """
        pass

    @abstractmethod
    def refund(self, transaction_id: str) -> PaymentResult:
        """
        退款
        前置條件: transaction_id 必須是有效的交易 ID
        後置條件: 返回包含 success 的結果
        異常: 可能拋出 PaymentError 或 TransactionNotFoundError
        """
        pass

class StripeGateway(PaymentGateway):
    def charge(self, amount: Decimal) -> PaymentResult:
        if amount <= 0:
            raise PaymentError("Amount must be positive")

        try:
            response = stripe.Charge.create(amount=amount)
            return PaymentResult(success=True, transaction_id=response.id)
        except stripe.error.StripeError as e:
            raise PaymentError(str(e))

    def refund(self, transaction_id: str) -> PaymentResult:
        try:
            stripe.Refund.create(charge=transaction_id)
            return PaymentResult(success=True)
        except stripe.error.InvalidRequestError:
            raise TransactionNotFoundError(transaction_id)
        except stripe.error.StripeError as e:
            raise PaymentError(str(e))
```

---

## 重構技巧

### 技巧 1: Extract Subclass → Extract Interface

```python
# Before: 用繼承表達變化
class Document:
    def __init__(self, content):
        self.content = content

    def save(self):
        raise NotImplementedError

class PDFDocument(Document):
    def save(self):
        # 保存為 PDF
        pass

class WordDocument(Document):
    def save(self):
        # 保存為 Word
        pass

# After: 用接口表達能力
class Document:
    def __init__(self, content, saver):
        self.content = content
        self.saver = saver

    def save(self):
        self.saver.save(self.content)

class DocumentSaver(ABC):
    @abstractmethod
    def save(self, content):
        pass

class PDFSaver(DocumentSaver):
    def save(self, content):
        # 保存為 PDF
        pass

class WordSaver(DocumentSaver):
    def save(self, content):
        # 保存為 Word
        pass

# 使用
doc = Document("content", PDFSaver())
doc.save()
```

### 技巧 2: Replace Conditional with Polymorphism

```python
# Before: 條件邏輯
class Bird:
    def __init__(self, bird_type):
        self.type = bird_type

    def fly(self):
        if self.type == "sparrow":
            return "Flying fast"
        elif self.type == "ostrich":
            return "Can't fly, running instead"
        elif self.type == "penguin":
            return "Can't fly, swimming instead"

# After: 多態
class Bird(ABC):
    @abstractmethod
    def move(self):
        pass

class Sparrow(Bird):
    def move(self):
        return "Flying fast"

class Ostrich(Bird):
    def move(self):
        return "Running"

class Penguin(Bird):
    def move(self):
        return "Swimming"
```

### 技巧 3: Pull Up / Push Down Method

```python
# Before: 重複代碼
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

class Square:
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side * self.side

    def perimeter(self):
        return 4 * self.side

# After: Pull Up 共同行為
class Shape(ABC):
    @abstractmethod
    def area(self):
        pass

    @abstractmethod
    def perimeter(self):
        pass

    def describe(self):  # 共同方法上提
        return f"Area: {self.area()}, Perimeter: {self.perimeter()}"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

    def perimeter(self):
        return 2 * (self.width + self.height)

class Square(Shape):
    def __init__(self, side):
        self.side = side

    def area(self):
        return self.side * self.side

    def perimeter(self):
        return 4 * self.side
```

---

## 無招勝有招 - is-a 的真諦

### 心法 1: 行為替換而非類型繼承

**從**: "正方形是矩形,所以繼承"
**到**: "正方形的行為與矩形不同,不應繼承"

真正的 is-a 關係:
```python
# 問自己: "我能在期望 A 的地方使用 B 而不出問題嗎?"

# ✓ Employee IS-A Person
# 所有期望 Person 的地方都可以用 Employee
def greet(person: Person):
    print(f"Hello, {person.name}")

greet(Employee("Alice"))  # 沒問題

# ✗ Square IS-NOT-A Rectangle (行為上)
def test_rectangle(rect: Rectangle):
    rect.set_width(5)
    rect.set_height(4)
    assert rect.area() == 20

test_rectangle(Square(3))  # 💥 失敗!
```

### 心法 2: 契約思維

不要只看方法簽名,要看契約:

```python
class Stack(ABC):
    """
    契約:
    - push 後 pop 會返回同一個元素
    - 空棧 pop 會拋出異常
    - 元素按 LIFO 順序
    """
    @abstractmethod
    def push(self, item):
        pass

    @abstractmethod
    def pop(self):
        pass

# ✓ 符合契約
class ArrayStack(Stack):
    def push(self, item):
        self.items.append(item)

    def pop(self):
        if not self.items:
            raise IndexError("Empty stack")
        return self.items.pop()

# ✗ 違反契約
class RandomStack(Stack):
    def push(self, item):
        self.items.append(item)

    def pop(self):
        if not self.items:
            raise IndexError("Empty stack")
        import random
        return self.items.pop(random.randint(0, len(self.items)-1))
        # ❌ 違反了 LIFO 契約!
```

### 心法 3: 最少驚訝原則

**子類的行為不應該讓使用基類的人感到驚訝**

```python
# ❌ 驚訝的行為
class FileReader:
    def read(self, path):
        with open(path) as f:
            return f.read()

class CachedFileReader(FileReader):
    def read(self, path):
        # 驚訝:第二次讀取返回快取,不是最新內容!
        if path in self.cache:
            return self.cache[path]
        content = super().read(path)
        self.cache[path] = content
        return content

# 使用者期望每次讀取都是最新內容
reader = FileReader()
content1 = reader.read("file.txt")
# ... 文件被其他程序修改 ...
content2 = reader.read("file.txt")  # 期望是新內容
# 如果是 CachedFileReader,content2 == content1!驚訝!

# ✅ 不驚訝的設計
class FileReader:
    def read(self, path):
        with open(path) as f:
            return f.read()

class CachedFileReader(FileReader):
    def read(self, path, use_cache=True):
        # 明確告知快取行為
        if use_cache and path in self.cache:
            return self.cache[path]
        content = super().read(path)
        if use_cache:
            self.cache[path] = content
        return content
```

### 心法 4: 組合優於繼承

**當你不確定是否該繼承時,用組合**

```python
# 猶豫: "User 應該繼承 Authenticator 嗎?"
# ❌ 不! User 不是 Authenticator
class Authenticator:
    def authenticate(self, password):
        pass

class User(Authenticator):  # ❌ User IS-A Authenticator? 奇怪
    pass

# ✅ User 有一個 Authenticator
class User:
    def __init__(self, authenticator: Authenticator):
        self.authenticator = authenticator

    def login(self, password):
        return self.authenticator.authenticate(password)
```

### 心法 5: 忘記 LSP

當你完全理解 LSP,你會發現:
- 不再需要檢查類型
- 不再需要拋出 NotImplementedError
- 不再需要空實現
- 繼承關係自然合理

這時候,你已經**內化了替換思維**:
- 設計基類時,自然考慮所有子類的共同行為
- 設計子類時,自然遵守基類的契約
- 選擇繼承或組合時,自然做出正確選擇

**這就是「無招」的境界** - LSP 不再是規則,而是思維方式。

---

## 實戰練習

### 練習 1: 識別違反 LSP

```python
class Account:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        self.balance -= amount

class SavingsAccount(Account):
    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        super().withdraw(amount)

# 這違反 LSP 嗎?為什麼?
```

<details>
<summary>答案</summary>

違反! SavingsAccount 加強了前置條件。

- 基類 Account.withdraw 接受任何 amount (允許透支)
- 子類 SavingsAccount.withdraw 要求 amount <= balance

修正:
```python
class Account(ABC):
    def __init__(self, balance):
        self.balance = balance

    @abstractmethod
    def withdraw(self, amount):
        pass

class CheckingAccount(Account):
    """允許透支的帳戶"""
    def withdraw(self, amount):
        self.balance -= amount

class SavingsAccount(Account):
    """不允許透支的帳戶"""
    def withdraw(self, amount):
        if amount > self.balance:
            raise ValueError("Insufficient funds")
        self.balance -= amount
```
</details>

### 練習 2: 重構違反 LSP 的代碼

```python
class Vehicle:
    def start(self):
        print("Starting vehicle")

    def add_fuel(self, amount):
        self.fuel += amount

class ElectricCar(Vehicle):
    def add_fuel(self, amount):
        raise NotImplementedError("Electric car doesn't use fuel")
```

<details>
<summary>參考答案</summary>

```python
class Vehicle(ABC):
    @abstractmethod
    def start(self):
        pass

    @abstractmethod
    def refuel(self, amount):
        pass

class FuelVehicle(Vehicle):
    def __init__(self):
        self.fuel = 0

    def start(self):
        print("Starting engine")

    def refuel(self, amount):
        self.fuel += amount

class ElectricVehicle(Vehicle):
    def __init__(self):
        self.battery = 0

    def start(self):
        print("Starting motor")

    def refuel(self, amount):
        self.battery += amount  # "refuel" 這裡表示補充能源
```
</details>

---

## 總結金句

1. **子類必須能替換基類** - 不只是類型,更是行為
2. **契約比簽名重要** - 前置條件、後置條件、不變條件
3. **組合優於繼承** - 不確定時選組合
4. **行為一致性** - is-a 是行為關係,不只是概念關係
5. **最少驚訝** - 子類不應該有令人驚訝的行為
6. **忘記類型檢查** - 如果需要 isinstance,設計可能有問題

---

## LSP 與其他原則的關係

- **與 OCP**: LSP 確保擴展(新子類)不會破壞現有行為
- **與 SRP**: 單一職責的類別更容易正確繼承
- **與 ISP**: 小接口減少子類需要實現不相關方法的問題
- **與 DIP**: 依賴抽象時,LSP 確保所有實現都可替換

---

## 延伸閱讀

- "A Behavioral Notion of Subtyping" (Barbara Liskov, Jeannette Wing)
- Effective Java (Joshua Bloch) - Item 18: Favor composition over inheritance
- Clean Code (Robert C. Martin) - Chapter 6: Objects and Data Structures

---

**記住**: LSP 的本質是**替換性思維** - 設計類別時,永遠要想「如果有人用子類替換基類,會發生什麼?」當你不再需要想這個問題,而是自然設計出可替換的類別時,你就達到了「無招勝有招」的境界。
