# Module 3: 分散式系統 (Distributed Systems)

> 理解大規模系統的複雜性 - 掌握分散式系統的核心挑戰與解決方案

## 學習目標

本模組深入探討分散式系統的核心概念和關鍵技術。這些是設計大規模系統時無法迴避的挑戰,也是FANG面試中資深工程師必須掌握的知識。

## 為什麼學習分散式系統?

當系統規模擴大到單機無法處理時,就需要分散式系統:
- **Netflix**: 全球800+個微服務
- **Uber**: 2200+個微服務
- **Amazon**: 數千個服務

**關鍵挑戰**:
- 網路不可靠 (延遲、丢包、分區)
- 部分故障 (某些節點失敗)
- 時鐘不同步
- 併發衝突

---

## 📚 課程內容

### 1. 分散式共識 (Consensus)

#### 1.1 共識問題基礎
- [共識問題介紹](./01-consensus-intro.ipynb)

**什麼是共識?**
在分散式系統中,多個節點就某個值達成一致的過程。

**為什麼需要共識?**
```
場景1: 選舉Leader
- 多個節點同時啟動
- 需要選出一個Leader處理寫入
- 如何保證只有一個Leader?

場景2: 分散式鎖
- 多個服務爭搶資源
- 需要保證同時只有一個持有鎖
- 如何避免腦裂?

場景3: 配置更新
- 配置變更需要所有節點同步
- 如何保證一致性?
```

**共識的要求**:
1. **Agreement (一致性)**: 所有節點決定相同值
2. **Validity (有效性)**: 決定的值必須是某個節點提議的
3. **Termination (終止性)**: 所有節點最終都會做出決定

**FLP不可能性定理**:
```
在異步網路中,即使只有一個節點故障,
也不存在同時滿足以下三點的共識算法:
- Safety (安全性)
- Liveness (活性)
- 容錯性

現實世界的共識算法都在這三者間權衡
```

#### 1.2 Paxos算法
- [Paxos詳解](./02-paxos.ipynb)
- [Paxos模擬實作](./02-paxos-simulation.ipynb)

**Paxos核心思想**:
分為兩個階段來達成共識

**Phase 1: Prepare階段**
```
Proposer (提議者):
1. 生成提案編號 n (遞增)
2. 向所有Acceptor發送 Prepare(n)

Acceptor (接受者):
3. 如果 n > 已見過的最大編號:
   - 承諾不再接受 < n 的提案
   - 返回已接受的最大提案 (如果有)
4. 否則拒絕
```

**Phase 2: Accept階段**
```
Proposer:
5. 如果收到多數派(majority)的Promise:
   - 如果有返回的已接受提案,使用最大編號的值
   - 否則使用自己的值
   - 向所有Acceptor發送 Accept(n, value)

Acceptor:
6. 如果 n >= 承諾的編號:
   - 接受提案
   - 返回Accepted
7. 否則拒絕
```

**範例流程**:
```
節點: A, B, C (共3個Acceptor)

時刻1: Proposer P1 提議 value="X"
  P1 → [Prepare(1)] → A, B, C
  A, B, C → [Promise(1)] → P1
  P1 → [Accept(1, "X")] → A, B, C
  A, B, C → [Accepted] → P1
  共識達成: X

時刻2: 並發提議
  P1 → [Prepare(2)] → A, B, C
  P2 → [Prepare(3)] → A, B, C

  A, B → [Promise(3)] → P2  (拒絕P1的Prepare(2))
  C → [Promise(2)] → P1

  P1無法獲得多數派,放棄
  P2 → [Accept(3, "Y")] → A, B, C
  共識達成: Y
```

**Paxos的問題**:
- ❌ 難以理解和實作
- ❌ 可能活鎖 (兩個Proposer互相搶佔)
- ❌ 效率不高 (兩階段)

#### 1.3 Raft算法
- [Raft詳解](./03-raft.ipynb)
- [Raft視覺化](./03-raft-visualization.ipynb)

**Raft核心思想**:
通過強Leader模式簡化共識過程

