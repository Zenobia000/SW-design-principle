# 12 - Library, Framework, API

## 本章目標
- 理解 Library, Framework, API 的差異
- 學會設計可重用的 Library
- 理解框架的控制反轉(IoC)
- 掌握 API 設計原則

## 核心概念

### 1. Library (函式庫)

**定義:** 提供特定功能的程式碼集合,由你的程式碼主動調用

**特點:**
- 你控制程式流程
- 主動調用 Library 的功能
- 可選擇性使用

**範例:**
```python
# requests 是一個 Library
import requests

# 你控制何時調用
response = requests.get('https://api.example.com')
data = response.json()
```

**常見 Python Library:**
- requests (HTTP)
- numpy (數值計算)
- pandas (數據分析)
- matplotlib (繪圖)

### 2. Framework (框架)

**定義:** 提供應用程式骨架,控制程式流程,你填充具體邏輯

**特點:**
- Framework 控制程式流程(控制反轉)
- 你實作特定的鉤子或回調
- 必須遵循 Framework 的規範

**範例:**
```python
# Django 是一個 Framework
from django.http import HttpResponse
from django.views import View

# Framework 決定何時調用你的程式碼
class MyView(View):
    def get(self, request):
        # Framework 調用這個方法
        return HttpResponse("Hello")
```

**常見 Python Framework:**
- Django (Web)
- Flask (Web)
- FastAPI (API)
- pytest (測試)

### 3. API (應用程式介面)

**定義:** 程式間溝通的介面和規範

**類型:**
- **Library API:** 函式和類別的使用介面
- **Web API:** HTTP 端點
- **System API:** 作業系統提供的介面

**範例:**
```python
# RESTful API 設計
class UserAPI:
    def get_user(self, user_id):
        """GET /users/{id}"""
        pass
    
    def create_user(self, data):
        """POST /users"""
        pass
    
    def update_user(self, user_id, data):
        """PUT /users/{id}"""
        pass
    
    def delete_user(self, user_id):
        """DELETE /users/{id}"""
        pass
```

## 差異對比

| 特性 | Library | Framework | API |
|------|---------|-----------|-----|
| 控制流程 | 你控制 | Framework 控制 | 定義介面 |
| 使用方式 | 主動調用 | 被動實作 | 遵循規範 |
| 靈活性 | 高 | 中 | 看設計 |
| 學習曲線 | 低 | 高 | 中 |

## 設計 Library 的原則

### 1. 單一職責
```python
# ✅ 好的設計
class JSONParser:
    def parse(self, json_string):
        pass

# ❌ 不好的設計
class DataProcessor:
    def parse_json(self): pass
    def parse_xml(self): pass
    def send_email(self): pass  # 職責混雜
```

### 2. 清晰的介面
```python
class DataValidator:
    def validate(self, data):
        """清晰的方法名和參數"""
        pass
    
    def is_valid_email(self, email):
        """描述性的方法名"""
        pass
```

### 3. 文檔齊全
```python
def calculate_discount(price, rate):
    """
    計算折扣價格
    
    Args:
        price (float): 原始價格
        rate (float): 折扣率 (0-1)
    
    Returns:
        float: 折扣後價格
    
    Raises:
        ValueError: 當 rate 不在 0-1 範圍時
    
    Example:
        >>> calculate_discount(1000, 0.1)
        900.0
    """
    if not 0 <= rate <= 1:
        raise ValueError("Rate must be between 0 and 1")
    return price * (1 - rate)
```

## API 設計最佳實踐

### 1. RESTful 設計
```python
# 資源導向的 URL 設計
GET    /users          # 獲取用戶列表
GET    /users/123      # 獲取單個用戶
POST   /users          # 創建用戶
PUT    /users/123      # 更新用戶
DELETE /users/123      # 刪除用戶
```

### 2. 一致性
```python
class API:
    # ✅ 一致的命名
    def get_user(self, id): pass
    def get_product(self, id): pass
    def get_order(self, id): pass
    
    # ❌ 不一致的命名
    def get_user(self, id): pass
    def fetch_product(self, id): pass
    def retrieve_order(self, id): pass
```

### 3. 錯誤處理
```python
class APIException(Exception):
    def __init__(self, message, status_code):
        self.message = message
        self.status_code = status_code

class UserAPI:
    def get_user(self, user_id):
        if not user_exists(user_id):
            raise APIException("User not found", 404)
        return get_user_data(user_id)
```

## 與 System Design 的連結

在系統設計中:

1. **Library** → 共享組件
2. **Framework** → 微服務模板
3. **API** → 服務間通訊介面

## 練習題

1. 設計一個簡單的 HTTP client library
2. 實作一個測試框架的基本結構
3. 設計一個 RESTful API 用於電商系統

## 總結

理解 Library, Framework, API 的差異有助於:
- 選擇合適的工具
- 設計可重用的程式碼
- 構建可維護的系統

**參考:** 現有教材 `00-library-framework-API.ipynb`

**下一章:** 從 OOP 到 System Design - 完整銜接
