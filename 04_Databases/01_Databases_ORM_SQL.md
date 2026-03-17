# Базы Данных: SQL, ORM и Архитектура

> **Назначение:** Фундаментальные и продвинутые концепции баз данных: реляционные и NoSQL БД, SQL запросы, индексы, транзакции, ORM. Включает практические примеры и оптимизацию.

---

## 1. Реляционные vs NoSQL базы данных

### 1.1. Сравнительная таблица

| Характеристика | Реляционные (SQL) | NoSQL |
|---------------|-------------------|-------|
| **Примеры** | PostgreSQL, MySQL, SQLite | MongoDB, Redis, Cassandra |
| **Структура** | Таблицы со строками/столбцами | Документы, ключ-значение, графы |
| **Схема** | Строгая, фиксированная | Динамическая, гибкая |
| **Связи** | JOIN между таблицами | Вложенные документы, ссылки |
| **Масштабирование** | Вертикальное (мощнее сервер) | Горизонтальное (больше серверов) |
| **Транзакции** | Полная поддержка ACID | Ограниченная или eventual consistency |
| **Когда использовать** | Финансы, учёт, сложные связи | Контент, кэш, аналитика, IoT |

---

### 1.2. PostgreSQL (Реляционная БД)

**Пример схемы:**
```sql
-- Таблица пользователей
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Таблица постов
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    published BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Индексы для ускорения
CREATE INDEX idx_posts_user_id ON posts(user_id);
CREATE INDEX idx_posts_published ON posts(published);
```

---

### 1.3. MongoDB (Документная NoSQL БД)

**Пример документа:**
```javascript
// Коллекция: users
{
    _id: ObjectId("..."),
    username: "john_doe",
    email: "john@example.com",
    posts: [  // Вложенные документы
        {
            title: "My First Post",
            content: "Hello World!",
            published: true,
            createdAt: ISODate("2024-01-01")
        }
    ],
    createdAt: ISODate("2024-01-01")
}
```

**Когда MongoDB лучше PostgreSQL:**
- ✅ Быстрое прототипирование (нет миграций схемы)
- ✅ Контент с переменной структурой (блоги, каталоги)
- ✅ Горизонтальное масштабирование (sharding)
- ✅ Хранение логов, аналитики

**Когда PostgreSQL лучше:**
- ✅ Сложные связи между данными
- ✅ Транзакции и консистентность (финансы)
- ✅ Строгая валидация данных
- ✅ Сложные аналитические запросы

---

## 2. ACID: Свойства транзакций

### 2.1. Детальное объяснение

**Atomicity (Атомарность):**
```
Перевод денег: $100 от Alice → Bob

1. Списать $100 у Alice
2. Зачислить $100 Bob

Если шаг 2 упал → шаг 1 откатывается
Итог: либо оба действия, либо ни одного
```

**Consistency (Согласованность):**
```sql
-- Ограничение: баланс не может быть отрицательным
ALTER TABLE accounts ADD CONSTRAINT check_balance 
CHECK (balance >= 0);

-- Транзакция нарушающая ограничение — откатится
BEGIN;
UPDATE accounts SET balance = balance - 200 WHERE id = 1;
-- Если balance было 100 → ошибка → ROLLBACK
COMMIT;
```

**Isolation (Изолированность):**
```
Транзакция A: Чтение баланса Alice
Транзакция B: Обновление баланса Alice

Изоляция гарантирует, что A не увидит
промежуточные результаты B
```

**Durability (Устойчивость):**
```
После COMMIT изменения записаны в:
1. Write-Ahead Log (WAL) — немедленно
2. Data Files — при checkpoint

Даже при сбое питания — данные не потеряны
```

---

## 3. Уровни изоляции транзакций

### 3.1. Аномалии при параллельном доступе

