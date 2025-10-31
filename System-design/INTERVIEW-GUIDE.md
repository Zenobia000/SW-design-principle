# System Design 面試完全指南

> FANG資深工程師的系統設計面試實戰手冊

## 🎯 面試流程概覽

### 典型45分鐘面試結構

```
┌─────────────────────────────────────────────────────┐
│  0-5分鐘   │ 需求澄清 (Requirements Clarification) │
├─────────────────────────────────────────────────────┤
│  5-13分鐘  │ 高層設計 (High-Level Design)         │
│            │ - API設計                            │
│            │ - 資料模型                           │
│            │ - 基本架構                           │
├─────────────────────────────────────────────────────┤
│ 13-33分鐘  │ 詳細設計 (Detailed Design)           │
│            │ - 核心組件                           │
│            │ - 資料流                             │
│            │ - 擴展性討論                         │
├─────────────────────────────────────────────────────┤
│ 33-42分鐘  │ 深入挖掘 (Deep Dive)                │
│            │ - 瓶頸分析                           │
│            │ - 故障處理                           │
│            │ - 優化方案                           │
├─────────────────────────────────────────────────────┤
│ 42-45分鐘  │ 總結 & Q&A                          │
└─────────────────────────────────────────────────────┘
```

---

## 📋 RADIO框架詳解

### R - Requirements (需求澄清) [5分鐘]

#### 要問的問題模板

**功能性需求**:
```
✓ 核心功能有哪些?
  "這個系統最重要的3個功能是什麼?"

✓ 用戶規模?
  "預期有多少日活躍用戶(DAU)?"

✓ 用戶行為?
  "典型用戶會如何使用系統?"

✓ 特殊需求?
  "是否需要支持XXX功能?" (如: 離線模式、實時通知)
```

**非功能性需求**:
```
✓ 可用性要求?
  "系統可用性目標是多少? 99.9%? 99.99%?"

✓ 一致性 vs 可用性?
  "在網路分區時,優先保證一致性還是可用性?"
  (CAP定理權衡)

✓ 延遲要求?
  "P99延遲需要在多少以內? 100ms? 1秒?"

✓ 資料持久性?
  "資料丟失是否可接受?"

✓ 擴展性?
  "預期未來會成長到多大規模?"
```

#### 容量估算 (Back-of-the-envelope)

**必須計算的4個數字**:

1. **QPS (Queries Per Second)**
```
DAU × 每用戶每天請求數 / 86400 = 平均QPS
平均QPS × 3 = 峰值QPS (一般用3倍係數)

範例:
100M DAU × 10請求/天 / 86400 ≈ 11.5K QPS
峰值: 35K QPS
```

2. **儲存容量**
```
每天新增資料量 × 保存年限 × 備份係數

範例:
DAU 100M, 每用戶每天產生1KB資料, 保存5年, 3副本
100M × 1KB × 365天 × 5年 × 3 = 500TB
```

3. **頻寬**
```
峰值QPS × 平均請求大小 = 入站頻寬
峰值QPS × 平均響應大小 = 出站頻寬

範例:
35K QPS × 1KB請求 = 35MB/s入站
35K QPS × 10KB響應 = 350MB/s出站
```

4. **記憶體 (快取)**
```
20/80原則: 20%的資料產生80%的流量

範例:
總資料 500TB × 20% = 100TB
實際快取熱點資料: 10-100GB (經濟考量)
```

#### 需求記錄模板

在白板/筆記上記錄:
```
功能需求:
- [ ] 功能1
- [ ] 功能2
- [ ] 功能3

非功能需求:
- 可用性: 99.9%
- 一致性: 最終一致性
- 延遲: P99 < 200ms

規模估算:
- DAU: 100M
- 峰值QPS: 35K
- 儲存: 500TB (5年)
- 快取: 50GB
```

---

### A - API Design [5分鐘]

#### API設計原則

