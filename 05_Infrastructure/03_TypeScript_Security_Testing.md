# TypeScript, Безопасность и Тестирование

> **Назначение:** Продвинутый TypeScript, веб-безопасность (OWASP), пирамида тестирования. Включает практические примеры защиты и лучшие практики.

---

## 1. TypeScript Core

### 1.1. interface vs type

```typescript
// interface — для объектов, поддерживает слияние
interface User {
    id: number;
    name: string;
}

// Слияние интерфейсов (declaration merging)
interface User {
    email: string;  // Добавляется к существующему
}

const user: User = {
    id: 1,
    name: 'John',
    email: 'john@example.com'
};

// type — универсальный алиас
type ID = string | number;  // Union

type Coordinates = [number, number];  // Tuple

type Point = {
    x: number;
    y: number;
};

// Пересечение (Intersection)
type UserWithRole = User & {
    role: 'admin' | 'user';
};

// НО: interface поддерживает extends, type — нет
interface Employee extends User {
    department: string;
}

// type поддерживает вычисляемые свойства
type Mapped<T> = {
    [K in keyof T]: T[K] | null;
};
```

**Когда что использовать:**
```
✅ interface:
- Описание формы объектов
- Расширение через extends
- Слияние деклараций
- Публичные API библиотек

✅ type:
- Union типы (A | B)
- Примитивы (string | number)
- Tuple
- Mapped types
- Utility types
```

---

### 1.2. Generics (Дженерики)

**Базовые дженерики:**
```typescript
// Функция с дженериком
function identity<T>(arg: T): T {
    return arg;
}

identity<string>('Hello');  // Явное указание
identity(42);  // Вывод типа (number)

// Ограничение дженерика
function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
}

getLength('Hello');  // OK (string имеет length)
getLength([1, 2, 3]);  // OK (array имеет length)
getLength(42);  // Error (number не имеет length)
```

**Дженерики в React:**
```tsx
// Generic компонент
interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
    return (
        <ul>
            {items.map((item, index) => (
                <li key={index}>{renderItem(item)}</li>
            ))}
        </ul>
    );
}

// Использование
<List
    items={[{ id: 1, name: 'John' }]}
    renderItem={(user) => <span>{user.name}</span>}
/>;

// Generic хук
function useFetch<T>(url: string) {
    const [data, setData] = useState<T | null>(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
        fetch(url)
            .then(res => res.json())
            .then(setData)
            .finally(() => setLoading(false));
    }, [url]);
    
    return { data, loading };
}

// Использование
const { data: user } = useFetch<User>('/api/user/1');
const { data: posts } = useFetch<Post[]>('/api/posts');
```

---

### 1.3. Utility Types

**Встроенные утилиты:**
```typescript
interface Product {
    id: number;
    name: string;
    price: number;
    description: string;
    createdAt: Date;
}

// Partial<T> — все свойства опциональны
type PartialProduct = Partial<Product>;
// { id?: number; name?: string; ... }

// Required<T> — все свойства обязательны
type RequiredProduct = Required<Product>;
// { id: number; name: string; ... }

// Pick<T, K> — выбрать свойства
type ProductPreview = Pick<Product, 'id' | 'name' | 'price'>;
// { id: number; name: string; price: number; }

// Omit<T, K> — исключить свойства
type ProductCreate = Omit<Product, 'id' | 'createdAt'>;
// { name: string; price: number; description: string; }

// Record<K, T> — словарь
type ProductPrices = Record<string, number>;
// { [key: string]: number }

// Exclude<T, U> — исключить из union
type Status = 'pending' | 'success' | 'error';
type NonErrorStatus = Exclude<Status, 'error'>;
// 'pending' | 'success'

// Extract<T, U> — оставить только совпадающие
type OnlyError = Extract<Status, 'error'>;
// 'error'

// ReturnType<T> — тип возвращаемого значения
function createProduct(): Product { /* ... */ }
type ProductType = ReturnType<typeof createProduct>;

// Parameters<T> — тип параметров функции
function updateUser(id: number, data: Partial<Product>) { /* ... */ }
type UpdateParams = Parameters<typeof updateUser>;
// [number, Partial<Product>]
```

