# Module 2: 核心組件設計 (Core Components)

> 掌握系統設計的構建塊 - 打造可擴展系統的武器庫

## 學習目標

本模組深入探討構建大規模分散式系統的核心組件。這些是FANG面試中最常被問到的技術棧,也是實際生產環境中的關鍵構建塊。

## 模組重要性

在系統設計面試中,面試官會評估你對各組件的理解深度:
- ❌ 表面: "加個Redis快取就好了"
- ✅ 深入: "我們用Redis作為cache-aside模式,需要處理快取穿透、雪崩和一致性問題"

**關鍵**: 理解**何時用**、**為何用**、**如何用**、**有何代價**

---

## 📚 課程內容

### 1. 負載均衡 (Load Balancing)

#### 1.1 負載均衡器基礎
- [負載均衡原理](./01-load-balancer-basics.ipynb)
- [實作簡易負載均衡器](./01-load-balancer-implementation.ipynb)

**核心概念**:
負載均衡器是分散流量到多台伺服器的關鍵組件,提升系統的可用性、可擴展性和容錯能力。

**負載均衡層級**:

```
用戶請求
    ↓
[DNS負載均衡] ← 地理位置分流
    ↓
[L4負載均衡器] ← 傳輸層 (TCP/UDP)
    ↓
[L7負載均衡器] ← 應用層 (HTTP/HTTPS)
    ↓
應用伺服器
```

#### 1.2 L4 vs L7 負載均衡

**L4負載均衡 (傳輸層)**:
- 基於IP、Port做路由決策
- 不解析應用層協議
- **性能**: 極快 (僅檢查TCP/UDP header)
- **功能**: 有限

```
優點:
✓ 高性能
✓ 協議無關
✓ 簡單穩定

缺點:
✗ 無法基於內容路由
✗ 不支持HTTP級別功能
✗ 無法做智能路由
```

**L7負載均衡 (應用層)**:
- 解析HTTP請求
- 基於URL、Header、Cookie路由
- **性能**: 較慢 (需要解析HTTP)
- **功能**: 豐富

```
優點:
✓ 智能路由 (基於URL路徑)
✓ SSL終止
✓ HTTP壓縮
✓ WAF功能

缺點:
✗ 性能開銷
✗ 協議綁定 (只能HTTP)
✗ 複雜度高
```

**選擇決策**:
```python
def choose_load_balancer(requirements):
    if requirements.need_content_based_routing:
        return "L7負載均衡器"  # 微服務、API Gateway
    elif requirements.need_high_performance:
        return "L4負載均衡器"  # 遊戲、即時通訊
    else:
        return "混合使用"  # L4處理流量,L7處理路由
```

#### 1.3 負載均衡算法

**1. Round-Robin (輪詢)**
```python
class RoundRobinLB:
    def __init__(self, servers):
        self.servers = servers
        self.current = 0

    def get_server(self):
        server = self.servers[self.current]
        self.current = (self.current + 1) % len(self.servers)
        return server
```
- 優點: 簡單、公平
- 缺點: 不考慮伺服器負載差異
- 適用: 伺服器性能均等

**2. Least Connections (最少連接)**
```python
class LeastConnectionLB:
    def __init__(self, servers):
        self.servers = {s: 0 for s in servers}

    def get_server(self):
        return min(self.servers, key=self.servers.get)

    def on_request(self, server):
        self.servers[server] += 1

    def on_complete(self, server):
        self.servers[server] -= 1
```
- 優點: 考慮實際負載
- 缺點: 需要維護連接計數
- 適用: 長連接、WebSocket

**3. Weighted Round-Robin (加權輪詢)**
```python
# 伺服器權重: server1=5, server2=3, server3=2
# 選擇順序: s1, s1, s2, s1, s3, s1, s2, s1, s3, s2
```
- 優點: 適應異構伺服器
- 適用: 伺服器性能差異大

**4. IP Hash**
```python
class IPHashLB:
    def __init__(self, servers):
        self.servers = servers

    def get_server(self, client_ip):
        hash_value = hash(client_ip)
        index = hash_value % len(self.servers)
        return self.servers[index]
```
- 優點: 同一客戶端固定到同一伺服器
- 適用: 有狀態應用、Session親和性

**5. Consistent Hashing (一致性雜湊)**
- [一致性雜湊詳解](./01-consistent-hashing.ipynb)
- 優點: 添加/移除節點時最小化資料遷移
- 適用: 快取、分散式儲存

