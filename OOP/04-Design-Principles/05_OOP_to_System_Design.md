# 從OOP到System Design

> 橋接微觀程式設計與宏觀系統架構的關鍵一章

## 📋 章節導覽

**預計學習時間**: 4-6 小時 (建議分3次完成)
**難度等級**: ⭐⭐⭐⭐ (中高級)
**前置知識**: OOP四大支柱 + SOLID原則

---

## 🎯 本章學習目標

完成本章後,你將能夠:

- [ ] 理解OOP原則如何擴展到系統設計 (60分鐘)
- [ ] 掌握從類別圖到架構圖的思維轉換 (90分鐘)
- [ ] 應用SOLID原則進行系統架構設計 (120分鐘)
- [ ] 獨立設計中等複雜度的分散式系統 (60分鐘)
- [ ] 為FANG System Design面試建立紮實基礎

**學習路徑**:
```
理論理解 → 對照表記憶 → 案例分析 → 實戰演練 → 面試應用
  (30分)      (30分)       (120分)     (90分)      (60分)
```

---

## 💡 為什麼這一章如此重要?

### 這是什麼樣的一章?

這一章是**思維躍遷**的關鍵橋樑:
- 從"寫代碼"到"畫架構圖"
- 從"類別設計"到"系統設計"
- 從"單機應用"到"分散式系統"

### 真實場景

**面試官**: "設計一個Twitter系統"
**候選人A** (沒學過本章): "呃...需要User表、Tweet表..."
**候選人B** (學過本章): "這類似於OOP中的觀察者模式,我們可以設計User Service和Timeline Service,應用Fan-out策略..."

---

## 第一部分: 建立思維模型 (30分鐘)

### 🚨 常見的認知誤區

<details>
<summary><b>誤區1: OOP和System Design是兩個完全不同的領域</b></summary>

❌ **錯誤思維**: "OOP是寫代碼的,System Design是畫架構圖的,兩者無關"
✅ **正確思維**: "OOP的原則和模式直接應用於系統架構設計"

**類比**:
- OOP ≈ 建築內部設計 (房間佈局、功能分區)
- System Design ≈ 城市規劃 (建築群佈局、交通網絡)
- **原則相同,只是尺度不同!**
</details>

<details>
<summary><b>誤區2: 學完OOP就能直接做System Design</b></summary>

❌ **錯誤思維**: "我會寫類別了,應該能設計系統吧?"
✅ **正確思維**: "需要理解如何將類別設計思維擴展到系統層次"

**需要的思維轉換**:
- 類別 → 服務
- 方法調用 → API調用
- 內存 → 分散式存儲
- 線程 → 分散式進程
</details>

---

### 🗺️ 核心對照表 (建議記憶)

> **學習建議**: 先理解左側OOP概念,再類比到右側System Design概念

| OOP概念 | System Design概念 | 具體範例 | 面試關鍵字 |
|---------|-------------------|----------|-----------|
| **Class** | **Service/Component** | `User` 類別 → `User Service` | Microservice |
| **Method** | **API Endpoint** | `user.save()` → `POST /api/users` | REST API |
| **Interface** | **API Contract** | `PaymentInterface` → Payment API Spec | API Gateway |
| **Encapsulation** | **Service Boundary** | 私有方法 → 內部API (不對外) | Service Isolation |
| **Inheritance** | **Service Extension** | `VIPUser extends User` → VIP Service | Service Hierarchy |
| **Composition** | **Service Integration** | `Order` 有 `User` → Order調用User Service | Service Mesh |
| **SOLID原則** | **System Design原則** | 單一職責 → 微服務拆分 | Design Principles |
| **設計模式** | **架構模式** | 觀察者 → Event-Driven | Architecture Pattern |

<details>
<summary>💡 <b>記憶技巧</b></summary>

**口訣**: "類變服,方變API,組合變調用,原則永不變"
- 類(Class) → 服(Service)
- 方(Method) → API
- 組合(Composition) → 調用(RPC/HTTP)
- 原則(SOLID) → 永不變(適用所有層次)
</details>

---

### ✅ 知識檢查點 1

在繼續之前,請確認你理解:
- [ ] OOP和System Design使用相同的設計原則
- [ ] 類別設計 ≈ 小規模系統設計
- [ ] 系統設計 ≈ 大規模OOP設計
- [ ] 能夠說出至少3個對照關係

<details>
<summary>查看答案</summary>

**三個核心對照**:
1. Class → Service (實體對應)
2. Method → API (行為對應)
3. Composition → Service Integration (關係對應)
</details>

---

