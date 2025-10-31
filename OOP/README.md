# Python OOP 速成課程

> 從零基礎到系統設計的完整OOP學習路徑

## 🎯 課程目標

本課程將帶你從Python OOP基礎,逐步進階到能夠運用SOLID原則設計大型系統。這是通向System Design的第一步,也是最重要的基礎。

**學完本課程你將能夠**:
- ✅ 深入理解OOP四大支柱 (封裝、繼承、多型、抽象)
- ✅ 掌握Python特有的OOP特性 (魔術方法、裝飾器、迭代器)
- ✅ 應用SOLID原則設計可維護的代碼
- ✅ 理解OOP如何幫助構建大型系統
- ✅ 為System Design面試打下堅實基礎

---

## 📚 課程結構

### 🟢 階段一: OOP基礎 (Foundation) - 2週

學習目標: 建立紮實的OOP基礎概念

#### [01 - OOP核心概念](./01-OOP-Core-Concepts.md) ⭐ 必讀
- 什麼是物件導向程式設計?
- 類別 vs 實例 vs 物件
- 屬性與方法
- `__init__` 與 self
- 實戰: 設計一個簡單的系統

**現有教材**:
- 📓 [python-object-oriented-programming.ipynb](./python-object-oriented-programming.ipynb) - 核心OOP概念

#### [02 - OOP四大支柱](./02-OOP-Four-Pillars.md) ⭐ 必讀
- 封裝 (Encapsulation) - 資料隱藏與@property
- 繼承 (Inheritance) - 程式碼重用
- 多型 (Polymorphism) - 統一介面
- 抽象 (Abstraction) - ABC與abstractmethod

**現有教材**:
- 📓 [03-inheritance.ipynb](./03-inheritance.ipynb) - 繼承詳解 (火影忍者範例)
- 📓 [04-abstract-Class.ipynb](./04-abstract-Class.ipynb) - 抽象類別

**為什麼這很重要?**
> 這四大支柱是設計可維護、可擴展系統的基礎。在System Design中,你會用這些原則設計微服務、API和系統組件。

---

### 🟡 階段二: Python特性 (Python Specifics) - 1.5週

學習目標: 掌握Python獨特的OOP機制

#### [03 - Python物件模型](./03-Python-Object-Model.md)
- Everything is an Object
- type() vs isinstance()
- Magic Methods (\_\_str\_\_, \_\_repr\_\_, \_\_add\_\_)
- object-type-class三角關係

**現有教材**:
- 📓 [02-underline.ipynb](./02-underline.ipynb) - 底線命名與魔術方法

#### [04 - 命名空間與作用域](./04-Namespace-Scope.md)
- LEGB規則
- global vs nonlocal
- 為什麼這很重要? (避免變數衝突)

**現有教材**:
- 📓 [01-name-space-and-scope.ipynb](./01-name-space-and-scope.ipynb)

#### [05 - 封裝的實踐](./05-Encapsulation-Practice.md)
- \_ (單底線) - protected
- \_\_ (雙底線) - private
- \_\_name\_\_ (dunder) - magic methods
- @property 的進階用法

**現有教材**:
- 📓 [02-underline.ipynb](./02-underline.ipynb)

**與System Design的連結**:
> 在設計API時,你需要明確哪些是公開介面 (public),哪些是內部實作 (private)。這直接對應到系統設計中的"封裝"概念。

---

### 🟠 階段三: 進階技巧 (Advanced Techniques) - 2週

學習目標: 掌握Python進階OOP特性

#### [06 - 繼承的深入探討](./06-Advanced-Inheritance.md)
- super()的正確用法
- MRO (Method Resolution Order)
- 多重繼承的陷阱與最佳實踐
- 組合 vs 繼承

**現有教材**:
- 📓 [03-inheritance.ipynb](./03-inheritance.ipynb)

#### [07 - 元類與抽象類別](./07-Metaclass-ABC.md)
- Metaclass概念
- ABC (Abstract Base Class)
- 為什麼需要抽象類別?
- 實戰: API介面設計

**現有教材**:
- 📓 [03.1-meta-Class.ipynb](./03.1-meta-Class.ipynb)
- 📓 [04-abstract-Class.ipynb](./04-abstract-Class.ipynb)

#### [08 - 裝飾器模式](./08-Decorators.md)
- @property, @classmethod, @staticmethod
- 自訂裝飾器
- 裝飾器的實用場景
- functools.wraps