#### 1.4 健康檢查機制

**主動健康檢查**:
```python
class HealthChecker:
    def __init__(self, servers, check_interval=5):
        self.servers = servers
        self.healthy_servers = set(servers)
        self.check_interval = check_interval

    async def health_check_loop(self):
        while True:
            for server in self.servers:
                try:
                    response = await ping(server)
                    if response.status == 200:
                        self.healthy_servers.add(server)
                    else:
                        self.healthy_servers.discard(server)
                except Exception:
                    self.healthy_servers.discard(server)

            await asyncio.sleep(self.check_interval)
```

**被動健康檢查**:
- 監控實際請求的成功率
- 連續失敗N次後標記為不健康
- 更快發現問題,但可能誤判

**FANG實踐**:
- Netflix Eureka: 心跳機制 (30秒)
- AWS ELB: 主動健康檢查 + 被動監控
- Google Cloud LB: 連續3次失敗標記不健康

---

### 2. 快取系統 (Caching)

#### 2.1 快取基礎與策略
- [快取原理](./02-caching-fundamentals.ipynb)
- [快取模式對比](./02-cache-patterns.ipynb)

**為什麼需要快取?**
```
資料庫查詢: 10-50ms
快取讀取: 1-5ms
性能提升: 10-50倍
```

**快取模式**:

**1. Cache-Aside (旁路快取)**
```python
def get_user(user_id):
    # 1. 先查快取
    user = cache.get(f"user:{user_id}")
    if user:
        return user

    # 2. 快取未命中,查資料庫
    user = db.query("SELECT * FROM users WHERE id = ?", user_id)

    # 3. 寫入快取
    cache.set(f"user:{user_id}", user, ttl=3600)
    return user

def update_user(user_id, data):
    # 更新資料庫
    db.update("UPDATE users SET ... WHERE id = ?", user_id, data)
    # 刪除快取 (lazy loading)
    cache.delete(f"user:{user_id}")
```
- **優點**: 應用掌控快取邏輯
- **缺點**: 快取和資料庫不一致窗口
- **適用**: 讀多寫少場景 (用戶資料、商品資訊)

**2. Read-Through (讀穿透)**
```python
# 快取層負責讀取資料庫
user = cache.get(user_id)  # 快取未命中時自動讀DB並快取
```
- 應用層不感知資料來源
- 快取層封裝資料存取邏輯

**3. Write-Through (寫穿透)**
```python
def update_user(user_id, data):
    # 同時寫入快取和資料庫
    cache.set(user_id, data)  # 快取層負責寫入DB
```
- **優點**: 快取和DB始終一致
- **缺點**: 寫入延遲高 (需等待DB確認)
- **適用**: 一致性要求高的場景

**4. Write-Back/Write-Behind (寫回)**
```python
def update_user(user_id, data):
    # 只寫快取,非同步寫DB
    cache.set(user_id, data)
    queue.enqueue(WriteTask(user_id, data))
```
- **優點**: 寫入性能極高
- **缺點**: 快取故障可能丟失資料
- **適用**: 寫入密集、可接受部分資料丟失 (計數器、瀏覽記錄)

#### 2.2 快取失效策略 (Eviction Policies)

**1. LRU (Least Recently Used)**
```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity):
        self.cache = OrderedDict()
        self.capacity = capacity

    def get(self, key):
        if key not in self.cache:
            return None
        # 移動到最後 (最近使用)
        self.cache.move_to_end(key)
        return self.cache[key]

    def put(self, key, value):
        if key in self.cache:
            self.cache.move_to_end(key)
        self.cache[key] = value
        if len(self.cache) > self.capacity:
            # 刪除最久未使用
            self.cache.popitem(last=False)
```
- **適用**: 存取模式有時間局部性
- **Redis實作**: `maxmemory-policy allkeys-lru`

**2. LFU (Least Frequently Used)**
- 淘汰使用頻率最低的項目
- **適用**: 熱點資料穩定的場景

**3. FIFO (First In First Out)**
- 最簡單,最先進入的先淘汰
- **適用**: 對淘汰策略不敏感的場景

**4. TTL (Time To Live)**
```python
cache.set("session:123", data, ttl=1800)  # 30分鐘過期
```
- 基於時間自動過期
- **適用**: 所有快取場景的基礎

