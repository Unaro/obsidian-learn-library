---
created: 2026-03-17
updated: 2026-03-17
category: Databases
type: topic
priority: high
status: completed
tags: [sql, optimization, n-plus-one, indexes, transactions, core-web-vitals]
difficulty: advanced
estimated_hours: 8
---

# Оптимизация Баз Данных и Производительность

> **Назначение:** Продвинутая оптимизация запросов, проблема N+1, индексы B-Tree, транзакции. Включает Core Web Vitals и frontend-оптимизацию.

---

## 1. Проблема N+1 Query

### 1.1. Суть проблемы

```
Сценарий: Получение списка пользователей с их постами

❌ НАИВНЫЙ ПОДХОД (N+1 запросов):

// Запрос 1: Получаем всех пользователей
const users = await db.query('SELECT * FROM users');
// 1 запрос

// Запросы N: Для каждого пользователя получаем посты
for (const user of users) {
    const posts = await db.query(
        'SELECT * FROM posts WHERE user_id = ?',
        [user.id]
    );
    user.posts = posts;
}
// N запросов (по одному на каждого пользователя)

Итого: 1 + N запросов
Для 100 пользователей: 101 запрос к БД!
```

**Визуализация:**
```
Время (мс)
│
├─ Query 1: SELECT * FROM users ────────█
│                                        │
├─ Query 2: SELECT * FROM posts WHERE ──█
│                                        │
├─ Query 3: SELECT * FROM posts WHERE ──█
│                                        │
├─ Query 4: SELECT * FROM posts WHERE ──█
│                                        │
├─ ... (ещё 97 запросов)
│
└─────────────────────────────────────────
   Общее время: ~5000мс (5 секунд!)
```

---

### 1.2. Решение на уровне SQL: JOIN

```sql
-- Один запрос вместо N+1
SELECT 
    u.id AS user_id,
    u.username,
    p.id AS post_id,
    p.title,
    p.content
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;

-- Время выполнения: ~50мс
-- Вместо 5000мс!
```

**В Node.js/TypeScript:**
```typescript
// Один запрос с JOIN
const results = await db.query(`
    SELECT 
        u.id AS user_id,
        u.username,
        JSON_AGG(
            JSON_BUILD_OBJECT(
                'id', p.id,
                'title', p.title,
                'content', p.content
            )
        ) FILTER (WHERE p.id IS NOT NULL) AS posts
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    GROUP BY u.id, u.username
`);

// Группировка результатов
const users = results.rows.map(row => ({
    id: row.user_id,
    username: row.username,
    posts: row.posts || []
}));
```

---

### 1.3. Решение в ORM

**Django ORM:**
```python
# ❌ N+1 проблема
users = User.objects.all()
for user in users:
    posts = user.posts.all()  # N запросов!

# ✅ select_related (для ForeignKey)
users = User.objects.select_related('profile').all()
# 1 запрос с JOIN

# ✅ prefetch_related (для ManyToMany)
users = User.objects.prefetch_related('posts').all()
# 2 запроса (отдельно к users и posts)
```

**Drizzle ORM:**
```typescript
// ❌ N+1 проблема
const users = await db.select().from(users);
for (const user of users) {
    const posts = await db.select().from(posts)
        .where(eq(posts.userId, user.id));
}

// ✅ with (eager loading)
const usersWithPosts = await db.query.users.findMany({
    with: {
        posts: true
    }
});
// 1 запрос с JOIN
```

**Prisma:**
```typescript
// ❌ N+1 проблема
const users = await prisma.user.findMany();
for (const user of users) {
    const posts = await prisma.post.findMany({
        where: { userId: user.id }
    });
}

// ✅ include (eager loading)
const usersWithPosts = await prisma.user.findMany({
    include: {
        posts: true
    }
});
```

---

### 1.4. DataLoader паттерн

