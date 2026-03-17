---
created: 2026-03-17
updated: 2026-03-17
category: React
type: topic
priority: high
status: completed
tags: [react, fiber, virtual-dom, hoc, portals, optimization]
difficulty: advanced
estimated_hours: 8
---

# React Internals & Оптимизация производительности

> **Назначение:** Глубокое понимание внутренней архитектуры React: Fiber, Reconciliation, рендеринг. Продвинутые техники оптимизации производительности.

---

## 1. Virtual DOM vs Shadow DOM

### 1.1. Virtual DOM под капотом

**Что такое Virtual DOM:**
```
┌─────────────────────────────────────────────────────────┐
│              React Rendering Pipeline                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  State Change → Re-render → Virtual DOM Tree           │
│                                      ↓                  │
│  ┌─────────────────────────────────────────────┐       │
│  │         Virtual DOM (JavaScript)            │       │
│  │                                             │       │
│  │  <div>                    <div>             │       │
│  │    <h1>Hello</h1>   →     <h1>Hello</h1>   │       │
│  │    <p>Old</p>             <p>New</p>       │       │
│  │  </div>                   </div>            │       │
│  │                                             │       │
│  └─────────────────────────────────────────────┘       │
│           ↓ Diffing Algorithm (O(n))                   │
│  ┌─────────────────────────────────────────────┐       │
│  │      Минимальные изменения в Real DOM       │       │
│  │      element.textContent = 'New'            │       │
│  └─────────────────────────────────────────────┘       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Пример работы Diffing:**
```tsx
// Старое дерево
const oldVTree = {
    type: 'div',
    props: {
        children: [
            { type: 'h1', props: { children: 'Hello' } },
            { type: 'p', props: { children: 'Old' } }
        ]
    }
};

// Новое дерево
const newVTree = {
    type: 'div',
    props: {
        children: [
            { type: 'h1', props: { children: 'Hello' } },
            { type: 'p', props: { children: 'New' } }
        ]
    }
};

// React сравнивает и находит различия:
// 1. type 'div' === 'div' → не меняем
// 2. type 'h1' === 'h1', children 'Hello' === 'Hello' → пропускаем
// 3. type 'p' === 'p', children 'Old' !== 'New' → обновляем!

// В Real DOM:
// document.querySelector('p').textContent = 'New';
```

---

### 1.2. Shadow DOM (для понимания отличий)

```tsx
// Shadow DOM — браузерная технология инкапсуляции
// НЕ связана с React!

class CustomSlider extends HTMLElement {
    constructor() {
        super();
        
        // Создание Shadow DOM
        const shadow = this.attachShadow({ mode: 'open' });
        
        // Инкапсулированные стили и разметка
        shadow.innerHTML = `
            <style>
                .slider { /* Стили не выйдут за пределы */ }
            </style>
            <div class="slider">
                <input type="range" min="0" max="100">
            </div>
        `;
    }
}

customElements.define('custom-slider', CustomSlider);

// Использование
// <custom-slider></custom-slider>
```

---

## 2. React Fiber и Reconciliation

### 2.1. Алгоритм согласования (Reconciliation)

**Правила Diffing:**

1. **Разные типы элементов → разное дерево:**
```tsx
// <div> → <span>
// React уничтожает старое дерево и создаёт новое
// Все дочерние компоненты будут пересозданы
```

2. **Стабильные key для списков:**
```tsx
// ❌ ПЛОХО: index как key
{items.map((item, index) => (
    <ListItem key={index} data={item} />
))}
// При изменении порядка — React думает что элементы те же

// ✅ ХОРОШО: уникальный ID
{items.map((item) => (
    <ListItem key={item.id} data={item} />
))}
// React точно знает какой элемент изменился
```

3. **Один корневой элемент:**
```tsx
// ❌ Ошибка
return [
    <Child1 />,
    <Child2 />
];