#### 2.3 快取問題與解決方案

**1. 快取穿透 (Cache Penetration)**
- **問題**: 查詢不存在的資料,快取和DB都沒有,導致每次都打DB
- **攻擊**: 惡意查詢大量不存在的key

**解決方案**:
```python
# 方案1: 快取空值
def get_user(user_id):
    user = cache.get(f"user:{user_id}")
    if user == "NULL":  # 快取的空值標記
        return None
    if user:
        return user

    user = db.query(...)
    if user is None:
        cache.set(f"user:{user_id}", "NULL", ttl=60)  # 短TTL
    else:
        cache.set(f"user:{user_id}", user, ttl=3600)
    return user

# 方案2: 布隆過濾器 (Bloom Filter)
from pybloom_live import BloomFilter

bf = BloomFilter(capacity=1000000, error_rate=0.001)
# 初始化時將所有存在的key加入BF
for user_id in db.query("SELECT id FROM users"):
    bf.add(user_id)

def get_user(user_id):
    if user_id not in bf:  # 快速判斷不存在
        return None
    # 後續正常查快取和DB
    ...
```

**2. 快取擊穿 (Cache Breakdown)**
- **問題**: 熱點key過期瞬間,大量請求打到DB
- **場景**: 熱門商品、明星微博

**解決方案**:
```python
import threading

locks = {}

def get_hot_product(product_id):
    data = cache.get(f"product:{product_id}")
    if data:
        return data

    # 獲取分散式鎖,只有一個請求查DB
    lock_key = f"lock:product:{product_id}"
    lock = locks.setdefault(lock_key, threading.Lock())

    with lock:
        # Double-check
        data = cache.get(f"product:{product_id}")
        if data:
            return data

        # 查詢DB
        data = db.query(...)
        cache.set(f"product:{product_id}", data, ttl=3600)
        return data

# 更好的方案: 熱點key永不過期
# 後台非同步更新
def refresh_hot_keys():
    while True:
        for key in HOT_KEYS:
            data = db.query(...)
            cache.set(key, data, ttl=7200)  # 長TTL
        time.sleep(3600)  # 每小時刷新
```

**3. 快取雪崩 (Cache Avalanche)**
- **問題**: 大量key同時過期,DB瞬間壓力巨大
- **場景**: 冷啟動、批量設置相同TTL

**解決方案**:
```python
import random

# 方案1: TTL加隨機值
def set_cache_with_jitter(key, value, base_ttl=3600):
    jitter = random.randint(0, 300)  # 0-5分鐘隨機
    cache.set(key, value, ttl=base_ttl + jitter)

# 方案2: 多級快取
# L1: 本地快取 (無過期)
# L2: Redis快取 (短TTL)
def get_with_multi_level_cache(key):
    # L1
    data = local_cache.get(key)
    if data:
        return data

    # L2
    data = redis.get(key)
    if data:
        local_cache.set(key, data, ttl=60)
        return data

    # DB
    data = db.query(...)
    redis.set(key, data, ttl=600)
    local_cache.set(key, data, ttl=60)
    return data

# 方案3: 限流 + 降級
# 當DB壓力過大時,返回預設值或舊資料
```

#### 2.4 多層快取架構

**典型架構**:
```
客戶端
    ↓
CDN快取 (靜態資源)
    ↓
Nginx本地快取 (頁面快取)
    ↓
應用內存快取 (熱點資料)
    ↓
Redis/Memcached (分散式快取)
    ↓
資料庫查詢快取
    ↓
資料庫
```

**各層特點**:
| 層級 | 容量 | 延遲 | 命中率 | 適用資料 |
|-----|------|------|--------|----------|
| CDN | TB級 | 50ms | 80%+ | 靜態資源 |
| Nginx | GB級 | 1ms | 60% | 頁面片段 |
| 應用記憶體 | MB級 | <1ms | 40% | 熱點資料 |
| Redis | 10-100GB | 1-5ms | 90% | 會話、計數 |
| DB查詢快取 | 取決於DB | 10ms | - | 複雜查詢 |

---

### 3. 資料庫設計 (Database Design)

#### 3.1 SQL vs NoSQL 選擇
- [資料庫選型指南](./03-database-selection.ipynb)

**SQL資料庫 (關聯式)**:
- **代表**: MySQL, PostgreSQL, Oracle
- **特點**: ACID事務、強一致性、JOIN操作
- **適用**:
  - 財務系統 (需要事務)
  - 複雜查詢 (多表JOIN)
  - 資料結構穩定

