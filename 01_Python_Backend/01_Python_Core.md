---
created: 2026-03-17
updated: 2026-03-17
category: Python
type: topic
priority: high
status: completed
tags: [python, core, types, oop, memory, gil]
difficulty: intermediate
estimated_hours: 8
---

# Python Core: Полное руководство для собеседования

> **Назначение:** Фундаментальные концепции Python от типов данных до многопоточности. Включает углублённые темы и частые вопросы на собеседованиях.

---

## 1. Типы данных и структуры данных

### 1.1. Мутабельность и иммутабельность

**Неизменяемые (Immutable) типы:**
- `int`, `float`, `complex` — числа
- `bool` — булевы значения
- `str` — строки
- `tuple` — кортежи
- `frozenset` — неизменяемые множества

**Изменяемые (Mutable) типы:**
- `list` — списки
- `set` — множества
- `dict` — словари

```python
# Пример с неизменяемым типом
a = 5
b = a
a = 10
print(b)  # 5 (создан новый объект)

# Пример с изменяемым типом
list1 = [1, 2, 3]
list2 = list1
list1.append(4)
print(list2)  # [1, 2, 3, 4] (изменился тот же объект)
```

**Важно для собеседования:**
- Неизменяемые объекты **потокобезопасны** по умолчанию
- Изменяемые объекты быстрее при модификации (не создают новый объект)
- Неизменяемые типы могут быть **хэшируемыми** (использоваться как ключи dict)

---

### 1.2. Списки vs Кортежи: Глубокое сравнение

| Характеристика | list | tuple |
|---------------|------|-------|
| **Синтаксис** | `[1, 2, 3]` | `(1, 2, 3)` |
| **Изменяемость** | Изменяемый | Неизменяемый |
| **Производительность** | Медленнее | Быстрее (кэшируются) |
| **Память** | Больше (48 байт + элементы) | Меньше (32 байта + элементы) |
| **Методы** | `append()`, `pop()`, `sort()` | Нет методов модификации |
| **Использование** | Коллекции одного типа | Разнородные данные (как struct) |

```python
# tuple быстрее в создании
import timeit
print(timeit.timeit("l = [1, 2, 3, 4, 5]", number=1000000))
print(timeit.timeit("t = (1, 2, 3, 4, 5)", number=1000000))  # ~25% быстрее
```

---

### 1.3. Словари (dict) под капотом

**Структура:** Хэш-таблица с открытой адресацией

**Сложность операций:**
- Вставка: **O(1)** в среднем
- Поиск: **O(1)** в среднем
- Удаление: **O(1)** в среднем
- В худшем случае: **O(n)** при коллизиях

```python
# Порядок сохранения (Python 3.7+)
d = {'z': 1, 'a': 2, 'b': 3}
print(list(d.keys()))  # ['z', 'a', 'b'] — порядок вставки сохраняется!
```

**Важные особенности:**
- Ключи должны быть **хэшируемыми** (неизменяемыми)
- При 2/3 заполнении происходит **ресайз** (удвоение размера)
- Удаление элементов **не уменьшает** память сразу

---

### 1.4. Копирование объектов

```python
import copy

# Поверхностное копирование (shallow)
list1 = [1, 2, [3, 4]]
list2 = list1.copy()  # или list1[:] или copy.copy(list1)
list2[2][0] = 999
print(list1)  # [1, 2, [999, 4]] — вложенный список изменился!

# Глубокое копирование (deep)
list3 = [1, 2, [3, 4]]
list4 = copy.deepcopy(list3)
list4[2][0] = 999
print(list3)  # [1, 2, [3, 4]] — оригинал не изменился
```

**Когда что использовать:**
- `copy()` — для плоских структур, экономия памяти
- `deepcopy()` — для вложенных структур, безопасность

---

## 2. Функции и функциональное программирование

### 2.1. Аргументы функций