**Raft的三個角色**:
```
Leader (領導者):
- 處理所有客戶端請求
- 將日誌複製到Follower
- 決定何時提交日誌

Follower (跟隨者):
- 被動接收Leader的日誌
- 響應Leader的心跳
- 不處理客戶端請求

Candidate (候選人):
- Leader選舉時的臨時角色
- Follower在超時後變成Candidate
- 向其他節點請求投票
```

**Leader選舉流程**:
```
初始狀態: 所有節點都是Follower

1. Follower超時 (150-300ms未收到Leader心跳)
   → 轉變為Candidate
   → term (任期號) +1
   → 投票給自己
   → 向其他節點發送RequestVote RPC

2. 其他節點收到RequestVote:
   if (candidate的term >= 自己的term)
      and (自己還沒投票 or 已投給該candidate)
      and (candidate的日誌 >= 自己的日誌):
       投票給candidate
   else:
       拒絕

3. Candidate收到投票:
   - 獲得多數票 → 成為Leader,發送心跳
   - 收到其他Leader心跳 → 轉為Follower
   - 超時無結果 → 重新選舉 (term+1)
```

**日誌複製流程**:
```
1. 客戶端寫入請求 → Leader

2. Leader將請求追加到本地日誌 (uncommitted)

3. Leader並行發送AppendEntries RPC到所有Follower

4. Follower收到後:
   - 檢查日誌一致性 (prevLogIndex, prevLogTerm)
   - 如果一致,追加日誌並返回成功
   - 如果不一致,返回失敗 (Leader會回退重試)

5. Leader收到多數派成功響應:
   - 標記日誌為committed
   - 應用到狀態機
   - 返回客戶端成功
   - 在下次心跳中告知Follower提交點

6. Follower收到提交點更新:
   - 應用日誌到狀態機
```

**Raft vs Paxos**:
| 特性 | Raft | Paxos |
|-----|------|-------|
| **理解難度** | 簡單 ⭐⭐ | 複雜 ⭐⭐⭐⭐⭐ |
| **實作難度** | 中等 ⭐⭐⭐ | 困難 ⭐⭐⭐⭐⭐ |
| **Leader** | 強Leader | 弱Leader或無 |
| **效率** | 高 (單輪) | 低 (兩階段) |
| **工業應用** | etcd, Consul | Google Chubby |

**實際應用**:
- **etcd**: Kubernetes的配置存儲 (Raft)
- **Consul**: 服務發現與配置 (Raft)
- **Google Spanner**: 全球分散式資料庫 (Paxos變種)
- **ZooKeeper**: 配置管理 (ZAB,類似Raft)

#### 1.4 複製模式

**1. 主從複製 (Leader-Follower Replication)**

```
        寫入
         ↓
    [Leader] ──────→ 複製 ──────→ [Follower 1]
                 └──────→ 複製 ──→ [Follower 2]
                                  [Follower 3]
        ↑                              ↑
    客戶端寫                        客戶端讀
```

**同步複製 vs 異步複製**:

**同步複製 (Synchronous)**:
```python
def write_sync(key, value):
    # 1. 寫入Leader
    leader.write(key, value)

    # 2. 等待所有Follower確認
    for follower in followers:
        follower.replicate(key, value)
        follower.wait_ack()  # 阻塞等待

    # 3. 返回成功
    return "OK"
```
- **優點**: 強一致性,Follower總是最新
- **缺點**: 慢,任一Follower延遲都影響寫入
- **適用**: 金融系統 (無法容忍資料丟失)

**異步複製 (Asynchronous)**:
```python
def write_async(key, value):
    # 1. 寫入Leader
    leader.write(key, value)

    # 2. 非同步複製 (不等待)
    for follower in followers:
        async_replicate(follower, key, value)

    # 3. 立即返回
    return "OK"
```
- **優點**: 快,寫入不受Follower影響
- **缺點**: 可能丟失資料 (Leader故障時)
- **適用**: 大多數Web應用

**半同步複製 (Semi-Synchronous)**:
```python
def write_semi_sync(key, value):
    leader.write(key, value)

    # 等待至少一個Follower確認
    wait_for_one_ack(followers)

    return "OK"
```
- **權衡方案**: MySQL的默認配置

