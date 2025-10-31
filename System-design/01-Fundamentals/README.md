# Module 1: 系統設計基礎概念 (Fundamentals)

> 建立系統設計的底層思維模型 - FANG面試的基石

## 學習目標

本模組將幫助你建立系統設計的核心思維框架,理解大規模系統的基本原理和權衡決策。這些概念是所有FANG面試的必考基礎。

## 為什麼這個模組重要?

在FANG面試中,面試官會持續評估你對基礎概念的理解深度:
- ❌ 錯誤: "我們用Redis快取就能解決所有效能問題"
- ✅ 正確: "我們可以用Redis快取,但需要考慮快取一致性、失效策略和記憶體限制"

**關鍵差異**: 理解權衡(Trade-offs)而非死記解決方案

## 📚 課程內容

### 1. 擴展性基礎 (Scalability Fundamentals)

#### 1.1 垂直擴展 vs 水平擴展
- [理論講解](./01-scalability-basics.ipynb)
- [實作練習](./01-scalability-exercise.ipynb)

**核心概念**:
- **垂直擴展 (Scale Up)**: 增加單機資源
  - 優點: 簡單、無需修改架構
  - 缺點: 有上限、單點故障、成本非線性增長
  - 適用場景: 單體應用、關聯式資料庫

- **水平擴展 (Scale Out)**: 增加機器數量
  - 優點: 理論上無限擴展、高可用性
  - 缺點: 複雜度高、資料一致性挑戰
  - 適用場景: 無狀態服務、微服務架構

**面試重點**:
```
面試官: "如何擴展你的系統以支持10倍流量?"

答題框架:
1. 先評估當前瓶頸 (CPU? Memory? Network? Database?)
2. 短期方案: 可能垂直擴展買時間
3. 長期方案: 水平擴展 + 架構調整
4. 討論權衡: 複雜度 vs 可擴展性
```

#### 1.2 有狀態 vs 無狀態架構
- [狀態管理策略](./02-stateful-vs-stateless.ipynb)

**核心概念**:
- **無狀態 (Stateless)**: 不儲存用戶會話資料
  - 任何實例可處理任何請求
  - 易於水平擴展
  - Session存於外部存儲 (Redis, DB)

- **有狀態 (Stateful)**: 儲存用戶會話資料
  - 需要sticky session或session replication
  - 擴展複雜
  - WebSocket連接、遊戲伺服器

**設計原則**: 盡可能無狀態化

#### 1.3 資料分片策略 (Data Sharding)
- [Sharding深度解析](./03-data-sharding.ipynb)
- [一致性雜湊實作](./03-consistent-hashing.ipynb)

**分片方法**:
1. **範圍分片 (Range-based)**: user_id 1-1000 → shard1
2. **雜湊分片 (Hash-based)**: hash(user_id) % N
3. **地理分片 (Geo-based)**: 按地區分片
4. **目錄分片 (Directory-based)**: 查找表映射

**一致性雜湊 (Consistent Hashing)**:
- 解決傳統雜湊在添加/移除節點時的大量資料遷移
- 虛擬節點 (Virtual Nodes)處理負載不均

---

### 2. 效能優化 (Performance)

#### 2.1 CAP定理深度解析
- [CAP定理詳解](./04-cap-theorem.ipynb)
- [實際案例分析](./04-cap-real-world.ipynb)

**CAP三角**:
```
        Consistency (一致性)
              / \
             /   \
            /     \
           /       \
    Availability   Partition Tolerance
    (可用性)        (分區容錯性)
```

**核心理解**:
- **只能同時滿足兩個**
- 在分散式系統中,網路分區(P)不可避免
- 實務上是在 CP vs AP 之間選擇

**典型系統分類**:
- **CP系統**: HBase, MongoDB (strong consistency), Zookeeper
  - 在分區時犧牲可用性保證一致性
  - 銀行交易系統

- **AP系統**: Cassandra, DynamoDB, CouchDB
  - 在分區時犧牲一致性保證可用性
  - 社交媒體、新聞推薦

