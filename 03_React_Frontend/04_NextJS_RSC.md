---
created: 2026-03-17
updated: 2026-03-17
category: React
type: topic
priority: high
status: completed
tags: [react, rsc, ssr, hydration, streaming, patterns]
difficulty: advanced
estimated_hours: 8
---

# React Server Components vs SSR: Полное сравнение

> **Назначение:** Глубокое понимание различий между RSC и SSR, гидратация, потоковая передача данных. Включает архитектурные паттерны и лучшие практики.

---

## 1. RSC vs SSR: Архитектурные различия

### 1.1. Сравнительная таблица

| Характеристика | SSR (Classic) | RSC (Next.js App Router) |
|---------------|---------------|-------------------------|
| **Где выполняется** | Сервер | Сервер |
| **JS в бандле** | Весь компонент | Только Client Components |
| **Доступ к БД** | Через API | Прямой |
| **Хуки** | Все | Только в Client Components |
| **Первый рендер** | HTML | HTML + RSC Payload |
| **Гидратация** | Полная | Частичная |
| **Размер бандла** | Большой | Минимальный |

---

### 1.2. SSR (Server-Side Rendering)

**Классический SSR (Next.js Pages Router):**
```tsx
// pages/products.tsx
export async function getServerSideProps() {
    const products = await db.products.findMany();
    
    return {
        props: { products }
    };
}

export default function ProductsPage({ products }) {
    return (
        <div>
            <h1>Products</h1>
            <ul>
                {products.map(p => (
                    <li key={p.id}>{p.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

**Поток данных SSR:**
```
┌─────────────┐
│   Client    │
│  Запрос /   │
│  products   │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   Server (getServerSideProps)   │
│   1. Запрос к БД                │
│   2. Рендер HTML                │
│   3. Сериализация props         │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   Response                      │
│   ┌─────────────────────────┐   │
│   │ HTML                    │   │
│   │ <ul><li>Product 1</li>  │   │
│   │ <li>Product 2</li></ul> │   │
│   └─────────────────────────┘   │
│   ┌─────────────────────────┐   │
│   │ JavaScript (бандл)      │   │
│   │ - React                 │   │
│   │ - Компоненты            │   │
│   │ - Логика                │   │
│   └─────────────────────────┘   │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   Client                        │
│   1. Парсинг HTML               │
│   2. Загрузка JS бандла         │
│   3. Гидратация (навешивание    │
│      обработчиков событий)      │
│   4. Интерактивный UI           │
└─────────────────────────────────┘
```

---

### 1.3. RSC (React Server Components)

**Next.js App Router:**
```tsx
// app/products/page.tsx — Server Component по умолчанию
import { db } from '@/lib/db';