## 第二部分: 思維層次的漸進式躍遷 (90分鐘)

> **學習策略**: 通過3個層次的演進,理解設計思維如何從代碼擴展到架構

### 📊 三層演進模型

```
Level 1: 單一類別 (Single Class)
   ↓ 添加依賴管理
Level 2: 模組/服務 (Module/Service)
   ↓ 添加網絡通信
Level 3: 分散式系統 (Distributed System)
```

---

### Level 1: 類別設計 (Class Design) ⭐

**場景**: 在單一進程中設計User功能

```python
class User:
    """簡單的User類別 - 所有邏輯在一個類別中"""
    def __init__(self, username, email):
        self.username = username
        self.email = email

    def save(self):
        """儲存到資料庫"""
        # 直接操作資料庫
        database.insert('users', self.__dict__)
```

**特點**:
- ✅ 簡單直接
- ✅ 適合小型應用
- ❌ 類別職責過多 (違反SRP)
- ❌ 難以測試 (依賴具體database)

---

### Level 2: 模組設計 (Module Design) ⭐⭐

**場景**: 分離關注點,引入依賴注入

```python
# user.py - 純數據模型
class User:
    """只負責數據表示"""
    def __init__(self, username, email):
        self.username = username
        self.email = email

# user_service.py - 業務邏輯
class UserService:
    """負責用戶業務邏輯,依賴於抽象接口"""
    def __init__(self, db: DatabaseInterface, cache: CacheInterface):
        self.db = db        # 依賴注入
        self.cache = cache  # 依賴注入

    def create_user(self, username, email):
        """創建用戶的完整流程"""
        # 1. 創建用戶對象
        user = User(username, email)

        # 2. 保存到數據庫
        self.db.save(user)

        # 3. 更新緩存
        self.cache.set(f"user:{user.id}", user)

        return user
```

**改進點**:
- ✅ 職責分離 (User vs UserService)
- ✅ 依賴抽象 (Interface)
- ✅ 易於測試 (可注入mock)
- ✅ 符合SOLID原則

**思考**: 這個設計已經很好了,為什麼還需要Level 3?

<details>
<summary>查看答案</summary>

**問題場景**:
- 當用戶量從1萬增長到1億時?
- 當需要多個服務器處理請求時?
- 當數據庫需要分片時?
- 當需要跨地區部署時?

→ 單機模組設計無法解決這些問題,需要System Design!
</details>

---

### Level 3: 系統設計 (System Design) ⭐⭐⭐

**場景**: 支持百萬級用戶的分散式系統

```
                    Internet (全球用戶)
                         │
                  ┌──────▼──────┐
                  │Load Balancer│ ← 分發請求到多台服務器
                  └──────┬──────┘
                         │
                  ┌──────▼──────┐
                  │ API Gateway │ ← 統一入口,路由,認證
                  └──────┬──────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
         ┌────▼────┐┌───▼────┐┌───▼────┐
         │ User    ││ User   ││ User   │ ← 多個UserService實例
         │ Service ││Service ││Service │   (水平擴展)
         │Instance1││Instance2││Instance3│
         └────┬────┘└───┬────┘└───┬────┘
              │          │          │
              └──────────┼──────────┘
                         │
                    ┌────┴────┐
                    │         │
               ┌────▼───┐ ┌──▼─────┐
               │Database│ │ Redis  │ ← 對應Level 2的db, cache
               │(Primary│ │Cluster │   但現在是分散式的!
               │+Replica│ └────────┘
               └────────┘
```

**對應關係** (這是關鍵!):

| Level 2 (OOP) | Level 3 (System Design) | 為什麼需要改變? |
|---------------|-------------------------|----------------|
| `UserService` 類別 | User Service (3個實例) | 處理高並發 |
| `self.db` 變量 | Database Cluster | 處理大數據量 |
| `self.cache` 變量 | Redis Cluster | 分散式緩存 |
| `create_user()` 方法 | `POST /api/users` API | 跨網絡調用 |
| 依賴注入 | Service Discovery | 動態查找服務 |

**關鍵洞察**:
```python
# Level 2: 在內存中調用
user_service.create_user("john", "john@example.com")

# Level 3: 跨網絡調用
HTTP POST https://api.example.com/users
{
  "username": "john",
  "email": "john@example.com"
}
```

---

### 🎯 三層演進總結