**Для GraphQL и сложных случаев:**
```typescript
import DataLoader from 'dataloader';

// Создание DataLoader
const userLoader = new DataLoader(async (userIds: string[]) => {
    // Один пакетный запрос вместо N
    const posts = await db.query(`
        SELECT * FROM posts 
        WHERE user_id IN (${userIds.join(',')})
    `);
    
    // Группировка по user_id
    const postsByUser = new Map();
    for (const post of posts) {
        if (!postsByUser.has(post.user_id)) {
            postsByUser.set(post.user_id, []);
        }
        postsByUser.get(post.user_id).push(post);
    }
    
    // Возврат в том же порядке, что и userIds
    return userIds.map(id => postsByUser.get(id) || []);
});

// Использование в GraphQL resolver
const resolvers = {
    User: {
        posts: async (user) => {
            return userLoader.load(user.id);
        }
    }
};

// Автоматическое batching:
// 10 запросов userLoader.load() → 1 SQL запрос!
```

---

## 2. Индексы B-Tree: Глубокое понимание

### 2.1. Структура B-Tree

```
                    [50]                    ← Корень (уровень 0)
                   /    \
                  /      \
        ┌───────[25]──────[75]───────┐      ← Внутренние узлы (уровень 1)
       /        /  \        \        \
      /        /    \        \        \
   [10] [20] [30] [40] [60] [70] [90] [100]  ← Листья (уровень 2)
   
   Высота дерева: 2
   Максимум сравнений для поиска: 2
   
   Для 1 млн записей:
   log₂(1,000,000) ≈ 20 сравнений
```

---

### 2.2. Операции B-Tree

**Поиск (O(log n)):**
```
Поиск 35:

1. 35 < 50 → влево
2. 35 > 25 → вправо
3. 35 > 30 → вправо
4. 35 < 40 → влево
5. Не найдено

Сравнений: 4 (для дерева из 10 элементов)
```

**Вставка (O(log n)):**
```
Вставка 35:

1. Поиск позиции (4 сравнения)
2. Вставка в лист [30][40]
3. Лист становится [30][35][40]
4. Балансировка если нужно

[30][35][40] — OK (≤ 3 ключей)
```

**Удаление (O(log n)):**
```
Удаление 35:

1. Поиск (4 сравнения)
2. Удаление из листа
3. Если лист пустой — слияние с соседом
4. Балансировка дерева
```

---

### 2.3. Trade-off: Чтение vs Запись

```
Сценарий: Таблица с 1 млн записей

Без индекса:
├─ SELECT: ~500мс (полное сканирование)
├─ INSERT: ~1мс (просто добавить в конец)
├─ UPDATE: ~500мс (поиск + обновление)
└─ DELETE: ~500мс (поиск + удаление)

С индексом:
├─ SELECT: ~5мс (O(log n) поиск) ✅
├─ INSERT: ~5мс (обновить дерево) ⚠️
├─ UPDATE: ~10мс (поиск + обновление индекса) ⚠️
└─ DELETE: ~10мс (поиск + удаление из индекса) ⚠️

Вывод: Индексы ускоряют чтение, но замедляют запись!
```

---

### 2.4. Когда НЕ создавать индексы

```sql
-- ❌ Не индексировать:

-- Маленькие таблицы (< 1000 строк)
-- Полное сканирование быстрее работы с индексом

-- Колонки с низкой селективностью
CREATE INDEX idx_gender ON users(gender);
-- 'M'/'F' — 50% строк совпадает → индекс бесполезен

-- Часто изменяемые колонки
CREATE INDEX idx_login_count ON users(login_count);
-- Каждый login → UPDATE → перестройка индекса

-- Колонки, которые не используются в WHERE/JOIN/ORDER BY
CREATE INDEX users_created_at ON users(created_at);
-- Если нет запросов с created_at → индекс мёртвый груз
```

---

### 2.5. Оптимизация запросов с индексами

