---
created: 2026-03-17
updated: 2026-03-17
category: Databases
type: topic
priority: medium
status: completed
tags: [postgresql, jsonb, cte, offline-first, indexeddb, patterns, star-method]
difficulty: advanced
estimated_hours: 8
---

# Продвинутые Базы Данных и Оффлайн-архитектура

> **Назначение:** Продвинутые возможности PostgreSQL (JSONB, CTE), оффлайн-архитектура с синхронизацией, паттерны проектирования. Включает практические примеры и метод STAR для собеседований.

---

## 1. PostgreSQL: Продвинутые возможности

### 1.1. JSONB: Гибридное хранение данных

**Что такое JSONB:**
```
JSONB (Binary JSON) — бинарное представление JSON в PostgreSQL

Преимущества перед обычным JSON:
✅ Быстрее парсинг (уже распарсен)
✅ Поддержка индексов (GIN)
✅ Эффективное хранение (сжатие)
❌ Медленнее вставка (нужно преобразование)
```

**Пример использования:**
```sql
-- Таблица с JSONB
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50),
    settings JSONB DEFAULT '{}'::jsonb,
    metadata JSONB DEFAULT '{}'::jsonb
);

-- Вставка
INSERT INTO users (username, settings) VALUES (
    'john',
    '{
        "theme": "dark",
        "notifications": {
            "email": true,
            "push": false
        },
        "language": "en"
    }'::jsonb
);

-- Запрос к JSONB
SELECT username, settings->>'theme' as theme
FROM users
WHERE settings->>'language' = 'en';

-- Обновление JSONB
UPDATE users 
SET settings = jsonb_set(settings, '{theme}', '"light"')
WHERE id = 1;

-- GIN индекс для быстрого поиска
CREATE INDEX idx_users_settings ON users USING GIN (settings);

-- Поиск по ключу (использует индекс)
SELECT * FROM users 
WHERE settings ? 'notifications';

-- Поиск по значению
SELECT * FROM users 
WHERE settings @> '{"theme": "dark"}';
```

---

### 1.2. CTE (Common Table Expressions)

**Базовый CTE:**
```sql
-- Без CTE (сложно читать)
SELECT 
    department,
    AVG(salary) as avg_salary
FROM (
    SELECT * FROM employees 
    WHERE hire_date > '2020-01-01'
) AS recent_employees
GROUP BY department;

-- С CTE (читаемее)
WITH recent_employees AS (
    SELECT * FROM employees 
    WHERE hire_date > '2020-01-01'
)
SELECT 
    department,
    AVG(salary) as avg_salary
FROM recent_employees
GROUP BY department;
```

---

### 1.3. Рекурсивные CTE

**Дерево комментариев:**
```sql
WITH RECURSIVE comment_tree AS (
    -- Базовый случай: корневые комментарии
    SELECT 
        id,
        parent_id,
        content,
        author_id,
        created_at,
        1 as depth,
        ARRAY[id] as path
    FROM comments
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Рекурсивный случай: дочерние комментарии
    SELECT 
        c.id,
        c.parent_id,
        c.content,
        c.author_id,
        c.created_at,
        ct.depth + 1,
        ct.path || c.id
    FROM comments c
    INNER JOIN comment_tree ct ON c.parent_id = ct.id
)
SELECT * FROM comment_tree
ORDER BY path;

-- Результат:
-- id | parent_id | content      | depth | path
-- 1  | NULL      | Root comment | 1     | {1}
-- 2  | 1         | Reply 1      | 2     | {1,2}
-- 3  | 1         | Reply 2      | 2     | {1,3}
-- 4  | 2         | Reply to 1   | 3     | {1,2,4}
```

---

### 1.4. Оконные функции (Window Functions)

