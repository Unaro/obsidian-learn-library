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
  - list
  - tuple
  - "#flashcard/python"
related:
  - - 01_Python_Backend/01_Python_Core#1-2-Списки-vs-Кортежи-Глубокое-сравнение
---

# List vs Tuple

## Вопрос
В чём разница между list и tuple в Python? Когда что использовать?
?
## Ответ

| Характеристика | list | tuple |
|---------------|------|-------|
| **Синтаксис** | `[1, 2, 3]` | `(1, 2, 3)` |
| **Изменяемость** | Изменяемый | Неизменяемый |
| **Производительность** | Медленнее | Быстрее (кэшируются) |
| **Память** | Больше | Меньше |
| **Методы** | `append()`, `pop()`, `sort()` | Нет методов модификации |

**Когда использовать:**
- **list** — динамические данные, нужно изменять размер
- **tuple** — фиксированные данные, ключи dict, возврат нескольких значений

**Пример:**
```python
# list — динамические данные
users = ['Alice', 'Bob', 'Charlie']
users.append('David')

# tuple — фиксированные данные
coordinates = (10.5, 20.3)
color_rgb = (255, 128, 0)

# tuple как ключ dict
locations = {
    (55.75, 37.62): 'Moscow',
    (40.71, -74.01): 'New York'
}
```

---

#flashcard #python #list #tuple