**RESTful API範例**:
```
POST   /api/v1/tweets        # 發推文
GET    /api/v1/tweets/{id}   # 獲取推文
DELETE /api/v1/tweets/{id}   # 刪除推文
GET    /api/v1/users/{id}/timeline  # 獲取時間線
POST   /api/v1/users/{id}/follow    # 關注用戶
```

**API設計檢查清單**:
```
✓ RESTful命名 (資源名詞,複數形式)
✓ HTTP動詞正確 (GET/POST/PUT/DELETE)
✓ 版本化 (/v1/)
✓ 錯誤處理定義
✓ 認證方式 (OAuth2, JWT)
✓ 限流策略
```

**請求/響應範例**:
```json
// POST /api/v1/tweets
Request:
{
  "content": "Hello World",
  "media_urls": ["https://..."],
  "visibility": "public"
}

Response: 201 Created
{
  "tweet_id": "1234567890",
  "user_id": "9876543210",
  "content": "Hello World",
  "created_at": "2025-10-31T10:00:00Z",
  "likes_count": 0,
  "retweets_count": 0
}

Error Response: 400 Bad Request
{
  "error": "CONTENT_TOO_LONG",
  "message": "Tweet content exceeds 280 characters",
  "max_length": 280
}
```

---

### D - Data Model [5分鐘]

#### 資料庫選擇決策樹

```
是否需要複雜事務(ACID)?
├─ 是 → SQL
│   ├─ 需要強一致性? → PostgreSQL
│   └─ 高讀取負載? → MySQL + 讀副本
│
└─ 否 → 考慮NoSQL
    ├─ 簡單鍵值存儲? → Redis, DynamoDB
    ├─ 文檔型資料? → MongoDB
    ├─ 時間序列/大量寫入? → Cassandra, InfluxDB
    ├─ 圖關係查詢? → Neo4j
    └─ 搜尋引擎? → Elasticsearch
```

#### Schema設計範例

**SQL (推特範例)**:
```sql
-- 用戶表
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    INDEX idx_username (username),
    INDEX idx_email (email)
);

-- 推文表
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY,
    user_id BIGINT NOT NULL,
    content TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    likes_count INT DEFAULT 0,
    retweets_count INT DEFAULT 0,
    FOREIGN KEY (user_id) REFERENCES users(id),
    INDEX idx_user_created (user_id, created_at DESC)
);

-- 關注關係表
CREATE TABLE relationships (
    follower_id BIGINT NOT NULL,
    following_id BIGINT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (follower_id, following_id),
    INDEX idx_follower (follower_id),
    INDEX idx_following (following_id)
);
```

**NoSQL (MongoDB範例)**:
```javascript
// 用戶文檔
{
  "_id": ObjectId("..."),
  "username": "john_doe",
  "email": "john@example.com",
  "profile": {
    "bio": "Software Engineer",
    "avatar_url": "https://..."
  },
  "following_count": 200,
  "followers_count": 1500,
  "created_at": ISODate("2025-01-01T00:00:00Z")
}

// 推文文檔 (嵌入式設計)
{
  "_id": ObjectId("..."),
  "user": {
    "id": "user123",
    "username": "john_doe",
    "avatar_url": "https://..."  // 反正規化,減少join
  },
  "content": "Hello World",
  "likes_count": 100,
  "created_at": ISODate("..."),
  "comments": [  // 嵌入式評論 (如果數量有限)
    {
      "user_id": "user456",
      "content": "Nice!",
      "created_at": ISODate("...")
    }
  ]
}
```

#### 分片策略

**何時需要分片?**
```
單機限制:
- PostgreSQL: 1-10TB, 10K-50K TPS
- MongoDB: 5-50TB, 100K TPS
- MySQL: 1-5TB, 5K-20K TPS

當達到這些限制的70%時,考慮分片
```

**常見分片鍵**:
```
✓ user_id: 用戶相關資料 (推文、訂單、會話)
✓ timestamp: 時間序列資料 (日誌、指標)
✓ geo_hash: 地理位置資料 (Uber、外送)
✓ hash(entity_id): 均勻分布
```

