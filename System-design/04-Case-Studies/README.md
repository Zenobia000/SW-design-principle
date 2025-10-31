# Module 4: 經典系統設計案例 (Real-World Case Studies)

> FANG面試真題實戰 - 從需求到架構的完整演練

## 學習目標

本模組以FANG公司真實面試題目為基礎,通過完整的系統設計案例,訓練你在45分鐘內從需求澄清到架構設計的完整流程。

## 面試準備重點

在FANG面試中,系統設計題目評估的不是你記住了多少架構圖,而是:
- ✅ **需求澄清能力**: 問對問題
- ✅ **權衡決策能力**: 解釋為什麼這樣設計
- ✅ **溝通能力**: 清晰表達思路
- ✅ **深度挖掘能力**: 處理面試官的追問

## RADIO框架 - 系統設計面試方法論

每個案例都遵循這個框架:

### R - Requirements (需求澄清) [5分鐘]
```
功能性需求:
- 核心功能有哪些?
- 用戶規模?
- 特殊需求?

非功能性需求:
- QPS多少?
- 資料量多大?
- 可用性要求? (99.9% vs 99.99%)
- 一致性要求?
- 延遲要求? (P99 < 200ms?)
```

### A - API Design (API設計) [5-8分鐘]
```
定義核心API:
- RESTful or GraphQL?
- 請求/響應格式
- 錯誤處理

範例:
POST /api/v1/tweets
GET /api/v1/users/{id}/timeline
```

### D - Data Model (資料模型) [5-8分鐘]
```
資料庫選擇:
- SQL or NoSQL?
- 為什麼?

Schema設計:
- 核心實體
- 關係
- 索引策略
```

### I - Infrastructure (基礎架構) [15-20分鐘]
```
高層架構:
- 畫出系統架構圖
- 解釋資料流

組件選擇:
- 為什麼用這個組件?
- 有什麼替代方案?
```

### O - Optimization (優化與深入) [10-12分鐘]
```
瓶頸分析:
- 系統的瓶頸在哪?
- 如何擴展?

深入討論:
- 面試官感興趣的部分
- 故障處理
- 監控告警
```

---

## 📚 案例分類