```sql
-- ✅ Запросы используют индекс:

-- Точное совпадение
SELECT * FROM users WHERE email = 'test@example.com';

-- Диапазон
SELECT * FROM posts WHERE created_at > '2024-01-01';

-- Сортировка (если индекс по тому же полю)
SELECT * FROM posts ORDER BY created_at DESC;

-- LIKE с префиксом
SELECT * FROM users WHERE username LIKE 'john%';

-- ❌ Запросы НЕ используют индекс:

-- Функции над колонкой
SELECT * FROM users WHERE LOWER(email) = 'test@example.com';
-- Решение: функциональный индекс
CREATE INDEX idx_lower_email ON users(LOWER(email));

-- Операторы NOT, <>
SELECT * FROM users WHERE status <> 'deleted';

-- LIKE с wildcard в начале
SELECT * FROM users WHERE username LIKE '%john%';

-- Вычисления над колонкой
SELECT * FROM products WHERE price * 1.2 > 100;
-- Решение: перенести вычисление
SELECT * FROM products WHERE price > 100 / 1.2;
```

---

## 3. Транзакции на практике

### 3.1. Базовый синтаксис

```sql
BEGIN;

-- Операции транзакции
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Фиксация изменений
COMMIT;

-- Или откат при ошибке
ROLLBACK;
```

---

### 3.2. Транзакции в коде

**Node.js (PostgreSQL):**
```typescript
import { db } from './db';

async function transferMoney(fromId: number, toId: number, amount: number) {
    const client = await db.connect();
    
    try {
        await client.query('BEGIN');
        
        // Проверка баланса
        const balanceResult = await client.query(
            'SELECT balance FROM accounts WHERE id = $1',
            [fromId]
        );
        const balance = balanceResult.rows[0].balance;
        
        if (balance < amount) {
            throw new Error('Insufficient funds');
        }
        
        // Списания
        await client.query(
            'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
            [amount, fromId]
        );
        
        // Зачисление
        await client.query(
            'UPDATE accounts SET balance = balance + $1 WHERE id = $1',
            [amount, toId]
        );
        
        // Логирование
        await client.query(
            'INSERT INTO transactions (from_id, to_id, amount) VALUES ($1, $2, $3)',
            [fromId, toId, amount]
        );
        
        await client.query('COMMIT');
        
    } catch (error) {
        await client.query('ROLLBACK');
        throw error;
    } finally {
        client.release();
    }
}
```

**Python (Django):**
```python
from django.db import transaction

@transaction.atomic
def transfer_money(from_id, to_id, amount):
    # Проверка баланса
    from_account = Account.objects.select_for_update().get(id=from_id)
    
    if from_account.balance < amount:
        raise ValueError('Insufficient funds')
    
    # Списания
    from_account.balance -= amount
    from_account.save()
    
    # Зачисление
    to_account = Account.objects.get(id=to_id)
    to_account.balance += amount
    to_account.save()
    
    # Логирование
    Transaction.objects.create(
        from_account=from_account,
        to_account=to_account,
        amount=amount
    )
    
    # COMMIT автоматически при выходе из функции
    # ROLLBACK при исключении
```

---

### 3.3. Savepoints (точки сохранения)

```sql
BEGIN;

-- Операция 1
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

SAVEPOINT sp1;

-- Операция 2 (может упасть)
UPDATE accounts SET balance = balance - 200 WHERE id = 2;
-- Ошибка: insufficient funds!

-- Откат до savepoint (операция 1 сохраняется)
ROLLBACK TO sp1;

-- Операция 3
UPDATE accounts SET balance = balance - 50 WHERE id = 3;

COMMIT;
```

---

## 4. Core Web Vitals

### 4.1. Метрики производительности

| Метрика | Описание | Хорошее | Требует улучшения | Плохое |
|---------|----------|---------|-------------------|--------|
| **LCP** | Largest Contentful Paint | < 2.5с | 2.5-4.0с | > 4.0с |
| **INP** | Interaction to Next Paint | < 200мс | 200-500мс | > 500мс |
| **CLS** | Cumulative Layout Shift | < 0.1 | 0.1-0.25 | > 0.25 |

---

### 4.2. LCP (Largest Contentful Paint)

**Что измеряет:** Время от начала загрузки до отрисовки крупнейшего видимого элемента.