---

### I - Infrastructure (基礎架構) [15分鐘]

#### 標準架構圖範本

**三層架構**:
```
┌─────────────────────────────────────────┐
│             Client Layer                │
│   (Web Browser / Mobile App)            │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│          Load Balancer Layer            │
│  (L7: Nginx, L4: AWS ALB/NLB)          │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│         Application Layer               │
│  ┌──────┐  ┌──────┐  ┌──────┐          │
│  │ API  │  │ API  │  │ API  │          │
│  │Server│  │Server│  │Server│          │
│  └──┬───┘  └──┬───┘  └──┬───┘          │
└─────┼────────┼────────┼────────────────┘
      │        │        │
┌─────▼────────▼────────▼────────────────┐
│          Cache Layer                    │
│    (Redis Cluster / Memcached)         │
└────────────────┬────────────────────────┘
                 │
┌────────────────▼────────────────────────┐
│        Data Layer                       │
│  ┌─────────────────┐                    │
│  │  Primary DB     │                    │
│  └────────┬────────┘                    │
│           │                             │
│  ┌────────▼────────┬────────────┐      │
│  │   Replica 1     │ Replica 2  │      │
│  └─────────────────┴────────────┘      │
└─────────────────────────────────────────┘
```

#### 組件選型對比

**負載均衡器**:
| 技術 | 類型 | 性能 | 功能 | 適用場景 |
|-----|------|-----|------|---------|
| Nginx | L7 | 10K-50K req/s | 豐富 | API路由 |
| HAProxy | L4/L7 | 100K+ req/s | 中等 | TCP負載 |
| AWS ALB | L7 | 彈性 | 豐富 | AWS環境 |
| AWS NLB | L4 | 極高 | 基本 | 極高性能 |

**快取**:
| 技術 | 類型 | 資料結構 | 持久化 | 適用場景 |
|-----|------|---------|--------|---------|
| Redis | 記憶體 | 豐富 | 支持 | 通用快取 |
| Memcached | 記憶體 | 簡單 | 不支持 | 簡單快取 |
| DynamoDB | 雲端 | K-V | 預設 | AWS生態 |

**資料庫**:
| 技術 | 類型 | 優勢 | 劣勢 | 適用場景 |
|-----|------|-----|------|---------|
| PostgreSQL | SQL | ACID, 功能豐富 | 寫入性能 | 複雜查詢 |
| MySQL | SQL | 成熟, 生態好 | 功能較少 | Web應用 |
| MongoDB | NoSQL | Schema靈活 | 無事務(舊版) | 文檔存儲 |
| Cassandra | NoSQL | 高寫入 | 複雜查詢弱 | 時間序列 |

**訊息佇列**:
| 技術 | 吞吐量 | 延遲 | 順序保證 | 適用場景 |
|-----|--------|------|---------|---------|
| Kafka | 極高 | 中 | 分區內保證 | 大資料管道 |
| RabbitMQ | 中 | 低 | 佇列內保證 | 企業應用 |
| AWS SQS | 高 | 高 | FIFO可選 | AWS生態 |

---

### O - Optimization (優化) [10分鐘]

#### 瓶頸識別檢查清單

**CPU瓶頸**:
```
症狀: CPU使用率>70%
原因: 複雜計算、低效算法
解決:
- 算法優化
- 非同步處理
- 快取計算結果
- 水平擴展
```

**記憶體瓶頸**:
```
症狀: 記憶體不足、OOM、頻繁GC
原因: 資料加載過多、記憶體洩漏
解決:
- 分頁查詢
- 流式處理
- 快取淘汰策略
- 垂直擴展記憶體
```

**資料庫瓶頸**:
```
症狀: 慢查詢、連接池耗盡
原因: 缺少索引、N+1查詢、鎖競爭
解決:
- 添加索引
- 查詢優化
- 讀寫分離
- 連接池調優
- 資料庫分片
```