// ✅ Правильно
return (
    <>
        <Child1 />
        <Child2 />
    </>
);
```

---

### 2.2. React Fiber Architecture

**До React 16 (Stack Reconciler):**
```
┌─────────────────────────────────────────┐
│  Stack Reconciler (Синхронный)          │
├─────────────────────────────────────────┤
│                                         │
│  Render: ████████████████████████████  │
│         (нельзя прервать)               │
│                                         │
│  Commit: ████                           │
│                                         │
│  Проблема: Блокировка Event Loop        │
│  для больших деревьев                   │
│                                         │
└─────────────────────────────────────────┘
```

**После React 16 (Fiber Reconciler):**
```
┌─────────────────────────────────────────┐
│  Fiber Reconciler (Асинхронный)         │
├─────────────────────────────────────────┤
│                                         │
│  Render: ███ █ ██ █ ███ █ ██ █ ███     │
│         (можно прервать)                │
│           ↑   ↑    ↑     ↑    ↑         │
│         Event Loop проверяет            │
│         приоритет задач                 │
│                                         │
│  Commit: ████ (одна фаза)               │
│                                         │
│  Преимущество: Не блокирует UI          │
│                                         │
└─────────────────────────────────────────┘
```

**Структура Fiber Node:**
```tsx
interface FiberNode {
    // Тип компонента
    type: string | Function;
    
    // Пропсы
    pendingProps: any;
    
    // Ссылки на другие Fiber
    return: FiberNode | null;      // Родитель
    child: FiberNode | null;       // Первый ребёнок
    sibling: FiberNode | null;     // Следующий брат
    
    // Для отложенной работы
    alternate: FiberNode | null;   // Копия для сравнения
    
    // Приоритет
    lanes: Lanes;  // Приоритет обновления
}
```

---

### 2.3. Фазы рендеринга

**Render Phase (может быть прервана):**
```tsx
// React обходит дерево компонентов
// Вызывает render / function component
// Создаёт/обновляет Virtual DOM

// Может быть прервано для:
// - Событий пользователя (клик, скролл)
// - Таймеров (setTimeout)
// - Сетевых запросов
```

**Commit Phase (не прерывается):**
```tsx
// Применение изменений в DOM
// Вызов useEffect
// Вызов useLayoutEffect

// Не может быть прервано — синхронно
```

---

## 3. Компоненты высшего порядка (HOC)

### 3.1. Паттерн HOC

**Базовый HOC:**
```tsx
function withLogging<P extends object>(
    WrappedComponent: React.ComponentType<P>
) {
    return function WithLogging(props: P) {
        useEffect(() => {
            console.log('Mounting:', WrappedComponent.name);
            return () => {
                console.log('Unmounting:', WrappedComponent.name);
            };
        }, []);
        
        return <WrappedComponent {...props} />;
    };
}

// Использование
const LoggedButton = withLogging(Button);
```

**HOC с параметрами:**
```tsx
function withAuth<P extends object>(
    options: { redirectTo: string }
) {
    return function (WrappedComponent: React.ComponentType<P>) {
        return function WithAuth(props: P) {
            const { user, loading } = useAuth();
            
            if (loading) {
                return <Spinner />;
            }
            
            if (!user) {
                return <Navigate to={options.redirectTo} />;
            }
            
            return <WrappedComponent {...props} user={user} />;
        };
    };
}

// Использование
const ProtectedDashboard = withAuth({ redirectTo: '/login' })(Dashboard);
```

**HOC для извлечения данных:**
```tsx
function withFetch<P extends object, Data>(
    WrappedComponent: React.ComponentType<P & { data: Data }>,
    url: string,
    getData: (response: any) => Data
) {
    return function WithFetch(props: P) {
        const [data, setData] = useState<Data | null>(null);
        const [loading, setLoading] = useState(true);
        const [error, setError] = useState<Error | null>(null);
        
        useEffect(() => {
            async function fetchData() {
                try {
                    const response = await fetch(url);
                    const json = await response.json();
                    setData(getData(json));
                } catch (err) {
                    setError(err as Error);
                } finally {
                    setLoading(false);
                }
            }
            
            fetchData();
        }, [url]);
        
        if (loading) return <Spinner />;
        if (error) return <Error message={error.message} />;
        
        return <WrappedComponent {...props} data={data} />;
    };
}

