# 07 - 元類與抽象類別

## 本章目標
- 理解元類(Metaclass)的概念
- 掌握 ABC (Abstract Base Class)
- 學會設計抽象介面
- 理解 type-object-class 關係

## 核心概念

### 1. 什麼是元類?

**元類是創建類別的類別**

```python
# 所有類別都是 type 的實例
class MyClass:
    pass

print(type(MyClass))  # <class 'type'>
print(isinstance(MyClass, type))  # True
```

### 2. 使用 type() 動態創建類別

```python
# 方法1: 正常定義
class Product:
    category = "Electronics"

# 方法2: 使用 type() 動態創建
Product = type('Product', (object,), {'category': 'Electronics'})
```

### 3. 自訂元類

```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        # 在創建類別時自動添加屬性
        dct['created_by'] = 'Metaclass'
        return super().__new__(cls, name, bases, dct)

class MyClass(metaclass=Meta):
    pass

print(MyClass.created_by)  # Metaclass
```

### 4. ABC (Abstract Base Class)

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# shape = Shape()  # TypeError: 無法實例化抽象類別
rect = Rectangle(10, 5)  # OK
```

## 實戰範例

### 動物園管理系統

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass
    
    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def make_sound(self):
        return "Woof!"
    
    def move(self):
        return "Running"

class Cat(Animal):
    def make_sound(self):
        return "Meow!"
    
    def move(self):
        return "Walking"

# 多型應用
animals = [Dog(), Cat()]
for animal in animals:
    print(f"{animal.make_sound()} - {animal.move()}")
```

### ORM 設計

```python
class ModelMeta(type):
    def __new__(cls, name, bases, dct):
        # 自動收集欄位
        fields = {}
        for key, value in dct.items():
            if isinstance(value, Field):
                fields[key] = value
        
        dct['_fields'] = fields
        return super().__new__(cls, name, bases, dct)

class Model(metaclass=ModelMeta):
    pass

class Field:
    pass

class User(Model):
    name = Field()
    email = Field()

print(User._fields)  # 自動收集的欄位
```

## 為什麼需要抽象類別?

1. **定義契約:** 確保子類別實作特定方法
2. **多型基礎:** 統一介面
3. **文檔化:** 清楚說明需要實作什麼
4. **早期錯誤檢測:** 在類別定義時而非執行時發現問題

## 常見陷阱

```python
# ❌ 忘記實作抽象方法
class BadShape(Shape):
    def area(self):  # 只實作一個
        return 0
    # 缺少 perimeter()

# shape = BadShape()  # TypeError!
```

## 延伸閱讀

- 詳細元類教材: `03.1-meta-Class.ipynb`
- 抽象類別實戰: `04-abstract-Class.ipynb`

**下一章:** 裝飾器模式
