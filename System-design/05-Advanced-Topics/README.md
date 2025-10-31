# Module 5: 進階主題 (Advanced Topics)

> 深入專業領域 - 成為系統架構專家的必經之路

## 學習目標

本模組涵蓋系統設計的進階主題,這些是資深工程師和架構師需要掌握的專業知識。在FANG面試中,Staff+級別的候選人經常需要展現對這些主題的深入理解。

## 模組定位

這不是基礎必修課,而是:
- 💼 **職業發展**: 從Senior到Staff的必經之路
- 🎯 **專業深化**: 選擇感興趣的領域深入研究
- 🚀 **實戰導向**: 每個主題都來自真實生產環境的挑戰

---

## 📚 課程內容

### 1. 監控與可觀測性 (Observability)

#### 1.1 可觀測性三支柱

**Metrics (指標) + Logs (日誌) + Traces (追蹤) = 可觀測性**

```
         系統健康度
              ↓
    ┌─────────┴─────────┐
    │                   │
Metrics              Logs              Traces
  ↓                  ↓                  ↓
"什麼時候出錯?"    "為什麼出錯?"     "哪裡出錯?"
```

#### 1.2 Metrics (指標)

**四大黃金指標 (Google SRE)**:

1. **Latency (延遲)**: 請求響應時間
```prometheus
# P50, P95, P99延遲
histogram_quantile(0.95,
  rate(http_request_duration_seconds_bucket[5m])
)
```

2. **Traffic (流量)**: 請求量
```prometheus
# QPS
rate(http_requests_total[1m])
```

3. **Errors (錯誤率)**: 失敗請求比例
```prometheus
# 錯誤率
rate(http_requests_total{status=~"5.."}[5m])
/
rate(http_requests_total[5m])
```

4. **Saturation (飽和度)**: 資源使用率
```prometheus
# CPU使用率
100 - (avg by (instance)
  (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**RED Method (微服務)**:
- **R**ate: 請求速率
- **E**rrors: 錯誤率
- **D**uration: 持續時間

**USE Method (資源監控)**:
- **U**tilization: 使用率
- **S**aturation: 飽和度
- **E**rrors: 錯誤數

**實作: Prometheus + Grafana**:
```python
from prometheus_client import Counter, Histogram, Gauge
import time

# 計數器: 請求總數
http_requests = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

# 直方圖: 請求延遲
http_duration = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

# 儀表: 當前活躍連接
active_connections = Gauge(
    'active_connections',
    'Number of active connections'
)

@app.route('/api/users')
def get_users():
    start_time = time.time()

    try:
        result = fetch_users()
        http_requests.labels('GET', '/api/users', '200').inc()
        return result
    except Exception as e:
        http_requests.labels('GET', '/api/users', '500').inc()
        raise
    finally:
        duration = time.time() - start_time
        http_duration.labels('GET', '/api/users').observe(duration)
```

#### 1.3 Logs (日誌)

**結構化日誌 vs 非結構化日誌**:

**非結構化 (不好)**:
```python
print(f"User {user_id} failed to login at {timestamp}")
# 難以解析和搜索
```

**結構化 (好)**:
```python
import structlog

logger = structlog.get_logger()

logger.info(
    "user_login_failed",
    user_id=user_id,
    ip_address=request.ip,
    reason="invalid_password",
    timestamp=datetime.now().isoformat()
)

# 輸出JSON,易於解析
{
  "event": "user_login_failed",
  "user_id": 12345,
  "ip_address": "192.168.1.1",
  "reason": "invalid_password",
  "timestamp": "2025-10-31T10:00:00Z"
}
```

**日誌級別策略**:
```python
# DEBUG: 調試信息 (僅開發環境)
logger.debug("Cache hit", key=cache_key, value=value)

# INFO: 正常業務流程
logger.info("Order created", order_id=order_id, user_id=user_id)

# WARNING: 潛在問題
logger.warning("Slow query detected", query_time=2.5, threshold=1.0)

# ERROR: 錯誤但服務可繼續
logger.error("Payment failed", user_id=user_id, error=str(e))