**Dirty Read (Грязное чтение):**
```
Время | Транзакция A         | Транзакция B
------|---------------------|---------------------
T1    | BEGIN;              |
T2    |                     | BEGIN;
T3    |                     | UPDATE accounts 
      |                     | SET balance = 200
      |                     | WHERE id = 1;
T4    | SELECT balance FROM | 
      | accounts WHERE id=1;| 
      | (читает 200)        |
T5    |                     | ROLLBACK;
T6    | (прочитало          |
      |  несуществующие     |
      |  данные!)           |
```

**Non-Repeatable Read:**
```
Время | Транзакция A         | Транзакция B
------|---------------------|---------------------
T1    | BEGIN;              |
T2    | SELECT balance FROM |
      | accounts WHERE id=1;| (100)
T3    |                     | BEGIN;
      |                     | UPDATE accounts
      |                     | SET balance = 200
      |                     | WHERE id = 1;
      |                     | COMMIT;
T4    | SELECT balance FROM |
      | accounts WHERE id=1;| (200 — изменилось!)
T5    | COMMIT;             |
```

**Phantom Read:**
```
Время | Транзакция A         | Транзакция B
------|---------------------|---------------------
T1    | BEGIN;              |
T2    | SELECT * FROM posts |
      | WHERE user_id = 1;  | (5 строк)
T3    |                     | INSERT INTO posts
      |                     | VALUES (..., user_id=1);
      |                     | COMMIT;
T4    | SELECT * FROM posts |
      | WHERE user_id = 1;  | (6 строк — фантом!)
T5    | COMMIT;             |
```

---

### 3.2. Уровни изоляции в PostgreSQL

```sql
-- READ UNCOMMITTED (не рекомендуется)
-- Может читать незафиксированные данные
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- READ COMMITTED (по умолчанию в PostgreSQL)
-- Читает только зафиксированные данные
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- REPEATABLE READ
-- Гарантирует повторное чтение тех же данных
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- SERIALIZABLE (максимальная изоляция)
-- Полная эмуляция последовательного выполнения
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Сравнение уровней:**

| Уровень | Dirty Read | Non-Repeatable | Phantom | Производительность |
|---------|-----------|----------------|---------|-------------------|
| READ UNCOMMITTED | ✅ Возможно | ✅ Возможно | ✅ Возможно | ⭐⭐⭐⭐⭐ |
| READ COMMITTED | ❌ Нет | ✅ Возможно | ✅ Возможно | ⭐⭐⭐⭐ |
| REPEATABLE READ | ❌ Нет | ❌ Нет | ✅ Возможно | ⭐⭐⭐ |
| SERIALIZABLE | ❌ Нет | ❌ Нет | ❌ Нет | ⭐⭐ |

---

## 4. Индексы и производительность

### 4.1. B-Tree структура

```
        [50]           ← Корень
       /    \
    [25]    [75]       ← Внутренние узлы
   /   \    /   \
 [10][30][60][90]     ← Листья (данные)

Поиск 30:
1. 30 < 50 → влево
2. 30 > 25 → вправо
3. 30 == 30 → найдено!

Сложность: O(log n)
Для 1 млн записей: ~20 сравнений
```

---

### 4.2. Типы индексов в PostgreSQL

**B-Tree (по умолчанию):**
```sql
-- Для равенства, диапазонов, сортировки
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created ON posts(created_at DESC);
```

**Hash:**
```sql
-- Только для равенства (=)
CREATE INDEX idx_session_token ON sessions 
USING HASH (token);
```

**GIN (Generalized Inverted Index):**
```sql
-- Для JSONB, массивов, полнотекстового поиска
CREATE INDEX idx_user_settings ON users 
USING GIN (settings);

-- Полнотекстовый поиск
CREATE INDEX idx_posts_search ON posts 
USING GIN (to_tsvector('english', title, content));
```

**GiST (Generalized Search Tree):**
```sql
-- Для геоданных, диапазонов
CREATE INDEX idx_location ON places 
USING GiST (coordinates);
```

---

### 4.3. Составные индексы

```sql
-- Индекс по нескольким колонкам
CREATE INDEX idx_posts_user_date 
ON posts(user_id, created_at DESC);

