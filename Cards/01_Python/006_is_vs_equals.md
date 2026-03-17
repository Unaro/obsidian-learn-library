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
  - is
  - equals
  - comparison
  - "#flashcard/python"
related:
  - - 01_Python_Backend/01_Python_Core#4-1-Операторы-is-и
---

# is vs == в Python

## Вопрос
В чём разница между операторами `is` и == в Python?
?
## Ответ

== — проверяет **равенство значений**:
```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (одинаковые значения)
```

**`is`** — проверяет **идентичность объектов** (один ли адрес в памяти):
```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a
print(a is b)  # False (разные объекты)
print(a is c)  # True (один объект)
```

**Interning (интернирование):**
- Малые целые числа (-5 до 256) интернируются
- Строки интернируются автоматически
```python
a = 256
b = 256
print(a is b)  # True

a = 257
b = 257
print(a is b)  # False (в разных строках кода)
```

**Когда что использовать:**
- == — сравнение значений (почти всегда)
- `is` — проверка на `None` (`if x is None`)

---

#flashcard #python #comparison #is #equals