**Собственные utility types:**
```typescript
// Nullable<T> — добавить null
type Nullable<T> = T | null;

type NullableUser = Nullable<User>;  // User | null

// DeepPartial<T> — рекурсивно все свойства опциональны
type DeepPartial<T> = {
    [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface Config {
    database: {
        host: string;
        port: number;
    };
}

type PartialConfig = DeepPartial<Config>;
// { database?: { host?: string; port?: number; } }

// Readonly<T> — все свойства только для чтения
type ReadonlyUser = Readonly<User>;
// { readonly id: number; readonly name: string; ... }
```

---

### 1.4. Type Guards и Narrowing

**Встроенные guards:**
```typescript
function process(value: string | number | null) {
    // typeof guard
    if (typeof value === 'string') {
        return value.toUpperCase();  // value: string
    }
    
    if (typeof value === 'number') {
        return value.toFixed(2);  // value: number
    }
    
    // null check
    if (value === null) {
        return 'No value';
    }
    
    // После всех проверок — never (недостижимо)
    return value;  // value: never
}

// in guard
interface Cat { meow(): void; }
interface Dog { bark(): void; }

function makeSound(animal: Cat | Dog) {
    if ('meow' in animal) {
        animal.meow();  // animal: Cat
    } else {
        animal.bark();  // animal: Dog
    }
}

// instanceof guard
function handleError(error: Error | string) {
    if (error instanceof Error) {
        return error.message;
    }
    return error;
}
```

**Кастомные type predicates:**
```typescript
interface User {
    id: number;
    name: string;
    email?: string;
}

// Type predicate: obj is User
function isUser(obj: unknown): obj is User {
    return (
        typeof obj === 'object' &&
        obj !== null &&
        'id' in obj &&
        'name' in obj &&
        typeof (obj as User).id === 'number' &&
        typeof (obj as User).name === 'string'
    );
}

// Использование
function processUser(data: unknown) {
    if (isUser(data)) {
        // TypeScript знает, что data — User
        console.log(data.id, data.name);
    }
}

// Asserts predicate
function assertsIsUser(obj: unknown): asserts obj is User {
    if (!isUser(obj)) {
        throw new Error('Not a valid User');
    }
}

function processUser2(data: unknown) {
    assertsIsUser(data);
    // После этой проверки data имеет тип User
    console.log(data.id);
}
```

---

## 2. Web-Безопасность (OWASP)

### 2.1. XSS (Cross-Site Scripting)

**Типы XSS:**
```
┌─────────────────────────────────────────────────────────┐
│              XSS Attack Types                           │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Reflected XSS                                       │
│     Злоумышленник → Сервер → Жертва                    │
│     https://site.com/search?q=<script>evil()</script>  │
│                                                         │
│  2. Stored XSS                                          │
│     Злоумышленник → Сервер (БД) → Жертва               │
│     Комментарий с вредоносным скриптом                 │
│                                                         │
│  3. DOM-based XSS                                       │
│     Злоумышленник → Жертва (через JS)                  │
│     document.innerHTML = userInput                     │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Защита в React:**
```tsx
// ❌ ОПАСНО: dangerouslySetInnerHTML
function Comment({ content }) {
    return <div dangerouslySetInnerHTML={{ __html: content }} />;
}
// Если content = '<script>steal()</script>' → XSS!

// ✅ БЕЗОПАСНО: экранирование по умолчанию
function Comment({ content }) {
    return <div>{content}</div>;  // React автоматически экранирует
}

// ✅ Для HTML используйте sanitize
import DOMPurify from 'dompurify';

function SafeHTML({ html }) {
    const clean = DOMPurify.sanitize(html);
    return <div dangerouslySetInnerHTML={{ __html: clean }} />;
}
```

**Content Security Policy (CSP):**
```html
<!-- Заголовок для защиты от XSS -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' https://trusted.com; 
               style-src 'self' 'unsafe-inline';
               img-src 'self' data: https:;">