**NoSQL資料庫**:

**1. 文件型 (Document Store)**
- **代表**: MongoDB, CouchDB
- **特點**: Schema靈活、嵌套文檔
- **適用**: CMS、用戶畫像、產品目錄

**2. 鍵值型 (Key-Value Store)**
- **代表**: Redis, DynamoDB
- **特點**: 極快、簡單
- **適用**: 快取、會話、計數器

**3. 列式 (Column-Family Store)**
- **代表**: Cassandra, HBase
- **特點**: 寫入快、大資料量
- **適用**: 時間序列、日誌、分析

**4. 圖資料庫 (Graph Database)**
- **代表**: Neo4j, Amazon Neptune
- **特點**: 複雜關係查詢
- **適用**: 社交網路、推薦系統

**選擇框架**:
```python
def choose_database(requirements):
    if requirements.needs_complex_transactions:
        return "PostgreSQL"  # 強ACID,複雜查詢
    elif requirements.needs_high_write_throughput:
        return "Cassandra"  # 高寫入,可擴展
    elif requirements.needs_flexible_schema:
        return "MongoDB"  # Schema靈活
    elif requirements.needs_graph_queries:
        return "Neo4j"  # 圖關係
    elif requirements.needs_caching:
        return "Redis"  # 快取,高性能
```

#### 3.2 資料庫索引 (Indexing)

**索引類型**:

**1. B-Tree索引 (預設)**
```sql
CREATE INDEX idx_user_email ON users(email);
```
- 適用: 等值查詢、範圍查詢
- 時間複雜度: O(log n)

**2. 雜湊索引 (Hash Index)**
```sql
CREATE INDEX idx_user_id_hash ON users USING HASH(id);
```
- 適用: 等值查詢 (WHERE id = 123)
- 不支持範圍查詢

**3. 複合索引 (Composite Index)**
```sql
CREATE INDEX idx_user_age_city ON users(age, city);

-- ✓ 可使用索引
SELECT * FROM users WHERE age = 25 AND city = 'Taipei';
SELECT * FROM users WHERE age = 25;  -- 前綴匹配

-- ✗ 無法使用索引 (跳過age)
SELECT * FROM users WHERE city = 'Taipei';
```
- 最左前綴原則

**4. 覆蓋索引 (Covering Index)**
```sql
CREATE INDEX idx_user_email_name ON users(email, name);

-- 無需回表查詢 (所有欄位都在索引中)
SELECT email, name FROM users WHERE email = 'user@example.com';
```

**索引的代價**:
```
優點:
+ 加速查詢 (10-100倍)

缺點:
- 增加儲存空間 (10-30%)
- 拖慢寫入速度 (INSERT/UPDATE/DELETE都需要更新索引)
- 維護成本
```

**FANG最佳實踐**:
1. 為WHERE、JOIN、ORDER BY欄位建索引
2. 選擇性高的欄位優先 (如email,不是gender)
3. 避免過度索引 (每個表<5個索引)
4. 定期分析慢查詢 (slow query log)

#### 3.3 讀寫分離 (Read Replicas)

**架構**:
```
應用
  ↓ (寫)
主資料庫 (Master)
  ↓ (複製)
從資料庫1 (Replica)
從資料庫2 (Replica)
  ↑ (讀)
應用
```

**實作**:
```python
class DatabaseRouter:
    def __init__(self):
        self.master = connect("master-db")
        self.replicas = [
            connect("replica-1"),
            connect("replica-2"),
        ]
        self.replica_index = 0

    def execute_write(self, sql):
        return self.master.execute(sql)

    def execute_read(self, sql):
        # 輪詢選擇從庫
        replica = self.replicas[self.replica_index]
        self.replica_index = (self.replica_index + 1) % len(self.replicas)
        return replica.execute(sql)

# 使用
db = DatabaseRouter()
db.execute_write("INSERT INTO users ...")  # 主庫
users = db.execute_read("SELECT * FROM users")  # 從庫
```