| 維度 | Level 1 | Level 2 | Level 3 |
|------|---------|---------|---------|
| **規模** | 單一類別 | 多個類別組成模組 | 多個服務組成系統 |
| **通信** | 方法調用 | 方法調用 | HTTP/RPC調用 |
| **數據** | 內存變量 | 內存+DB | 分散式存儲 |
| **並發** | 單線程 | 多線程 | 多進程/多機器 |
| **失敗處理** | 異常 | 異常+重試 | 超時+熔斷+降級 |
| **適用場景** | 原型/小項目 | 中型應用 | 大規模系統 |

---

### ✅ 知識檢查點 2

請回答:
1. UserService類別如何演變為User Service系統組件?
2. 為什麼需要從Level 2演進到Level 3?
3. `self.db` 和 Database Cluster 的本質區別是什麼?

<details>
<summary>查看參考答案</summary>

1. **演變過程**:
   - 將單個UserService實例部署為獨立服務
   - 通過容器(Docker)打包
   - 通過編排工具(K8s)管理多個實例
   - 通過API Gateway對外暴露

2. **為什麼需要Level 3**:
   - 單機性能上限 (CPU, Memory, Disk)
   - 高可用性需求 (單點故障)
   - 地理分布需求 (低延遲)
   - 團隊協作需求 (微服務獨立開發)

3. **本質區別**:
   - `self.db`: 內存引用,納秒級訪問,無網絡開銷
   - Database Cluster: 網絡調用,毫秒級訪問,需要考慮網絡分區、超時、重試
</details>

---

## 實戰演練: 電商系統從OOP到System Design

### 階段1: OOP類別設計

```python
# models.py
class User:
    def __init__(self, id, username, email):
        self.id = id
        self.username = username
        self.email = email

class Product:
    def __init__(self, id, name, price, stock):
        self.id = id
        self.name = name
        self.price = price
        self.stock = stock

class Order:
    def __init__(self, id, user, items):
        self.id = id
        self.user = user  # User實例
        self.items = items  # List[Product]

    def calculate_total(self):
        return sum(item.price for item in self.items)
```

**類別關係圖 (Class Diagram)**:
```
┌──────────┐
│   User   │
└──────────┘
      △
      │ 1
      │
┌─────┴─────┐
│   Order   │
└─────┬─────┘
      │ *
      ▼
┌──────────┐
│ Product  │
└──────────┘
```

### 階段2: 應用SOLID原則

**問題**: Order類別職責過多 (計算、儲存、通知)

**重構 - 單一職責原則 (SRP)**:
```python
# 拆分職責
class Order:
    """只負責資料"""
    def __init__(self, id, user, items):
        self.id = id
        self.user = user
        self.items = items

class OrderCalculator:
    """只負責計算"""
    def calculate_total(self, order):
        return sum(item.price for item in order.items)

class OrderRepository:
    """只負責儲存"""
    def save(self, order):
        db.save(order)

class OrderNotifier:
    """只負責通知"""
    def notify_user(self, order):
        send_email(order.user.email, f"Order {order.id} created")
```

**應用依賴倒置原則 (DIP)**:
```python
from abc import ABC, abstractmethod

# 定義抽象介面
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount):
        pass

# 具體實作
class StripePayment(PaymentGateway):
    def charge(self, amount):
        # Stripe API調用
        pass

class PayPalPayment(PaymentGateway):
    def charge(self, amount):
        # PayPal API調用
        pass

# 高層模組依賴抽象
class OrderService:
    def __init__(self, payment_gateway: PaymentGateway):
        self.payment = payment_gateway

    def process_order(self, order):
        total = OrderCalculator().calculate_total(order)
        self.payment.charge(total)
```

### 階段3: 轉換為系統架構

**微服務拆分** (對應SRP):
```
OrderService → Order Service (微服務)
OrderCalculator → Pricing Service
OrderRepository → Database Service
OrderNotifier → Notification Service
PaymentGateway → Payment Service
```

**系統架構圖**:
```
                    ┌──────────────┐
                    │ API Gateway  │
                    └──────┬───────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼─────┐      ┌────▼──────┐     ┌────▼─────┐
   │  User    │      │   Order   │     │ Product  │
   │ Service  │      │  Service  │     │ Service  │
   └────┬─────┘      └────┬──────┘     └────┬─────┘
        │                 │                  │
        │            ┌────┼────┐             │
        │            │    │    │             │
   ┌────▼─────┐ ┌───▼──┐ │ ┌──▼──────┐ ┌───▼──────┐
   │   DB     │ │  DB  │ │ │ Pricing │ │    DB    │
   │ (Users)  │ │(Order│ │ │ Service │ │(Products)│
   └──────────┘ └──────┘ │ └─────────┘ └──────────┘
                         │
                    ┌────▼──────────┐
                    │   Payment     │
                    │   Service     │
                    └────┬──────────┘
                         │
                    ┌────▼──────────┐
                    │ Notification  │
                    │   Service     │
                    └───────────────┘
```