```sql
-- Ранжирование пользователей по активности
SELECT 
    username,
    posts_count,
    RANK() OVER (ORDER BY posts_count DESC) as rank,
    ROW_NUMBER() OVER (ORDER BY posts_count DESC) as row_num
FROM users;

-- Скользящее среднее
SELECT 
    date,
    sales,
    AVG(sales) OVER (
        ORDER BY date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7days
FROM daily_sales;

-- Сравнение с предыдущим значением
SELECT 
    date,
    sales,
    LAG(sales, 1) OVER (ORDER BY date) as prev_day_sales,
    sales - LAG(sales, 1) OVER (ORDER BY date) as change
FROM daily_sales;
```

---

### 1.5. Full-Text Search

```sql
-- Создание индекса
CREATE INDEX idx_posts_search ON posts 
USING GIN (to_tsvector('english', title || ' ' || content));

-- Поиск
SELECT 
    title,
    ts_rank(to_tsvector('english', content), query) as rank
FROM posts,
     to_tsquery('english', 'database & optimization') query
WHERE to_tsvector('english', content) @@ query
ORDER BY rank DESC
LIMIT 10;

-- Подсветка результатов
SELECT 
    ts_headline(
        'english',
        content,
        to_tsquery('english', 'database'),
        'StartSel=<b>, StopSel=</b>'
    ) as highlighted_content
FROM posts
WHERE to_tsvector('english', content) @@ to_tsquery('english', 'database');
```

---

## 2. Оффлайн-архитектура

### 2.1. Архитектура Offline-First

```
┌─────────────────────────────────────────────────────────┐
│              Offline-First Architecture                 │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐         ┌─────────────────┐       │
│  │   Client (App)  │         │    Server       │       │
│  │                 │         │                 │       │
│  │  ┌───────────┐  │         │  ┌───────────┐  │       │
│  │  │IndexedDB  │  │◄───────►│  │   Database│  │       │
│  │  │(Local DB) │  │ Sync    │  │   (Postgres)│ │       │
│  │  └───────────┘  │ API     │  └───────────┘  │       │
│  │                 │         │                 │       │
│  │  ┌───────────┐  │         │  ┌───────────┐  │       │
│  │  │Service    │  │         │  │  Sync     │  │       │
│  │  │Worker     │  │         │  │  Service  │  │       │
│  │  └───────────┘  │         │  └───────────┘  │       │
│  │                 │         │                 │       │
│  │  ┌───────────┐  │         │                 │       │
│  │  │Background │  │         │                 │       │
│  │  │Sync API   │  │         │                 │       │
│  │  └───────────┘  │         │                 │       │
│  └─────────────────┘         └─────────────────┘       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 2.2. Схема данных для синхронизации

```typescript
// Типы для оффлайн-очереди
type SyncStatus = 'pending' | 'syncing' | 'synced' | 'failed';
type OperationType = 'CREATE' | 'UPDATE' | 'DELETE';

interface SyncQueueItem {
    id: string;
    operation: OperationType;
    tableName: string;
    recordId: string;
    data: Record<string, any>;
    timestamp: number;
    retryCount: number;
    status: SyncStatus;
}

// IndexedDB схема
const DB_NAME = 'OfflineApp';
const DB_VERSION = 1;

const stores = {
    users: { keyPath: 'id', indexes: ['username', 'sync_status'] },
    posts: { keyPath: 'id', indexes: ['userId', 'sync_status'] },
    sync_queue: { keyPath: 'id', indexes: ['status', 'timestamp'] }
};
```

---

### 2.3. Service Worker с Background Sync

```javascript
// sw.js
const CACHE_NAME = 'offline-app-v1';

// Кэширование статики
self.addEventListener('install', (event) => {
    event.waitUntil(
        caches.open(CACHE_NAME).then((cache) => {
            return cache.addAll([
                '/',
                '/static/js/main.js',
                '/static/css/main.css',
                '/offline.html'
            ]);
        })
    );
});

// Offline fallback
self.addEventListener('fetch', (event) => {
    event.respondWith(
        fetch(event.request)
            .catch(() => {
                if (event.request.mode === 'navigate') {
                    return caches.match('/offline.html');
                }
                return caches.match(event.request);
            })
    );
});

