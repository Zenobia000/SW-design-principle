# 08 - 裝飾器模式

## 本章目標
- 理解裝飾器的本質
- 掌握 @property, @classmethod, @staticmethod
- 學會撰寫自訂裝飾器
- 理解 functools.wraps 的作用

## 核心概念

### 1. 什麼是裝飾器?

裝飾器是一個函數,接收另一個函數並返回修改後的函數。

```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
# Before
# Hello!
# After
```

### 2. 內建裝飾器

#### @property
```python
class Product:
    def __init__(self, price):
        self._price = price
    
    @property
    def price(self):
        return self._price
    
    @price.setter
    def price(self, value):
        if value < 0:
            raise ValueError("Price cannot be negative")
        self._price = value

p = Product(100)
print(p.price)  # 100
p.price = 200   # 使用 setter
```

#### @classmethod 和 @staticmethod
```python
class MyClass:
    count = 0
    
    @classmethod
    def increment_count(cls):
        cls.count += 1
    
    @staticmethod
    def utility_function():
        return "Utility"
```

### 3. 實用裝飾器

#### 計時裝飾器
```python
import time
from functools import wraps

def timer(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end-start:.2f}s")
        return result
    return wrapper

@timer
def slow_function():
    time.sleep(1)
    return "Done"
```

#### 快取裝飾器
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

#### 日誌裝飾器
```python
def log(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper
```

### 4. 帶參數的裝飾器

```python
def repeat(times):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello {name}")

greet("Alice")
# Hello Alice
# Hello Alice
# Hello Alice
```

## 實戰範例

### 權限檢查裝飾器
```python
def require_auth(func):
    @wraps(func)
    def wrapper(user, *args, **kwargs):
        if not user.is_authenticated:
            raise PermissionError("Not authenticated")
        return func(user, *args, **kwargs)
    return wrapper

@require_auth
def delete_user(user, user_id):
    # 刪除用戶
    pass
```

### 重試裝飾器
```python
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Retry {attempt + 1}/{max_attempts}")
            return wrapper
    return decorator

@retry(3)
def unstable_api_call():
    # 可能失敗的 API 調用
    pass
```

## 為什麼使用 functools.wraps?

```python
def bad_decorator(func):
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

def good_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@bad_decorator
def my_function():
    """My docstring"""
    pass

print(my_function.__name__)  # wrapper (錯誤!)
print(my_function.__doc__)   # None (遺失!)

@good_decorator  
def my_function():
    """My docstring"""
    pass

print(my_function.__name__)  # my_function (正確!)
print(my_function.__doc__)   # My docstring (保留!)
```

## 延伸閱讀

- 完整範例: `05-decorator.ipynb`
- Python functools 文檔

**下一章:** 迭代器與生成器