**API設計** (對應方法):
```python
# Order類別的方法
order.calculate_total()  →  GET /api/orders/{id}/total
order.save()            →  POST /api/orders
order.get_status()      →  GET /api/orders/{id}/status
```

---

## 第三部分: SOLID原則在System Design中的應用 (120分鐘)

> **學習目標**: 理解如何將SOLID五大原則從代碼層次應用到系統架構層次

**學習建議**:
- 每個原則分開學習,間隔15分鐘休息
- 對比OOP和System Design的雙重應用
- 完成每個原則後的思考題

---

### 1️⃣ 單一職責原則 (SRP) → 微服務拆分

**核心思想**: 一個類別/服務只做一件事,只有一個改變的理由

#### 📝 OOP層面實現

```python
# ❌ 違反SRP - User類別職責過多
class User:
    def __init__(self, name, email):
        self.name = name
        self.email = email

    def save_to_db(self):
        """職責1: 數據持久化"""
        db.insert('users', self.__dict__)

    def send_welcome_email(self):
        """職責2: 發送郵件"""
        smtp.send(self.email, "Welcome!")

    def generate_report(self):
        """職責3: 生成報表"""
        return f"User: {self.name}"
```

**問題**: 修改郵件模板需要改User類別,修改數據庫需要改User類別,改報表格式也要改User類別!

```python
# ✅ 遵循SRP - 職責分離
class User:
    """只負責表示用戶數據"""
    def __init__(self, name, email):
        self.name = name
        self.email = email

class UserRepository:
    """只負責數據持久化"""
    def save(self, user: User):
        db.insert('users', user.__dict__)

class EmailService:
    """只負責郵件發送"""
    def send_welcome_email(self, user: User):
        smtp.send(user.email, "Welcome!")

class ReportGenerator:
    """只負責報表生成"""
    def generate_user_report(self, user: User):
        return f"User: {user.name}"
```

**好處**: 每個類別只有一個改變的理由!

---

#### 🏗️ System Design層面實現

```
❌ 違反SRP - 單體應用 (Monolith)
┌────────────────────────────────┐
│     All-in-One Service         │
│                                │
│  👤 User Management            │ ← 改用戶邏輯
│  📦 Order Processing           │ ← 改訂單邏輯  } 都要重啟整個服務!
│  💳 Payment                    │ ← 改支付邏輯
│  📧 Notification               │ ← 改通知邏輯
└────────────────────────────────┘
    問題: 團隊衝突、部署風險、擴展困難

✅ 遵循SRP - 微服務架構 (Microservices)
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│   User   │ │  Order   │ │ Payment  │ │  Notify  │
│ Service  │ │ Service  │ │ Service  │ │ Service  │
│          │ │          │ │          │ │          │
│ Team A   │ │ Team B   │ │ Team C   │ │ Team D   │
└──────────┘ └──────────┘ └──────────┘ └──────────┘
   獨立部署    獨立擴展     獨立開發     獨立升級
```

**對應關係**:
- User類別 → User Service
- UserRepository類別 → Order Service的數據層
- EmailService類別 → Notification Service

---

#### 💭 思考題: SRP

<details>
<summary>為什麼Netflix要將Billing Service和Streaming Service分開?</summary>

**答案**:
- **不同改變頻率**: 計費邏輯很少改,但推薦算法頻繁改
- **不同擴展需求**: 播放服務需要1000台服務器,計費只需10台
- **不同團隊**: 計費團隊(財務背景) vs 播放團隊(視頻技術)
- **故障隔離**: 計費掛了不影響用戶繼續看片

**這就是SRP在System Design的體現!**
</details>

---

### 2. 開放封閉原則 (OCP) → 插件架構

**OOP層面**:
```python
# ❌ 違反OCP - 每次新增支付方式都要修改
class PaymentProcessor:
    def process(self, method, amount):
        if method == 'credit_card':
            # ...
        elif method == 'paypal':
            # ...
        elif method == 'bitcoin':  # 新增需修改
            # ...

# ✅ 遵循OCP - 擴展不修改
class PaymentGateway(ABC):
    @abstractmethod
    def pay(self, amount): pass

class CreditCardPayment(PaymentGateway): pass
class PayPalPayment(PaymentGateway): pass
class BitcoinPayment(PaymentGateway): pass  # 新增無需修改原代碼
```