**網路瓶頸**:
```
症狀: 高延遲、丟包
原因: 跨區域調用、大資料傳輸
解決:
- CDN
- 資料壓縮
- 多區域部署
- HTTP/2、gRPC
```

#### 擴展策略

**10倍流量擴展計畫**:
```
當前: 1K QPS
目標: 10K QPS

第1階段 (2x): 垂直擴展
- 資料庫升級配置
- 增加快取記憶體
- API Server加CPU

第2階段 (5x): 水平擴展 + 優化
- API Server: 3台 → 15台
- 讀副本: 1台 → 5台
- Redis集群化
- 添加CDN

第3階段 (10x): 架構改造
- 資料庫分片
- 微服務拆分
- 非同步處理
- 多區域部署
```

#### 故障處理討論

**常見故障場景**:

1. **資料庫主庫宕機**
```
檢測: 健康檢查失敗 (5秒)
故障轉移:
- Patroni/Orchestrator自動提升從庫為主庫 (30秒)
- 更新DNS或配置中心
- 應用重連
總停機時間: <1分鐘
```

2. **Redis集群節點故障**
```
檢測: Redis Sentinel檢測到節點down
故障轉移:
- Sentinel自動選舉新主節點
- 客戶端重定向
影響: 部分快取未命中,降級到資料庫查詢
```

3. **級聯故障**
```
場景: 資料庫慢 → API超時 → 連接堆積 → 系統崩潰
預防:
- 熔斷器 (Circuit Breaker)
- 限流 (Rate Limiting)
- 超時設置
- 服務降級
```

---

## 💡 面試實戰技巧

### 溝通技巧

**✅ 好的表達**:
```
"我會選擇Redis作為快取,因為它提供豐富的資料結構,
並且支持持久化。不過如果只需要簡單的K-V快取,
Memcached也是一個選項,性能會稍微好一點。
您覺得我們需要快取的資料結構複雜度如何?"
```

**❌ 差的表達**:
```
"用Redis。"
(沒有解釋為什麼,沒有權衡分析)
```

### 畫圖技巧

**架構圖要素**:
```
✓ 清晰的箭頭表示資料流方向
✓ 標註QPS/延遲等關鍵數字
✓ 用顏色區分不同層級
✓ 標註關鍵組件的數量 (3台API Server)
```

**範例**:
```
       10K QPS
User ─────────→ [LB] ──┬── 3K QPS → [API-1]
                       ├── 3K QPS → [API-2]
                       └── 4K QPS → [API-3]
                              │
                       1-5ms  ↓
                         [Redis] (95% hit rate)
                              │
                      5% miss ↓ 10-20ms
                         [PostgreSQL]
```

### 時間管理

**如果時間不夠**:
```
優先順序:
1. 需求澄清 (必須完成)
2. 高層架構圖 (必須完成)
3. 核心組件設計 (至少1-2個)
4. 容量估算 (如果還沒做)
5. 深入討論 (看時間)
```

**如果時間太多**:
```
主動提出:
- "我們可以深入討論一下XXX組件的設計嗎?"
- "關於監控和告警,我有一些想法..."
- "這個系統在XXX場景下可能會有問題,我們可以討論一下嗎?"
```

### 處理追問

**面試官: "如果資料庫成為瓶頸怎麼辦?"**
```
✅ 好的回答:
"首先我會分析是讀瓶頸還是寫瓶頸。

如果是讀瓶頸:
1. 短期: 增加讀副本,提高快取命中率
2. 中期: 引入搜尋引擎(Elasticsearch)分流查詢
3. 長期: 資料庫分片

如果是寫瓶頸:
1. 短期: 批次寫入,非同步處理
2. 中期: 垂直擴展資料庫配置
3. 長期: 資料庫分片,或考慮NoSQL

您希望我深入討論哪個方案?"

❌ 差的回答:
"加機器"
(沒有分析,沒有層次)
```

---

## 📚 常見題目快速參考

