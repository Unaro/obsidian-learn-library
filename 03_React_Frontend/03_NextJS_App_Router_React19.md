---
created: 2026-03-17
updated: 2026-03-17
category: React
type: topic
priority: high
status: completed
tags: [nextjs, app-router, react19, caching, server-actions, hooks]
difficulty: advanced
estimated_hours: 10
---

# Next.js App Router & React 19: Полное руководство

> **Назначение:** Продвинутые возможности Next.js 14+ и React 19: кэширование, Server Actions, новые хуки. Включает практические примеры и лучшие практики.

---

## 1. Четыре уровня кэширования в Next.js

### 1.1. Архитектура кэширования

```
┌─────────────────────────────────────────────────────────┐
│           Next.js Caching Architecture                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  1. Request Memoization (на уровне запроса)     │   │
│  │  ┌─────────────────────────────────────────┐    │   │
│  │  │ 同一 request → один fetch               │    │   │
│  │  └─────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
│                        ↓                                │
│  ┌─────────────────────────────────────────────────┐   │
│  │  2. Data Cache (персистентный кэш на сервере)   │   │
│  │  ┌─────────────────────────────────────────┐    │   │
│  │  │ .next/cache/ → передеployment          │    │   │
│  │  └─────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
│                        ↓                                │
│  ┌─────────────────────────────────────────────────┐   │
│  │  3. Full Route Cache (статический HTML)         │   │
│  │  ┌─────────────────────────────────────────┐    │   │
│  │  │ HTML + RSC Payload для каждого route   │    │   │
│  │  └─────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
│                        ↓                                │
│  ┌─────────────────────────────────────────────────┐   │
│  │  4. Router Cache (кэш в браузере клиента)       │   │
│  │  ┌─────────────────────────────────────────┐    │   │
│  │  │ Prefetch + Cache для навигации         │    │   │
│  │  └─────────────────────────────────────────┘    │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 1.2. Request Memoization

**Как работает:**
```tsx
// app/products/page.tsx

// Эти fetch вызовут реальный запрос только 1 раз!
async function ProductsPage() {
    const products = await fetch('https://api.example.com/products', {
        cache: 'force-cache'  // По умолчанию в App Router
    });
    
    const categories = await fetch('https://api.example.com/categories', {
        cache: 'force-cache'
    });
    
    // Если в другом компоненте тоже есть этот fetch
    // → результат возьмётся из памяти
    
    return <div>...</div>;
}

// app/components/ProductList.tsx
async function ProductList() {
    // Тот же URL → тот же результат из кэша
    const products = await fetch('https://api.example.com/products', {
        cache: 'force-cache'
    });
    
    return <div>...</div>;
}
```

**Важно:** Request Memoization работает только в рамках одного запроса к одному маршруту.

---

### 1.3. Data Cache

**Настройка кэширования:**
```tsx
// Кэширование по времени
export const revalidate = 3600;  // Ре-валидация через 1 час

// Или на уровне fetch
const data = await fetch('https://api.example.com/data', {
    next: {
        revalidate: 3600  // 1 час
    }
});

// Без кэширования (динамический рендеринг)
const data = await fetch('https://api.example.com/data', {
    cache: 'no-store'
});

// Force cache (статическая генерация)
const data = await fetch('https://api.example.com/data', {
    next: { revalidate: false }
});
```

**Теги для кэширования:**
```tsx
// app/blog/page.tsx
export const revalidate = 3600;

export default async function BlogPage() {
    const posts = await fetch('https://api.example.com/posts', {
        next: {
            revalidate: 3600,
            tags: ['posts']  // Тег для инвалидации
        }
    });
    
    return <div>...</div>;
}

// app/blog/[slug]/page.tsx
export default async function BlogPost({ params }) {
    const post = await fetch(
        `https://api.example.com/posts/${params.slug}`,
        {
            next: {
                revalidate: 3600,
                tags: ['posts', `post-${params.slug}`]
            }
        }
    );
    
    return <div>...</div>;
}
```

---

### 1.4. Full Route Cache

**Статическая генерация:**
```tsx
// app/about/page.tsx
// По умолчанию статическая генерация

export default async function AboutPage() {
    const content = await fetch('https://api.example.com/about', {
        next: { revalidate: 86400 }  // Ре-валидация раз в сутки
    });
    
    // На этапе build генерируется HTML
    // При запросе — отдаётся статический HTML
    
    return <div dangerouslySetInnerHTML={{ __html: content.html }} />;
}
```

**Динамический рендеринг:**
```tsx
// app/dashboard/page.tsx
export const dynamic = 'force-dynamic';  // Всегда динамически

