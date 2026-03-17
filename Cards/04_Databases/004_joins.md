---
created: 2026-03-17
category: Databases
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [sql, join, flashcard/databases]
related: [[04_Databases/01_Databases_ORM_SQL#5-JOIN-и-связи]]
---

# Типы JOIN в SQL

## Вопрос
Какие типы JOIN существуют в SQL? В чём разница?
?
## Ответ

**INNER JOIN:**
- Только совпадающие записи из обеих таблиц
```sql
SELECT * FROM users
INNER JOIN orders ON users.id = orders.user_id;
-- Только пользователи с заказами
```

**LEFT JOIN (LEFT OUTER JOIN):**
- Все записи из левой таблицы + совпадающие из правой
```sql
SELECT * FROM users
LEFT JOIN orders ON users.id = orders.user_id;
-- Все пользователи, даже без заказов
```

**RIGHT JOIN:**
- Все записи из правой таблицы + совпадающие из левой
```sql
SELECT * FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
-- Все заказы, даже без пользователей
```

**FULL JOIN:**
- Все записи из обеих таблиц
```sql
SELECT * FROM users
FULL JOIN orders ON users.id = orders.user_id;
-- Все пользователи и все заказы
```

**Визуализация:**
```
INNER:   ∩ (пересечение)
LEFT:    ← (всё слева + пересечение)
RIGHT:   → (всё справа + пересечение)
FULL:    ∪ (объединение)
```

---

#flashcard #sql #join #relational
