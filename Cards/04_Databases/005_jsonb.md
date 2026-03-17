---
created: 2026-03-17
category: Databases
type: flashcard
difficulty: advanced
frequency: low
status: new
next_review: 
tags: [postgresql, jsonb, nosql, flashcard/databases]
related: [[04_Databases/03_Advanced_Database_Offline#1-1-JSONB-Гибридное-хранение-данных]]
---

# Что такое JSONB в PostgreSQL?

## Вопрос
Что такое JSONB в PostgreSQL? Чем отличается от JSON?
?
## Ответ

**JSONB (Binary JSON)** — бинарное представление JSON в PostgreSQL.

**Отличия от JSON:**
| Характеристика | JSON | JSONB |
|---------------|------|-------|
| **Хранение** | Текст | Бинарный формат |
| **Парсинг** | При каждом запросе | Распарсен заранее |
| **Индексы** | Нет | ✅ GIN индексы |
| **Запись** | Быстрее | Медленнее (преобразование) |
| **Чтение** | Медленнее | Быстрее |

**Пример использования:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    settings JSONB DEFAULT '{}'::jsonb
);

-- Вставка
INSERT INTO users (username, settings) VALUES (
    'john',
    '{"theme": "dark", "notifications": {"email": true}}'::jsonb
);

-- Запрос к JSONB
SELECT username, settings->>'theme' as theme
FROM users
WHERE settings->>'language' = 'en';

-- GIN индекс для быстрого поиска
CREATE INDEX idx_users_settings ON users USING GIN (settings);
```

**Когда использовать:**
- ✅ Динамические атрибуты
- ✅ Кастомные настройки пользователя
- ✅ Сырые логи
- ❌ Когда нужна строгая схема

---

#flashcard #postgresql #jsonb #nosql