<!-- Или в HTTP заголовке -->
Content-Security-Policy: default-src 'self'; script-src 'self'
```

---

### 2.2. SQL Injection

**Уязвимый код:**
```typescript
// ❌ ОПАСНО: конкатенация SQL
async function getUser(username: string) {
    const query = `SELECT * FROM users WHERE username = '${username}'`;
    return db.query(query);
}

// Атака:
// username = "admin' OR '1'='1"
// SQL: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// → Вернёт всех пользователей!
```

**Защита: параметризованные запросы:**
```typescript
// ✅ БЕЗОПАСНО: параметризованный запрос
async function getUser(username: string) {
    return db.query(
        'SELECT * FROM users WHERE username = $1',
        [username]  // Параметры экранируются автоматически
    );
}

// ✅ С ORM (Drizzle)
import { eq } from 'drizzle-orm';

async function getUser(username: string) {
    return db.select().from(users)
        .where(eq(users.username, username));
}

// ✅ С Django ORM
def get_user(username):
    return User.objects.get(username=username)  # Автоматическая защита
```

---

### 2.3. CORS

**Настройка сервера:**
```typescript
import cors from 'cors';
import express from 'express';

const app = express();

// ❌ Опасно: разрешает всё
app.use(cors({ origin: '*' }));

// ✅ Безопасно: конкретные домены
app.use(cors({
    origin: (origin, callback) => {
        const allowed = [
            'https://myapp.com',
            'https://www.myapp.com'
        ];
        
        if (!origin || allowed.includes(origin)) {
            callback(null, true);
        } else {
            callback(new Error('Not allowed by CORS'));
        }
    },
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization']
}));
```

**Preflight запрос:**
```
Запрос браузера:
OPTIONS /api/data HTTP/1.1
Origin: https://myapp.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization

Ответ сервера:
HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://myapp.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true
```

---

### 2.4. OWASP Top 10

```
┌─────────────────────────────────────────────────────────┐
│              OWASP Top 10 2021                          │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  A01:2021 Broken Access Control                        │
│  - Проверка прав доступа на сервере                    │
│  - Principle of Least Privilege                         │
│                                                         │
│  A02:2021 Cryptographic Failures                        │
│  - HTTPS обязательно                                    │
│  - Хэширование паролей (bcrypt, argon2)                │
│  - Не хранить секреты в коде                            │
│                                                         │
│  A03:2021 Injection                                     │
│  - Параметризованные запросы (SQL, NoSQL)              │
│  - Валидация входных данных                             │
│                                                         │
│  A04:2021 Insecure Design                               │
│  - Threat modeling на этапе проектирования              │
│  - Secure by default                                    │
│                                                         │
│  A05:2021 Security Misconfiguration                     │
│  - Отключить debug в production                         │
│  - Удалить тестовые данные                              │
│  - Обновлять зависимости                                │
│                                                         │
│  A06:2021 Vulnerable Components                         │
│  - npm audit, yarn audit                                │
│  - Snyk, Dependabot для мониторинга                     │
│                                                         │
│  A07:2021 Authentication Failures                       │
│  - MFA (Multi-Factor Authentication)                    │
│  - Rate limiting для login                              │
│  - Secure session management                            │
│                                                         │
│  A08:2021 Software and Data Integrity Failures          │
│  - Подпись кода и зависимостей                          │
│  - CI/CD security                                       │
│                                                         │
│  A09:2021 Security Logging and Monitoring               │
│  - Логирование всех попыток доступа                     │
│  - Alert system для аномалий                            │
│                                                         │
│  A10:2021 Server-Side Request Forgery (SSRF)            │
│  - Валидация URL от пользователя                        │
│  - Whitelist допустимых доменов                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Пирамида тестирования

### 3.1. Уровни тестирования