**Факторы влияния:**
```
┌─────────────────────────────────────────┐
│  Факторы, влияющие на LCP               │
├─────────────────────────────────────────┤
│                                         │
│  1. Медленная загрузка сервера          │
│     (TTFB > 600мс)                      │
│                                         │
│  2. Блокирующий JavaScript/CSS          │
│     (render-blocking resources)         │
│                                         │
│  3. Медленная загрузка ресурсов         │
│     (картинки, шрифты, видео)           │
│                                         │
│  4. Клиентский рендеринг                │
│     (долгий JS парсинг)                 │
│                                         │
└─────────────────────────────────────────┘
```

**Оптимизация LCP:**
```tsx
// ✅ Оптимизация картинок
<img 
    src="hero.jpg"
    alt="Hero"
    loading="eager"  /* Приоритетная загрузка */
    fetchpriority="high"
    width="1200"
    height="630"
/>

// ✅ Предзагрузка критических ресурсов
<link 
    rel="preload" 
    as="image" 
    href="/hero.webp" 
/>

// ✅ Серверный рендеринг (Next.js)
export default async function Page() {
    const data = await fetchData();  // На сервере
    return <Hero data={data} />;     // Готовый HTML
}

// ✅ Избегать lazy loading для LCP элемента
// ❌ Не делать lazy на главной картинке
```

---

### 4.3. INP (Interaction to Next Paint)

**Что измеряет:** Задержка между действием пользователя и визуальным откликом.

**Фазы взаимодействия:**
```
Время (мс)
│
├─ Input Delay (ожидание) ─────█
│                                │
├─ Processing (обработка) ──────█
│                                │
├─ Presentation (отрисовка) ────█
│
└────────────────────────────────
   INP = Input Delay + Processing + Presentation
```

**Оптимизация INP:**
```tsx
// ✅ Разделение тяжелых задач
// ❌ Блокировка главного потока
function handleSubmit() {
    validateForm();      // 50мс
    calculateTotal();    // 200мс ← Блокировка!
    updateCart();        // 100мс
    showNotification();  // 50мс
}

// ✅ Асинхронное выполнение
async function handleSubmit() {
    validateForm();
    
    // Неблокирующая обработка
    await new Promise(resolve => setTimeout(resolve, 0));
    
    const total = await calculateTotal();
    updateCart(total);
    showNotification();
}

// ✅ Использование Web Workers
const worker = new Worker('calculator.js');
worker.postMessage(data);
worker.onmessage = (e) => {
    updateUI(e.data);  // Не блокирует UI
};
```

---

### 4.4. CLS (Cumulative Layout Shift)

**Что измеряет:** Суммарный сдвиг макета во время загрузки.

**Причины CLS:**
```
1. Картинки без размеров
   <img src="hero.jpg">  ← Нет width/height!

2. Динамически добавляемый контент
   - Реклама
   - Виджеты
   - Lazy-loaded изображения

3. Шрифты (FOUT/FOIT)
   - Текст отображается одним шрифтом
   - Затем загружается кастомный шрифт
   - Текст "прыгает"

4. Асинхронные UI компоненты
   - Баннеры
   - Уведомления
   - Модальные окна
```

**Оптимизация CLS:**
```tsx
// ✅ Указание размеров
<img 
    src="hero.jpg" 
    width="1200" 
    height="630" 
    alt="Hero"
/>

// ✅ Резервирование места
<div className="ad-container" style={{ height: '250px' }}>
    {/* Реклама загрузится позже */}
</div>

// ✅ Shimmer UI (скелетные экраны)
function ProductCard() {
    const [product, setProduct] = useState(null);
    
    if (!product) {
        return (
            <div className="skeleton">
                <div className="skeleton-image" />
                <div className="skeleton-title" />
                <div className="skeleton-price" />
            </div>
        );
    }
    
    return <ProductCard data={product} />;
}

// ✅ font-display: optional
@font-face {
    font-family: 'CustomFont';
    src: url('font.woff2') format('woff2');
    font-display: optional;  /* Не вызывать сдвиг */
}
```

---

## 🔗 Связанные темы

- [[04_Databases/01_Databases_ORM_SQL]] — SQL основы
- [[04_Databases/03_Advanced_Database_Offline]] — Продвинутый PostgreSQL
- [[03_React_Frontend/02_React_Internals_Optimization]] — React оптимизация

---

*Файл обновлён: 17 марта 2026*