**System Design層面**:
```
✅ 插件式支付架構
┌──────────────┐
│Payment Service│
└──────┬───────┘
       │ 通過配置選擇
   ┌───┴────┬────────┬─────────┐
   │        │        │         │
┌──▼──┐ ┌──▼──┐ ┌───▼──┐ ┌────▼────┐
│Stripe│ │PayPal││Bitcoin││ 新插件  │
└──────┘ └──────┘ └──────┘ └─────────┘
```

### 3. 里氏替換原則 (LSP) → 服務可替換性

**OOP層面**:
```python
class CacheInterface(ABC):
    @abstractmethod
    def get(self, key): pass
    @abstractmethod
    def set(self, key, value): pass

class RedisCache(CacheInterface):
    def get(self, key): return redis.get(key)
    def set(self, key, value): redis.set(key, value)

class MemcachedCache(CacheInterface):
    def get(self, key): return memcached.get(key)
    def set(self, key, value): memcached.set(key, value)

# 可互換使用
cache: CacheInterface = RedisCache()  # 或 MemcachedCache()
```

**System Design層面**:
```
✅ 快取層可替換
┌────────────┐
│   Service  │
└─────┬──────┘
      │ CacheInterface
      ▼
┌─────────────┐
│   Redis     │ ← 可替換為 Memcached 或其他
└─────────────┘
```

### 4. 介面隔離原則 (ISP) → API設計

**OOP層面**:
```python
# ❌ 違反ISP - 胖介面
class Worker(ABC):
    @abstractmethod
    def work(self): pass
    @abstractmethod
    def eat(self): pass

class Robot(Worker):
    def work(self): pass
    def eat(self): pass  # Robot不需要eat,被迫實作

# ✅ 遵循ISP - 介面隔離
class Workable(ABC):
    @abstractmethod
    def work(self): pass

class Eatable(ABC):
    @abstractmethod
    def eat(self): pass

class Human(Workable, Eatable): pass
class Robot(Workable): pass
```

**System Design層面**:
```
❌ 胖API (所有客戶端都要實作所有方法)
┌────────────────────────┐
│   Huge API             │
│  - getUserProfile()    │
│  - updateUserProfile() │
│  - deleteUser()        │
│  - adminResetPassword()│ ← 普通客戶端不需要
└────────────────────────┘

✅ 細粒度API (按需使用)
┌──────────────┐ ┌──────────────┐
│   User API   │ │  Admin API   │
│  - getProfile│ │ - deleteUser │
│  - update    │ │ - reset      │
└──────────────┘ └──────────────┘
```

### 5. 依賴倒置原則 (DIP) → 服務解耦

**OOP層面**:
```python
# ❌ 違反DIP - 高層依賴低層
class OrderService:
    def __init__(self):
        self.db = MySQLDatabase()  # 直接依賴具體實作

# ✅ 遵循DIP - 依賴抽象
class OrderService:
    def __init__(self, db: DatabaseInterface):
        self.db = db  # 依賴抽象介面
```

**System Design層面**:
```
❌ 緊耦合
┌──────────────┐
│Order Service │
└──────┬───────┘
       │ 直接調用
   ┌───▼────┐
   │ MySQL  │
   └────────┘

✅ 鬆耦合 (通過抽象層)
┌──────────────┐
│Order Service │
└──────┬───────┘
       │ 通過Interface
   ┌───▼────────┐
   │ DB Adapter │ ← 可切換不同資料庫
   └───┬────────┘
       ▼
   MySQL/PostgreSQL/MongoDB
```

---

## 設計模式在System Design中的應用

### 1. 工廠模式 → Service Factory

**OOP**:
```python
class PaymentFactory:
    @staticmethod
    def create_payment(method):
        if method == 'credit_card':
            return CreditCardPayment()
        elif method == 'paypal':
            return PayPalPayment()
```

**System Design**:
```
┌──────────────────┐
│ Payment Factory  │
│    Service       │
└────────┬─────────┘
         │ 根據配置動態創建
    ┌────┴────┬─────────┐
    ▼         ▼         ▼
┌───────┐ ┌───────┐ ┌────────┐
│Stripe │ │PayPal │ │Alipay  │
│Service│ │Service│ │Service │
└───────┘ └───────┘ └────────┘
```

### 2. 單例模式 → 配置中心

**OOP**:
```python
class Config:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance
```

**System Design**:
```
所有服務 → 單一配置中心 (Consul/etcd)
確保配置一致性
```

### 3. 觀察者模式 → 事件驅動架構

**OOP**:
```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def notify(self):
        for observer in self._observers:
            observer.update()
```