**現有教材**:
- 📓 [05-decorator.ipynb](./05-decorator.ipynb)

#### [09 - 迭代器與生成器](./09-Iterator-Generator.md)
- \_\_iter\_\_ 與 \_\_next\_\_
- yield關鍵字
- 記憶體效率
- 實戰: 大數據處理

**現有教材**:
- 📓 [06-iterator_and_generator.ipynb](./06-iterator_and_generator.ipynb)

**與System Design的連結**:
> 生成器在處理大規模資料流時非常重要。在設計如Twitter Timeline、日誌處理系統時,你會用到這些技術來優化記憶體使用。

---

### 🔴 階段四: 實戰應用 (Practical Application) - 2週

學習目標: 將OOP應用於實際系統設計

#### [10 - SOLID原則](./10-SOLID-Principles.md) ⭐⭐⭐ 核心
- S - Single Responsibility (單一職責)
- O - Open/Closed (開放封閉)
- L - Liskov Substitution (里氏替換)
- I - Interface Segregation (介面隔離)
- D - Dependency Inversion (依賴倒置)

**現有教材**:
- 📂 [../System-design/SOLID/](../System-design/SOLID/) - 完整SOLID教材

#### [11 - 設計模式入門](./11-Design-Patterns.md)
- 創建型模式: 工廠模式、單例模式
- 結構型模式: 裝飾器模式、適配器模式
- 行為型模式: 策略模式、觀察者模式

#### [12 - Library vs Framework vs API](./12-Library-Framework-API.md)
- 三者的差異
- 如何設計Library
- 如何設計API
- 實戰範例

**現有教材**:
- 📓 [00-library-framework-API.ipynb](./00-library-framework-API.ipynb)

---

### 🚀 階段五: 銜接System Design (Bridge to System Design) - 1週

#### [13 - 從OOP到System Design](./13-OOP-to-System-Design.md) ⭐⭐⭐ 橋接章節
- OOP如何幫助系統設計?
- 類別設計 vs 系統架構
- 從Class Diagram到Architecture Diagram
- 實戰: 設計一個完整的電商系統

**貫穿性專案**: 電商系統設計
- 階段一: User, Product, Order類別設計
- 階段二: 購物車、支付模組
- 階段三: 庫存管理、優惠券系統
- 階段四: 應用SOLID原則重構
- 階段五: 擴展到分散式系統

---

## 🗺️ 學習路徑圖

```
第1-2週: OOP基礎
  ├─ 01 OOP核心概念
  └─ 02 四大支柱
       ↓
第3-4週: Python特性
  ├─ 03 Python物件模型
  ├─ 04 命名空間與作用域
  └─ 05 封裝實踐
       ↓
第5-6週: 進階技巧
  ├─ 06 繼承深入
  ├─ 07 元類與ABC
  ├─ 08 裝飾器
  └─ 09 迭代器與生成器
       ↓
第7-8週: 實戰應用
  ├─ 10 SOLID原則 ⭐
  ├─ 11 設計模式
  └─ 12 Library/Framework/API
       ↓
第9週: 銜接System Design
  └─ 13 OOP to System Design ⭐
       ↓
    System Design課程
```

---

## 📖 現有教材對照表

| 新編號 | 新標題 | 現有教材 | 狀態 |
|--------|--------|----------|------|
| 01 | OOP核心概念 | python-object-oriented-programming.ipynb | ✅ 可用 |
| 02 | OOP四大支柱 | 03-inheritance.ipynb, 04-abstract-Class.ipynb | ✅ 可用 |
| 03 | Python物件模型 | 02-underline.ipynb (部分) | ⚠️ 需擴充 |
| 04 | 命名空間與作用域 | 01-name-space-and-scope.ipynb | ✅ 可用 |
| 05 | 封裝的實踐 | 02-underline.ipynb | ✅ 可用 |
| 06 | 繼承深入 | 03-inheritance.ipynb | ⚠️ 需補充MRO |
| 07 | 元類與ABC | 03.1-meta-Class.ipynb, 04-abstract-Class.ipynb | ✅ 可用 |
| 08 | 裝飾器 | 05-decorator.ipynb | ⚠️ 需補充 |
| 09 | 迭代器與生成器 | 06-iterator_and_generator.ipynb | ⚠️ 需補充 |
| 10 | SOLID原則 | ../System-design/SOLID/ | ✅ 已有 |
| 11 | 設計模式 | - | ❌ 需新增 |
| 12 | Library/Framework/API | 00-library-framework-API.ipynb | ⚠️ 需擴充 |
| 13 | OOP to System Design | - | ❌ 需新增 |