-- Запрос использует индекс (левые колонки!)
SELECT * FROM posts 
WHERE user_id = 1 
ORDER BY created_at DESC;

-- Запрос НЕ использует индекс (пропущена левая колонка)
SELECT * FROM posts 
WHERE created_at > '2024-01-01';
```

**Правило:** Индекс работает слева направо. Порядок колонок важен!

---

### 4.4. EXPLAIN ANALYZE

```sql
-- Анализ запроса
EXPLAIN ANALYZE
SELECT * FROM posts 
WHERE user_id = 1 
ORDER BY created_at DESC 
LIMIT 10;

-- Вывод:
-- Limit  (cost=0.43..1.23 rows=10)
--   ->  Index Scan Backward using idx_posts_user_date 
--       on posts  (cost=0.43..100.00 rows=1000)
--         Index Cond: (user_id = 1)

-- Seq Scan — полное сканирование (плохо для больших таблиц)
-- Index Scan — использование индекса (хорошо)
```

---

## 5. JOIN и связи

### 5.1. Типы JOIN

```sql
-- INNER JOIN (только совпадающие)
SELECT u.username, p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id;
-- Alice | Post 1
-- Bob   | Post 2

-- LEFT JOIN (все из левой + совпадающие)
SELECT u.username, p.title
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
-- Alice | Post 1
-- Bob   | Post 2
-- Carol | NULL  ← нет постов

-- RIGHT JOIN (все из правой + совпадающие)
SELECT u.username, p.title
FROM users u
RIGHT JOIN posts p ON u.id = p.user_id;
-- Alice | Post 1
-- Bob   | Post 2
-- NULL  | Post 3  ← пост без автора

-- FULL JOIN (все записи)
SELECT u.username, p.title
FROM users u
FULL JOIN posts p ON u.id = p.user_id;
-- Alice | Post 1
-- Bob   | Post 2
-- Carol | NULL
-- NULL  | Post 3
```

---

### 5.2. Связи в SQL

**One-to-One:**
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50)
);

CREATE TABLE profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE REFERENCES users(id),
    bio TEXT,
    avatar_url VARCHAR(200)
);
```

**One-to-Many:**
```sql
CREATE TABLE authors (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE books (
    id SERIAL PRIMARY KEY,
    author_id INTEGER REFERENCES authors(id),
    title VARCHAR(200)
);
```

**Many-to-Many:**
```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100)
);

-- Промежуточная таблица
CREATE TABLE enrollments (
    student_id INTEGER REFERENCES students(id),
    course_id INTEGER REFERENCES courses(id),
    enrolled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (student_id, course_id)
);
```

---

## 6. Drizzle ORM (TypeScript)

### 6.1. Определение схемы

```typescript
// db/schema.ts
import { pgTable, serial, text, integer, timestamp } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
    id: serial('id').primaryKey(),
    username: text('username').notNull().unique(),
    email: text('email').notNull().unique(),
    createdAt: timestamp('created_at').defaultNow()
});

export const posts = pgTable('posts', {
    id: serial('id').primaryKey(),
    userId: integer('user_id').references(() => users.id),
    title: text('title').notNull(),
    content: text('content'),
    published: text('status').default('draft'),
    createdAt: timestamp('created_at').defaultNow()
});
```

---

### 6.2. Базовые запросы

```typescript
import { db } from './db';
import { users, posts } from './schema';
import { eq, and, desc, sql } from 'drizzle-orm';

// INSERT
await db.insert(users).values({
    username: 'john',
    email: 'john@example.com'
});

// SELECT
const allUsers = await db.select().from(users);

// WHERE
const user = await db.select().from(users)
    .where(eq(users.id, 1));

// UPDATE
await db.update(users)
    .set({ email: 'new@example.com' })
    .where(eq(users.id, 1));

// DELETE
await db.delete(users)
    .where(eq(users.id, 1));
```