// Background Sync
self.addEventListener('sync', (event) => {
    if (event.tag === 'sync-data') {
        event.waitUntil(syncData());
    }
});

async function syncData() {
    const db = await openDatabase();
    const pendingItems = await db.getAllFromIndex('sync_queue', 'status', 'pending');
    
    for (const item of pendingItems) {
        try {
            // Обновление статуса
            await db.put('sync_queue', { ...item, status: 'syncing' });
            
            // Отправка на сервер
            await fetch('/api/sync', {
                method: 'POST',
                body: JSON.stringify(item)
            });
            
            // Успех
            await db.put('sync_queue', { ...item, status: 'synced' });
            
        } catch (error) {
            // Ошибка — увеличиваем retry count
            const retryCount = (item.retryCount || 0) + 1;
            
            if (retryCount >= 3) {
                await db.put('sync_queue', { ...item, status: 'failed', retryCount });
            } else {
                await db.put('sync_queue', { ...item, retryCount });
            }
        }
    }
}
```

---

### 2.4. Клиентская библиотека для синхронизации

```typescript
// lib/offline-sync.ts
class OfflineSync {
    private db: IDBDatabase | null = null;
    private syncInProgress = false;
    
    async init() {
        this.db = await this.openDB();
        this.registerSync();
    }
    
    private async openDB(): Promise<IDBDatabase> {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open('OfflineApp', 1);
            
            request.onupgradeneeded = () => {
                const db = request.result;
                
                if (!db.objectStoreNames.contains('sync_queue')) {
                    const store = db.createObjectStore('sync_queue', {
                        keyPath: 'id',
                        autoIncrement: true
                    });
                    store.createIndex('status', 'status');
                    store.createIndex('timestamp', 'timestamp');
                }
            };
            
            request.onsuccess = () => resolve(request.result);
            request.onerror = () => reject(request.error);
        });
    }
    
    private registerSync() {
        if ('serviceWorker' in navigator && 'sync' in window.registration) {
            navigator.serviceWorker.ready.then((registration) => {
                registration.sync.register('sync-data');
            });
        }
    }
    
    async queueOperation(
        operation: OperationType,
        tableName: string,
        recordId: string,
        data: any
    ) {
        if (!this.db) throw new Error('DB not initialized');
        
        return new Promise((resolve, reject) => {
            const tx = this.db.transaction('sync_queue', 'readwrite');
            const store = tx.objectStore('sync_queue');
            
            store.add({
                operation,
                tableName,
                recordId,
                data,
                timestamp: Date.now(),
                status: 'pending',
                retryCount: 0
            });
            
            tx.oncomplete = () => {
                this.triggerSync();
                resolve(true);
            };
            tx.onerror = () => reject(tx.error);
        });
    }
    
    private triggerSync() {
        if ('serviceWorker' in navigator && 'sync' in window.registration) {
            navigator.serviceWorker.ready.then((registration) => {
                registration.sync.register('sync-data');
            });
        }
    }
    
    // Разрешение конфликтов: Last Write Wins
    resolveConflict(localData: any, serverData: any): any {
        if (localData.updatedAt > serverData.updatedAt) {
            return localData;
        }
        return serverData;
    }
}

export const offlineSync = new OfflineSync();
```

---

### 2.5. Использование в приложении

```typescript
// components/PostEditor.tsx
'use client';

import { useState, useEffect } from 'react';
import { offlineSync } from '@/lib/offline-sync';