# CRITICAL: 嚴重錯誤,服務可能不可用
logger.critical("Database connection lost", db_host=db_host)
```

**日誌聚合: ELK Stack**:
```
應用 → Logstash → Elasticsearch → Kibana
  ↓ (收集)  ↓ (解析)    ↓ (存儲)     ↓ (可視化)
```

**日誌最佳實踐**:
1. ✅ 使用結構化日誌 (JSON)
2. ✅ 包含上下文 (request_id, user_id)
3. ✅ 合理設置日誌級別
4. ✅ 不要記錄敏感信息 (密碼、信用卡)
5. ✅ 日誌採樣 (高流量時避免過載)

#### 1.4 Traces (分散式追蹤)

**問題**: 微服務架構中,一個請求可能經過10+個服務,如何定位性能瓶頸?

**解決**: 分散式追蹤 (Distributed Tracing)

**核心概念**:
```
Trace (追蹤): 一個完整的請求鏈路
  ↓
Span (跨度): 一個操作單元

範例:
Trace: 用戶下單
├─ Span 1: API Gateway (50ms)
├─ Span 2: Order Service (200ms)
│  ├─ Span 2.1: 查詢用戶 (30ms)
│  ├─ Span 2.2: 檢查庫存 (100ms) ← 瓶頸!
│  └─ Span 2.3: 創建訂單 (70ms)
└─ Span 3: Payment Service (80ms)
```

**OpenTelemetry實作**:
```python
from opentelemetry import trace
from opentelemetry.exporter.jaeger import JaegerExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# 配置Tracer
trace.set_tracer_provider(TracerProvider())
tracer = trace.get_tracer(__name__)

jaeger_exporter = JaegerExporter(
    agent_host_name="localhost",
    agent_port=6831,
)
span_processor = BatchSpanProcessor(jaeger_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# 使用Tracer
@app.route('/api/orders')
def create_order():
    with tracer.start_as_current_span("create_order") as span:
        span.set_attribute("user_id", user_id)

        # 子Span
        with tracer.start_as_current_span("check_inventory"):
            inventory = check_inventory(product_id)

        with tracer.start_as_current_span("process_payment"):
            payment = process_payment(amount)

        with tracer.start_as_current_span("save_order"):
            order = save_order(order_data)

        return order
```

**追蹤工具對比**:
| 工具 | 優點 | 缺點 | 適用 |
|-----|------|------|------|
| Jaeger | 開源、功能完整 | 部署複雜 | 自建 |
| Zipkin | 簡單易用 | 功能較少 | 中小規模 |
| AWS X-Ray | 雲端整合 | 綁定AWS | AWS用戶 |
| Datadog APM | 商業支持 | 昂貴 | 企業級 |

---

### 2. 安全性設計 (Security)

#### 2.1 認證 (Authentication)

**認證方式對比**:

**1. Session-Based (傳統)**:
```python
# 登入
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)
    if user:
        session['user_id'] = user.id
        return "Login success"
    return "Login failed", 401

# 檢查認證
@app.route('/api/profile')
def profile():
    if 'user_id' not in session:
        return "Unauthorized", 401
    user = get_user(session['user_id'])
    return user
```
- **優點**: 服務端控制,可隨時撤銷
- **缺點**: 不適合分散式系統 (Session共享問題)

**2. Token-Based (JWT)**:
```python
import jwt
from datetime import datetime, timedelta

SECRET_KEY = "your-secret-key"

# 登入
@app.route('/login', methods=['POST'])
def login():
    user = authenticate(username, password)
    if user:
        # 生成JWT
        payload = {
            'user_id': user.id,
            'username': user.username,
            'exp': datetime.utcnow() + timedelta(hours=24)
        }
        token = jwt.encode(payload, SECRET_KEY, algorithm='HS256')
        return {'token': token}
    return "Login failed", 401

# 驗證JWT
def verify_token(token):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
        return payload
    except jwt.ExpiredSignatureError:
        return None  # Token過期
    except jwt.InvalidTokenError:
        return None  # Token無效