// Использование
const UserList = withFetch(
    UsersTable,
    '/api/users',
    (response) => response.users
);
```

---

### 3.2. HOC vs Custom Hooks

**HOC (старый подход):**
```tsx
function withUserData<P extends object>(
    WrappedComponent: React.ComponentType<P & { user: User }>
) {
    return function WithUserData(props: P) {
        const [user, setUser] = useState<User | null>(null);
        
        useEffect(() => {
            fetchUser().then(setUser);
        }, []);
        
        return <WrappedComponent {...props} user={user} />;
    };
}

const Profile = withUserData(ProfileComponent);
```

**Custom Hooks (новый подход):**
```tsx
function useUserData() {
    const [user, setUser] = useState<User | null>(null);
    
    useEffect(() => {
        fetchUser().then(setUser);
    }, []);
    
    return user;
}

function Profile() {
    const user = useUserData();
    return <ProfileComponent user={user} />;
}
```

---

## 4. React Portals

### 4.1. Как работают Portals

```tsx
import { createPortal } from 'react-dom';

// HTML структура
// <div id="root"></div>
// <div id="modal-root"></div>

function Modal({ children, onClose }) {
    return createPortal(
        <div className="modal-overlay" onClick={onClose}>
            <div className="modal-content" onClick={(e) => e.stopPropagation()}>
                {children}
            </div>
        </div>,
        document.getElementById('modal-root')  // Рендер вне #root
    );
}

// Использование
function App() {
    const [showModal, setShowModal] = useState(false);
    
    return (
        <div>
            <button onClick={() => setShowModal(true)}>
                Open Modal
            </button>
            
            {showModal && (
                <Modal onClose={() => setShowModal(false)}>
                    <h2>Modal Title</h2>
                    <p>Modal Content</p>
                </Modal>
            )}
        </div>
    );
}
```

---

### 4.2. Практические кейсы

**Tooltip с Portal:**
```tsx
function Tooltip({ target, children }) {
    const [position, setPosition] = useState({ top: 0, left: 0 });
    
    useEffect(() => {
        const rect = target.getBoundingClientRect();
        setPosition({
            top: rect.top - 10,
            left: rect.left + rect.width / 2
        });
    }, [target]);
    
    return createPortal(
        <div 
            className="tooltip"
            style={{
                position: 'fixed',
                top: position.top,
                left: position.left,
                zIndex: 9999  // Поверх всего
            }}
        >
            {children}
        </div>,
        document.body
    );
}
```

**Dropdown внутри таблицы:**
```tsx
// Проблема: overflow: hidden обрезает dropdown
function TableCell({ children }) {
    return (
        <td style={{ overflow: 'hidden' }}>
            {children}
        </td>
    );
}

// Решение: Portal для dropdown
function DropdownCell({ options }) {
    const [isOpen, setIsOpen] = useState(false);
    
    return (
        <TableCell>
            <button onClick={() => setIsOpen(true)}>
                Options
            </button>
            
            {isOpen && (
                <DropdownPortal
                    options={options}
                    onClose={() => setIsOpen(false)}
                />
            )}
        </TableCell>
    );
}
```

---

## 5. Оптимизация производительности

### 5.1. Code Splitting и Lazy Loading

**Базовый lazy loading:**
```tsx
import { lazy, Suspense } from 'react';

// Компонент загружается только при использовании
const HeavyChart = lazy(() => import('./HeavyChart'));
const Dashboard = lazy(() => import('./Dashboard'));

function App() {
    return (
        <Suspense fallback={<Spinner />}>
            <Dashboard />
            <HeavyChart />
        </Suspense>
    );
}
```

**Маршрутная загрузка (React Router):**
```tsx
import { lazy } from 'react';
import { createBrowserRouter, RouterProvider } from 'react-router-dom';

const HomePage = lazy(() => import('./pages/HomePage'));
const ProductPage = lazy(() => import('./pages/ProductPage'));
const CheckoutPage = lazy(() => import('./pages/CheckoutPage'));