```python
# Позиционные аргументы
def func(a, b):
    return a + b

# Именованные аргументы
func(a=1, b=2)

# *args — переменное число позиционных
def sum_all(*args):
    return sum(args)
sum_all(1, 2, 3, 4)  # 10

# **kwargs — переменное число именованных
def print_kwargs(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")
print_kwargs(name="John", age=30)

# Комбинирование
def complex_func(a, b, *args, c=10, **kwargs):
    pass
```

**⚠️ Ловушка с изменяемыми аргументами по умолчанию:**

```python
# НЕПРАВИЛЬНО
def append_to_list(value, lst=[]):  # Список создаётся ОДИН раз при определении!
    lst.append(value)
    return lst

print(append_to_list(1))  # [1]
print(append_to_list(2))  # [1, 2] — а не [2]!

# ПРАВИЛЬНО
def append_to_list(value, lst=None):
    if lst is None:
        lst = []
    lst.append(value)
    return lst
```

---

### 2.2. Lambda-функции

```python
# Синтаксис: lambda аргументы: выражение
square = lambda x: x ** 2
print(square(5))  # 25

# Использование с sorted()
students = [('Alice', 25), ('Bob', 20), ('Charlie', 23)]
sorted_students = sorted(students, key=lambda x: x[1])

# Использование с filter()
numbers = [1, 2, 3, 4, 5, 6]
even = list(filter(lambda x: x % 2 == 0, numbers))
```

**Ограничения:**
- Только **одно выражение**
- Не подходят для сложной логики
- Сложнее отлаживать

---

### 2.3. Итераторы и Генераторы

**Итератор** — объект с методами `__iter__()` и `__next__()`:

```python
class Counter:
    def __init__(self, start, end):
        self.current = start
        self.end = end
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.current >= self.end:
            raise StopIteration
        self.current += 1
        return self.current - 1

counter = Counter(0, 5)
for num in counter:
    print(num)  # 0, 1, 2, 3, 4
```

**Генератор** — функция с `yield`:

```python
def counter_generator(start, end):
    current = start
    while current < end:
        yield current  # Сохраняет состояние и возвращает значение
        current += 1

# Генераторное выражение (экономия памяти)
squares = (x**2 for x in range(1000000))  # Не создаёт список в памяти
```

**Сравнение:**

| Характеристика | Итератор             | Генератор         |
| -------------- | -------------------- | ----------------- |
| **Синтаксис**  | Класс с `__next__()` | Функция с `yield` |
| **Состояние**  | Вручную в атрибутах  | Автоматически     |
| **Память**     | Экономно             | Очень экономно    |
| **Сложность**  | Сложнее              | Проще             |

---

### 2.4. Область видимости (LEGB Rule)

**Порядок поиска имён:**

```python
# L — Local (локальная)
# E — Enclosing (замыкание)
# G — Global (глобальная)
# B — Built-in (встроенная)

x = "global"

def outer():
    x = "enclosing"
    
    def inner():
        x = "local"
        print(x)  # L: "local"
    
    inner()
    print(x)  # E: "enclosing"

outer()
print(x)  # G: "global"
```

**Ключевые слова `global` и `nonlocal`:**

```python
# global — изменение глобальной переменной
count = 0
def increment():
    global count
    count += 1

# nonlocal — изменение переменной из замыкания
def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1
    return inner
```

---

### 2.5. Декораторы

**Базовый декоратор:**

```python
def logger(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"Finished {func.__name__}")
        return result
    return wrapper

@logger
def say_hello(name):
    print(f"Hello, {name}!")

say_hello("Alice")
```

**Декоратор с параметрами:**

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            for _ in range(times):
                func(*args, **kwargs)
        return wrapper
    return decorator

@repeat(3)
def greet():
    print("Hello!")

greet()  # Напечатает "Hello!" 3 раза
```

**Декоратор с сохранением метаданных:**

```python
from functools import wraps

def logger(func):
    @wraps(func)  # Сохраняет __name__ и __doc__
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

## 3. Объектно-ориентированное программирование (ООП)