```
┌─────────────────────────────────────────────────────────┐
│              Testing Pyramid                            │
├─────────────────────────────────────────────────────────┤
│                                                         │
│           /\                                              │
│          /  \   E2E Tests (10%)                         │
│         /____\  Cypress, Playwright                     │
│        /      \                                          │
│       /        \ Integration Tests (20%)                │
│      /__________\ React Testing Library, Supertest      │
│     /            \                                       │
│    /              \ Unit Tests (70%)                     │
│   /________________\ Jest, Vitest, PyTest                │
│                                                         │
│  Скорость:  Быстро ←───────────→ Медленно              │
│  Стоимость: Дёшево ←───────────→ Дорого                │
│  Надёжность: Высокая ←─────────→ Низкая                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 3.2. Unit тесты (Jest)

```typescript
// math.test.ts
import { sum, divide } from './math';

describe('sum', () => {
    it('adds two numbers', () => {
        expect(sum(1, 2)).toBe(3);
    });
    
    it('handles negative numbers', () => {
        expect(sum(-1, -2)).toBe(-3);
    });
});

describe('divide', () => {
    it('divides two numbers', () => {
        expect(divide(10, 2)).toBe(5);
    });
    
    it('throws on division by zero', () => {
        expect(() => divide(10, 0)).toThrow('Division by zero');
    });
});

// Моки
import { fetchUser } from './api';

jest.mock('./api');

describe('UserService', () => {
    it('fetches user data', async () => {
        (fetchUser as jest.Mock).mockResolvedValue({
            id: 1,
            name: 'John'
        });
        
        const user = await fetchUser('1');
        
        expect(user).toEqual({ id: 1, name: 'John' });
        expect(fetchUser).toHaveBeenCalledWith('1');
    });
});

// Spy
describe('Logger', () => {
    it('logs messages', () => {
        const consoleSpy = jest.spyOn(console, 'log');
        
        logger.log('Hello');
        
        expect(consoleSpy).toHaveBeenCalledWith('Hello');
        consoleSpy.mockRestore();
    });
});
```

---

### 3.3. Integration тесты (React Testing Library)

```tsx
// LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { LoginForm } from './LoginForm';
import { MemoryRouter } from 'react-router-dom';

function renderWithProviders(component: React.ReactElement) {
    return render(
        <MemoryRouter>
            {component}
        </MemoryRouter>
    );
}

describe('LoginForm', () => {
    it('renders form fields', () => {
        renderWithProviders(<LoginForm />);
        
        expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
        expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
    });
    
    it('shows validation errors', async () => {
        renderWithProviders(<LoginForm />);
        
        fireEvent.click(screen.getByRole('button', { name: /login/i }));
        
        await waitFor(() => {
            expect(screen.getByText(/email is required/i)).toBeInTheDocument();
        });
    });
    
    it('submits form successfully', async () => {
        global.fetch = jest.fn().mockResolvedValue({
            ok: true,
            json: async () => ({ token: 'abc' })
        });
        
        renderWithProviders(<LoginForm />);
        
        fireEvent.change(screen.getByLabelText(/email/i), {
            target: { value: 'test@example.com' }
        });
        fireEvent.change(screen.getByLabelText(/password/i), {
            target: { value: 'password123' }
        });
        
        fireEvent.click(screen.getByRole('button', { name: /login/i }));
        
        await waitFor(() => {
            expect(global.fetch).toHaveBeenCalledWith(
                '/api/login',
                expect.objectContaining({
                    method: 'POST',
                    body: expect.any(String)
                })
            );
        });
    });
});
```

---

### 3.4. E2E тесты (Playwright)

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Login Flow', () => {
    test.beforeEach(async ({ page }) => {
        await page.goto('/login');
    });
    
    test('successful login', async ({ page }) => {
        await page.fill('[name="email"]', 'test@example.com');
        await page.fill('[name="password"]', 'password123');
        await page.click('button[type="submit"]');
        
        await expect(page).toHaveURL('/dashboard');
        await expect(page.locator('text=Welcome')).toBeVisible();
    });
    
    test('invalid credentials', async ({ page }) => {
        await page.fill('[name="email"]', 'wrong@example.com');
        await page.fill('[name="password"]', 'wrong');
        await page.click('button[type="submit"]');
        
        await expect(page.locator('text=Invalid credentials')).toBeVisible();
        await expect(page).toHaveURL('/login');
    });
    
    test('form validation', async ({ page }) => {
        await page.click('button[type="submit"]');
        
        await expect(page.locator('text=Email is required')).toBeVisible();
        await expect(page.locator('text=Password is required')).toBeVisible();
    });
});

// tests/e2e/checkout.spec.ts
test.describe('Checkout Process', () => {
    test('complete purchase', async ({ page }) => {
        // Добавить товар в корзину
        await page.goto('/products/1');
        await page.click('button:has-text("Add to Cart")');
        
        // Перейти к оформлению
        await page.click('[data-testid="cart-button"]');
        await page.click('button:has-text("Checkout")');
        
        // Заполнить форму доставки
        await page.fill('[name="address"]', '123 Main St');
        await page.fill('[name="city"]', 'New York');
        await page.fill('[name="zip"]', '10001');
        
        // Оплатить
        await page.click('button:has-text("Pay Now")');
        
        // Проверка успеха
        await expect(page.locator('text=Order confirmed')).toBeVisible();
        await expect(page).toHaveURL('/orders/\\d+');
    });
});
```

