# 快速開始指南 (Quick Start)

> 5分鐘了解課程,立即開始學習之旅

## 🎯 課程定位

這是一套**完整的軟體工程實務課程**,專為準備進入**FANG/FAANG**等頂尖科技公司的工程師設計。

**學完後你將能夠**:
- ✅ 在45分鐘內完整設計一個大規模分散式系統
- ✅ 清楚解釋設計決策的權衡(Trade-offs)
- ✅ 通過FANG的System Design面試

---

## 📖 課程結構概覽

```
SW-Design-principle/
│
├── OOP/                          # Level 0: 基礎建設 (2週)
│   ├── 封裝、繼承、多型、抽象
│   └── 裝飾器、迭代器、生成器
│
├── System-design/
│   ├── SOLID/                    # Level 0: 設計原則 (2週)
│   │   ├── 單一職責原則 (SRP)
│   │   ├── 開放封閉原則 (OCP)
│   │   ├── 里氏替換原則 (LSP)
│   │   ├── 介面隔離原則 (ISP)
│   │   └── 依賴倒置原則 (DIP)
│   │
│   ├── 01-Fundamentals/          # Level 1: 基礎概念 (3週)
│   │   ├── 擴展性 (Scale Up vs Scale Out)
│   │   ├── CAP定理
│   │   ├── 一致性模型
│   │   └── 容量估算
│   │
│   ├── 02-Core-Components/       # Level 2: 核心組件 (4週)
│   │   ├── 負載均衡器
│   │   ├── 快取系統
│   │   ├── 資料庫設計
│   │   └── 訊息佇列
│   │
│   ├── 03-Distributed-Systems/   # Level 3: 分散式系統 (4週)
│   │   ├── 分散式共識
│   │   ├── 資料一致性
│   │   └── 服務發現
│   │
│   ├── 04-Case-Studies/          # Level 4: 實戰案例 (6週)
│   │   ├── 短網址服務 ⭐⭐
│   │   ├── Twitter/Instagram ⭐⭐⭐⭐
│   │   └── YouTube/Google Docs ⭐⭐⭐⭐⭐
│   │
│   └── 05-Advanced-Topics/       # Level 5: 進階主題 (持續)
│       ├── 安全性設計
│       ├── 成本優化
│       └── 微服務架構
│
├── README.md                      # 課程總覽
├── LEARNING-PATH.md               # 詳細學習路徑
├── INTERVIEW-GUIDE.md             # 面試完全指南
└── QUICK-START.md                 # 本文件
```

---

## 🚀 三種學習路線

### 路線A: 初學者 (0基礎 → 面試ready)
**適合**: 1-3年經驗,沒有系統設計經驗
**時間**: 4-6個月

```
Month 1: OOP + SOLID
Month 2: 系統設計基礎 + 核心組件
Month 3-4: 實戰基礎案例
Month 5: 複雜案例
Month 6: 密集訓練
```