---

## 🎓 每章統一結構

所有新建章節將遵循以下結構:

```markdown
## 本章目標 (Learning Objectives)
- 明確的學習目標清單

## 為什麼需要這個? (Motivation)
- 實際場景引入
- 問題的產生

## 核心概念 (Core Concepts)
- 理論解釋
- 關鍵術語

## 實戰範例 (Hands-on Examples)
- 完整可執行代碼
- 從簡單到複雜

## 常見陷阱 (Common Pitfalls)
- 錯誤示範
- 最佳實踐

## 與System Design的連結 (Connection to System Design)
- 如何應用於大型系統
- 實際案例

## 練習題 (Exercises)
- 基礎題
- 進階題
- 專案練習

## 延伸閱讀 (Further Reading)
- 相關文章
- 官方文檔
```

---

## 🚀 快速開始

### 適合初學者
1. **從頭開始**: [01 - OOP核心概念](./01-OOP-Core-Concepts.md)
2. **現有教材**: [python-object-oriented-programming.ipynb](./python-object-oriented-programming.ipynb)
3. **練習**: 完成每章的練習題

### 適合有基礎者
1. **快速複習**: [02 - OOP四大支柱](./02-OOP-Four-Pillars.md)
2. **直接進階**: [06 - 繼承深入](./06-Advanced-Inheritance.md)
3. **重點學習**: [10 - SOLID原則](./10-SOLID-Principles.md)

### 準備System Design面試
1. **必讀**: [10 - SOLID原則](./10-SOLID-Principles.md)
2. **必讀**: [13 - OOP到System Design](./13-OOP-to-System-Design.md)
3. **實戰**: 完成電商系統專案

---

## 📝 學習建議

### 每日學習計畫 (1-2小時/天)

**Week 1-2: 基礎階段**
```
Day 1: OOP核心概念 (理論)
Day 2: OOP核心概念 (實作)
Day 3: 封裝 (Encapsulation)
Day 4: 繼承 (Inheritance)
Day 5: 多型 (Polymorphism)
Day 6: 抽象 (Abstraction)
Day 7: 複習與練習
```

**Week 3-4: Python特性**
```
Day 1: Python物件模型
Day 2: Magic Methods實戰
Day 3-4: 命名空間與作用域
Day 5-6: 封裝實踐
Day 7: 專案練習
```

**Week 5-6: 進階技巧**
```
Day 1-2: 繼承深入 + MRO
Day 3: 元類
Day 4: 抽象類別
Day 5-6: 裝飾器
Day 7: 迭代器與生成器
```

**Week 7-8: 實戰應用**
```
Day 1-3: SOLID原則 ⭐
Day 4-5: 設計模式
Day 6-7: 電商系統專案
```

**Week 9: 銜接**
```
Day 1-5: OOP to System Design
Day 6-7: 系統設計基礎預習
```

---

## 🎯 貫穿性專案: 電商系統

我們將用一個電商系統作為貫穿整個課程的範例:

### 階段一: 基礎類別設計
```python
class User:
    """用戶類別 - 封裝"""

class Product:
    """商品類別 - 屬性與方法"""

class Order:
    """訂單類別 - 組合關係"""
```

### 階段二: 繼承與多型
```python
class RegularUser(User):
    """普通用戶"""

class VIPUser(User):
    """VIP用戶 - 繼承與方法覆寫"""

class DigitalProduct(Product):
    """數位商品 - 多型"""

class PhysicalProduct(Product):
    """實體商品 - 多型"""
```

### 階段三: 進階特性
```python
class ShoppingCart:
    """購物車 - 迭代器"""
    def __iter__(self): ...

@discount_decorator
def calculate_price(self):
    """價格計算 - 裝飾器"""
```

### 階段四: SOLID重構
```python
# 應用單一職責原則拆分類別
class OrderService:  # 訂單處理
class PaymentService:  # 支付處理
class InventoryService:  # 庫存管理

# 應用依賴倒置
class PaymentInterface(ABC):
    @abstractmethod
    def pay(self): ...

class CreditCardPayment(PaymentInterface):
    def pay(self): ...
```