### 3.1. Создание экземпляра: `__new__` vs `__init__`

```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("1. __new__ — создание экземпляра")
        instance = super().__new__(cls)
        return instance
    
    def __init__(self, value):
        print("2. __init__ — инициализация")
        self.value = value

obj = MyClass(42)
```

**Когда использовать `__new__`:**
- Наследование от неизменяемых типов (`str`, `int`, `tuple`)
- Паттерн **Singleton**
- Метапрограммирование

---

### 3.2. Инкапсуляция и модификаторы доступа

```python
class BankAccount:
    def __init__(self):
        self.public = 100      # Публичный атрибут
        self._protected = 200  # Protected (конвенция, не защита!)
        self.__private = 300   # Private (name mangling!)
    
    def get_private(self):
        return self.__private

account = BankAccount()
print(account.public)      # 100
print(account._protected)  # 200 (можно, но не рекомендуется)
# print(account.__private)  # AttributeError!
print(account._BankAccount__private)  # 300 (обход mangling)
```

**Name Mangling:** `__private` → `_ClassName__private`

---

### 3.3. Декораторы методов класса

```python
class MyClass:
    class_var = "I'm a class variable"
    
    def __init__(self, value):
        self.instance_var = value
    
    # Обычный метод (работает с экземпляром)
    def instance_method(self):
        return self.instance_var
    
    # Метод класса (работает с классом)
    @classmethod
    def class_method(cls):
        return cls.class_var
    
    # Статический метод (не зависит от класса/экземпляра)
    @staticmethod
    def static_method(x, y):
        return x + y
    
    # Property (геттер/сеттер)
    @property
    def computed_property(self):
        return self.instance_var * 2
    
    @computed_property.setter
    def computed_property(self, value):
        self.instance_var = value // 2

print(MyClass.class_method())  # Доступ без экземпляра
print(MyClass.static_method(2, 3))  # 5
```

---

### 3.4. Дандер-методы (Магические методы)

**Сравнение:**
```python
class Number:
    def __init__(self, value):
        self.value = value
    
    def __eq__(self, other):      # ==
        return self.value == other.value
    
    def __lt__(self, other):      # <
        return self.value < other.value
    
    def __le__(self, other):      # <=
        return self.value <= other.value
```

**Арифметика:**
```python
    def __add__(self, other):     # +
        return Number(self.value + other.value)
    
    def __sub__(self, other):     # -
        return Number(self.value - other.value)
    
    def __mul__(self, other):     # *
        return Number(self.value * other.value)
```

**Представление:**
```python
    def __str__(self):            # str(), print()
        return f"Number({self.value})"
    
    def __repr__(self):           # repr(), в интерпретаторе
        return f"Number({self.value})"
    
    def __len__(self):            # len()
        return self.value
```

**Контекстный менеджер:**
```python
    def __enter__(self):
        print("Entering context")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        print("Exiting context")
```

---

### 3.5. Абстрактные классы

```python
from abc import ABC, abstractmethod

class Shape(ABC):  # Абстрактный базовый класс
    @abstractmethod
    def area(self):
        pass
    
    @abstractmethod
    def perimeter(self):
        pass
    
    def describe(self):  # Обычный метод (не абстрактный)
        return "I'm a shape"

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height
    
    def perimeter(self):
        return 2 * (self.width + self.height)

# shape = Shape()  # TypeError: нельзя создать экземпляр абстрактного класса
rect = Rectangle(5, 3)  # OK
```

---

## 4. Управление памятью

### 4.1. Операторы `is` и ==

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)  # True (равные значения)
print(a is b)  # False (разные объекты в памяти)
print(a is c)  # True (один объект)
```

**Interning (интернирование):**
```python
# Малые целые числа интернируются (-5 до 256)
a = 256
b = 256
print(a is b)  # True

a = 257
b = 257
print(a is b)  # False (в разных строках кода)
print(a is b)  # True (в одной строке — оптимизация компилятора)