export default async function DashboardPage() {
    const stats = await db.stats.findMany();  // Запрос к БД при каждом запросе
    
    return <Dashboard stats={stats} />;
}
```

---

### 1.5. Router Cache (клиентский)

**Prefetch ссылок:**
```tsx
import Link from 'next/link';

// Автоматический prefetch при появлении в viewport
<Link href="/products">Products</Link>

// Отключение prefetch
<Link href="/products" prefetch={false}>Products</Link>

// Принудительный prefetch
import { useRouter } from 'next/navigation';

function Navigation() {
    const router = useRouter();
    
    useEffect(() => {
        router.prefetch('/products');
    }, []);
    
    return <button onClick={() => router.push('/products')}>Go</button>;
}
```

---

## 2. Инвалидация кэша

### 2.1. Time-based Revalidation

```tsx
// app/products/page.tsx
export const revalidate = 300;  // 5 минут

export default async function ProductsPage() {
    const products = await fetch('https://api.example.com/products', {
        next: { revalidate: 300 }
    });
    
    return <ProductList products={products} />;
}

// Через 5 минут следующий запрос вызовет:
// 1. Отдачу старого кэша
// 2. Фоновую ре-валидацию
// 3. Обновление кэша
```

---

### 2.2. On-demand Revalidation

**revalidatePath:**
```tsx
// app/products/actions.ts
'use server';

import { revalidatePath } from 'next/cache';

export async function updateProduct(formData: FormData) {
    const id = formData.get('id');
    const name = formData.get('name');
    
    await db.products.update({ id, name });
    
    // Ре-валидация конкретной страницы
    revalidatePath('/products');
    
    // Ре-валидация всех страниц, использующих этот layout
    revalidatePath('/products', 'layout');
    
    // Ре-валидация динамического маршрута
    revalidatePath(`/products/${id}`);
}
```

**revalidateTag:**
```tsx
// app/blog/actions.ts
'use server';

import { revalidateTag } from 'next/cache';

export async function createPost(formData: FormData) {
    const title = formData.get('title');
    const content = formData.get('content');
    
    await db.posts.create({ title, content });
    
    // Ре-валидация всех запросов с тегом 'posts'
    revalidateTag('posts');
    
    // Можно несколько тегов
    revalidateTag('blog');
    revalidateTag('homepage');
}
```

**Использование в компоненте:**
```tsx
// app/blog/page.tsx
export default async function BlogPage() {
    const posts = await fetch('https://api.example.com/posts', {
        next: {
            tags: ['posts']
        }
    });
    
    return <PostList posts={posts} />;
}

// После createPost() → все страницы с тегом 'posts' обновятся
```

---

### 2.3. Комбинированная инвалидация

```tsx
// app/admin/products/actions.ts
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function deleteProduct(id: string) {
    await db.products.delete(id);
    
    // Ре-валидация путей
    revalidatePath('/products');
    revalidatePath(`/products/${id}`);
    
    // Ре-валидация по тегам
    revalidateTag('products');
    revalidateTag(`product-${id}`);
    
    // Ре-валидация homepage
    revalidatePath('/');
    revalidateTag('homepage');
}
```

---

## 3. Server Actions

### 3.1. Базовое использование

**Форма с Server Action:**
```tsx
// app/login/page.tsx
import { login } from './actions';

export default function LoginPage() {
    return (
        <form action={login}>
            <input name="email" type="email" required />
            <input name="password" type="password" required />
            <button type="submit">Login</button>
        </form>
    );
}

// app/login/actions.ts
'use server';

import { redirect } from 'next/navigation';
import { signIn } from '@/auth';

export async function login(formData: FormData) {
    const email = formData.get('email') as string;
    const password = formData.get('password') as string;
    
    try {
        await signIn('credentials', { email, password });
        redirect('/dashboard');
    } catch (error) {
        // Обработка ошибки
        return { error: 'Invalid credentials' };
    }
}
```

---

### 3.2. Server Actions с валидацией

```tsx
// app/register/actions.ts
'use server';

import { z } from 'zod';
import { redirect } from 'next/navigation';

const registerSchema = z.object({
    email: z.string().email('Invalid email'),
    password: z.string().min(8, 'Password must be at least 8 characters'),
    confirmPassword: z.string()
}).refine(data => data.password === data.confirmPassword, {
    message: "Passwords don't match",
    path: ['confirmPassword']
});