### 階段五: 系統設計擴展
```
從單機 → 分散式系統
- 如何拆分微服務?
- 如何設計API?
- 如何處理高併發?
```

---

## ⚠️ 現有教材使用說明

### 推薦使用順序

1. **先讀新教材 (Markdown)**: 理論講解與系統化知識
2. **再看Notebook**: 實作練習與範例
3. **完成練習題**: 鞏固理解

### 現有Notebook的使用建議

| Notebook | 推薦使用時機 | 注意事項 |
|---------|-------------|---------|
| python-object-oriented-programming.ipynb | 第1週 | 核心概念,必讀 |
| 01-name-space-and-scope.ipynb | 第3週 | LEGB規則重要 |
| 02-underline.ipynb | 第3-4週 | 封裝實踐 |
| 03-inheritance.ipynb | 第2週 & 第5週 | 火影忍者範例有趣 |
| 03.1-meta-Class.ipynb | 第6週 | 進階,可選 |
| 04-abstract-Class.ipynb | 第6週 | 配合SOLID學習 |
| 05-decorator.ipynb | 第6週 | 需補充functools.wraps |
| 06-iterator_and_generator.ipynb | 第6週 | 需補充yield from |
| 07-lambda.ipynb | 補充閱讀 | 簡短,快速學習 |

### 已知問題與修正

1. **重複內容**: `物件導向程式設計.ipynb` 與 `python-object-oriented-programming.ipynb` 相同,選一即可
2. **圖片顯示**: 部分notebook使用attachment格式,可能無法顯示
3. **錯誤範例**: 部分有未處理的異常,建議加上try-except
4. **缺少MRO**: 繼承章節需補充Method Resolution Order

---

## 📚 延伸資源

### 官方文檔
- [Python OOP Tutorial](https://docs.python.org/3/tutorial/classes.html)
- [Python Data Model](https://docs.python.org/3/reference/datamodel.html)
- [abc — Abstract Base Classes](https://docs.python.org/3/library/abc.html)

### 推薦書籍
- **Python Object-Oriented Programming** - Dusty Phillips
- **Fluent Python** - Luciano Ramalho (進階)
- **Clean Code** - Robert C. Martin (SOLID原則)

### 線上資源
- [Real Python - OOP](https://realpython.com/python3-object-oriented-programming/)
- [Corey Schafer - OOP Tutorials](https://www.youtube.com/playlist?list=PL-osiE80TeTsqhIuOqKhwlXsIBIdSeYtc)

---

## 🎓 學習檢查清單

### 基礎階段
- [ ] 能解釋類別、物件、實例的差異
- [ ] 能使用 `__init__` 初始化物件
- [ ] 理解self的含義
- [ ] 能實作封裝、繼承、多型、抽象
- [ ] 完成電商系統基礎類別

### 進階階段
- [ ] 理解LEGB作用域規則
- [ ] 能正確使用底線命名慣例
- [ ] 能解釋MRO並使用super()
- [ ] 能實作自訂裝飾器
- [ ] 能使用生成器優化記憶體
- [ ] 完成電商系統進階功能

### 實戰階段
- [ ] 能應用SOLID五大原則
- [ ] 認識常見設計模式
- [ ] 能設計Library和API
- [ ] 能將OOP應用於系統設計
- [ ] 完成完整電商系統專案

---

## 💡 常見問題 (FAQ)

**Q: 我需要先學會哪些Python基礎?**
A: 變數、資料型別、函數、if/for/while、list/dict基本操作即可。

**Q: OOP和System Design有什麼關係?**
A: OOP的SOLID原則直接應用於系統設計。類別設計的思維會幫助你設計微服務架構。

**Q: 要花多久時間學完?**
A: 初學者約2個月(每天1-2小時),有基礎者約1個月。

**Q: 現有的Notebook和新教材有衝突嗎?**
A: 沒有,新教材是系統化的理論,Notebook是實作練習,兩者互補。

**Q: 需要全部學完才能進入System Design嗎?**
A: 建議至少完成階段一、二、四 (SOLID原則最重要)。

---

**下一步**: [開始學習 - OOP核心概念](./01-OOP-Core-Concepts.md) 或 [查看現有教材](./python-object-oriented-programming.ipynb)

**返回**: [主課程目錄](../README.md)