**2. 多主複製 (Multi-Leader Replication)**

```
      [Leader 1] ←──────→ 複製 ←──────→ [Leader 2]
          ↓                                  ↓
      Followers                          Followers
```

**適用場景**:
- 多數據中心部署 (每個DC一個Leader)
- 離線應用 (手機App本地寫入)
- 協作編輯 (Google Docs)

**衝突問題**:
```
時刻1:
  用戶A在DC1: UPDATE users SET name='Alice' WHERE id=1
  用戶B在DC2: UPDATE users SET name='Bob' WHERE id=1

時刻2: 複製同步
  DC1收到: name='Bob'
  DC2收到: name='Alice'

問題: 最終應該是Alice還是Bob?
```

**衝突解決策略**:

**1. Last Write Wins (LWW)**:
```python
def resolve_conflict_lww(value1, value2):
    if value1.timestamp > value2.timestamp:
        return value1
    else:
        return value2
```
- 簡單但可能丟失更新

**2. 應用層解決**:
```python
# 保留兩個版本,讓用戶選擇
def resolve_conflict_app(value1, value2):
    return {
        "conflict": True,
        "options": [value1, value2]
    }
```
- Amazon購物車使用此方法

**3. CRDT (Conflict-free Replicated Data Types)**:
- 數學上保證衝突自動解決
- Riak, Redis等使用

**3. 無主複製 (Leaderless Replication)**

```
客戶端 ──→ [節點1]
       ├─→ [節點2]
       └─→ [節點3]
```

**Quorum機制**:
```
N = 總節點數
W = 寫入確認數 (Write quorum)
R = 讀取節點數 (Read quorum)

一致性條件: W + R > N

範例: N=3, W=2, R=2
寫入: 3個節點中至少2個確認
讀取: 讀取2個節點,取最新值
```

**代表系統**:
- **Cassandra**: N=3, W=2, R=2 (默認)
- **DynamoDB**: 可配置W和R

---

### 2. 資料一致性 (Data Consistency)

#### 2.1 分散式事務

**ACID vs BASE**:

**ACID (強一致性)**:
- **A**tomicity: 原子性,全部成功或全部失敗
- **C**onsistency: 一致性,滿足約束條件
- **I**solation: 隔離性,併發事務互不影響
- **D**urability: 持久性,提交後永久保存

**BASE (最終一致性)**:
- **B**asically **A**vailable: 基本可用
- **S**oft state: 軟狀態,允許中間狀態
- **E**ventual consistency: 最終一致性

#### 2.2 兩階段提交 (2PC)

**角色**:
- **協調者 (Coordinator)**: 協調事務
- **參與者 (Participants)**: 執行事務

**流程**:

**Phase 1: Prepare (準備階段)**
```
1. Coordinator → "Prepare" → All Participants
2. 每個Participant:
   - 執行事務但不提交
   - 鎖定資源
   - 返回 "Yes" or "No"
```

**Phase 2: Commit (提交階段)**
```
3. Coordinator收集投票:
   - 全部Yes → 發送 "Commit"
   - 任一No → 發送 "Abort"

4. Participants收到決定:
   - Commit → 提交事務,釋放鎖
   - Abort → 回滾事務,釋放鎖
```

**範例: 跨行轉帳**:
```python
# 從銀行A轉100元到銀行B

# Phase 1: Prepare
coordinator.send_prepare([bank_a, bank_b])

bank_a.prepare():  # 檢查餘額 >= 100
    if balance >= 100:
        balance -= 100  # 暫扣,未提交
        return "Yes"
    return "No"

bank_b.prepare():  # 準備接收
    balance += 100  # 暫增,未提交
    return "Yes"

# Phase 2: Commit
if all_yes:
    coordinator.send_commit([bank_a, bank_b])
    bank_a.commit()  # 確認扣款
    bank_b.commit()  # 確認入賬
else:
    coordinator.send_abort([bank_a, bank_b])
```

**2PC的問題**:

**1. 阻塞問題**:
```
Participant在Prepare後等待Commit/Abort期間,
資源被鎖定,其他事務無法訪問
```

