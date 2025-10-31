# 09 - 迭代器與生成器

## 本章目標
- 理解迭代器協議
- 掌握 yield 關鍵字
- 學會使用生成器優化記憶體
- 理解惰性求值的優勢

## 核心概念

### 1. 迭代器 (Iterator)

實作 `__iter__()` 和 `__next__()` 的物件

```python
class Counter:
    def __init__(self, max):
        self.max = max
        self.current = 0
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current < self.max:
            self.current += 1
            return self.current
        raise StopIteration

counter = Counter(3)
for num in counter:
    print(num)  # 1, 2, 3
```

### 2. 生成器 (Generator)

使用 `yield` 關鍵字的函數

```python
def count_up_to(max):
    count = 1
    while count <= max:
        yield count
        count += 1

for num in count_up_to(3):
    print(num)  # 1, 2, 3
```

### 3. 生成器表達式

```python
# 列表推導式 (一次性創建所有元素)
squares_list = [x**2 for x in range(1000000)]

# 生成器表達式 (惰性求值)
squares_gen = (x**2 for x in range(1000000))
```

### 4. yield from

```python
def flatten(nested_list):
    for item in nested_list:
        if isinstance(item, list):
            yield from flatten(item)  # 遞迴
        else:
            yield item

nested = [1, [2, 3, [4, 5]], 6]
flat = list(flatten(nested))
print(flat)  # [1, 2, 3, 4, 5, 6]
```

## 實戰範例

### 分頁查詢生成器
```python
def paginated_query(query_func, page_size=100):
    page = 0
    while True:
        results = query_func(page, page_size)
        if not results:
            break
        
        for item in results:
            yield item
        
        page += 1

# 使用
for user in paginated_query(get_users, page_size=50):
    process_user(user)
```

### 檔案讀取生成器
```python
def read_large_file(file_path):
    with open(file_path, 'r') as f:
        for line in f:
            # 惰性讀取,不會一次載入整個檔案
            yield line.strip()

for line in read_large_file('huge_file.txt'):
    process_line(line)
```

### 無限序列
```python
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# 取前10個費波那契數
fib = fibonacci()
first_10 = [next(fib) for _ in range(10)]
print(first_10)
```

## 記憶體優勢

```python
import sys

# 列表:一次性創建所有元素
numbers_list = [x for x in range(1000000)]
print(sys.getsizeof(numbers_list))  # ~8MB

# 生成器:惰性求值
numbers_gen = (x for x in range(1000000))
print(sys.getsizeof(numbers_gen))   # ~128 bytes
```

## 迭代器 vs 生成器

| 特性 | 迭代器 | 生成器 |
|------|--------|--------|
| 定義方式 | 實作 `__iter__` 和 `__next__` | 使用 `yield` |
| 程式碼量 | 較多 | 較少 |
| 靈活性 | 高 | 中 |
| 易用性 | 中 | 高 |

## 常見陷阱

```python
# 生成器只能遍歷一次
gen = (x for x in range(3))
list(gen)  # [0, 1, 2]
list(gen)  # [] (已耗盡)

# 需要重新創建
gen = (x for x in range(3))
```

## 延伸閱讀

- 完整範例: `06-iterator_and_generator.ipynb`
- Python Iterator Protocol 文檔

**下一章:** SOLID 原則
