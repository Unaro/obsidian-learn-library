---
created: 2026-03-17
category: Python
type: flashcard
difficulty: beginner
frequency: high
status: new
next_review:
tags:
  - python
  - decorators
  - flashcard
  - "#flashcard/python"
related:
  - - 01_Python_Backend/01_Python_Core#2-5-Декораторы
---

# Что такое декоратор?

## Вопрос
Что такое декоратор в Python? Как написать свой декоратор?
?
## Ответ

**Декоратор** — функция, которая принимает другую функцию и расширяет её поведение без изменения её кода.

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
```

**С сохранением метаданных:**
```python
from functools import wraps

def logger(func):
    @wraps(func)  # Сохраняет __name__ и __doc__
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper
```

---

#flashcard #python #decorators