**2. 單點故障**:
```
如果Coordinator在Phase 2之前掛了:
- Participants不知道該Commit還是Abort
- 資源一直被鎖定 (阻塞)

解決: 3PC (三階段提交)
```

**3. 性能問題**:
```
- 兩次網路往返
- 同步阻塞
- 不適合高併發場景
```

#### 2.3 Saga模式

**核心思想**: 將長事務分解為多個本地事務,每個本地事務都有補償操作

**範例: 電商下單**
```
訂單事務 = 創建訂單 + 扣庫存 + 扣款 + 發貨

Saga流程:
1. 創建訂單 (成功)
   補償: 取消訂單

2. 扣庫存 (成功)
   補償: 恢復庫存

3. 扣款 (失敗!) ← 支付失敗

4. 執行補償:
   - 恢復庫存
   - 取消訂單
```

**實作方式**:

**1. 編排式 (Choreography)**:
```
創建訂單 → 發送事件 "OrderCreated"
           ↓
庫存服務監聽 → 扣庫存 → 發送 "InventoryReserved"
                        ↓
支付服務監聽 → 扣款 → 成功: "PaymentCompleted"
                      失敗: "PaymentFailed"
                        ↓
庫存服務監聽 "PaymentFailed" → 恢復庫存
訂單服務監聽 "PaymentFailed" → 取消訂單
```

**2. 協調式 (Orchestration)**:
```python
class OrderSaga:
    def execute(self):
        try:
            # Step 1: 創建訂單
            order_id = order_service.create_order()

            # Step 2: 扣庫存
            inventory_service.reserve(order_id)

            # Step 3: 扣款
            payment_service.charge(order_id)

            # Step 4: 發貨
            shipping_service.ship(order_id)

        except InventoryException:
            order_service.cancel(order_id)

        except PaymentException:
            inventory_service.release(order_id)
            order_service.cancel(order_id)

        except ShippingException:
            payment_service.refund(order_id)
            inventory_service.release(order_id)
            order_service.cancel(order_id)
```

**Saga vs 2PC**:
| 特性 | Saga | 2PC |
|-----|------|-----|
| **一致性** | 最終一致 | 強一致 |
| **性能** | 高 | 低 |
| **隔離性** | 無 (其他事務可見中間狀態) | 有 |
| **複雜度** | 高 (需設計補償) | 中 |
| **適用場景** | 微服務、長事務 | 單體、短事務 |

#### 2.4 事件溯源 (Event Sourcing)

**核心思想**: 不存儲最終狀態,存儲所有改變狀態的事件

**傳統方式 vs 事件溯源**:

**傳統方式**:
```sql
-- 只存儲最終狀態
users表:
id | name  | balance
1  | Alice | 150

-- 看不到歷史
```

**事件溯源**:
```sql
events表:
id | user_id | event_type      | amount | timestamp
1  | 1       | AccountCreated  | 0      | T1
2  | 1       | MoneyDeposited  | 100    | T2
3  | 1       | MoneyWithdrawn  | 50     | T3
4  | 1       | MoneyDeposited  | 100    | T4

-- 當前狀態 = 重放所有事件
balance = 0 + 100 - 50 + 100 = 150
```

**優點**:
- ✅ 完整的審計日誌
- ✅ 時間旅行 (查看任意時刻狀態)
- ✅ 調試友好 (可重放bug)
- ✅ 事件驅動架構的基礎

**缺點**:
- ❌ 查詢慢 (需要重放)
- ❌ 儲存空間大
- ❌ 事件版本管理複雜

**解決方案: CQRS + Event Sourcing**

#### 2.5 CQRS (Command Query Responsibility Segregation)

**核心思想**: 讀寫分離,使用不同的模型

```
        Command (寫入)
             ↓
        [Event Store] → 事件流
             ↓
        事件處理器
             ↓
        [Read Model] ← Query (讀取)
```

**範例: 電商訂單**

**寫入側 (Command)**:
```python
class CreateOrderCommand:
    def execute(self, order_data):
        # 1. 驗證
        validate(order_data)

        # 2. 創建事件
        event = OrderCreatedEvent(
            order_id=generate_id(),
            user_id=order_data['user_id'],
            items=order_data['items'],
            timestamp=now()
        )

        # 3. 存儲事件
        event_store.append(event)

        # 4. 發布事件
        event_bus.publish(event)
```