**複製延遲問題**:
```python
# 問題: 讀己之寫不一致
user_id = db.execute_write("INSERT INTO users ... RETURNING id")
user = db.execute_read(f"SELECT * FROM users WHERE id = {user_id}")
# user 可能為 None (從庫未同步)

# 解決方案1: 短期內讀主庫
class SmartRouter:
    def __init__(self):
        self.last_write_time = {}

    def execute_write(self, sql, user_id):
        result = self.master.execute(sql)
        self.last_write_time[user_id] = time.time()
        return result

    def execute_read(self, sql, user_id):
        # 寫入後5秒內讀主庫
        if time.time() - self.last_write_time.get(user_id, 0) < 5:
            return self.master.execute(sql)
        return self.replica.execute(sql)
```

#### 3.4 資料分片 (Sharding)

**水平分片 (Horizontal Sharding)**:
```
users表 (1億記錄)
    ↓
Shard 1: user_id 0-25M
Shard 2: user_id 25M-50M
Shard 3: user_id 50M-75M
Shard 4: user_id 75M-100M
```

**分片鍵選擇**:
```python
# 差的分片鍵: 會導致熱點
shard_id = user_id % 4  # 新用戶都在最後一個shard

# 好的分片鍵: 均勻分布
shard_id = hash(user_id) % 4
```

**查詢路由**:
```python
class ShardedDatabase:
    def __init__(self, shards):
        self.shards = shards

    def get_shard(self, user_id):
        shard_id = hash(user_id) % len(self.shards)
        return self.shards[shard_id]

    def get_user(self, user_id):
        shard = self.get_shard(user_id)
        return shard.query("SELECT * FROM users WHERE id = ?", user_id)

    def get_users_by_city(self, city):
        # 問題: 需要查詢所有shard (scatter-gather)
        results = []
        for shard in self.shards:
            results.extend(shard.query(
                "SELECT * FROM users WHERE city = ?", city
            ))
        return results
```

**Sharding挑戰**:
1. **跨Shard查詢**: 效率低,需要掃描所有shard
2. **跨Shard事務**: 需要2PC,複雜且慢
3. **Rebalancing**: 添加shard時資料遷移
4. **Shard鍵變更**: 幾乎不可能

**FANG實踐**:
- Instagram: 按user_id分片
- Pinterest: 按board_id分片
- Uber: 按地理位置分片

---

### 4. 訊息佇列 (Message Queues)

#### 4.1 訊息佇列基礎
- [訊息佇列原理](./04-message-queue-basics.ipynb)

**為什麼需要訊息佇列?**

1. **非同步處理**: 不阻塞主流程
2. **削峰填谷**: 平滑流量波動
3. **解耦服務**: 服務間鬆耦合
4. **可靠性**: 保證訊息不丟失

**經典場景**:
```python
# 同步處理 (差)
def create_order(order_data):
    order = save_order(order_data)  # 100ms
    send_email(order)  # 2000ms ← 用戶等待
    update_inventory(order)  # 500ms
    return order  # 總共2600ms

# 非同步處理 (好)
def create_order(order_data):
    order = save_order(order_data)  # 100ms
    queue.publish("order.created", order)  # 5ms
    return order  # 總共105ms

# 背景消費者
def order_created_handler(order):
    send_email(order)
    update_inventory(order)
```

#### 4.2 訊息語義 (Delivery Semantics)

**1. At-Most-Once (最多一次)**
- 訊息可能丟失,不會重複
- **實作**: 發送後不確認,不重試
- **適用**: 可容忍丟失 (日誌收集、監控指標)

**2. At-Least-Once (至少一次)**
- 訊息不會丟失,可能重複
- **實作**: 確認機制 + 重試
- **適用**: 大多數場景 (需要冪等處理)

```python
class ReliableQueue:
    def consume(self, handler):
        while True:
            message = self.receive()
            try:
                handler(message)
                self.acknowledge(message)  # 確認成功
            except Exception:
                # 不確認,訊息會重新投遞
                pass
```

**3. Exactly-Once (精確一次)**
- 訊息不丟失,不重複
- **實作**: 複雜 (分散式事務、去重機制)
- **適用**: 金融交易、支付

```python
# 冪等處理 (實現Exactly-Once語義)
def process_payment(message):
    payment_id = message['payment_id']

    # 去重檢查
    if redis.exists(f"processed:{payment_id}"):
        return  # 已處理過

    # 處理支付
    process_payment_logic(message)

    # 標記已處理
    redis.set(f"processed:{payment_id}", 1, ex=86400)  # 24小時
```

#### 4.3 背壓處理 (Backpressure)

**問題**: 生產者速度 > 消費者速度