**System Design**:
```
┌──────────────┐
│ Event Source │
│ (Subject)    │
└──────┬───────┘
       │ publish
   ┌───▼──────┐
   │  Kafka   │
   └───┬──────┘
       │ subscribe
   ┌───┴───┬─────────┬─────────┐
   │       │         │         │
┌──▼──┐ ┌─▼──┐ ┌────▼──┐ ┌────▼──┐
│Obs 1│ │Obs2│ │Obs 3  │ │Obs 4  │
└─────┘ └────┘ └───────┘ └───────┘
```

### 4. 代理模式 → API Gateway

**OOP**:
```python
class Proxy:
    def __init__(self, real_subject):
        self._real_subject = real_subject

    def request(self):
        # 前置處理
        self._real_subject.request()
        # 後置處理
```

**System Design**:
```
Client → API Gateway (Proxy) → Backend Services
       ↓
    - 認證
    - 限流
    - 日誌
    - 快取
```

---

## 從類別圖到架構圖的轉換

### 步驟1: 識別核心實體

**類別圖**:
```
User, Product, Order, Payment
```

### 步驟2: 分組為服務

**按業務領域分組**:
- User Management: User相關
- Catalog: Product相關
- Order Processing: Order相關
- Payment: Payment相關

### 步驟3: 定義服務邊界

**每個服務的職責**:
- User Service: 用戶CRUD, 認證
- Product Service: 商品管理, 庫存
- Order Service: 訂單創建, 狀態管理
- Payment Service: 支付處理

### 步驟4: 設計服務間通信

**同步 vs 異步**:
```python
# 同步 (REST API)
order_service.create_order()
  → user_service.get_user()  # HTTP調用

# 異步 (Message Queue)
order_service.create_order()
  → publish_event('OrderCreated')
  → notification_service.listen('OrderCreated')
```

### 步驟5: 添加基礎設施組件

```
- API Gateway (入口)
- Load Balancer (負載均衡)
- Database (每服務一個)
- Cache (Redis)
- Message Queue (Kafka)
- Service Discovery (Consul)
```

### 完整架構圖

```
                    Internet
                       │
                ┌──────▼──────┐
                │ API Gateway │
                └──────┬──────┘
                       │
            ┌──────────┼──────────┐
            │          │          │
       ┌────▼───┐ ┌───▼────┐ ┌──▼──────┐
       │  User  │ │Product │ │ Order   │
       │Service │ │Service │ │ Service │
       └────┬───┘ └───┬────┘ └──┬──────┘
            │         │         │
       ┌────▼───┐ ┌──▼─────┐ ┌─▼──────┐
       │UserDB  │ │ProductDB│ │OrderDB │
       └────────┘ └─────────┘ └────────┘
                       │
                  ┌────▼────┐
                  │  Kafka  │ (Event Bus)
                  └────┬────┘
                       │
                ┌──────▼────────┐
                │  Notification │
                │    Service    │
                └───────────────┘
```

---

## 實戰練習: 設計Twitter系統

### Level 1: OOP類別設計

```python
class User:
    def __init__(self, id, username):
        self.id = id
        self.username = username
        self.followers = []  # List[User]
        self.following = []  # List[User]

class Tweet:
    def __init__(self, id, user, content, timestamp):
        self.id = id
        self.user = user
        self.content = content
        self.timestamp = timestamp

class Timeline:
    def __init__(self, user):
        self.user = user

    def get_tweets(self):
        # 獲取關注用戶的推文
        tweets = []
        for followed_user in self.user.following:
            tweets.extend(followed_user.tweets)
        return sorted(tweets, key=lambda t: t.timestamp, reverse=True)
```

### Level 2: 應用SOLID重構

```python
# 單一職責
class User: pass
class Tweet: pass

class FollowService:
    def follow(self, user, target): pass
    def unfollow(self, user, target): pass

class TweetService:
    def post_tweet(self, user, content): pass

class TimelineService:
    def get_timeline(self, user): pass

# 依賴倒置
class StorageInterface(ABC):
    @abstractmethod
    def save_tweet(self, tweet): pass

class MySQLStorage(StorageInterface): pass
class CassandraStorage(StorageInterface): pass
```

### Level 3: System Design架構

```
Client
  ↓
API Gateway
  ↓
┌────────┬─────────┬──────────┐
│        │         │          │
User    Tweet   Timeline   Follow
Service Service  Service   Service
  │        │         │          │
UserDB  TweetDB   Redis     FollowDB
                 (Cache)
```

**關鍵設計決策**:

1. **Fan-out策略** (對應Timeline.get_tweets()):
```python
# 寫時Fan-out (Push Model)
def post_tweet(user, content):
    tweet = Tweet(user, content)
    save_tweet(tweet)
    # 推送到所有粉絲的Timeline
    for follower in user.followers:
        redis.lpush(f"timeline:{follower.id}", tweet.id)
```

2. **資料分片** (對應大規模User):
```
user_id → hash(user_id) % N → ShardN
```

---

## 常見陷阱與最佳實踐

### ❌ 陷阱1: 過度設計

```python
# 不要在一開始就設計10層抽象
class AbstractBaseFactorySingletonProxyDecorator: pass
```

**建議**: 從簡單開始,根據需求演進

### ❌ 陷阱2: 忽略性能

```python
# OOP優雅但可能慢
for user in all_users:  # 100萬用戶
    user.calculate_score()  # 每次都計算
```

**System Design考量**: 快取、批次處理、異步

### ❌ 陷阱3: 類別直接對應服務

```python
# 不是每個類別都要成為一個微服務
class UserAddress: pass  ← 不需要獨立服務
```

**建議**: 按業務領域劃分,不是按類別

### ✅ 最佳實踐

1. **從單體開始,逐步拆分**
2. **SOLID原則在兩個層次都適用**
3. **類別的依賴關係 = 服務的依賴關係**
4. **介面 = API Contract**

---

## 練習題

### 基礎題

1. **設計一個Blog系統**
   - OOP: User, Post, Comment類別設計
   - System Design: 拆分為微服務

2. **應用SRP**
   - 找出違反單一職責的類別並重構
   - 對應到系統設計中的服務拆分

### 進階題

3. **設計Uber系統**
   - OOP: Driver, Rider, Trip類別
   - SOLID重構
   - 微服務架構設計
   - 處理實時定位 (GPS)

4. **設計Instagram**
   - OOP: User, Photo, Feed類別
   - 圖片儲存策略
   - Feed生成算法
   - 系統架構設計

---

## 📚 本章總結與複習

### ✅ 學習成果檢查清單

完成本章後,你應該能夠:

#### 理論理解
- [ ] 說出OOP和System Design的5個核心對應關係
- [ ] 解釋為什麼需要從Level 2演進到Level 3
- [ ] 描述SOLID原則在兩個層次的應用

#### 實踐能力
- [ ] 將一個類別設計轉換為系統架構圖
- [ ] 識別違反SOLID原則的設計並重構
- [ ] 設計一個中等複雜度的微服務架構

#### 面試準備
- [ ] 用RADIO框架回答"設計Twitter"
- [ ] 能說出3個設計模式的System Design對應
- [ ] 能討論微服務拆分的權衡(Trade-offs)

---

### 🎯 核心要點回顧 (30分鐘速記)

#### 1. 基本對應關係
```
Class      → Service
Method     → API Endpoint
Interface  → API Contract
Composition → Service Integration
SOLID      → Architecture Principles
```

#### 2. 三層演進
```
Level 1 (單一類別)
  → 添加依賴管理 →
Level 2 (模組設計)
  → 添加分散式 →
Level 3 (系統設計)
```

#### 3. SOLID精髓
- **SRP**: 一個服務一個職責 → 微服務拆分
- **OCP**: 擴展不修改 → 插件架構
- **LSP**: 子類可替換 → 服務可替換
- **ISP**: 介面隔離 → API細粒度化
- **DIP**: 依賴抽象 → 服務解耦

#### 4. 設計模式應用
- 工廠模式 → Service Factory
- 單例模式 → 配置中心
- 觀察者模式 → Event-Driven
- 代理模式 → API Gateway

---

### 🧠 思維導圖

```
從OOP到System Design
│
├─ 理論基礎
│   ├─ 對照表 (Class→Service, Method→API...)
│   ├─ 三層演進 (Level 1/2/3)
│   └─ 為什麼需要橋接
│
├─ SOLID原則
│   ├─ SRP → 微服務拆分
│   ├─ OCP → 插件架構
│   ├─ LSP → 服務可替換性
│   ├─ ISP → API設計
│   └─ DIP → 服務解耦
│
├─ 設計模式
│   ├─ 創建型 → Service Factory
│   ├─ 結構型 → API Gateway
│   └─ 行為型 → Event-Driven
│
└─ 實戰案例
    ├─ 電商系統 (完整演進)
    ├─ Twitter (Fan-out策略)
    └─ 練習題 (Blog, Uber, Instagram)
```

---

### 📖 延伸閱讀建議

