# Software Design Principles & System Design Interview Prep

> 以FANG資深工程師視角設計的軟體工程實務課程
> 從基礎原則到系統設計面試的完整學習路徑

## 課程定位

本課程專為準備進入頂尖科技公司(FANG/FAANG)的工程師設計,涵蓋從基礎物件導向程式設計到大規模分散式系統設計的完整知識體系。

## 學習路徑

```
OOP基礎 → SOLID原則 → 設計模式 → 系統設計基礎 → 分散式系統 → 實戰案例
```

## 課程結構

### 📚 Phase 1: 基礎建設 (Foundation)

#### [OOP - Python物件導向程式設計速成](./OOP) ⭐ 全新優化
**學習時間**: 2個月 (初學者) | 3週 (有基礎者)

**階段一: OOP基礎** (2週)
- OOP核心概念 (類別、物件、實例)
- 四大支柱 (封裝、繼承、多型、抽象)
- [現有教材: 12個Jupyter Notebooks](./OOP)

**階段二: Python特性** (1.5週)
- Python物件模型 (Everything is an Object)
- 命名空間與作用域 (LEGB規則)
- 封裝實踐 (\_, \_\_, magic methods)

**階段三: 進階技巧** (2週)
- 繼承深入 (super(), MRO)
- 元類與抽象類別
- 裝飾器模式
- 迭代器與生成器

**階段四: 實戰應用** (2週)
- [SOLID原則](./System-design/SOLID) ⭐ 核心
- 設計模式入門
- Library vs Framework vs API

**階段五: 銜接System Design** (1週)
- [從OOP到System Design](./OOP/13-OOP-to-System-Design.md) ⭐⭐⭐
- 貫穿性專案: 電商系統設計
- 類別圖 → 架構圖的轉換

**學習檢查清單**:
- [ ] 能應用OOP四大支柱設計類別
- [ ] 理解並應用SOLID五大原則
- [ ] 能從類別設計思維轉換到系統架構思維
- [ ] 完成電商系統專案 (從單機到分散式)

**詳細學習路徑**: [OOP README](./OOP/README.md)

### 🏗️ Phase 2: 系統設計核心 (System Design Core)

#### [Module 1: 基礎概念](./System-design/01-Fundamentals)
**目標**: 建立系統設計的底層思維模型

- **擴展性基礎 (Scalability Fundamentals)**
  - 垂直擴展 vs 水平擴展
  - 無狀態 vs 有狀態架構
  - 資料分片策略 (Sharding)

- **效能優化 (Performance)**
  - CAP定理深度解析
  - 一致性模型 (Strong/Eventual Consistency)
  - 延遲 vs 吞吐量的權衡

- **可靠性設計 (Reliability)**
  - 故障模式分析 (Failure Modes)
  - 冗餘與備援策略
  - 降級與熔斷機制

#### [Module 2: 核心組件](./System-design/02-Core-Components)
**目標**: 掌握系統設計的構建塊

- **負載均衡 (Load Balancing)**
  - L4 vs L7負載均衡
  - 負載均衡算法 (Round-Robin, Least-Connection, Consistent Hashing)
  - 健康檢查機制

- **快取系統 (Caching)**
  - 快取策略 (Cache-Aside, Write-Through, Write-Back)
  - 快取失效 (Eviction Policies: LRU, LFU, FIFO)
  - 多層快取架構
  - 快取一致性問題

- **資料庫設計 (Database Design)**
  - SQL vs NoSQL選擇決策
  - 資料庫索引原理
  - 讀寫分離 (Read Replicas)
  - 資料分片 (Horizontal Partitioning)

- **訊息佇列 (Message Queues)**
  - 非同步處理模式
  - 訊息語義 (At-most-once, At-least-once, Exactly-once)
  - 背壓處理 (Backpressure)

#### [Module 3: 分散式系統](./System-design/03-Distributed-Systems)
**目標**: 理解大規模系統的複雜性

- **分散式共識 (Consensus)**
  - Paxos vs Raft算法
  - 主從複製 (Leader-Follower Replication)
  - 多主複製 (Multi-Leader Replication)

- **資料一致性 (Data Consistency)**
  - 分散式交易 (2PC, 3PC, Saga)
  - 事件溯源 (Event Sourcing)
  - CQRS模式