# Строки интернируются автоматически
a = "hello"
b = "hello"
print(a is b)  # True
```

---

### 4.2. Сборщик мусора (Garbage Collection)

**Алгоритмы:**
1. **Reference Counting** — подсчёт ссылок (основной)
2. **Generational GC** — поколенческая сборка (для циклических ссылок)

```python
import sys
import gc

a = []
print(sys.getrefcount(a))  # 2 (ссылка + аргумент getrefcount)

# Циклическая ссылка
class Node:
    def __init__(self):
        self.ref = None

node1 = Node()
node2 = Node()
node1.ref = node2
node2.ref = node1

del node1, node2  # Reference counting не освободит память!
gc.collect()  # Принудительный запуск GC
```

---

### 4.3. `__slots__` для оптимизации памяти

```python
class WithoutSlots:
    def __init__(self):
        self.x = 1
        self.y = 2

class WithSlots:
    __slots__ = ['x', 'y']  # Фиксированный набор атрибутов
    
    def __init__(self):
        self.x = 1
        self.y = 2

# WithSlots использует ~40% меньше памяти
# НО: нельзя добавлять новые атрибуты динамически
```

---

## 5. Конкурентность и многопоточность

### 5.1. GIL (Global Interpreter Lock)

**Проблема:**
```python
import threading
import time

counter = 0

def increment():
    global counter
    for _ in range(1000000):
        counter += 1  # Не атомарная операция!

t1 = threading.Thread(target=increment)
t2 = threading.Thread(target=increment)
t1.start()
t2.start()
t1.join()
t2.join()

print(counter)  # Может быть меньше 2000000 из-за гонки!
```

**Решение — Lock:**
```python
from threading import Lock

lock = Lock()
counter = 0

def increment():
    global counter
    for _ in range(1000000):
        with lock:  # Блокировка
            counter += 1
```

---

### 5.2. Threading vs Multiprocessing

| Характеристика | Threading | Multiprocessing |
|---------------|-----------|-----------------|
| **GIL** | Затрагивает | Обходит |
| **Память** | Общая | Изолированная |
| **Производительность** | I/O-bound | CPU-bound |
| **Создание** | Быстрое | Медленное |
| **IPC** | Не нужно | Нужно (Queue, Pipe) |

**Threading (I/O-bound):**
```python
from threading import Thread
import requests

def fetch(url):
    response = requests.get(url)
    print(f"{url}: {len(response.content)}")

urls = ['http://example.com'] * 5
threads = [Thread(target=fetch, args=(url,)) for url in urls]
for t in threads: t.start()
for t in threads: t.join()
```

**Multiprocessing (CPU-bound):**
```python
from multiprocessing import Process
import os

def cpu_intensive(n):
    return sum(i * i for i in range(n))

processes = [Process(target=cpu_intensive, args=(10**7,)) for _ in range(4)]
for p in processes: p.start()
for p in processes: p.join()
```

---

### 5.3. Asyncio (асинхронность)

```python
import asyncio

async def fetch_data(delay):
    print("Start fetching")
    await asyncio.sleep(delay)  # Не блокирует!
    print("Done fetching")
    return {"data": "result"}

async def main():
    # Параллельное выполнение
    results = await asyncio.gather(
        fetch_data(2),
        fetch_data(1),
        fetch_data(3)
    )
    print(results)

asyncio.run(main())
```

**Сравнение подходов:**

| Задача | Решение |
|--------|---------|
| Сетевые запросы | `asyncio` / `threading` |
| Работа с файлами | `multiprocessing` |
| Вычисления | `multiprocessing` |
| Смешанная нагрузка | `asyncio` + `multiprocessing` |

---

## 🔗 Связанные темы

- [[01_Python_Backend/02_Django_REST_API]] — Django ORM
- [[04_Databases/01_Databases_ORM_SQL]] — Базы данных
- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Event Loop для сравнения

---

*Файл обновлён: 17 марта 2026*