---

### 6.3. JOIN с Drizzle

```typescript
import { relations } from 'drizzle-orm';

// Связи
export const usersRelations = relations(users, ({ many }) => ({
    posts: many(posts)
}));

export const postsRelations = relations(posts, ({ one }) => ({
    user: one(users, {
        fields: [posts.userId],
        references: [users.id]
    })
}));

// Запрос с связями
const usersWithPosts = await db.query.users.findMany({
    with: {
        posts: true
    }
});

// Фильтрация связей
const usersWithPublishedPosts = await db.query.users.findMany({
    with: {
        posts: {
            where: eq(posts.published, true)
        }
    }
});
```

---

## 7. IndexedDB (Браузерная БД)

### 7.1. Базовое использование

```typescript
// Открытие базы
const request = indexedDB.open('MyDatabase', 1);

request.onupgradeneeded = (event) => {
    const db = (event.target as IDBOpenDBRequest).result;
    
    // Создание хранилища
    const store = db.createObjectStore('users', {
        keyPath: 'id',
        autoIncrement: true
    });
    
    // Индексы
    store.createIndex('username', 'username', { unique: true });
    store.createIndex('email', 'email', { unique: true });
};

request.onsuccess = (event) => {
    const db = (event.target as IDBOpenDBRequest).result;
    
    // Транзакция
    const transaction = db.transaction('users', 'readwrite');
    const store = transaction.objectStore('users');
    
    // Добавление
    store.add({ username: 'john', email: 'john@example.com' });
    
    // Поиск по индексу
    const index = store.index('username');
    const getRequest = index.get('john');
    
    getRequest.onsuccess = () => {
        console.log(getRequest.result);
    };
};
```

---

### 7.2. Обёртка над IndexedDB

```typescript
// lib/db.ts
class Database {
    private db: IDBDatabase | null = null;
    
    async connect(): Promise<IDBDatabase> {
        if (this.db) return this.db;
        
        return new Promise((resolve, reject) => {
            const request = indexedDB.open('AppDB', 1);
            
            request.onupgradeneeded = (e) => {
                const db = (e.target as IDBOpenDBRequest).result;
                
                if (!db.objectStoreNames.contains('cache')) {
                    const store = db.createObjectStore('cache', {
                        keyPath: 'key'
                    });
                    store.createIndex('timestamp', 'timestamp');
                }
            };
            
            request.onsuccess = () => {
                this.db = request.result;
                resolve(this.db);
            };
            
            request.onerror = () => reject(request.error);
        });
    }
    
    async set(key: string, value: any): Promise<void> {
        const db = await this.connect();
        
        return new Promise((resolve, reject) => {
            const tx = db.transaction('cache', 'readwrite');
            const store = tx.objectStore('cache');
            
            store.put({ key, value, timestamp: Date.now() });
            
            tx.oncomplete = () => resolve();
            tx.onerror = () => reject(tx.error);
        });
    }
    
    async get(key: string): Promise<any> {
        const db = await this.connect();
        
        return new Promise((resolve, reject) => {
            const tx = db.transaction('cache', 'readonly');
            const store = tx.objectStore('cache');
            
            const request = store.get(key);
            
            request.onsuccess = () => resolve(request.result?.value);
            request.onerror = () => reject(request.error);
        });
    }
}

export const db = new Database();
```

---

## 🔗 Связанные темы

- [[04_Databases/02_Database_Optimization_N1]] — Оптимизация N+1
- [[04_Databases/03_Advanced_Database_Offline]] — Продвинутый PostgreSQL
- [[01_Python_Backend/02_Django_REST_API]] — Django ORM

---

*Файл обновлён: 17 марта 2026*