@app.route('/api/profile')
def profile():
    token = request.headers.get('Authorization').split(' ')[1]
    payload = verify_token(token)
    if not payload:
        return "Unauthorized", 401
    user = get_user(payload['user_id'])
    return user
```

**3. OAuth 2.0 (第三方登入)**:
```
用戶 → 點擊"用Google登入"
  ↓
重定向到Google授權頁
  ↓
用戶同意授權
  ↓
Google重定向回應用 (帶Authorization Code)
  ↓
應用用Code換取Access Token
  ↓
用Access Token訪問Google API獲取用戶信息
```

**OAuth 2.0 Grant Types**:
- **Authorization Code**: Web應用 (最安全)
- **Implicit**: 單頁應用 (已棄用)
- **Client Credentials**: 服務間調用
- **Password**: 信任的第一方應用

#### 2.2 授權 (Authorization)

**RBAC (Role-Based Access Control)**:
```python
# 定義角色和權限
roles = {
    'admin': ['read', 'write', 'delete'],
    'editor': ['read', 'write'],
    'viewer': ['read']
}

users = {
    'alice': 'admin',
    'bob': 'editor',
    'charlie': 'viewer'
}

def check_permission(user, action):
    role = users.get(user)
    if not role:
        return False
    return action in roles.get(role, [])

# 使用
@app.route('/api/posts/<id>', methods=['DELETE'])
def delete_post(id):
    user = get_current_user()
    if not check_permission(user, 'delete'):
        return "Forbidden", 403
    delete_post_by_id(id)
    return "Deleted"
```

**ABAC (Attribute-Based Access Control)**:
```python
# 基於屬性的訪問控制
def check_access(user, resource, action):
    # 規則: 作者可以編輯自己的文章
    if action == 'edit' and resource.author_id == user.id:
        return True

    # 規則: 管理員可以編輯所有文章
    if user.role == 'admin':
        return True

    # 規則: 編輯可以編輯草稿狀態的文章
    if user.role == 'editor' and resource.status == 'draft':
        return True

    return False
```

#### 2.3 API安全

**1. Rate Limiting (限流)**:
```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=lambda: request.headers.get('X-API-Key'),
    default_limits=["100 per hour"]
)

@app.route('/api/search')
@limiter.limit("10 per minute")
def search():
    return perform_search()
```

**2. CORS (跨域資源共享)**:
```python
from flask_cors import CORS

# 允許特定域名
CORS(app, resources={
    r"/api/*": {
        "origins": ["https://example.com"],
        "methods": ["GET", "POST"],
        "allow_headers": ["Content-Type", "Authorization"]
    }
})
```

**3. Input Validation (輸入驗證)**:
```python
from marshmallow import Schema, fields, validate, ValidationError

class UserSchema(Schema):
    username = fields.Str(
        required=True,
        validate=validate.Length(min=3, max=20)
    )
    email = fields.Email(required=True)
    age = fields.Int(
        validate=validate.Range(min=0, max=120)
    )

@app.route('/api/users', methods=['POST'])
def create_user():
    schema = UserSchema()
    try:
        data = schema.load(request.json)
        user = create_user_in_db(data)
        return user, 201
    except ValidationError as e:
        return {"errors": e.messages}, 400
```

**4. SQL Injection防護**:
```python
# ✗ 危險: SQL注入
username = request.args.get('username')
query = f"SELECT * FROM users WHERE username = '{username}'"
# 惡意輸入: username = "' OR '1'='1"

# ✓ 安全: 參數化查詢
username = request.args.get('username')
query = "SELECT * FROM users WHERE username = ?"
cursor.execute(query, (username,))
```

**5. XSS (跨站腳本)防護**:
```python
from markupsafe import escape

# 轉義用戶輸入
user_input = request.form.get('comment')
safe_input = escape(user_input)

# 使用Content Security Policy
@app.after_request
def set_csp(response):
    response.headers['Content-Security-Policy'] = \
        "default-src 'self'; script-src 'self' 'unsafe-inline'"
    return response
```

#### 2.4 加密

**對稱加密 vs 非對稱加密**:

**對稱加密 (AES)**:
```python
from cryptography.fernet import Fernet