**如果你想深入某個主題**:

1. **微服務架構**:
   - 書籍: *Building Microservices* by Sam Newman
   - 重點: Service boundaries, Communication patterns

2. **分散式系統**:
   - 書籍: *Designing Data-Intensive Applications* by Martin Kleppmann
   - 重點: CAP定理, Consensus algorithms

3. **API設計**:
   - 資源: REST API Design Rulebook
   - 重點: RESTful principles, GraphQL vs REST

4. **真實案例**:
   - ByteByteGo Newsletter
   - Engineering blogs: Netflix, Uber, Airbnb

---

### 🎓 下一步學習路徑

完成本章後,你已經具備:
- ✅ OOP紮實基礎
- ✅ SOLID原則雙層應用能力
- ✅ 從類別到系統的思維轉換能力
- ✅ 基本的分散式系統概念

**接下來的學習路徑**:

```
你現在在這裡 ▼
─────────────────────────────────────────────
OOP基礎 → SOLID → [OOP to System Design] → System Design基礎 → 進階主題
                          ✓完成
```

**推薦進入**: [System Design基礎概念](../System-design/01-Fundamentals/README.md)

**System Design課程預告**:
- 📊 容量估算 (Capacity Estimation)
- 🎯 CAP定理 (Consistency, Availability, Partition Tolerance)
- ⚖️ 負載均衡 (Load Balancing)
- 💾 快取策略 (Caching Strategies)
- 🌐 分散式系統 (Distributed Systems)

**所有這些概念都建立在你現在的OOP基礎之上!**

---

### 💡 給面試者的建議

**在System Design面試中**:

1. **一定要提到OOP背景**:
   ```
   面試官: "設計Twitter的Timeline系統"
   你: "這類似於OOP中的Observer Pattern,我們可以..."
   ```

2. **用SOLID解釋架構決策**:
   ```
   "我選擇將Timeline Service和Tweet Service分開,
   是基於Single Responsibility Principle..."
   ```

3. **展示漸進式思維**:
   ```
   "讓我先設計一個簡單版本(Level 2),
   然後我們可以討論如何擴展到百萬用戶(Level 3)..."
   ```

4. **討論Trade-offs**:
   ```
   "微服務提供了靈活性但增加了複雜度,
   在初期我建議用Modular Monolith..."
   ```

---

### 🎯 最後的練習挑戰

**挑戰**: 在45分鐘內完成一個System Design

**題目**: 設計一個短網址服務 (like bit.ly)

**要求**:
1. 從OOP類別設計開始
2. 應用SOLID原則重構
3. 轉換為微服務架構
4. 討論擴展性和Trade-offs

**提示**:
- 想想需要哪些類別? (URL, User, Analytics?)
- 如何應用SRP拆分服務?
- 使用什麼資料庫? (想想OOP的依賴倒置!)
- 如何處理高並發? (想想Level 3的擴展!)

<details>
<summary>查看參考架構</summary>

**Level 2: 模組設計**
```python
class URLShortener:
    def __init__(self, db: DatabaseInterface):
        self.db = db

    def shorten(self, long_url: str) -> str:
        short_code = generate_code()
        self.db.save(short_code, long_url)
        return short_code
```

**Level 3: 系統架構**
```
Client → API Gateway
           ↓
    URL Shortener Service
           ↓
    ┌──────┴──────┐
    │             │
Database      Redis Cache
(Cassandra)   (讀優化)
```

**關鍵設計**:
- SRP: 分離URL生成和Analytics
- DIP: Database Interface允許切換DB
- Cache-Aside Pattern for reads
- Consistent Hashing for sharding
</details>

---

## 📎 快速參考卡片

### OOP → System Design 速查表

| 當你看到OOP中的... | 在System Design中思考... |
|------------------|------------------------|
| 類別有多個方法 | 是否需要拆分為多個服務? (SRP) |
| 使用繼承 | 能否用Composition替代? (LSP) |
| 定義Interface | 這是我的API Contract嗎? (ISP) |
| 依賴注入 | 如何做Service Discovery? (DIP) |
| 創建新對象 | 使用Factory Service? |
| 觀察者模式 | 考慮Event-Driven架構? |
| 單例模式 | 需要配置中心嗎? |

---

**上一章**: [SOLID原則](./10-SOLID-Principles.md)
**下一章**: [System Design基礎](../System-design/01-Fundamentals/README.md)
**返回**: [OOP目錄](./README.md)

---

**最後更新**: 2025-10-31
**作者**: FANG Senior Engineers
**反饋**: 歡迎提Issue改進本章內容
