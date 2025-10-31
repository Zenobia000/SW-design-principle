# 06 - 繼承進階

## 本章目標
- 深入理解 super() 的工作原理
- 掌握 MRO (Method Resolution Order)
- 學會處理多重繼承
- 理解組合 vs 繼承的選擇

## 核心概念

### 1. super() 的正確用法

```python
class Parent:
    def __init__(self, name):
        self.name = name
        print(f"Parent.__init__({name})")

class Child(Parent):
    def __init__(self, name, age):
        super().__init__(name)  # 調用父類別
        self.age = age
        print(f"Child.__init__({name}, {age})")

c = Child("Alice", 10)
# Parent.__init__(Alice)
# Child.__init__(Alice, 10)
```

### 2. MRO (Method Resolution Order)

**C3 線性化算法:**

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

### 3. 多重繼承

```python
class Flyable:
    def fly(self):
        print("Flying")

class Swimmable:
    def swim(self):
        print("Swimming")

class Duck(Flyable, Swimmable):
    pass

duck = Duck()
duck.fly()   # Flying
duck.swim()  # Swimming
```

### 4. 組合 vs 繼承

**繼承:** "is-a" 關係
```python
class Car(Vehicle):  # Car is a Vehicle
    pass
```

**組合:** "has-a" 關係  
```python
class Car:
    def __init__(self):
        self.engine = Engine()  # Car has an Engine
```

## 火影忍者範例 (詳細版)

參考現有教材: `03-inheritance.ipynb`

- 大筒木輝夜 (Kaguya)
- 大筒木羽衣 (Hagoromo) - 六道仙人
- 阿修羅 (Asura) vs 因陀羅 (Indra)
- 漩渦鳴人 (Naruto) vs 宇智波佐助 (Sasuke)
- 旋渦慕留人 (Boruto) - 多重繼承

## 常見陷阱

### 1. 菱形繼承問題
```python
class A:
    def method(self):
        print("A")

class B(A):
    def method(self):
        print("B")
        super().method()

class C(A):
    def method(self):
        print("C")
        super().method()

class D(B, C):
    def method(self):
        print("D")
        super().method()

d = D()
d.method()
# D -> B -> C -> A (根據 MRO)
```

### 2. super() 不帶參數的陷阱
```python
# Python 3 推薦寫法
class Child(Parent):
    def method(self):
        super().method()  # 自動推斷

# Python 2 相容寫法  
class Child(Parent):
    def method(self):
        super(Child, self).method()  # 明確指定
```

## 最佳實踐

1. **優先使用組合而非繼承**
2. **保持繼承層次簡單(不超過3層)**
3. **使用 super() 而非直接調用父類**
4. **理解 MRO 避免意外行為**

## 延伸閱讀

- 完整範例: `03-inheritance.ipynb`
- Python MRO 官方文檔

**下一章:** 元類與抽象類別