# 生成密鑰
key = Fernet.generate_key()
cipher = Fernet(key)

# 加密
plaintext = b"Secret message"
ciphertext = cipher.encrypt(plaintext)

# 解密
decrypted = cipher.decrypt(ciphertext)
```
- 優點: 快速
- 缺點: 密鑰分發問題
- 適用: 資料加密

**非對稱加密 (RSA)**:
```python
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

# 生成密鑰對
private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=2048
)
public_key = private_key.public_key()

# 公鑰加密 (用於發送方)
from cryptography.hazmat.primitives.asymmetric import padding

ciphertext = public_key.encrypt(
    b"Secret message",
    padding.OAEP(...)
)

# 私鑰解密 (用於接收方)
plaintext = private_key.decrypt(ciphertext, padding.OAEP(...))
```
- 優點: 無需共享密鑰
- 缺點: 慢
- 適用: 數字簽名、密鑰交換

**HTTPS/TLS**:
```
客戶端 → [ClientHello] → 服務端
       ← [ServerHello + 證書] ←
       → [驗證證書]
       → [生成會話密鑰] →
       ← [確認] ←
       ↔ [對稱加密通信] ↔
```

---

### 3. 成本優化 (Cost Optimization)

#### 3.1 資源利用率優化

**1. Right-Sizing (合理配置)**:
```python
# 監控實際使用情況
actual_cpu_usage = get_cpu_metrics()  # 平均30%
actual_memory_usage = get_memory_metrics()  # 平均2GB

# 當前配置: 4 vCPU, 8GB RAM
# 優化後: 2 vCPU, 4GB RAM
# 節省: 50%成本
```

**2. Auto-Scaling (自動擴展)**:
```yaml
# Kubernetes HPA配置
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-server
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 2  # 最小2個Pod
  maxReplicas: 10  # 最大10個Pod
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70  # CPU>70%時擴展
```

**3. Spot Instances (競價實例)**:
```python
# AWS Spot實例可節省70-90%成本
# 適用於可中斷的工作負載

# 批次處理任務
spot_config = {
    'instance_type': 'm5.large',
    'max_price': '0.05',  # 最高出價
    'interruption_behavior': 'terminate'
}

# 混合使用On-Demand和Spot
{
    'on_demand': 2,  # 保底容量
    'spot': 8,       # 彈性容量
    'total': 10
}
```

#### 3.2 資料存儲優化

**1. 冷熱資料分離**:
```python
# 熱資料 (最近30天): 快速存儲
hot_data_storage = "SSD (AWS EBS gp3)"
cost_per_gb = 0.08  # USD/GB/月

# 溫資料 (30-90天): 標準存儲
warm_data_storage = "HDD (AWS EBS st1)"
cost_per_gb = 0.045

# 冷資料 (>90天): 歸檔存儲
cold_data_storage = "AWS S3 Glacier"
cost_per_gb = 0.004

# 自動轉移策略
def archive_old_data():
    # 將90天前的資料移至Glacier
    old_data = query("SELECT * FROM logs WHERE created_at < NOW() - INTERVAL 90 DAY")
    for record in old_data:
        s3.put_object(
            Bucket='archive-bucket',
            Key=f'logs/{record.id}',
            Body=record.data,
            StorageClass='GLACIER'
        )
        db.delete(record)
```

**2. 資料壓縮**:
```python
import gzip
import json

# 壓縮日誌
data = json.dumps(log_data)
compressed = gzip.compress(data.encode('utf-8'))

# 壓縮比: 通常5-10倍
original_size = len(data)
compressed_size = len(compressed)
ratio = original_size / compressed_size  # 約7x

# 節省存儲成本: 85%
```

**3. TTL (Time To Live)**:
```python
# Redis自動過期
redis.setex(
    f"session:{user_id}",
    3600,  # 1小時後過期
    session_data
)

# MongoDB TTL索引
db.sessions.create_index(
    "createdAt",
    expireAfterSeconds=3600
)
```

#### 3.3 網路成本優化

**1. CDN使用**:
```
無CDN:
用戶 → (跨區域) → 源伺服器
流量成本: $0.09/GB