**讀取側 (Query)**:
```python
# 專門為查詢優化的模型
orders_view表:
order_id | user_name | total | status | created_at

class GetOrderQuery:
    def execute(self, order_id):
        # 直接從優化的讀取模型查詢
        return orders_view.get(order_id)
```

**事件處理器 (更新讀取模型)**:
```python
@event_handler('OrderCreatedEvent')
def on_order_created(event):
    # 更新讀取模型
    orders_view.insert({
        'order_id': event.order_id,
        'user_name': get_user_name(event.user_id),
        'total': calculate_total(event.items),
        'status': 'CREATED',
        'created_at': event.timestamp
    })
```

**CQRS的好處**:
- ✅ 讀寫獨立優化
- ✅ 讀取模型可以有多個 (不同視角)
- ✅ 高性能 (讀取不受寫入影響)

---

### 3. 服務發現與協調 (Service Discovery & Coordination)

#### 3.1 服務發現模式

**問題**: 微服務架構中,服務實例動態變化,如何找到服務?

**方案1: 客戶端發現 (Client-Side Discovery)**
```
1. 服務啟動 → 註冊到服務註冊表 (Consul/Eureka)
2. 客戶端查詢註冊表 → 獲取服務列表
3. 客戶端選擇一個實例 (負載均衡)
4. 直接調用服務

優點: 客戶端控制負載均衡
缺點: 客戶端邏輯複雜
```

**方案2: 服務端發現 (Server-Side Discovery)**
```
1. 服務啟動 → 註冊到註冊表
2. 客戶端 → Load Balancer
3. Load Balancer查詢註冊表 → 選擇實例
4. Load Balancer轉發請求

優點: 客戶端簡單
缺點: Load Balancer成為單點
```

**Netflix Eureka範例**:
```java
// 服務註冊
@EnableEurekaClient
public class UserService {
    // 啟動時自動註冊到Eureka
}

// 服務發現
@Autowired
private DiscoveryClient discoveryClient;

List<ServiceInstance> instances =
    discoveryClient.getInstances("user-service");

ServiceInstance instance = instances.get(0);
String url = instance.getUri() + "/api/users/1";
```

#### 3.2 配置管理

**集中式配置 vs 分散式配置**:

**分散式配置 (etcd/Consul)**:
```python
# 服務讀取配置
config = etcd.get('/services/user-service/config')
db_url = config['database']['url']

# 配置更新
etcd.put('/services/user-service/config', new_config)

# 監聽配置變化
@watch('/services/user-service/config')
def on_config_change(new_config):
    reload_config(new_config)
```

#### 3.3 分散式鎖

**為什麼需要分散式鎖?**
```
場景: 定時任務在多台機器運行
問題: 如何保證只有一台執行?

解決: 分散式鎖
```

**Redis實作 (Redlock)**:
```python
import redis
import time
import uuid

class RedisLock:
    def __init__(self, redis_client, key, ttl=10):
        self.redis = redis_client
        self.key = f"lock:{key}"
        self.ttl = ttl
        self.token = str(uuid.uuid4())

    def acquire(self):
        # SET key token NX EX ttl
        return self.redis.set(
            self.key,
            self.token,
            nx=True,  # Only set if not exists
            ex=self.ttl
        )

    def release(self):
        # Lua腳本保證原子性
        script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        self.redis.eval(script, 1, self.key, self.token)

# 使用
lock = RedisLock(redis_client, "cron-job-1")

if lock.acquire():
    try:
        # 執行任務
        run_cron_job()
    finally:
        lock.release()
else:
    print("Another instance is running")
```