export function PostEditor() {
    const [title, setTitle] = useState('');
    const [content, setContent] = useState('');
    const [isOnline, setIsOnline] = useState(navigator.onLine);
    
    useEffect(() => {
        const handleOnline = () => setIsOnline(true);
        const handleOffline = () => setIsOnline(false);
        
        window.addEventListener('online', handleOnline);
        window.addEventListener('offline', handleOffline);
        
        return () => {
            window.removeEventListener('online', handleOnline);
            window.removeEventListener('offline', handleOffline);
        };
    }, []);
    
    const handleSubmit = async (e: React.FormEvent) => {
        e.preventDefault();
        
        const postData = { title, content, createdAt: Date.now() };
        
        if (isOnline) {
            // Онлайн — отправка на сервер
            await fetch('/api/posts', {
                method: 'POST',
                body: JSON.stringify(postData)
            });
        } else {
            // Оффлайн — сохранение в очередь
            await offlineSync.queueOperation(
                'CREATE',
                'posts',
                Date.now().toString(),
                postData
            );
            alert('Post saved locally. Will sync when online.');
        }
    };
    
    return (
        <form onSubmit={handleSubmit}>
            {!isOnline && (
                <div className="offline-notice">
                    📴 You're offline. Changes will sync later.
                </div>
            )}
            
            <input
                value={title}
                onChange={(e) => setTitle(e.target.value)}
                placeholder="Post title"
            />
            
            <textarea
                value={content}
                onChange={(e) => setContent(e.target.value)}
                placeholder="Post content"
            />
            
            <button type="submit">
                {isOnline ? 'Publish' : 'Save Offline'}
            </button>
        </form>
    );
}
```

---

## 3. Паттерны проектирования ООП

### 3.1. Dependency Injection

```typescript
// ❌ Без DI (тесная связь)
class UserService {
    private db = new PostgreSQLDatabase();  // Конкретная реализация
    
    async getUser(id: string) {
        return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    }
}

// ✅ С DI (слабая связь)
interface Database {
    query(sql: string, params: any[]): Promise<any[]>;
}

class UserService {
    constructor(private db: Database) {}  // Зависимость внедряется
    
    async getUser(id: string) {
        return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
    }
}

// Использование
const postgresDB = new PostgreSQLDatabase();
const userService = new UserService(postgresDB);

// В тестах — мок
const mockDB = {
    query: jest.fn().mockResolvedValue([{ id: '1', name: 'Test' }])
};
const testService = new UserService(mockDB);
```

---

### 3.2. Strategy Pattern

```typescript
// Контекст
class MetricCalculator {
    constructor(private strategy: CalculationStrategy) {}
    
    calculate(data: number[]): number {
        return this.strategy.execute(data);
    }
    
    setStrategy(strategy: CalculationStrategy) {
        this.strategy = strategy;
    }
}

// Стратегии
interface CalculationStrategy {
    execute(data: number[]): number;
}

class SumStrategy implements CalculationStrategy {
    execute(data: number[]): number {
        return data.reduce((sum, n) => sum + n, 0);
    }
}

class AverageStrategy implements CalculationStrategy {
    execute(data: number[]): number {
        return data.reduce((sum, n) => sum + n, 0) / data.length;
    }
}

class MedianStrategy implements CalculationStrategy {
    execute(data: number[]): number {
        const sorted = [...data].sort((a, b) => a - b);
        const mid = Math.floor(sorted.length / 2);
        return sorted.length % 2 !== 0 
            ? sorted[mid] 
            : (sorted[mid - 1] + sorted[mid]) / 2;
    }
}

// Использование
const calculator = new MetricCalculator(new SumStrategy());
calculator.calculate([1, 2, 3, 4, 5]);  // 15

calculator.setStrategy(new AverageStrategy());
calculator.calculate([1, 2, 3, 4, 5]);  // 3

calculator.setStrategy(new MedianStrategy());
calculator.calculate([1, 2, 3, 4, 5]);  // 3
```

---

### 3.3. Singleton Pattern

```typescript
class DatabaseConnection {
    private static instance: DatabaseConnection;
    private connection: any;
    
    private constructor() {
        // Приватный конструктор
        this.connection = this.createConnection();
    }
    
    private createConnection() {
        // Создание подключения к БД
        return createPool({
            host: 'localhost',
            database: 'myapp',
            user: 'user',
            password: 'pass'
        });
    }
    