有CDN:
用戶 → (就近) → CDN → (少量) → 源伺服器
流量成本: $0.02/GB (CDN) + $0.01/GB (回源)
節省: 67%
```

**2. 資料傳輸優化**:
```python
# 壓縮HTTP響應
from flask_compress import Compress

app = Flask(__name__)
Compress(app)  # 自動壓縮>500字節的響應

# 節省: 70-80%頻寬
```

**3. 同區域部署**:
```
跨區域傳輸: $0.02/GB
同區域傳輸: 免費

策略: 資料庫和應用在同一可用區
```

#### 3.4 成本監控與分析

**AWS Cost Explorer範例**:
```python
import boto3

ce = boto3.client('ce')

# 查詢本月成本
response = ce.get_cost_and_usage(
    TimePeriod={
        'Start': '2025-10-01',
        'End': '2025-10-31'
    },
    Granularity='DAILY',
    Metrics=['UnblendedCost'],
    GroupBy=[
        {'Type': 'DIMENSION', 'Key': 'SERVICE'}
    ]
)

# 分析哪個服務花費最多
for result in response['ResultsByTime']:
    print(f"Date: {result['TimePeriod']['Start']}")
    for group in result['Groups']:
        service = group['Keys'][0]
        cost = group['Metrics']['UnblendedCost']['Amount']
        print(f"  {service}: ${cost}")
```

**成本優化清單**:
- [ ] Right-sizing: 調整實例大小
- [ ] Auto-scaling: 根據負載自動調整
- [ ] Reserved Instances: 長期使用打折
- [ ] Spot Instances: 批次任務使用競價實例
- [ ] 冷熱資料分離: 舊資料歸檔
- [ ] CDN: 靜態資源使用CDN
- [ ] 壓縮: 資料和網路壓縮
- [ ] 清理: 刪除未使用資源

---

### 4. 微服務架構 (Microservices Architecture)

#### 4.1 微服務 vs 單體架構

**單體架構 (Monolithic)**:
```
┌─────────────────────────────────┐
│      Monolithic Application     │
│  ┌────┬────┬────┬────┬────┐    │
│  │User│Ord │Inv │Pay │Ship│    │
│  │Mgmt│er  │ent │ment│ping│    │
│  └────┴────┴────┴────┴────┘    │
│         Shared Database         │
└─────────────────────────────────┘
```
- 優點: 簡單、開發快、易於部署
- 缺點: 難以擴展、技術棧鎖定、一個bug影響全局

**微服務架構 (Microservices)**:
```
┌────────┐  ┌────────┐  ┌────────┐
│ User   │  │ Order  │  │Payment │
│Service │  │Service │  │Service │
│   DB   │  │   DB   │  │   DB   │
└────────┘  └────────┘  └────────┘
```
- 優點: 獨立擴展、技術自由、故障隔離
- 缺點: 複雜度高、分散式事務、運維成本

**何時使用微服務?**
```
團隊規模 > 20人
系統複雜度高
需要獨立擴展
多語言需求
```

#### 4.2 API Gateway

**作用**:
- 統一入口
- 路由轉發
- 認證授權
- 限流熔斷
- 協議轉換

**實作範例 (Kong)**:
```yaml
# Kong配置
services:
  - name: user-service
    url: http://user-service:8080
    routes:
      - paths:
          - /api/users

  - name: order-service
    url: http://order-service:8080
    routes:
      - paths:
          - /api/orders

plugins:
  - name: rate-limiting
    config:
      minute: 100
  - name: jwt
    config:
      key_claim_name: iss
```

#### 4.3 Service Mesh

**問題**: 服務間通信的可靠性、安全性、可觀測性

**解決**: Service Mesh (服務網格)

**Istio架構**:
```
服務A → [Sidecar Proxy] ←→ [Sidecar Proxy] ← 服務B
            ↓                      ↓
       Control Plane (Istiod)
       - 流量管理
       - 安全策略
       - 遙測收集
