---
created: 2026-03-17
category: Python
type: flashcard
difficulty: beginner
frequency: medium
status: new
next_review:
tags:
  - python
  - args
  - kwargs
  - "#flashcard/python"
related:
  - - 01_Python_Backend/01_Python_Core#2-1-Аргументы-функций
---

# Что такое *args и **kwargs?

## Вопрос
Что означают `*args` и `**kwargs` в Python? Когда используются?
?
## Ответ

**`*args`** — переменное число позиционных аргументов (передаются как кортеж):
```python
def sum_all(*args):
    return sum(args)

sum_all(1, 2, 3, 4)  # 10
```

**`**kwargs`** — переменное число именованных аргументов (передаются как словарь):
```python
def print_kwargs(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")

print_kwargs(name="John", age=30)
# name: John
# age: 30
```

**Комбинирование:**
```python
def complex_func(a, b, *args, c=10, **kwargs):
    pass
```

**Порядок параметров:**
1. Позиционные (`a`, `b`)
2. `*args`
3. Именованные (`c=10`)
4. `**kwargs`

---

#flashcard #python #functions #args