**面試陷阱**:
```
面試官: "我們需要一個既高可用又強一致的系統"

錯誤回答: "可以,我們用XXX技術就能實現"
正確回答: "根據CAP定理,在網路分區時必須在兩者間權衡。
          我們可以討論業務場景,看哪個更重要..."
```

#### 2.2 一致性模型 (Consistency Models)
- [一致性光譜](./05-consistency-models.ipynb)

**一致性強度排序** (從強到弱):

1. **強一致性 (Strong Consistency)**
   - 讀取保證看到最新寫入
   - 性能代價高
   - 範例: 關聯式資料庫 ACID

2. **最終一致性 (Eventual Consistency)**
   - 系統最終會達到一致狀態
   - 高性能、高可用
   - 範例: DNS、Amazon S3

3. **因果一致性 (Causal Consistency)**
   - 保證因果關係的操作順序
   - 折衷方案
   - 範例: Facebook News Feed

4. **讀己之寫一致性 (Read-Your-Writes)**
   - 用戶立即看到自己的更新
   - 常見需求
   - 範例: 社交媒體發文

**設計決策框架**:
```python
def choose_consistency_model(use_case):
    if use_case.requires_correctness():  # 金融交易
        return "強一致性"
    elif use_case.requires_availability():  # 社交媒體
        return "最終一致性"
    elif use_case.requires_user_experience():  # 評論系統
        return "讀己之寫一致性"
```

#### 2.3 延遲 vs 吞吐量 (Latency vs Throughput)
- [性能指標解析](./06-latency-throughput.ipynb)

**核心概念**:
- **延遲 (Latency)**: 單個請求的響應時間
  - P50, P99, P999延遲
  - 用戶體驗直接相關

- **吞吐量 (Throughput)**: 單位時間處理請求數
  - QPS (Queries Per Second)
  - 系統容量相關

**權衡關係**:
- 提升吞吐量可能增加延遲 (批次處理)
- 降低延遲可能犧牲吞吐量 (減少批次大小)

**優化策略**:
```
低延遲優化:
- 快取
- CDN
- 資料庫索引
- 非同步處理

高吞吐量優化:
- 連接池
- 批次處理
- 負載均衡
- 水平擴展
```

---

### 3. 可靠性設計 (Reliability)

#### 3.1 故障模式分析 (Failure Modes)
- [常見故障類型](./07-failure-modes.ipynb)

**故障分類**:

1. **硬體故障**
   - 伺服器宕機
   - 磁碟損壞
   - 網路故障

2. **軟體故障**
   - Bug
   - 記憶體洩漏
   - Deadlock

3. **級聯故障 (Cascading Failure)**
   - 一個服務故障導致連鎖反應
   - 最危險的故障模式

4. **人為錯誤**
   - 配置錯誤
   - 錯誤部署

**防禦策略**:
- 預期故障會發生 (Design for Failure)
- 隔離故障影響範圍
- 快速恢復 > 避免故障

#### 3.2 冗餘與備援策略
- [高可用架構設計](./08-redundancy.ipynb)

**冗餘類型**:

1. **主動-被動 (Active-Passive)**
   - 主節點處理請求,備節點待命
   - 成本低
   - 故障轉移時間較長

2. **主動-主動 (Active-Active)**
   - 所有節點同時處理請求
   - 高資源利用率
   - 複雜度高

3. **N+1冗餘**
   - N個節點承載流量,+1備份
   - 平衡成本與可靠性

**可用性計算**:
```
單機可用性: 99% (每年3.65天不可用)
雙機主備: 1 - (1-0.99)^2 = 99.99% (52.56分鐘)
三機: 99.9999% (31.5秒)

公式: 可用性 = (總時間 - 停機時間) / 總時間
```

#### 3.3 降級與熔斷機制
- [服務降級策略](./09-degradation.ipynb)
- [熔斷器模式](./09-circuit-breaker.ipynb)

**降級 (Degradation)**:
- 在系統壓力大時犧牲次要功能保證核心功能
- 範例:
  - 電商: 關閉推薦系統,保證購物功能
  - 社交媒體: 暫停高清圖片,提供低解析度版本

**熔斷器 (Circuit Breaker)**:
- 防止級聯故障
- 三狀態: 關閉(正常) → 開啟(故障) → 半開(測試恢復)

