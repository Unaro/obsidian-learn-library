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
  - types
  - mutability
  - "#flashcard/python"
related:
  - - 01_Python_Backend/01_Python_Core#1-Типы-данных-и-структуры-данных
---

# Мутабельность типов в Python

## Вопрос
В чём разница между изменяемыми и неизменяемыми типами в Python? Приведите примеры.
?
## Ответ

**Неизменяемые (Immutable):**
- При изменении создаётся **новый объект** в памяти
- Примеры: `int`, `float`, `str`, `tuple`, `frozenset`, `bool`

**Изменяемые (Mutable):**
- Изменяются **на месте** без создания нового объекта
- Примеры: `list`, `dict`, `set`

**Пример:**
```python
# Неизменяемый
a = 5
b = a
a = 10
print(b)  # 5 (старый объект не изменился)

# Изменяемый
list1 = [1, 2, 3]
list2 = list1
list1.append(4)
print(list2)  # [1, 2, 3, 4] (изменился тот же объект)
```

**Важно для собеседования:**
- Неизменяемые типы **потокобезопасны** по умолчанию
- Неизменяемые типы могут быть **ключами dict** (хэшируемые)
- Изменяемые быстрее при модификации

---

#flashcard #python #types