    public static getInstance(): DatabaseConnection {
        if (!DatabaseConnection.instance) {
            DatabaseConnection.instance = new DatabaseConnection();
        }
        return DatabaseConnection.instance;
    }
    
    public query(sql: string, params: any[]) {
        return this.connection.query(sql, params);
    }
}

// Использование
const db1 = DatabaseConnection.getInstance();
const db2 = DatabaseConnection.getInstance();

console.log(db1 === db2);  // true (один экземпляр)
```

---

## 4. Метод STAR для поведенческих интервью

### 4.1. Структура ответа

```
┌─────────────────────────────────────────────────────────┐
│              STAR Method Framework                      │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  S — Situation (Ситуация)                               │
│  Опишите контекст и проблему                            │
│                                                         │
│  T — Task (Задача)                                      │
│  Ваша конкретная задача/цель                            │
│                                                         │
│  A — Action (Действия)                                  │
│  Что ВЫ сделали (не "мы", а "я")                        │
│                                                         │
│  R — Result (Результат)                                 │
│  Измеримый результат с цифрами                          │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 4.2. Пример ответа

**Вопрос:** "Расскажите о самой сложной технической проблеме, которую вы решили"

**Ответ по STAR:**

**Situation:**
> "На проекте Urban Analytics мы столкнулись с серьёзной проблемой производительности. Пользователи загружали Excel-файлы с данными до 100,000 строк для аналитики. Изначально архитектура предполагала отправку всех данных на бэкенд для обработки, что приводило к таймаутам запросов и перегрузке сервера."

**Task:**
> "Мне нужно было перепроектировать систему так, чтобы:
> - Убрать нагрузку с бэкенда
> - Обеспечить мгновенную отрисовку дашбордов
> - Сохранить возможность вычисления кастомных метрик
> - Гарантировать безопасность вычислений"

**Action:**
> "Я принял решение перенести обработку данных на клиентскую сторону:
> 
> 1. **IndexedDB для хранения:** Внедрил локальную базу данных для хранения сырых данных прямо в браузере. Это позволило работать с большими объёмами без блокировки UI.
> 
> 2. **AST-парсер для формул:** Написал парсер абстрактных синтаксических деревьев для безопасного вычисления пользовательских формул. Это исключило уязвимость через `eval()`.
> 
> 3. **Zustand для состояния:** Использовал легковесный стейт-менеджер для управления состоянием дашбордов, что упростило архитектуру по сравнению с Redux.
> 
> 4. **Виртуализация списков:** Внедрил react-window для рендеринга только видимых строк таблицы."

**Result:**
> "В результате:
> - Время загрузки дашбордов сократилось с 5-8 секунд до ~200мс
> - Нагрузка на бэкенд снизилась на 80%
> - Браузер перестал зависать при рендеринге больших таблиц
> - Безопасность вычислений обеспечена через AST-валидацию
> 
> Пользователи получили мгновенный отклик интерфейса, а компания сэкономила на серверных ресурсах."

---

### 4.3. Шаблоны для других вопросов

**Вопрос:** "Как вы работали в команде над сложным проектом?"

```
S: "В команде из 5 разработчиков мы делали платформу для..."
T: "Моя задача была реализовать систему авторизации с ролями..."
A: "Я использовал RBAC паттерн, создал middleware для проверки..."
R: "В результате система поддерживает 5 ролей с гранулярными правами..."
```

**Вопрос:** "Как вы оптимизировали производительность?"

```
S: "Приложение загружалось 8+ секунд, пользователи уходили..."
T: "Нужно было сократить время загрузки до 2-3 секунд..."
A: "Я внедрил code splitting, lazy loading, оптимизировал картинки..."
R: "LCP улучшился с 8с до 1.8с, bounce rate снизился на 40%..."
```

---

## 🔗 Связанные темы

- [[04_Databases/01_Databases_ORM_SQL]] — SQL основы
- [[04_Databases/02_Database_Optimization_N1]] — Оптимизация N+1
- [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding]] — AST парсинг

---

*Файл обновлён: 17 марта 2026*