### 🔷 Level 1: 基礎案例 (適合中階工程師)
- [設計短網址服務 (URL Shortener)](#案例1-設計短網址服務)
- [設計限流器 (Rate Limiter)](#案例2-設計限流器)
- [設計鍵值存儲 (Key-Value Store)](#案例3-設計鍵值存儲)

### 🔶 Level 2: 中級案例 (適合資深工程師)
- [設計Twitter/微博](#案例4-設計twitter)
- [設計Instagram](#案例5-設計instagram)
- [設計Uber後端](#案例6-設計uber後端)

### 🔴 Level 3: 高級案例 (適合Staff+工程師)
- [設計YouTube視頻平台](#案例7-設計youtube)
- [設計Google Docs協作編輯](#案例8-設計google-docs)
- [設計分散式交易系統](#案例9-設計分散式交易系統)

---

## 案例1: 設計短網址服務

> **難度**: ⭐⭐☆☆☆ (基礎)
> **面試公司**: Google, Amazon, Uber, Airbnb
> **預計時間**: 35-45分鐘

### 完整解答: [url-shortener.md](./url-shortener.md)

### R - Requirements

**面試對話示範**:
```
候選人: "我想先確認一下功能需求。這個短網址服務需要支持哪些功能?"

面試官: "主要兩個功能:生成短網址和重定向。"

候選人: "好的。關於規模,預期有多少用戶?"
面試官: "假設每月100M新URL,讀寫比100:1。"

候選人: "了解。對於可用性,我們目標是多少?"
面試官: "99.9%即可。"

候選人: "短網址的過期策略呢?"
面試官: "假設永不過期,但可以預留這個擴展點。"
```

**需求總結**:

**功能性需求**:
1. 給定長URL,生成短URL
2. 給定短URL,重定向到原始URL
3. (可選) 自訂短碼
4. (可選) 過期時間
5. (可選) 訪問統計

**非功能性需求**:
```
寫入: 100M URL/月 = 100M / (30*24*60*60) ≈ 40 URL/秒
讀取: 100:1 比例 = 4000 重定向/秒
峰值: 3倍 = 120寫入/秒, 12000讀取/秒

儲存: 100M URL/月 * 12月 * 5年 = 6B URL
     每條 1KB (URL + metadata) = 6TB

可用性: 99.9% (重定向不能down)
延遲: P99 < 100ms
```

### A - API Design

```python
# API 1: 生成短網址
POST /api/v1/shorten
Request:
{
  "long_url": "https://example.com/very/long/url",
  "custom_alias": "my-link",  // 可選
  "expire_at": "2025-12-31T23:59:59Z"  // 可選
}

Response:
{
  "short_url": "https://short.ly/abc123",
  "long_url": "https://example.com/very/long/url",
  "created_at": "2025-10-31T10:00:00Z"
}

Error Codes:
- 400: Invalid URL
- 409: Custom alias already exists
- 429: Rate limit exceeded

# API 2: 重定向
GET /{short_code}
Response: 301/302 Redirect to long_url

# API 3: 統計 (可選)
GET /api/v1/stats/{short_code}
Response:
{
  "short_code": "abc123",
  "total_clicks": 12345,
  "unique_clicks": 5678,
  "referrers": {...}
}
```

**API設計要點**:
- ✅ RESTful風格
- ✅ 版本化 (/v1/)
- ✅ 明確的錯誤碼
- ✅ 301 vs 302: 301可快取,302不可快取

### D - Data Model

**資料庫選擇**: SQL (PostgreSQL)

**理由**:
- ✅ 資料結構簡單且固定
- ✅ ACID事務保證短碼唯一性
- ✅ 關聯式查詢 (統計功能)
- ✅ 成熟的備份和複製方案

**Schema設計**:

```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(7) UNIQUE NOT NULL,  -- 索引
    long_url TEXT NOT NULL,
    user_id BIGINT,  -- 外鍵
    created_at TIMESTAMP DEFAULT NOW(),
    expire_at TIMESTAMP,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);

CREATE TABLE url_stats (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(7) NOT NULL,
    clicked_at TIMESTAMP DEFAULT NOW(),
    referrer TEXT,
    user_agent TEXT,
    ip_address INET,
    INDEX idx_short_code (short_code),
    INDEX idx_clicked_at (clicked_at)
);
```

**短碼生成策略**:

**方案1: 雜湊 + 碰撞處理**
```python
import hashlib

def generate_short_code(long_url):
    hash_value = hashlib.md5(long_url.encode()).hexdigest()
    short_code = hash_value[:7]  # 取前7位

    # 碰撞處理
    suffix = 0
    while db.exists(short_code):
        short_code = hash_value[:6] + str(suffix)
        suffix += 1

    return short_code
```
- 缺點: 相同URL產生相同短碼 (可能是優點?)
- 缺點: 碰撞時需要多次DB查詢

**方案2: Base62編碼 + 自增ID** (推薦)
```python
BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

def base62_encode(num):
    if num == 0:
        return BASE62[0]

    result = []
    while num:
        result.append(BASE62[num % 62])
        num //= 62

    return ''.join(reversed(result))

# 使用:
# id=1 → 短碼 "1"
# id=62 → 短碼 "10"
# id=916132832 → 短碼 "Abc123"

def shorten_url(long_url):
    # 1. 插入資料庫獲取自增ID
    id = db.insert("INSERT INTO urls (long_url) VALUES (?)", long_url)

    # 2. Base62編碼
    short_code = base62_encode(id)

    # 3. 更新短碼
    db.update("UPDATE urls SET short_code = ? WHERE id = ?", short_code, id)

    return f"https://short.ly/{short_code}"
```

**容量計算**:
```
短碼長度7位,Base62編碼:
62^7 = 3.5萬億 (夠用)

如果每秒1000寫入:
3.5T / (1000 * 86400 * 365) ≈ 110年
```

**方案3: 預生成ID池** (高性能方案)
```python
# ZooKeeper維護ID範圍
# Server1: 1-1000000
# Server2: 1000001-2000000

class IDGenerator:
    def __init__(self):
        self.current_id = self.fetch_id_range_from_zk()
        self.max_id = self.current_id + 1000000

    def get_next_id(self):
        if self.current_id >= self.max_id:
            # 獲取新的ID範圍
            self.current_id = self.fetch_id_range_from_zk()
            self.max_id = self.current_id + 1000000

        self.current_id += 1
        return self.current_id
```

### I - Infrastructure

**高層架構**:

```
                      ┌─────────────┐
                      │  Client     │
                      └──────┬──────┘
                             │
                    ┌────────▼────────┐
                    │   DNS / CDN     │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  Load Balancer  │
                    │   (Nginx L7)    │
                    └────────┬────────┘
                             │
            ┌────────────────┼────────────────┐
            │                │                │
      ┌─────▼─────┐    ┌────▼────┐    ┌─────▼─────┐
      │   API     │    │  API    │    │   API     │
      │ Server 1  │    │ Server 2│    │ Server 3  │
      └─────┬─────┘    └────┬────┘    └─────┬─────┘
            │               │               │
            └───────┬───────┴───────┬───────┘
                    │               │
           ┌────────▼──────┐   ┌────▼──────────┐
           │  Redis Cluster│   │  PostgreSQL   │
           │   (Cache)     │   │   (Primary)   │
           └───────────────┘   └────┬──────────┘
                                    │
                         ┌──────────┼──────────┐
                    ┌────▼────┐ ┌───▼────┐ ┌───▼────┐
                    │Replica 1│ │Replica2│ │Replica3│
                    └─────────┘ └────────┘ └────────┘
```

**組件說明**:

**1. CDN / DNS**
- **作用**: 地理分流,就近訪問
- **技術**: Cloudflare, AWS CloudFront
- **優化**: 將301重定向快取在CDN

**2. Load Balancer**
- **類型**: L7 (Nginx)
- **算法**: Least Connections
- **健康檢查**: HTTP /health每5秒

**3. API Servers**
- **無狀態**: 可水平擴展
- **語言**: Go (高併發) 或 Python (快速開發)
- **部署**: Kubernetes Pods

**4. Redis Cluster**
- **模式**: Cache-Aside
- **資料**: 短碼 → 長URL映射
- **TTL**: 熱點永不過期,冷資料1天
- **容量**: 10GB (可存1000萬熱點URL)

**5. PostgreSQL**
- **Primary**: 處理寫入
- **Replicas**: 3個,處理讀取
- **連接池**: PgBouncer
- **分片**: 未來按短碼雜湊分片

**資料流**:

**寫入流程 (生成短網址)**:
```
1. Client → Load Balancer
2. LB → API Server
3. API Server → PostgreSQL (插入,獲取ID)
4. API Server → Base62編碼
5. API Server → PostgreSQL (更新短碼)
6. API Server → 返回短網址
   (不寫Redis,lazy loading)
```

**讀取流程 (重定向)**:
```
1. Client → CDN (301快取,直接返回)
   ↓ (未命中)
2. CDN → Load Balancer → API Server
3. API Server → Redis (查短碼)
   ↓ (未命中)
4. API Server → PostgreSQL Replica
5. API Server → Redis (寫入快取)
6. API Server → 301重定向
7. CDN → 快取301響應
```

### O - Optimization

**瓶頸分析**:

**瓶頸1: 資料庫寫入**
- **現狀**: 單機PostgreSQL, 1000 TPS極限
- **解決**:
  - 短期: 優化索引,使用SSD
  - 長期: 分片 (按短碼雜湊)

**瓶頸2: 資料庫讀取**
- **現狀**: 12000 QPS峰值
- **解決**:
  - Redis快取命中率>95%
  - 讀副本分流
  - CDN快取301響應

**瓶頸3: 短碼生成衝突**
- **問題**: 高併發下自增ID鎖競爭
- **解決**: 預生成ID池 (ZooKeeper/etcd)

**擴展策略**:

**10倍流量 (1200寫/秒, 120K讀/秒)**:
```
1. API Server: 3台 → 30台 (Kubernetes HPA)
2. Redis: 單機 → 集群 (10個節點,分片)
3. PostgreSQL:
   - 主庫: 垂直擴展 (更多vCPU)
   - 讀副本: 3台 → 10台
4. CDN: 提高快取命中率到99%
```

**100倍流量** (需要重新架構):
```
1. 資料庫分片 (16 shards)
2. 多區域部署 (US, EU, APAC)
3. NoSQL考慮 (Cassandra寫入更快)
```

**深入討論**:

**Q: 如何防止濫用?**
```
1. Rate Limiting: 每IP每分鐘10個短網址
2. API Key認證: 需要註冊才能使用
3. CAPTCHA: 可疑行為要求驗證
```

**Q: 如何處理短碼衝突?**
```
Base62方案: 不會衝突 (基於自增ID)
雜湊方案: 重新雜湊或加後綴
```

**Q: 如何刪除短網址?**
```
DELETE /api/v1/{short_code}

軟刪除:
UPDATE urls SET deleted_at = NOW() WHERE short_code = ?
查詢時過濾: WHERE deleted_at IS NULL
```

**Q: 如何實現分析功能?**
```
方案1: 同步寫DB (影響性能)
方案2: 非同步寫 (Kafka → Flink → ClickHouse)
方案3: 日誌收集 (Nginx access log → ELK)
```

**Q: 如何保證高可用?**
```
1. 多可用區部署 (Multi-AZ)
2. 資料庫主從自動故障轉移 (Patroni)
3. Redis Sentinel (自動failover)
4. 監控告警 (Prometheus + Grafana)
```

---

## 案例2: 設計限流器

> **難度**: ⭐⭐☆☆☆ (基礎)
> **面試公司**: Amazon, Microsoft, Stripe
> **預計時間**: 30-40分鐘
> **核心考點**: 限流算法、分散式限流、Redis應用

### 完整解答: [rate-limiter.md](./rate-limiter.md)

### 快速概覽

**需求**: 限制API調用頻率 (如: 每用戶每分鐘100次請求)

**核心算法對比**:

| 算法 | 優點 | 缺點 | 適用場景 |
|-----|------|------|---------|
| **固定窗口** | 簡單 | 邊界突刺 | 簡單限流 |
| **滑動窗口** | 平滑 | 記憶體高 | 精確限流 |
| **令牌桶** | 允許突發 | 複雜 | API Gateway |
| **漏桶** | 平滑流量 | 無法突發 | 訊息佇列 |

**架構**:
```
API Request → Rate Limiter (Redis) → API Server
                  ↓ (超限)
              429 Too Many Requests
```

**Redis實作 (滑動窗口)**:
```python
import redis
import time

class RateLimiter:
    def __init__(self, redis_client, max_requests, window_seconds):
        self.redis = redis_client
        self.max_requests = max_requests
        self.window_seconds = window_seconds

    def is_allowed(self, user_id):
        key = f"rate_limit:{user_id}"
        now = time.time()
        window_start = now - self.window_seconds

        # 使用Redis Sorted Set
        pipe = self.redis.pipeline()

        # 1. 移除窗口外的請求
        pipe.zremrangebyscore(key, 0, window_start)

        # 2. 統計當前窗口請求數
        pipe.zcard(key)

        # 3. 添加當前請求
        pipe.zadd(key, {now: now})

        # 4. 設置過期時間
        pipe.expire(key, self.window_seconds + 1)

        results = pipe.execute()
        request_count = results[1]

        return request_count < self.max_requests

# 使用
limiter = RateLimiter(redis.Redis(), max_requests=100, window_seconds=60)
if limiter.is_allowed("user123"):
    # 處理請求
    pass
else:
    # 返回429
    return {"error": "Rate limit exceeded"}, 429
```

---

## 案例4: 設計Twitter

> **難度**: ⭐⭐⭐⭐☆ (中高級)
> **面試公司**: Google, Facebook, Twitter, Amazon
> **預計時間**: 45-60分鐘
> **核心考點**: Timeline生成、Fan-out策略、快取設計

### 完整解答: [twitter.md](./twitter.md)

### 快速概覽

**核心功能**:
1. 發推文
2. 關注/取消關注
3. Timeline (首頁時間線)

**規模估算**:
```
DAU: 200M
平均關注: 200人
發推頻率: 平均每天2條
讀寫比: 100:1

寫入: 200M * 2 / 86400 = 4.6K 推文/秒
讀取: 4.6K * 100 = 460K Timeline請求/秒
```

**核心挑戰: Timeline生成**

**方案1: Pull Model (讀時生成)**
```python
def get_timeline(user_id):
    # 查詢所有關注的人
    following = db.query(
        "SELECT following_id FROM relationships WHERE follower_id = ?",
        user_id
    )

    # 查詢每個人的推文,合併排序
    tweets = db.query(
        "SELECT * FROM tweets WHERE user_id IN (?) ORDER BY created_at DESC LIMIT 100",
        following
    )
    return tweets
```
- **優點**: 發推文快 (只寫入自己的表)
- **缺點**: 讀取慢 (需要join多個用戶的推文)
- **適用**: 關注數少的用戶

**方案2: Push Model (寫時生成, Fan-out)**
```python
def post_tweet(user_id, content):
    # 1. 保存推文
    tweet = db.insert("INSERT INTO tweets ...", user_id, content)

    # 2. 獲取所有粉絲
    followers = db.query(
        "SELECT follower_id FROM relationships WHERE following_id = ?",
        user_id
    )

    # 3. 推送到每個粉絲的Timeline
    for follower_id in followers:
        redis.lpush(f"timeline:{follower_id}", tweet.id)
        redis.ltrim(f"timeline:{follower_id}", 0, 999)  # 保留最新1000條
```
- **優點**: 讀取極快 (Redis預先計算)
- **缺點**: 寫入慢 (大V發一條推文需推送給百萬粉絲)
- **適用**: 粉絲數適中的用戶

**方案3: 混合模型 (Twitter實際方案)**
```python
def post_tweet(user_id, content):
    tweet = db.insert(...)

    # 判斷是否大V
    if is_celebrity(user_id):
        # 大V不fan-out,粉絲讀時生成
        cache.set(f"celebrity_tweet:{user_id}", tweet.id)
    else:
        # 普通用戶fan-out到粉絲Timeline
        fan_out_to_followers(user_id, tweet)

def get_timeline(user_id):
    # 1. 從Redis獲取預生成Timeline
    timeline = redis.lrange(f"timeline:{user_id}", 0, 99)

    # 2. 獲取關注的大V最新推文
    celebrities = get_following_celebrities(user_id)
    celebrity_tweets = get_latest_tweets(celebrities)

    # 3. 合併排序
    return merge_sort_by_time(timeline, celebrity_tweets)
```

**架構圖**:
```
                       寫入流程 (發推文)
User → API → [推文服務] → PostgreSQL (推文表)
                ↓
         [Fan-out服務] → Redis (Timeline快取)
                ↓
            Kafka → [非同步處理]


                      讀取流程 (Timeline)
User → API → [Timeline服務] → Redis (快取)
                ↓ (未命中)
          PostgreSQL → 生成Timeline → 寫入Redis
```

---

## 案例5: 設計Instagram

> **難度**: ⭐⭐⭐⭐☆ (中高級)
> **面試公司**: Facebook, Google, Snapchat
> **核心考點**: 圖片儲存、Feed生成、Sharding策略

### 完整解答: [instagram.md](./instagram.md)

**核心差異 vs Twitter**:
1. **媒體處理**: 圖片/影片上傳、壓縮、多尺寸生成
2. **儲存**: 物件儲存 (S3) vs 資料庫
3. **Feed**: 個性化推薦 vs 時間序

**圖片處理流程**:
```
用戶上傳
   ↓
[API Server] → 生成預簽名URL
   ↓
用戶 → S3直接上傳原圖
   ↓
[Lambda觸發] → 生成縮圖 (多尺寸)
   ↓
[存入S3] → 通知API Server
   ↓
[更新資料庫] → 記錄圖片URL
   ↓
[CDN快取] → 加速圖片訪問
```

---

## 案例7: 設計YouTube

> **難度**: ⭐⭐⭐⭐⭐ (高級)
> **面試公司**: Google, Netflix, Amazon
> **核心考點**: 影片編碼、CDN、推薦系統

### 完整解答: [youtube.md](./youtube.md)

**核心挑戰**:
1. **影片上傳**: 斷點續傳、分片上傳
2. **轉碼**: 多解析度 (1080p, 720p, 480p...)
3. **分發**: CDN、自適應串流 (HLS/DASH)
4. **推薦**: 個性化推薦算法

**影片處理管道**:
```
上傳 → [分片上傳服務] → S3原始影片
         ↓
    [SQS佇列]
         ↓
    [轉碼集群] → FFmpeg並行轉碼
         ↓
    S3多解析度影片
         ↓
    [生成HLS切片] → m3u8播放列表
         ↓
    CloudFront CDN
         ↓
    用戶播放器 (自適應碼率)
```

---

## 案例8: 設計Google Docs

> **難度**: ⭐⭐⭐⭐⭐ (高級)
> **面試公司**: Google, Microsoft, Dropbox
> **核心考點**: 協作編輯、衝突解決、OT/CRDT

### 完整解答: [google-docs.md](./google-docs.md)

**核心挑戰: 實時協作**

**問題**: 多人同時編輯同一文檔,如何保證一致性?

**方案1: Operational Transformation (OT)**
```python
# 用戶A: 在位置0插入"H"
op_a = Insert(0, "H")

# 用戶B: 在位置0插入"W" (同時發生)
op_b = Insert(0, "W")

# 衝突解決: 轉換操作
# Server收到op_a: 文檔="H"
# Server收到op_b: 需要轉換 → Insert(1, "W")
# 最終文檔: "HW" (兩個客戶端一致)
```

**方案2: CRDT (Conflict-free Replicated Data Types)**
- 無中心化協作
- 保證最終一致性
- Figma、Notion使用

**架構**:
```
用戶瀏覽器
   ↓ WebSocket
[協作服務器] → Redis Pub/Sub
   ↓
[OT引擎] → 操作轉換
   ↓
[資料庫] → 持久化
   ↓
[版本歷史]
```

---

## 📝 面試實戰Tips

### 時間分配建議 (45分鐘面試)

```
0-5分鐘: 需求澄清 (提問、記錄)
5-13分鐘: API + 資料模型 (快速定義)
13-33分鐘: 架構設計 (畫圖、解釋)
33-42分鐘: 深入討論 (瓶頸、優化)
42-45分鐘: 總結、回答問題
```

### 常見錯誤

❌ **直接開始畫架構圖** → 先問清楚需求!
❌ **過度設計** → 從簡單開始,逐步優化
❌ **不說話** → 大聲思考,讓面試官跟上你的思路
❌ **固執己見** → 聽取面試官提示,靈活調整
❌ **忽略數字** → 容量估算很重要!

### 加分項

✅ 主動提出多種方案並對比權衡
✅ 考慮故障處理和監控
✅ 討論安全性 (認證、授權、加密)
✅ 提到實際技術棧 (Kubernetes, Kafka, Redis...)
✅ 畫清晰的架構圖

---

## 🎓 練習建議

### Week 1-2: 基礎案例
- 每天1個案例,從頭到尾獨立完成
- 限時45分鐘
- 錄音回放,檢討思路

### Week 3-4: 中級案例
- 每天1個,重點練習Fan-out、分片
- 模擬面試 (找朋友當面試官)

### Week 5-6: 高級案例
- 每2天1個,深入研究技術細節
- 閱讀實際系統的技術博客

### 持續學習
- 訂閱 [ByteByteGo Newsletter](https://blog.bytebytego.com/)
- 閱讀各大公司技術博客
- 參與開源項目

---

**上一模組**: [02-Core-Components](../02-Core-Components/README.md)
**下一模組**: [05-Advanced-Topics](../05-Advanced-Topics/README.md)