- **服務發現 (Service Discovery)**
  - 註冊中心模式
  - DNS vs Consul/Etcd
  - 健康檢查與心跳機制

### 🎯 Phase 3: 實戰演練 (Real-World Cases)

#### [Module 4: 經典系統設計案例](./System-design/04-Case-Studies)
**目標**: 以FANG面試真題為基礎的實戰訓練

- **社交媒體類**
  - 設計Twitter/微博
  - 設計Instagram/抖音
  - 設計Facebook News Feed

- **電商平台類**
  - 設計Amazon商品推薦系統
  - 設計秒殺系統
  - 設計購物車系統

- **內容分發類**
  - 設計YouTube視頻平台
  - 設計Netflix推薦引擎
  - 設計CDN系統

- **協作工具類**
  - 設計Google Docs協作編輯
  - 設計Slack即時通訊
  - 設計Zoom視訊會議

- **基礎設施類**
  - 設計短網址服務 (URL Shortener)
  - 設計分散式鎖
  - 設計限流器 (Rate Limiter)
  - 設計網頁爬蟲

#### [Module 5: 進階主題](./System-design/05-Advanced-Topics)
**目標**: 深入理解大規模系統的細節

- **監控與可觀測性 (Observability)**
  - Metrics, Logs, Traces三支柱
  - 分散式追蹤 (Distributed Tracing)
  - 告警策略設計

- **安全性設計 (Security)**
  - 認證與授權 (OAuth 2.0, JWT)
  - API安全最佳實踐
  - DDoS防護策略

- **成本優化 (Cost Optimization)**
  - 資源利用率分析
  - Auto-scaling策略
  - 冷熱資料分離

## 🎓 學習方法論

### 面試準備框架 (Interview Framework)

**RADIO框架** - FANG工程師常用的系統設計面試方法:

1. **R**equirements (需求澄清)
   - 功能性需求
   - 非功能性需求 (規模、效能、可用性)
   - 約束條件

2. **A**PI Design (API設計)
   - 核心接口定義
   - 資料模型設計

3. **D**ata Model (資料模型)
   - 資料庫選擇
   - Schema設計

4. **I**nfrastructure (基礎架構)
   - 高層架構圖
   - 組件選擇與權衡

5. **O**ptimization (優化與深入)
   - 瓶頸分析
   - 擴展性討論

### 每個模組的學習流程

```
理論學習 → 程式碼實作 → 案例分析 → 面試模擬
```

## 📖 推薦學習順序

### 初學者路徑 (3-6個月)
1. OOP基礎 (2週)
2. SOLID原則 (2週)
3. 系統設計基礎概念 (4週)
4. 核心組件學習 (6週)
5. 經典案例演練 (8週)

### 進階路徑 (2-3個月)
1. 複習SOLID原則 (1週)
2. 系統設計核心組件 (4週)
3. 分散式系統 (4週)
4. 實戰案例密集訓練 (4週)

## 🔧 技術棧參考

- **程式語言**: Python, Go, Java
- **資料庫**: PostgreSQL, MySQL, MongoDB, Redis, Cassandra
- **訊息佇列**: Kafka, RabbitMQ, AWS SQS
- **快取**: Redis, Memcached
- **搜尋引擎**: Elasticsearch
- **雲服務**: AWS, GCP, Azure

## 📚 參考資源

### 必讀書籍
- **Designing Data-Intensive Applications** by Martin Kleppmann
- **System Design Interview** by Alex Xu (Vol 1 & 2)
- **Clean Architecture** by Robert C. Martin

### 線上資源
- [System Design Primer](https://github.com/donnemartin/system-design-primer)
- [ByteByteGo Newsletter](https://blog.bytebytego.com/)
- [Grokking the System Design Interview](https://www.educative.io/courses/grokking-the-system-design-interview)

## 🎯 面試成功指標

完成本課程後,你應該能夠:

✅ 在45分鐘內完整設計一個中等複雜度的系統
✅ 清楚說明設計決策的權衡 (Trade-offs)
✅ 估算系統的容量需求 (Capacity Estimation)
✅ 識別並解決系統的瓶頸
✅ 討論系統的擴展性和可靠性

## 🤝 貢獻指南

歡迎提交Pull Request或Issue來改進課程內容!

## 📝 授權

MIT License

---

**Last Updated**: 2025-10-31
**Maintained by**: FANG Senior Engineers