export async function register(prevState: any, formData: FormData) {
    // Валидация
    const result = registerSchema.safeParse({
        email: formData.get('email'),
        password: formData.get('password'),
        confirmPassword: formData.get('confirmPassword')
    });
    
    if (!result.success) {
        return {
            errors: result.error.flatten().fieldErrors,
            message: 'Invalid input'
        };
    }
    
    // Регистрация
    try {
        await createUser(result.data);
        redirect('/login');
    } catch (error) {
        return { message: 'Failed to create account' };
    }
}
```

```tsx
// app/register/page.tsx
'use client';

import { useFormState } from 'react-dom';
import { register } from './actions';

export default function RegisterPage() {
    const [state, formAction] = useFormState(register, {
        errors: {},
        message: null
    });
    
    return (
        <form action={formAction}>
            <input name="email" type="email" />
            {state.errors?.email && (
                <span className="error">{state.errors.email[0]}</span>
            )}
            
            <input name="password" type="password" />
            {state.errors?.password && (
                <span className="error">{state.errors.password[0]}</span>
            )}
            
            <input name="confirmPassword" type="password" />
            {state.errors?.confirmPassword && (
                <span className="error">{state.errors.confirmPassword[0]}</span>
            )}
            
            {state.message && (
                <div className="message">{state.message}</div>
            )}
            
            <button type="submit">Register</button>
        </form>
    );
}
```

---

### 3.3. Оптимистичные обновления

```tsx
// app/todos/page.tsx
'use client';

import { useOptimistic } from 'react';
import { addTodo } from './actions';