👉 **開始**: [LEARNING-PATH.md - 初學者路徑](./LEARNING-PATH.md#-初學者路徑-0基礎--面試ready)

---

### 路線B: 進階者 (有經驗 → 進階面試)
**適合**: 3-5年經驗,有部分系統設計經驗
**時間**: 2-3個月

```
Month 1: 核心組件精通
Month 2: 案例密集訓練
Month 3: 實戰準備
```

👉 **開始**: [LEARNING-PATH.md - 進階路徑](./LEARNING-PATH.md#-進階路徑-有經驗--進階面試)

---

### 路線C: 衝刺者 (面試前2週衝刺)
**適合**: 資深工程師,已有系統設計基礎
**時間**: 2週

```
Week 1:
├─ Day 1: 複習RADIO框架
├─ Day 2-3: 核心組件快速回顧
├─ Day 4-5: 基礎案例 (每天2個)
├─ Day 6-7: 中級案例 (每天1個)

Week 2:
├─ Day 1-3: 高級案例 (每2天1個)
├─ Day 4-7: 模擬面試 (每天2次)
```

👉 **開始**: [INTERVIEW-GUIDE.md](./System-design/INTERVIEW-GUIDE.md)

---

## 📋 立即開始的3個步驟

### Step 1: 選擇你的路線 (5分鐘)

**我是初學者** → 從[OOP基礎](./OOP)開始
**我有經驗** → 從[系統設計基礎](./System-design/01-Fundamentals)開始
**我要衝刺** → 直接看[面試指南](./System-design/INTERVIEW-GUIDE.md)

---

### Step 2: 完成第一個練習 (30分鐘)

**初學者**:
```python
# 練習: 用Python實作一個簡單的類別
class BankAccount:
    def __init__(self, balance=0):
        self._balance = balance  # 封裝

    def deposit(self, amount):
        if amount > 0:
            self._balance += amount

    def get_balance(self):
        return self._balance

# 測試
account = BankAccount(100)
account.deposit(50)
print(account.get_balance())  # 150
```
👉 完成後,繼續學習[OOP/01-name-space-and-scope.ipynb](./OOP/01-name-space-and-scope.ipynb)

---

**進階者**:
```
練習: 容量估算

假設設計Twitter:
- DAU: 200M
- 每用戶每天發2條推文
- 讀寫比: 100:1

計算:
1. 寫入QPS?
2. 讀取QPS?
3. 5年儲存容量? (每條推文1KB)
```

<details>
<summary>查看答案</summary>

```
1. 寫入QPS:
   200M用戶 × 2推文/天 / 86400秒 ≈ 4.6K QPS
   峰值: 4.6K × 3 = 14K QPS

2. 讀取QPS:
   4.6K × 100 = 460K QPS
   峰值: 1.4M QPS

3. 儲存容量:
   每天: 200M × 2 × 1KB = 400GB/天
   5年: 400GB × 365 × 5 = 730TB
   3副本: 2.2PB
```
</details>

👉 完成後,繼續學習[系統設計基礎](./System-design/01-Fundamentals/README.md)

---

**衝刺者**:
```
挑戰: 45分鐘設計短網址服務

需求:
- 100M DAU
- 每月100M新URL
- 讀寫比 100:1

按RADIO框架設計:
1. Requirements (5min)
2. API Design (5min)
3. Data Model (5min)
4. Infrastructure (20min)
5. Optimization (10min)
```

👉 完成後,對照[標準答案](./System-design/04-Case-Studies/README.md#案例1-設計短網址服務)

---

### Step 3: 設定學習計畫 (10分鐘)

**選擇學習時間**:
- [ ] 每天1小時 (4-6個月完成)
- [ ] 每天2小時 (2-3個月完成)
- [ ] 全職學習 (1個月完成)

**設定每週目標**:
```
Week 1目標:
- [ ] 完成OOP基礎 (初學者)
- [ ] 完成3個案例 (進階者)
- [ ] 完成10個模擬面試 (衝刺者)
```

**找學習夥伴** (可選):
- 找同事/朋友一起學習
- 互相Mock Interview
- 討論技術細節

---

## 🎯 學習里程碑

### 第1週後
✅ 完成基礎知識學習
✅ 理解核心概念
✅ 完成第一個練習

### 第1個月後
✅ 掌握SOLID原則
✅ 理解系統設計基礎
✅ 完成3-5個案例

### 第3個月後
✅ 精通核心組件
✅ 完成15+案例
✅ 能設計中等複雜度系統

### 第6個月後
✅ 理解分散式系統
✅ 完成30+案例
✅ **FANG面試Ready** 🎉

---

## 📚 關鍵文檔快速導航

### 必讀文檔
1. **[README.md](./README.md)** - 課程總覽與結構
2. **[LEARNING-PATH.md](./LEARNING-PATH.md)** - 詳細學習路徑
3. **[INTERVIEW-GUIDE.md](./System-design/INTERVIEW-GUIDE.md)** - 面試完全指南

### 核心模組
1. **[系統設計基礎](./System-design/01-Fundamentals/README.md)** - CAP定理、一致性、容量估算
2. **[核心組件](./System-design/02-Core-Components/README.md)** - 負載均衡、快取、資料庫、MQ
3. **[實戰案例](./System-design/04-Case-Studies/README.md)** - Twitter、Instagram、YouTube等

---

## 💡 學習建議

### 有效學習的5個原則

1. **主動學習**: 不要只看,要動手實作
2. **費曼技巧**: 能教會別人才是真懂
3. **刻意練習**: 練習舒適區邊緣的內容
4. **立即反饋**: 模擬面試,獲得反饋
5. **間隔重複**: 定期複習,鞏固記憶

### 常見問題

**Q: 我需要先學會所有資料結構和算法嗎?**
A: 不需要。系統設計重點是架構思維,不是算法。但基本的資料結構(Hash, Tree)要了解。

**Q: 每個案例需要練習幾次?**
A: 至少2次。第1次獨立設計(可能不完美),第2次對照答案優化。重要案例(Twitter, YouTube)建議3次以上。

**Q: 我需要實際寫代碼嗎?**
A: 面試時不需要寫完整代碼,但學習時強烈建議實作核心組件(LRU快取、限流器等),加深理解。

**Q: 多久能達到面試水平?**
A: 取決於基礎和投入時間:
- 初學者: 4-6個月 (每天1-2小時)
- 進階者: 2-3個月 (每天2-3小時)
- 衝刺者: 2週 (全職學習)

---

## 🎓 成功學習的檢查清單

### 開始前
- [ ] 選擇適合自己的學習路線
- [ ] 設定明確的目標和時間表
- [ ] 準備學習環境(筆記本、白板筆、電腦)
- [ ] (可選)找到學習夥伴

### 學習中
- [ ] 每天至少學習1小時
- [ ] 完成所有練習題
- [ ] 記錄學習筆記
- [ ] 定期自我測試
- [ ] 參加模擬面試

### 面試前
- [ ] 完成30+案例練習
- [ ] 模擬面試10+次
- [ ] 複習RADIO框架
- [ ] 準備常見問題的答案

---

## 🚀 現在就開始!

**選擇你的第一步**:

### 👉 初學者
從這裡開始: [OOP基礎](./OOP/python-object-oriented-programming.ipynb)

### 👉 進階者
從這裡開始: [系統設計基礎](./System-design/01-Fundamentals/README.md)

### 👉 衝刺者
從這裡開始: [面試指南](./System-design/INTERVIEW-GUIDE.md)

---

## 📞 需要幫助?

- 📖 **技術問題**: 查看各模組的README
- 💬 **學習交流**: 找學習夥伴討論
- 🎯 **面試準備**: 參考[INTERVIEW-GUIDE.md](./System-design/INTERVIEW-GUIDE.md)

---

**祝你學習順利,面試成功! 💪🎉**

記住: **行動 > 完美** - 現在就開始第一步!
