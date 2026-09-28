---
source_sha: 06b3e7f0f7ef0f6608495df323452a4c3435d9407a436b6f4d49e1cbd2383cfb
translated: machine
---

# callable() 函数的复杂度

`callable()` 函数检查一个对象是否可调用（即能否以参数形式调用）。

## 复杂度分析

| 情况 | 时间 | 空间 | 备注 |
|------|------|-------|-------|
| 检查可调用状态 | O(1) | O(1) | 简单的属性检查 |
| 定义了 `__call__` 的对象 | O(1) | O(1) | 直接属性查找 |
| 内置类型 | O(1) | O(1) | 常数时间检查 |

## 基本用法

### 检查函数与方法

```python
# O(1) - constant time check
def my_function():
    pass

callable(my_function)      # True
callable(len)              # True
callable(str.upper)        # True (bound method)
callable(str.split)        # True (unbound method)
```

### 检查不可调用对象

```python
# O(1) - returns False
callable(42)               # False
callable("string")         # False
callable([1, 2, 3])        # False
callable({"key": "value"}) # False
callable(None)             # False
```

### 类与实例

```python
# O(1) - classes are callable
class MyClass:
    pass

callable(MyClass)          # True (can instantiate)

instance = MyClass()
callable(instance)         # False (unless __call__ defined)

# With __call__ method
class Callable:
    def __call__(self):
        return "called"

obj = Callable()
callable(obj)              # True (has __call__)
```

## 复杂度细节

### callable() 的工作原理

```python
# O(1) - checks for __call__ method in type hierarchy
# Note: This is a simplified conceptual model; CPython has optimized checks
def is_callable(obj):
    return hasattr(type(obj), '__call__')

# callable() is more efficient than this Python equivalent
```

### 属性查找

```python
# O(1) - single attribute check
# callable() essentially checks:
# 1. obj has __call__ method
# 2. obj is a type/class
# 3. obj is a built-in function

# All constant time operations
```

## 常见模式

### 函数分发

```python
# O(1) - check before calling
def execute_if_callable(func, *args):
    if callable(func):
        return func(*args)
    else:
        return func

result = execute_if_callable(len, [1, 2, 3])  # 3
value = execute_if_callable(42)  # 42
```

### 回调校验

```python
# O(1) - validate callback parameter
def register_callback(callback):
    if not callable(callback):
        raise TypeError(f"callback must be callable, got {type(callback)}")
    # Register callback
    return callback

# Valid
register_callback(lambda x: x + 1)  # OK

# Invalid
try:
    register_callback(42)  # TypeError
except TypeError as e:
    print(e)
```

### 多态处理

```python
# O(1) - per item, handle both callables and values
def process(items):
    results = []
    for item in items:
        if callable(item):
            results.append(item())  # Call it
        else:
            results.append(item)    # Use as-is
    return results

values = [42, lambda: 100, "string", len]
# results = [42, 100, "string", <function len>]
```

## Python 中的可调用对象

### 函数

```python
# All are callable O(1)

# Regular function
def func():
    pass
callable(func)  # True

# Lambda
f = lambda x: x + 1
callable(f)  # True

# Built-in function
callable(len)  # True
callable(print)  # True

# Method
obj = "hello"
callable(obj.upper)  # True
```

### 类

```python
# O(1) - classes are callable (they instantiate)

class MyClass:
    pass

callable(MyClass)  # True

# Built-in types
callable(int)      # True
callable(str)      # True
callable(list)     # True
callable(dict)     # True
```

### 定义了 __call__ 的对象

```python
# O(1) - check for __call__ method

class Callable:
    def __call__(self):
        return "I was called"

obj = Callable()
callable(obj)  # True

result = obj()  # "I was called"
```

## 性能模式

### 与 isinstance() 检查的对比

```python
# Both O(1), but different purposes
import types

obj = lambda x: x + 1

# callable() - high level
if callable(obj):
    pass

# isinstance() - more specific
if isinstance(obj, types.FunctionType):
    pass

# callable() is cleaner for just checking if callable
```

### 批量处理

```python
# O(n) - check n items
# n calls to callable(), each O(1)

items = [42, lambda: 1, "str", len, [1, 2]]
callables = [x for x in items if callable(x)]
# O(n) total - [<lambda>, len]
```

## 边界情况

### 偏函数

```python
from functools import partial

# O(1) - partial objects are callable
add_five = partial(lambda x, y: x + y, 5)
callable(add_five)  # True

result = add_five(10)  # 15
```

### 类方法与静态方法

```python
# O(1) - decorators preserve callability

class Example:
    @classmethod
    def class_method(cls):
        return "class"
    
    @staticmethod
    def static_method():
        return "static"
    
    def instance_method(self):
        return "instance"

callable(Example.class_method)      # True
callable(Example.static_method)     # True
callable(Example.instance_method)   # True
```

### 生成器与迭代器

```python
# O(1) - functions are callable, but generators are not

def generator():
    yield 1

callable(generator)     # True (it's the function)
callable(generator())   # False (generator object not callable)

# Generator object has __iter__ and __next__, not __call__
```

### 内置类型

```python
# O(1) - type check

class NotType:
    pass

# Instances - not callable
instance = NotType()
callable(instance)  # False

# Class - callable
callable(NotType)  # True
```

## 实用示例

### 惰性求值

```python
# O(1) - check if value or factory
def lazy_value(obj):
    if callable(obj):
        return obj()  # Call factory
    else:
        return obj    # Return value directly

# Usage
value1 = lazy_value(42)              # 42
value2 = lazy_value(lambda: 42)      # 42 (factory called)

# Useful for configuration
config = {
    'static': 100,
    'dynamic': lambda: expensive_calculation()
}

for key, val in config.items():
    print(f"{key}: {lazy_value(val)}")
```

### 函数组合

```python
# O(1) - validate functions before composing
def compose(f, g):
    if not (callable(f) and callable(g)):
        raise TypeError("Both arguments must be callable")
    
    return lambda x: f(g(x))

square = lambda x: x ** 2
double = lambda x: x * 2

composed = compose(square, double)
result = composed(5)  # (5 * 2) ** 2 = 100
```

### 事件处理器注册

```python
# O(1) - validate event handler
class EventManager:
    def __init__(self):
        self.handlers = []
    
    def register(self, handler):
        if not callable(handler):
            raise TypeError("Handler must be callable")
        self.handlers.append(handler)
    
    def emit(self, event):
        for handler in self.handlers:
            handler(event)

manager = EventManager()
manager.register(lambda e: print(f"Event: {e}"))
manager.emit("click")  # Output: Event: click
```

## 最佳实践

✅ **应该**：

- 在调用未知对象之前用 `callable()` 做检查
- 在回调/处理器校验中使用
- 在文档中说明哪些参数应当是可调用的
- 用于防御式编程

❌ **避免**：

- 不加检查就假设对象可调用
- 用 `callable()` 做类型区分（应该用 `isinstance()`）
- 简单函数就够用时把事情复杂化成 `__call__`
- 在性能关键路径上添加不必要的 `callable()` 检查

## 相关函数

- **[hasattr()](index.md)** - 检查属性是否存在
- **[isinstance()](isinstance.md)** - 检查对象类型
- **[type()](type_func.md)** - 获取对象类型
- **[inspect.isfunction()](https://docs.python.org/3/library/inspect.html)** - 详细的类型检查

## 版本说明

- **Python 2.x**：`callable()` 作为内置函数可用
- **Python 3.0-3.1**：被移除（一个有争议的决定）
- **Python 3.2+**：重新作为内置函数加入
- **所有版本**：O(1) 常数时间操作