export default function TodosPage({ initialTodos }) {
    const [optimisticTodos, addOptimisticTodo] = useOptimistic(
        initialTodos,
        (state, newTodo) => [...state, { ...newTodo, pending: true }]
    );
    
    const handleSubmit = async (formData: FormData) => {
        const todo = {
            id: Date.now(),
            text: formData.get('text') as string,
            completed: false
        };
        
        // Мгновенное обновление UI
        addOptimisticTodo(todo);
        
        // Отправка на сервер
        await addTodo(formData);
    };
    
    return (
        <div>
            <form action={handleSubmit}>
                <input name="text" placeholder="Add todo..." />
                <button type="submit">Add</button>
            </form>
            
            <ul>
                {optimisticTodos.map(todo => (
                    <li key={todo.id} className={todo.pending ? 'pending' : ''}>
                        {todo.text}
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

---

## 4. Новые хуки React 19

### 4.1. useActionState

**Базовое использование:**
```tsx
'use client';

import { useActionState } from 'react';

async function increment(previousState: number) {
    return previousState + 1;
}

function Counter() {
    const [state, formAction, isPending] = useActionState(increment, 0);
    
    return (
        <form action={formAction}>
            <p>Count: {state}</p>
            <button type="submit" disabled={isPending}>
                {isPending ? 'Incrementing...' : 'Increment'}
            </button>
        </form>
    );
}
```

**С аргументами:**
```tsx
interface State {
    errors?: Record<string, string[]>;
    message?: string;
}

async function submitForm(prevState: State, formData: FormData): Promise<State> {
    const email = formData.get('email');
    const password = formData.get('password');
    
    // Валидация
    if (!email) {
        return { errors: { email: ['Email is required'] } };
    }
    
    // API вызов
    try {
        await api.register({ email, password });
        return { message: 'Registration successful!' };
    } catch (error) {
        return { message: 'Registration failed' };
    }
}

function RegisterForm() {
    const [state, formAction, isPending] = useActionState(submitForm, {
        errors: {},
        message: null
    });
    
    return (
        <form action={formAction}>
            <input name="email" />
            {state.errors?.email && <span>{state.errors.email[0]}</span>}
            
            <input name="password" type="password" />
            
            <button type="submit" disabled={isPending}>
                {isPending ? 'Submitting...' : 'Register'}
            </button>
            
            {state.message && <p>{state.message}</p>}
        </form>
    );
}
```

---

### 4.2. useFormStatus

```tsx
'use client';

import { useFormStatus } from 'react-dom';

function SubmitButton() {
    const { pending, data, method, action } = useFormStatus();
    
    return (
        <button type="submit" disabled={pending}>
            {pending ? (
                <>
                    <Spinner />
                    Submitting...
                </>
            ) : (
                'Submit'
            )}
        </button>
    );
}

function Form() {
    return (
        <form action={submitForm}>
            <input name="email" required />
            <input name="password" type="password" required />
            
            {/* useFormStatus работает только внутри form */}
            <SubmitButton />
        </form>
    );
}
```

---

### 4.3. useOptimistic

```tsx
'use client';

import { useOptimistic, useState } from 'react';

function MessageList({ messages, sendMessage }) {
    const [optimisticMessages, addOptimisticMessage] = useOptimistic(
        messages,
        (state, newMessage) => [
            ...state,
            { ...newMessage, sending: true }
        ]
    );
    
    const handleSubmit = async (formData: FormData) => {
        const message = {
            id: Date.now(),
            text: formData.get('message'),
            sender: 'me'
        };
        
        // Оптимистичное обновление
        addOptimisticMessage(message);
        
        // Отправка на сервер
        try {
            await sendMessage(message);
        } catch (error) {
            // Откат при ошибке (автоматически через state)
            console.error('Failed to send:', error);
        }
    };
    
    return (
        <div>
            <ul>
                {optimisticMessages.map(msg => (
                    <li key={msg.id} className={msg.sending ? 'sending' : ''}>
                        {msg.text}
                    </li>
                ))}
            </ul>
            
            <form action={handleSubmit}>
                <input name="message" />
                <button type="submit">Send</button>
            </form>
        </div>
    );
}
```

---

### 4.4. use() — работа с Promise и Context

**Чтение Promise:**
```tsx
'use client';

import { use, Suspense } from 'react';

function Message({ messagePromise }) {
    // use() читает Promise прямо в компоненте
    const message = use(messagePromise);
    
    return <p>{message.text}</p>;
}

function Page() {
    return (
        <Suspense fallback={<Spinner />}>
            <Message messagePromise={fetchMessage()} />
        </Suspense>
    );
}
```

**Чтение Context (с условием):**
```tsx
'use client';

import { use, createContext } from 'react';

const ThemeContext = createContext('light');

function ThemedText() {
    const theme = use(ThemeContext);
    
    // Можно использовать в условиях!
    if (theme === 'dark') {
        return <p className="dark">Dark text</p>;
    }
    
    return <p className="light">Light text</p>;
}
```

---

## 5. Работа со списками

### 5.1. Виртуализация

**@tanstack/react-virtual:**
```tsx
'use client';

import { useVirtualizer } from '@tanstack/react-virtual';
import { useRef } from 'react';

function VirtualList({ items }) {
    const parentRef = useRef<HTMLDivElement>(null);
    
    const virtualizer = useVirtualizer({
        count: items.length,
        getScrollElement: () => parentRef.current,
        estimateSize: () => 50,
        overscan: 3
    });
    
    return (
        <div
            ref={parentRef}
            style={{
                height: '600px',
                overflow: 'auto'
            }}
        >
            <div
                style={{
                    height: `${virtualizer.getTotalSize()}px`,
                    position: 'relative'
                }}
            >
                {virtualizer.getVirtualItems().map((virtualItem) => (
                    <div
                        key={virtualItem.key}
                        style={{
                            position: 'absolute',
                            top: 0,
                            left: 0,
                            width: '100%',
                            height: `${virtualItem.size}px`,
                            transform: `translateY(${virtualItem.start}px)`
                        }}
                    >
                        {items[virtualItem.index].name}
                    </div>
                ))}
            </div>
        </div>
    );
}
```

---

### 5.2. React.memo для списков

```tsx
'use client';

import { memo, useCallback } from 'react';

const MemoizedItem = memo(function Item({ item, onUpdate }) {
    console.log('Render item:', item.id);
    
    return (
        <div>
            {item.name}
            <button onClick={() => onUpdate(item.id)}>Update</button>
        </div>
    );
});

function List({ items }) {
    const handleUpdate = useCallback((id: string) => {
        console.log('Update:', id);
    }, []);
    
    return (
        <ul>
            {items.map(item => (
                <MemoizedItem
                    key={item.id}
                    item={item}
                    onUpdate={handleUpdate}
                />
            ))}
        </ul>
    );
}
```

---

## 🔗 Связанные темы

- [[03_React_Frontend/01_React_Architecture_FSD]] — Архитектура React
- [[03_React_Frontend/02_React_Internals_Optimization]] — React оптимизация
- [[03_React_Frontend/04_NextJS_RSC]] — RSC детали

---

*Файл обновлён: 17 марта 2026*