const router = createBrowserRouter([
    {
        path: '/',
        element: (
            <Suspense fallback={<PageLoader />}>
                <HomePage />
            </Suspense>
        )
    },
    {
        path: '/product/:id',
        element: (
            <Suspense fallback={<PageLoader />}>
                <ProductPage />
            </Suspense>
        )
    },
    {
        path: '/checkout',
        element: (
            <Suspense fallback={<PageLoader />}>
                <CheckoutPage />
            </Suspense>
        )
    }
]);

function App() {
    return <RouterProvider router={router} />;
}
```

---

### 5.2. React.memo для оптимизации рендеров

**Базовое использование:**
```tsx
// Без memo — ререндер при каждом изменении родителя
function ListItem({ item }) {
    console.log('Render:', item.id);
    return <div>{item.name}</div>;
}

function List({ items }) {
    return (
        <ul>
            {items.map(item => (
                <ListItem key={item.id} item={item} />
            ))}
        </ul>
    );
}

// С memo — ререндер только при изменении props
const MemoListItem = React.memo(ListItem);

function List({ items }) {
    return (
        <ul>
            {items.map(item => (
                <MemoListItem key={item.id} item={item} />
            ))}
        </ul>
    );
}
```

**Custom comparison function:**
```tsx
const ExpensiveComponent = React.memo(
    function ExpensiveComponent({ data, config }) {
        // Дорогие вычисления
        return <div>{/* ... */}</div>;
    },
    (prevProps, nextProps) => {
        // Глубокое сравнение для config
        return (
            prevProps.data === nextProps.data &&
            JSON.stringify(prevProps.config) === 
            JSON.stringify(nextProps.config)
        );
    }
);
```

---

### 5.3. Shimmer UI (Скелетные экраны)

**CSS анимация:**
```css
.shimmer {
    background: linear-gradient(
        90deg,
        #f0f0f0 25%,
        #e0e0e0 50%,
        #f0f0f0 75%
    );
    background-size: 200% 100%;
    animation: shimmer 1.5s infinite;
}

@keyframes shimmer {
    0% { background-position: 200% 0; }
    100% { background-position: -200% 0; }
}
```

**Компонент скелета:**
```tsx
function CardSkeleton() {
    return (
        <div className="card">
            <div className="shimmer" style={{ width: '100%', height: '200px' }} />
            <div className="shimmer" style={{ width: '60%', height: '20px', marginTop: '16px' }} />
            <div className="shimmer" style={{ width: '40%', height: '16px', marginTop: '8px' }} />
        </div>
    );
}

function CardList() {
    const [cards, setCards] = useState(null);
    
    return (
        <div>
            {cards ? (
                cards.map(card => <Card key={card.id} data={card} />)
            ) : (
                <>
                    <CardSkeleton />
                    <CardSkeleton />
                    <CardSkeleton />
                </>
            )}
        </div>
    );
}
```

---

### 5.4. Виртуализация списков

**react-window:**
```tsx
import { FixedSizeList } from 'react-window';

function Row({ index, style }) {
    return (
        <div style={style}>
            Row {index}
        </div>
    );
}

function VirtualizedList({ items }) {
    return (
        <FixedSizeList
            height={600}  // Высота контейнера
            itemCount={items.length}
            itemSize={35}  // Высота одного элемента
            width="100%"
        >
            {({ index, style }) => (
                <Row key={items[index].id} index={index} style={style} />
            )}
        </FixedSizeList>
    );
}

// Рендерятся только видимые элементы (~20 из 10000)
```

**@tanstack/react-virtual:**
```tsx
import { useVirtualizer } from '@tanstack/react-virtual';

function VirtualList({ items }) {
    const parentRef = useRef(null);
    
    const virtualizer = useVirtualizer({
        count: items.length,
        getScrollElement: () => parentRef.current,
        estimateSize: () => 35,
        overscan: 5  // Запас элементов
    });
    
    return (
        <div ref={parentRef} style={{ height: '600px', overflow: 'auto' }}>
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

## 🔗 Связанные темы

- [[03_React_Frontend/01_React_Architecture_FSD]] — Архитектура React
- [[03_React_Frontend/03_NextJS_App_Router_React19]] — Next.js оптимизация
- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Event Loop для понимания

---

*Файл обновлён: 17 марта 2026*