```python
class CircuitBreaker:
    def __init__(self, failure_threshold, timeout):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN

    def call(self, func):
        if self.state == "OPEN":
            raise CircuitBreakerError("Service unavailable")

        try:
            result = func()
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise e

    def on_failure(self):
        self.failure_count += 1
        if self.failure_count >= self.failure_threshold:
            self.state = "OPEN"
```

---

## 🎯 容量估算 (Capacity Estimation)

在FANG面試中,你需要快速估算系統規模:

### 估算框架

**1. 流量估算**
```
假設: 日活躍用戶(DAU) = 100M
     每用戶每天請求 = 10次

每日請求: 100M * 10 = 1B requests/day
每秒請求: 1B / 86400 ≈ 11.5K QPS
峰值QPS: 11.5K * 3 = 35K QPS (3倍峰值係數)
```

**2. 儲存估算**
```
假設: 每次請求產生 1KB 資料
     保存3年

日儲存: 1B * 1KB = 1TB/day
3年儲存: 1TB * 365 * 3 ≈ 1PB
考慮備份(3副本): 3PB
```

**3. 頻寬估算**
```
峰值QPS: 35K
請求大小: 1KB
響應大小: 10KB

入站頻寬: 35K * 1KB = 35MB/s
出站頻寬: 35K * 10KB = 350MB/s
```

### 常用數字 (Numbers Everyone Should Know)

```
L1 cache reference:           0.5 ns
Branch mispredict:            5 ns
L2 cache reference:           7 ns
Mutex lock/unlock:            25 ns
Main memory reference:        100 ns
Compress 1KB with Zippy:      3,000 ns = 3 µs
Send 1KB over 1 Gbps network: 10,000 ns = 10 µs
SSD random read:              150,000 ns = 150 µs
Read 1MB sequentially from memory: 250,000 ns = 250 µs
Round trip within same datacenter: 500,000 ns = 500 µs
Read 1MB sequentially from SSD: 1,000,000 ns = 1 ms
Disk seek:                    10,000,000 ns = 10 ms
Read 1MB sequentially from disk: 20,000,000 ns = 20 ms
Send packet CA→Netherlands→CA: 150,000,000 ns = 150 ms
```

**記憶技巧**:
- 記憶體 < 100ns
- SSD < 1ms
- 磁碟 < 10ms
- 跨大陸 < 150ms

---

## 📝 練習題

### 基礎題

1. **擴展性**
   - 一個Web服務器能處理1000 QPS,現在流量是10000 QPS,提出3種擴展方案並分析權衡

2. **一致性**
   - 設計一個聊天應用,討論應該選擇強一致性還是最終一致性,為什麼?

3. **容量估算**
   - 估算Twitter的儲存需求 (提示: 3億DAU, 每人每天發10條推文, 每條280字符)

### 進階題

4. **系統設計**
   - 設計一個全球化的內容分發系統,討論如何在不同地區保證低延遲和高可用性

5. **故障處理**
   - 你的資料庫在高峰期響應變慢,可能的原因和解決方案?

6. **權衡決策**
   - 在設計支付系統時,如何在一致性和可用性之間做權衡?

---

## 🎓 學習檢查清單

完成本模組後,你應該能夠:

- [ ] 解釋垂直擴展和水平擴展的差異及適用場景
- [ ] 說明CAP定理並舉出實際系統的例子
- [ ] 理解不同一致性模型的權衡
- [ ] 在5分鐘內完成基本的容量估算
- [ ] 識別系統的潛在故障點並提出防禦策略
- [ ] 設計冗餘機制以提高系統可用性
- [ ] 實作簡單的熔斷器模式

---

## 📚 延伸閱讀

### 必讀文章
- [CAP Twelve Years Later: How the "Rules" Have Changed](https://www.infoq.com/articles/cap-twelve-years-later-how-the-rules-have-changed/)
- [Fallacies of Distributed Computing](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)

### 推薦影片
- [Scalability Lecture - Harvard CS75](https://www.youtube.com/watch?v=-W9F__D3oY4)

---

**下一模組**: [02-Core-Components - 核心組件設計](../02-Core-Components/README.md)