**ZooKeeper實作**:
```python
from kazoo.client import KazooClient

zk = KazooClient(hosts='127.0.0.1:2181')
zk.start()

# 創建臨時順序節點
lock_path = "/locks/my-lock"
zk.ensure_path(lock_path)

# 獲取鎖
node = zk.create(f"{lock_path}/lock-", ephemeral=True, sequence=True)

# 檢查是否最小節點
children = sorted(zk.get_children(lock_path))
if node.split('/')[-1] == children[0]:
    # 獲得鎖
    run_critical_section()
    zk.delete(node)
else:
    # 等待前一個節點釋放
    pass
```

**分散式鎖對比**:
| 方案 | 優點 | 缺點 | 適用場景 |
|-----|------|------|---------|
| Redis | 高性能 | 可能出現腦裂 | 對一致性要求不高 |
| ZooKeeper | 強一致性 | 性能較低 | 對一致性要求高 |
| etcd | 強一致性 | 複雜度高 | Kubernetes生態 |

---

## 🎯 綜合案例: 設計分散式配置中心

**需求**:
- 集中管理所有服務的配置
- 配置更新實時推送
- 高可用 (99.99%)
- 配置版本管理
- 權限控制

**架構設計**:

```
                [Web Console] ← 運維人員
                      ↓
                [API Server]
                      ↓
            [etcd Cluster] ← 存儲配置 (Raft保證一致性)
             /     |     \
            /      |      \
       [Watch] [Watch] [Watch]
         /        |        \
[Service A] [Service B] [Service C]
```

**核心組件**:

1. **存儲層 (etcd)**:
```go
// 存儲配置
etcd.Put("/config/user-service/prod/db.url",
         "mysql://prod-db:3306")

// 版本管理
etcd.Put("/config/user-service/prod/db.url",
         "mysql://new-db:3306",
         ClientV3.WithPrevKV())  // 保留舊版本
```

2. **推送層 (Watch)**:
```go
// 服務監聽配置變化
watchChan := etcd.Watch(context.Background(),
                        "/config/user-service/prod/")

for watchResp := range watchChan {
    for _, event := range watchResp.Events {
        fmt.Printf("Config changed: %s = %s\n",
                   event.Kv.Key, event.Kv.Value)
        reloadConfig()
    }
}
```

3. **高可用**:
```
etcd集群 (3或5個節點)
- Raft保證一致性
- 任一節點故障,集群繼續服務
- 多數派(majority)存活即可
```

---

## 📝 練習題

### 基礎題

1. **共識算法**:
   - 解釋Raft的Leader選舉過程
   - Paxos和Raft有什麼區別?

2. **複製模式**:
   - 同步複製和異步複製的權衡?
   - 什麼情況下使用多主複製?

3. **分散式事務**:
   - 2PC和Saga的適用場景?
   - 如何設計補償操作?

### 進階題

4. **系統設計**:
   - 設計一個分散式鎖服務 (要求高可用)
   - 設計一個分散式配置中心

5. **故障處理**:
   - etcd集群中2個節點故障,會發生什麼?
   - 如何處理網路分區?

6. **一致性**:
   - 設計一個最終一致性的購物車系統
   - 如何在CQRS中保證讀寫一致性?

---

## 🎓 學習檢查清單

完成本模組後,你應該能夠:

- [ ] 解釋Raft算法的完整流程
- [ ] 理解Paxos和Raft的差異
- [ ] 選擇合適的複製模式 (主從/多主/無主)
- [ ] 設計分散式事務 (2PC/Saga/Event Sourcing)
- [ ] 實作CQRS模式
- [ ] 使用etcd/ZooKeeper實作服務發現
- [ ] 實作分散式鎖
- [ ] 處理網路分區和腦裂問題

---

## 📚 延伸閱讀

### 論文
- **Paxos Made Simple** - Leslie Lamport
- **In Search of an Understandable Consensus Algorithm (Raft)** - Diego Ongaro

### 書籍
- **Designing Data-Intensive Applications** - Chapter 5, 7, 8, 9
- **Database Internals** - Alex Petrov

### 實作
- [Raft可視化](https://raft.github.io/)
- [etcd源碼](https://github.com/etcd-io/etcd)
- [Consul源碼](https://github.com/hashicorp/consul)

---

**上一模組**: [02-Core-Components](../02-Core-Components/README.md)
**下一模組**: [04-Case-Studies](../04-Case-Studies/README.md)