### 設計短網址 (URL Shortener)
**關鍵點**: Base62編碼, 快取策略, 資料庫索引
**時間**: 30-35分鐘
**難度**: ⭐⭐☆☆☆

### 設計限流器 (Rate Limiter)
**關鍵點**: 限流算法(Token Bucket/Sliding Window), Redis實作
**時間**: 25-30分鐘
**難度**: ⭐⭐☆☆☆

### 設計Twitter
**關鍵點**: Fan-out策略, Timeline生成, 快取設計
**時間**: 45-50分鐘
**難度**: ⭐⭐⭐⭐☆

### 設計Instagram
**關鍵點**: 圖片儲存(S3), CDN, Feed生成
**時間**: 45-50分鐘
**難度**: ⭐⭐⭐⭐☆

### 設計Uber
**關鍵點**: 地理位置索引(Geohash), 配對算法, WebSocket
**時間**: 45-50分鐘
**難度**: ⭐⭐⭐⭐☆

### 設計YouTube
**關鍵點**: 影片處理, CDN, 推薦系統
**時間**: 50-60分鐘
**難度**: ⭐⭐⭐⭐⭐

### 設計Google Docs
**關鍵點**: OT/CRDT, WebSocket, 衝突解決
**時間**: 50-60分鐘
**難度**: ⭐⭐⭐⭐⭐

---

## 🎯 面試評分標準 (面試官視角)

### Hire (錄取)
```
✓ 清楚澄清需求,問對問題
✓ 設計合理,考慮權衡
✓ 溝通清晰,思路連貫
✓ 主動識別瓶頸並提出優化
✓ 深入理解至少2-3個核心組件
```

### No Hire (不錄取)
```
✗ 不澄清需求直接設計
✗ 設計不合理,無法擴展
✗ 溝通混亂,邏輯跳躍
✗ 對組件只有表面理解
✗ 無法處理追問
```

### Strong Hire (強烈推薦)
```
✓ Hire的所有條件
✓ 主動提出多種方案並對比
✓ 深入討論故障處理、監控
✓ 考慮安全性、成本優化
✓ 展現實際系統設計經驗
```

---

## 📖 推薦學習資源

### 必讀書籍
1. **Designing Data-Intensive Applications** - Martin Kleppmann
   - 深入理解分散式系統原理
2. **System Design Interview Vol 1 & 2** - Alex Xu
   - 面試案例詳解
3. **Clean Architecture** - Robert C. Martin
   - 架構設計原則

### 線上課程
1. **Grokking the System Design Interview** (Educative.io)
2. **System Design Primer** (GitHub開源)
3. **ByteByteGo Newsletter** - Alex Xu

### 技術博客
- **Netflix Tech Blog**: https://netflixtechblog.com/
- **Uber Engineering**: https://eng.uber.com/
- **Meta Engineering**: https://engineering.fb.com/
- **AWS Architecture Blog**: https://aws.amazon.com/blogs/architecture/

### YouTube頻道
- **Gaurav Sen** - System Design
- **Tech Dummies** - System Design
- **Hussein Nasser** - Backend Engineering

---

## ✅ 面試前檢查清單

### 1週前
- [ ] 完成至少10個案例練習
- [ ] 熟悉RADIO框架
- [ ] 複習核心組件 (快取、資料庫、MQ等)

### 1天前
- [ ] 複習容量估算公式
- [ ] 準備白板筆和紙
- [ ] 測試視訊會議工具 (遠程面試)

### 面試當天
- [ ] 提前10分鐘登入/到達
- [ ] 準備好筆記本記錄需求
- [ ] 深呼吸,保持冷靜
- [ ] 大聲思考,主動溝通

---

## 🚀 祝你面試成功!

記住這三點:
1. **溝通 > 設計**: 面試評估的是你的思考過程,不是記憶架構圖
2. **權衡 > 完美**: 沒有完美的設計,只有合適的權衡
3. **練習 > 理論**: 多做模擬面試,熟能生巧

**Good luck! 💪**