---

### 3.5. Конфигурация Playwright

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
    testDir: './tests/e2e',
    timeout: 30000,
    expect: {
        timeout: 5000
    },
    fullyParallel: true,
    forbidOnly: !!process.env.CI,
    retries: process.env.CI ? 2 : 0,
    workers: process.env.CI ? 1 : undefined,
    reporter: 'html',
    
    use: {
        baseURL: 'http://localhost:3000',
        trace: 'on-first-retry',
        screenshot: 'only-on-failure'
    },
    
    projects: [
        {
            name: 'chromium',
            use: { ...devices['Desktop Chrome'] }
        },
        {
            name: 'firefox',
            use: { ...devices['Desktop Firefox'] }
        },
        {
            name: 'webkit',
            use: { ...devices['Desktop Safari'] }
        },
        {
            name: 'Mobile Chrome',
            use: { ...devices['Pixel 5'] }
        },
        {
            name: 'Mobile Safari',
            use: { ...devices['iPhone 12'] }
        }
    ],
    
    webServer: {
        command: 'npm run dev',
        port: 3000,
        reuseExistingServer: !process.env.CI
    }
});
```

---

## 4. TailwindCSS

### 4.1. Utility-First подход

```tsx
// ❌ Традиционный CSS
// Button.css
.button {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    padding: 0.75rem 1.5rem;
    font-weight: 600;
    background-color: #3b82f6;
    color: white;
    border-radius: 0.5rem;
    transition: background-color 0.2s;
}

.button:hover {
    background-color: #2563eb;
}

// Button.tsx
import './Button.css';
export function Button({ children }) {
    return <button className="button">{children}</button>;
}

// ✅ TailwindCSS
export function Button({ children }) {
    return (
        <button className="
            inline-flex items-center justify-center
            px-6 py-3 font-semibold
            bg-blue-500 text-white
            rounded-lg
            transition-colors
            hover:bg-blue-600
        ">
            {children}
        </button>
    );
}
```

---

### 4.2. JIT компилятор

```
┌─────────────────────────────────────────────────────────┐
│              Tailwind JIT Compilation                   │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Сканирование исходных файлов                        │
│     src/**/*.tsx, src/**/*.ts                           │
│                                                         │
│  2. Извлечение классов                                  │
│     "bg-red-500", "text-white", "p-4"                   │
│                                                         │
│  3. Генерация CSS только для используемых классов       │
│     .bg-red-500 { background-color: #ef4444; }          │
│     .text-white { color: #ffffff; }                     │
│     .p-4 { padding: 1rem; }                             │
│                                                         │
│  4. Результат: ~5-10 KB вместо ~3 MB                    │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## 🔗 Связанные темы

- [[05_Infrastructure/01_Infrastructure_Networks_Docker]] — Docker, сети
- [[05_Infrastructure/02_System_Analysis_BPMN_UML]] — Системный анализ
- [[03_React_Frontend/01_React_Architecture_FSD]] — React архитектура

---

*Файл обновлён: 17 марта 2026*
