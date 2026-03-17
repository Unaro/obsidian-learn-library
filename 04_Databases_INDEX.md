# 🗄️ Базы Данных & Оптимизация

> **Назначение:** Навигация по темам баз данных, ORM и оптимизации запросов

---

## 📁 Файлы категории

| № | Файл | Описание |
|---|------|----------|
| 01 | [[04_Databases/01_Databases_ORM_SQL]] | SQL основы, ACID, индексы, Drizzle, IndexedDB |
| 02 | [[04_Databases/02_Database_Optimization_N1]] | N+1 проблема, DataLoader, метрики производительности |
| 03 | [[04_Databases/03_Advanced_Database_Offline]] | PostgreSQL JSONB, CTE, синхронизация, паттерны ООП |

---

## 📚 Основы баз данных

### Из файла [[04_Databases/01_Databases_ORM_SQL]]

#### 1. Реляционные vs NoSQL
| Характеристика | Реляционные (SQL) | NoSQL |
|---------------|-------------------|-------|
| **Примеры** | PostgreSQL, MySQL | MongoDB, Redis |
| **Структура** | Таблицы, строки, столбцы | Коллекции, документы (JSON) |
| **Схема** | Строгая | Динамическая |
| **Масштабирование** | Вертикальное | Горизонтальное |

#### 2. ACID (свойства транзакций)
- **A**tomicity — неделимость (всё или ничего)
- **C**onsistency — согласованность
- **I**solation — изолированность
- **D**urability — устойчивость

#### 3. Уровни изоляции транзакций
1. **READ UNCOMMITTED** — грязное чтение возможно
2. **READ COMMITTED** — только зафиксированные данные
3. **REPEATABLE READ** — данные не меняются в процессе
4. **SERIALIZABLE** — полная изоляция, низкая производительность

#### 4. Индексы и ключи
- **B-Tree** — сбалансированное дерево, O(log n) поиск
- **Плюсы:** ускоряют SELECT, WHERE, ORDER BY
- **Минусы:** замедляют INSERT/UPDATE/DELETE, занимают место
- **Foreign Key** — ссылочная целостность
- **JOIN:** INNER, LEFT, RIGHT, FULL

#### 5. Drizzle ORM
- Type-safe для TypeScript
- Без промежуточного движка
- Прямой контроль над SQL

#### 6. IndexedDB
- Браузерная NoSQL БД
- Асинхронная, не блокирует UI
- Для оффлайн-приложений и кэширования

---

## ⚡ Оптимизация запросов

### Из файла [[04_Databases/02_Database_Optimization_N1]]

#### 1. Проблема N+1
- **Суть:** 1 запрос на список + N запросов в цикле
- **Решение на уровне SQL:** `JOIN`
- **Решение на уровне кода:** **DataLoader**
  - Собирает ID в массив
  - Делает один запрос: `WHERE id IN (...)`

#### 2. Индексы B-Tree под капотом
- Поиск за O(log n)
- **Trade-off:** чтение vs запись

#### 3. Транзакции на практике
```sql
BEGIN;
-- операции
COMMIT;    -- успешно
ROLLBACK;  -- откат при ошибке
```

---

## 🔧 Продвинутые возможности PostgreSQL

### Из файла [[04_Databases/03_Advanced_Database_Offline]]

#### 1. JSONB
- **Binary JSON** — бинарный формат
- Для динамических атрибутов
- Поддержка **GIN-индексов** для быстрого поиска по ключам

#### 2. CTE (Common Table Expressions)
```sql
WITH temp_table AS (
  SELECT ...
)
SELECT * FROM temp_table;
```
- Улучшает читаемость сложных запросов
- **WITH RECURSIVE** для деревьев и иерархий

---

## 📡 Оффлайн-архитектура

### Из файла [[04_Databases/03_Advanced_Database_Offline]]

#### 1. Стратегия синхронизации
1. **Флаги состояния:** `sync_status: 'pending' | 'synced' | 'deleted'`
2. **Background Sync API / Service Workers** — отслеживание сети
3. **Пакетная отправка** при восстановлении связи

#### 2. Разрешение конфликтов
- **Last Write Wins** — побеждает последняя запись по timestamp
- **Ручное слияние** — модальное окно пользователю

---

## 🎯 Паттерны ООП в базах данных

### Из файла [[04_Databases/03_Advanced_Database_Offline]]

#### 1. Dependency Injection
- Зависимости передаются извне (через конструктор)
- Для тестируемости (Mock БД)

#### 2. Strategy
- Семейство алгоритмов
- Пример: `SumStrategy`, `AverageStrategy`

#### 3. Singleton
- Один экземпляр подключения к БД
- Глобальная точка доступа

---

## 📊 Core Web Vitals

### Из файла [[04_Databases/02_Database_Optimization_N1]]

| Метрика | Описание | Норма |
|---------|----------|-------|
| **LCP** (Largest Contentful Paint) | Время отрисовки крупнейшего элемента | ≤ 2.5 сек |
| **INP** (Interaction to Next Paint) | Время реакции на взаимодействие | — |
| **CLS** (Cumulative Layout Shift) | Накопительный сдвиг макета | Минимальный |

**Борьба с CLS:**
- `width`/`height` для медиа
- Shimmer UI

---

## 🔗 Связанные темы

- [[01_Python_Backend/02_Django_REST_API]] — ORM оптимизация (`select_related`, `prefetch_related`)
- [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]] — DataLoader паттерн
- [[03_React_Frontend/01_React_Architecture_FSD]] — IndexedDB в браузере

---

*Категория: Базы Данных*