export default async function ProductsPage() {
    const products = await db.products.findMany();
    
    return (
        <div>
            <h1>Products</h1>
            <ul>
                {products.map(p => (
                    <li key={p.id}>{p.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

**Поток данных RSC:**
```
┌─────────────┐
│   Client    │
│  Навигация  │
│  /products  │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────┐
│   Server (Route Handler)        │
│   1. Выполнение Server          │
│      Components                 │
│   2. Запрос к БД                │
│   3. Сериализация в RSC         │
│      Payload                    │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   Response                      │
│   ┌─────────────────────────┐   │
│   │ HTML (статический)      │   │
│   │ <ul><li>Product 1</li>  │   │
│   │ <li>Product 2</li></ul> │   │
│   └─────────────────────────┘   │
│   ┌─────────────────────────┐   │
│   │ RSC Payload (бинарный)  │   │
│   │ [                       │   │
│   │   {type: 'ul', props},  │   │
│   │   {type: 'li', props},  │   │
│   │   ...                   │   │
│   │ ]                       │   │
│   └─────────────────────────┘   │
│   ┌─────────────────────────┐   │
│   │ JS (только Client       │   │
│   │     Components)         │   │
│   │ ~5KB вместо ~500KB      │   │
│   └─────────────────────────┘   │
└──────┬──────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│   Client                        │
│   1. Парсинг HTML               │
│   2. Применение RSC Payload     │
│   3. Гидратация только Client   │
│      Components                 │
│   4. Интерактивный UI           │
└─────────────────────────────────┘
```

---

## 2. Гидратация (Hydration)

### 2.1. Как работает гидратация

**Процесс:**
```tsx
// 1. Сервер отдаёт HTML
// <button onclick="alreadyAttached">Click</button>

// 2. React на клиенте создаёт Virtual DOM
const virtualDOM = {
    type: 'button',
    props: {
        onClick: handleClick,
        children: 'Click'
    }
};

// 3. Сравнение с реальным DOM
// HTML: <button onclick="alreadyAttached">Click</button>
// VDOM: {type: 'button', props: {onClick: handleClick}}

// 4. Навешивание обработчиков
realButton.addEventListener('click', handleClick);

// 5. Гидратация завершена — UI интерактивен
```

---

### 2.2. Hydration Mismatch

**Причины ошибок:**

```tsx
// ❌ Math.random() — разное значение на сервере и клиенте
export default function Page() {
    return <div>{Math.random()}</div>;
}
// Сервер: 0.123456
// Клиент: 0.789012
// ❌ Hydration failed

// ❌ Date.now() — разное время
export default function Page() {
    return <div>{Date.now()}</div>;
}
// Сервер: 1710000000000
// Клиент: 1710000000100
// ❌ Hydration failed

// ❌ window — нет на сервере
export default function Page() {
    const height = window.innerHeight;  // ReferenceError на сервере!
    return <div>{height}</div>;
}

// ❌ localStorage — нет на сервере
export default function Page() {
    const theme = localStorage.getItem('theme');  // ReferenceError!
    return <div className={theme}>Content</div>;
}
```

---

### 2.3. Решение проблем с гидратацией

**useEffect для клиентских данных:**
```tsx
'use client';

import { useState, useEffect } from 'react';

// ✅ Правильно: данные только на клиенте
export default function ClientTime() {
    const [time, setTime] = useState<string>('');
    
    useEffect(() => {
        // Выполняется только на клиенте
        setTime(new Date().toLocaleTimeString());
    }, []);
    
    return <div>{time || 'Loading...'}</div>;
}

// ✅ Правильно: window API
export default function WindowHeight() {
    const [height, setHeight] = useState<number>(0);
    
    useEffect(() => {
        setHeight(window.innerHeight);
    }, []);
    
    return <div>Window height: {height}px</div>;
}
```

**Suppress Hydration Warnings:**
```tsx
// Для контента, который заведомо разный
export default function Page() {
    return (
        <div suppressHydrationWarning>
            {Math.random()}
        </div>
    );
}

// Работает только на одном уровне глубины
```

---

### 2.4. Progressive Hydration

```tsx
// app/dashboard/page.tsx
import { Suspense } from 'react';
import { db } from '@/lib/db';

// Серверный компонент
async function StatsCard() {
    const stats = await db.stats.findFirst();
    return <div>Stats: {stats.value}</div>;
}

// Клиентский компонент с загрузкой
'use client';

function Chart() {
    const [data, setData] = useState(null);
    
    useEffect(() => {
        fetch('/api/chart-data')
            .then(res => res.json())
            .then(setData);
    }, []);
    
    if (!data) return <ChartSkeleton />;
    return <ChartComponent data={data} />;
}

export default function Dashboard() {
    return (
        <div>
            {/* Гидратируется сразу */}
            <StatsCard />
            
            {/* Гидратируется когда данные готовы */}
            <Suspense fallback={<ChartSkeleton />}>
                <Chart />
            </Suspense>
        </div>
    );
}
```

---

## 3. Потоковая передача данных

### 3.1. Streaming SSR

```tsx
// app/products/page.tsx
import { Suspense } from 'react';
import { db } from '@/lib/db';

async function ProductList() {
    const products = await db.products.findMany();
    return (
        <ul>
            {products.map(p => (
                <li key={p.id}>{p.name}</li>
            ))}
        </ul>
    );
}

async function CategoryList() {
    const categories = await db.categories.findMany();
    return (
        <ul>
            {categories.map(c => (
                <li key={c.id}>{c.name}</li>
            ))}
        </ul>
    );
}

export default function ProductsPage() {
    return (
        <div>
            <h1>Products</h1>
            
            {/* Отправляется сразу */}
            <Suspense fallback={<div>Loading products...</div>}>
                <ProductList />
            </Suspense>
            
            {/* Отправляется когда готов */}
            <Suspense fallback={<div>Loading categories...</div>}>
                <CategoryList />
            </Suspense>
        </div>
    );
}
```

**Поток ответа:**
```
T=0ms:   <html><head>...</head><body>
         <h1>Products</h1>
         <div>Loading products...</div>
         <div>Loading categories...</div>
T=100ms: <ul><li>Product 1</li>...</ul>
         <!-- ProductList готов -->
T=250ms: <ul><li>Category 1</li>...</ul>
         <!-- CategoryList готов -->
T=250ms: </body></html>
```

---

### 3.2. React.use() с потоковой передачей

```tsx
// app/product/[id]/page.tsx
import { use, Suspense } from 'react';
import { db } from '@/lib/db';

function ProductDetails({ productPromise }) {
    // use() приостанавливает рендер пока Promise не разрешится
    const product = use(productPromise);
    
    return (
        <div>
            <h1>{product.name}</h1>
            <p>{product.description}</p>
            <p>${product.price}</p>
        </div>
    );
}

export default function ProductPage({ params }) {
    const productPromise = db.products.findUnique({
        where: { id: params.id }
    });
    
    return (
        <Suspense fallback={<ProductSkeleton />}>
            <ProductDetails productPromise={productPromise} />
        </Suspense>
    );
}
```

---

## 4. Паттерны композиции RSC + Client Components

### 4.1. Паттерн: Slot Composition

```tsx
// ✅ ХОРОШО: Client Component получает children с сервера
// app/layout.tsx
import { db } from '@/lib/db';

async function HeaderData() {
    const categories = await db.categories.findMany();
    return <Header categories={categories} />;
}

export default function RootLayout({ children }) {
    return (
        <html>
            <body>
                <HeaderData />  // Server → Client
                {children}
            </body>
        </html>
    );
}

// components/Header.tsx
'use client';

export function Header({ categories }) {
    const [isOpen, setIsOpen] = useState(false);
    
    return (
        <header>
            <nav>
                {categories.map(c => (
                    <a key={c.id} href={c.slug}>{c.name}</a>
                ))}
            </nav>
            <button onClick={() => setIsOpen(!isOpen)}>
                Menu
            </button>
        </header>
    );
}
```

---

### 4.2. Паттерн: Render Props

```tsx
// ✅ ХОРОШО: Server Component передаёт данные в Client
// app/page.tsx
import { db } from '@/lib/db';
import { DataConsumer } from './DataConsumer';

export default async function HomePage() {
    const data = await db.data.findMany();
    
    return <DataConsumer data={data} />;
}

// app/DataConsumer.tsx
'use client';

export function DataConsumer({ data }) {
    const [filter, setFilter] = useState('all');
    
    const filtered = data.filter(item => 
        filter === 'all' || item.category === filter
    );
    
    return (
        <div>
            <select value={filter} onChange={e => setFilter(e.target.value)}>
                <option value="all">All</option>
                <option value="a">A</option>
                <option value="b">B</option>
            </select>
            
            <ul>
                {filtered.map(item => (
                    <li key={item.id}>{item.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

---

### 4.3. Паттерн: Wrapper Composition

```tsx
// ✅ ХОРОШО: Обёртка вокруг Client Component
// app/page.tsx
import { getServerData } from '@/lib/data';
import { ClientWrapper } from './ClientWrapper';

export default async function Page() {
    const data = await getServerData();
    
    return (
        <ClientWrapper initialData={data}>
            <PageContent />
        </ClientWrapper>
    );
}

// app/ClientWrapper.tsx
'use client';

export function ClientWrapper({ children, initialData }) {
    const [data, setData] = useState(initialData);
    
    return (
        <DataContext.Provider value={{ data, setData }}>
            {children}
        </DataContext.Provider>
    );
}
```

---

## 5. Сравнение подходов к рендерингу

### 5.1. Когда что использовать

**SSR (классический):**
- ✅ Приложения на Next.js Pages Router
- ✅ Нужна поддержка старого оборудования
- ✅ Простые приложения без сложной интерактивности

**RSC (Next.js App Router):**
- ✅ Новые проекты на Next.js 13+
- ✅ Тяжёлые библиотеки (markdown, charts)
- ✅ Прямой доступ к БД без API
- ✅ Минимальный размер бандла

**CSR (Client-Side Rendering):**
- ✅ Дашборды с частыми обновлениями
- ✅ Real-time приложения
- ✅ Персонализированный контент

**SSG (Static Site Generation):**
- ✅ Блоги, документация
- ✅ Маркетинговые страницы
- ✅ Контент, который редко меняется

---

### 5.2. Гибридный подход

```tsx
// app/product/[id]/page.tsx
import { Suspense } from 'react';
import { db } from '@/lib/db';

// Статическая генерация
export async function generateStaticParams() {
    const products = await db.products.findMany({
        select: { id: true }
    });
    
    return products.map(p => ({ id: p.id }));
}

// ISR с ре-валидацией
export const revalidate = 3600;  // 1 час

export default async function ProductPage({ params }) {
    const product = await db.products.findUnique({
        where: { id: params.id }
    });
    
    return (
        <div>
            {/* Статический контент */}
            <h1>{product.name}</h1>
            <p>{product.description}</p>
            
            {/* Динамический контент */}
            <Suspense fallback={<ReviewsSkeleton />}>
                <Reviews productId={product.id} />
            </Suspense>
            
            {/* Клиентский интерактив */}
            <AddToCart productId={product.id} />
        </div>
    );
}
```

---

## 🔗 Связанные темы

- [[03_React_Frontend/01_React_Architecture_FSD]] — Архитектура React
- [[03_React_Frontend/03_NextJS_App_Router_React19]] — Next.js кэширование
- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Event Loop

---

*Файл обновлён: 17 марта 2026*