**解決方案**:

**1. 佇列長度限制**
```python
class BoundedQueue:
    def __init__(self, max_size=1000):
        self.queue = []
        self.max_size = max_size

    def publish(self, message):
        if len(self.queue) >= self.max_size:
            raise QueueFullError("Queue is full")
        self.queue.append(message)
```

**2. 動態調整消費者數量**
```python
# Kubernetes HPA based on queue length
if queue_length > 10000:
    scale_consumers(count=20)
elif queue_length < 1000:
    scale_consumers(count=5)
```

**3. 限流 (Rate Limiting)**
```python
from ratelimit import limits

@limits(calls=100, period=1)  # 每秒最多100個訊息
def publish_message(message):
    queue.publish(message)
```

#### 4.4 訊息佇列對比

| 特性 | RabbitMQ | Kafka | AWS SQS |
|-----|----------|-------|---------|
| **模型** | 傳統MQ | 分散式日誌 | 雲端服務 |
| **吞吐量** | 中 (萬/秒) | 極高 (百萬/秒) | 高 |
| **延遲** | 低 (ms) | 中 (10ms) | 較高 |
| **順序保證** | 部分 | 分區內保證 | FIFO佇列 |
| **持久化** | 可選 | 預設持久化 | 預設持久化 |
| **複雜度** | 中 | 高 | 低 (託管) |
| **適用場景** | 企業內部 | 大資料管道 | AWS生態系 |

**選擇指南**:
```python
if requirements.needs_ultra_high_throughput:
    return "Kafka"  # 日誌收集、事件溯源
elif requirements.needs_cloud_managed:
    return "AWS SQS"  # 無需維護
elif requirements.needs_complex_routing:
    return "RabbitMQ"  # 靈活的路由規則
```

---

## 🎯 綜合案例: 設計一個URL Shortener

讓我們應用本模組所學的組件設計一個短網址服務:

**需求**:
- 10M DAU
- 100M URL創建/月
- 1B URL訪問/月

**系統架構**:
```
用戶
  ↓
[L7 LB - Nginx] ← 基於路徑路由
  ↓
[API Servers] ← 無狀態,水平擴展
  ↓
[Redis Cluster] ← 快取短網址映射 (Cache-Aside)
  ↓
[PostgreSQL] ← 主庫 (寫)
  ↓
[Read Replicas] ← 從庫 (讀)
```

**核心組件設計**:

1. **負載均衡**:
   - L7 Nginx
   - 算法: Least Connections (API伺服器負載可能不均)

2. **快取**:
   - Redis Cluster (水平擴展)
   - LRU淘汰策略
   - TTL: 熱點URL永不過期,冷URL 1天

3. **資料庫**:
   - PostgreSQL主庫 (寫)
   - 3個讀副本 (讀)
   - 索引: `short_code` (唯一索引)

4. **分片策略** (未來擴展):
   - 按短碼雜湊分片
   - 16個shard → 支持100B+ URL

---

## 📝 練習題

### 基礎題

1. **負載均衡**:
   - 比較Round-Robin和Least-Connections在長連接場景下的差異

2. **快取**:
   - 實作一個LRU快取 (不使用OrderedDict)
   - 處理快取穿透的3種方法

3. **資料庫**:
   - 為社交媒體用戶表設計合適的索引
   - 計算讀寫分離能提升多少讀取容量

### 進階題

4. **系統設計**:
   - 設計一個支持10億用戶的會話管理系統 (快取 + 資料庫)

5. **權衡分析**:
   - Write-Through vs Write-Back快取策略在電商訂單系統的選擇

6. **故障處理**:
   - Redis集群某個節點宕機,如何保證服務可用?

---

## 🎓 學習檢查清單

完成本模組後,你應該能夠:

- [ ] 說明L4和L7負載均衡的差異並選擇合適場景
- [ ] 實作至少3種負載均衡算法
- [ ] 解釋5種快取策略的權衡
- [ ] 處理快取穿透、擊穿、雪崩問題
- [ ] 選擇SQL或NoSQL資料庫並說明理由
- [ ] 設計資料庫索引並優化慢查詢
- [ ] 實作讀寫分離架構
- [ ] 理解訊息佇列的3種語義保證
- [ ] 設計一個使用所有核心組件的中型系統

---

**下一模組**: [03-Distributed-Systems - 分散式系統](../03-Distributed-Systems/README.md)