```

**功能**:
1. **流量管理**: 負載均衡、灰度發布
2. **安全**: mTLS、授權策略
3. **可觀測性**: 自動追蹤、指標收集

**灰度發布範例**:
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
    - reviews
  http:
  - match:
    - headers:
        user-type:
          exact: internal  # 內部用戶
    route:
    - destination:
        host: reviews
        subset: v2  # 新版本
      weight: 100
  - route:
    - destination:
        host: reviews
        subset: v1  # 舊版本
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10  # 10%流量到新版本
```

---

### 5. 多租戶架構 (Multi-Tenancy)

#### 5.1 多租戶模式

**1. 資料庫級隔離 (Database per Tenant)**:
```
Tenant A → DB_A
Tenant B → DB_B
Tenant C → DB_C
```
- 優點: 隔離性好、易於備份和遷移
- 缺點: 成本高、維護複雜

**2. Schema級隔離 (Schema per Tenant)**:
```
Database
├─ schema_tenant_a
├─ schema_tenant_b
└─ schema_tenant_c
```
- 優點: 成本適中、隔離性好
- 缺點: 單庫限制

**3. 行級隔離 (Row-Level Isolation)**:
```sql
users表:
id | tenant_id | name
1  | tenant_a  | Alice
2  | tenant_b  | Bob
3  | tenant_a  | Charlie

-- 查詢時過濾
SELECT * FROM users WHERE tenant_id = 'tenant_a'
```
- 優點: 成本低、易於管理
- 缺點: 資料洩漏風險、性能挑戰

**選擇決策**:
```python
def choose_tenancy_model(requirements):
    if requirements.security_level == "high":
        return "資料庫級隔離"
    elif requirements.tenant_count < 100:
        return "Schema級隔離"
    else:
        return "行級隔離"
```

#### 5.2 租戶識別

**方法1: Subdomain**:
```
tenant-a.saas-app.com → Tenant A
tenant-b.saas-app.com → Tenant B
```

**方法2: Path**:
```
saas-app.com/tenant-a → Tenant A
saas-app.com/tenant-b → Tenant B
```

**方法3: Header**:
```
X-Tenant-ID: tenant-a
```

---

## 🎯 綜合案例: 設計SaaS平台

**需求**:
- 支持1000+租戶
- 每租戶10-100用戶
- 高可用 (99.9%)
- 資料隔離
- 成本優化

**架構設計**:

```
                  [Tenant識別中間件]
                         ↓
            [API Gateway - Kong]
                         ↓
        ┌────────────────┼────────────────┐
        ↓                ↓                ↓
   [User Service]  [Billing Svc]   [Analytics Svc]
        ↓                ↓                ↓
   [PostgreSQL]     [PostgreSQL]      [ClickHouse]
   (行級隔離)        (Schema隔離)      (資料倉儲)
```

**關鍵設計**:
1. **租戶識別**: Subdomain + Header
2. **資料隔離**: 行級 (降低成本)
3. **計費**: 獨立Schema (隔離財務資料)
4. **分析**: 單獨資料倉儲 (不影響業務)
5. **成本優化**: 共享基礎設施 + Auto-scaling

---

## 📝 練習題

1. **監控**: 為電商系統設計監控方案 (Metrics + Logs + Traces)
2. **安全**: 設計OAuth 2.0認證流程
3. **成本**: 分析AWS帳單,提出優化建議
4. **微服務**: 將單體電商系統拆分為微服務
5. **多租戶**: 設計支持10000租戶的SaaS平台

---

## 🎓 學習檢查清單

- [ ] 實作Prometheus + Grafana監控系統
- [ ] 配置分散式追蹤 (Jaeger/Zipkin)
- [ ] 實作JWT認證和RBAC授權
- [ ] 分析並優化AWS成本
- [ ] 設計微服務架構並實作API Gateway
- [ ] 理解Service Mesh (Istio)概念
- [ ] 設計多租戶SaaS架構

---

## 📚 延伸閱讀

- **Site Reliability Engineering** - Google
- **Building Microservices** - Sam Newman
- **Observability Engineering** - Charity Majors

---

**上一模組**: [03-Distributed-Systems](../03-Distributed-Systems/README.md)
**返回**: [主目錄](../../README.md)
