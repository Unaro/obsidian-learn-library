# ⚛️ React & Frontend Architecture

> **Назначение:** Навигация по темам React, Next.js и архитектуры фронтенда

---

## 📁 Файлы категории

| № | Файл | Описание |
|---|------|----------|
| 01 | [[03_React_Frontend/01_React_Architecture_FSD]] | RSC, FSD, Zustand, AST-анализ |
| 02 | [[03_React_Frontend/02_React_Internals_Optimization]] | Virtual DOM, Fiber, HOC, Portals |
| 03 | [[03_React_Frontend/03_NextJS_App_Router_React19]] | Кэширование, Server Actions, хуки React 19 |
| 04 | [[03_React_Frontend/04_NextJS_RSC]] | RSC vs SSR, гидратация |

---

## 🏗️ Frontend Architecture

### Из файла [[03_React_Frontend/01_React_Architecture_FSD]]

#### 1. React Server Components (RSC)
- Рендерятся **только на сервере**
- **Не попадают** в клиентский бандл
- Прямой доступ к БД и файловой системе
- Не поддерживают `useState`, `useEffect`, обработчики

#### 2. Client Components
- Директива `'use client'`
- SSR + гидратация на клиенте
- Для интерактивного UI

#### 3. Паттерны рендеринга
- **SSR** — динамическая генерация при запросе
- **SSG** — генерация на этапе сборки
- **CSR** — рендеринг в браузере

#### 4. Server Actions
- Асинхронные функции на сервере
- Вызов из клиентских компонентов
- Обработка форм без API routes

#### 5. Feature-Sliced Design (FSD)
- **Слои (сверху вниз):**
  1. `app` — инициализация, провайдеры
  2. `pages` — композиция страниц
  3. `widgets` — самостоятельные блоки (Header, Dashboard)
  4. `features` — бизнес-сценарии (форма, создание метрики)
  5. `entities` — бизнес-сущности (User, Metric)
  6. `shared` — переиспользуемый код (UI-kit, utils)
- **Правило:** импорт только снизу вверх
- **Public API** (`index.ts`) — инкапсуляция

#### 6. Zustand
- Однонаправленный поток (Flux)
- Работа вне React компонентов
- Селективная подписка

#### 7. AST-анализ
- **Уязвимость `eval()`** — XSS
- **Безопасный парсинг:**
  1. Токенизация
  2. Построение AST
  3. Валидация и вычисление

---

## 🔬 React под капотом

### Из файла [[03_React_Frontend/02_React_Internals_Optimization]]

#### 1. Virtual DOM vs Shadow DOM
- **Virtual DOM** — JS-репрезентация, Diffing, точечное обновление
- **Shadow DOM** — инкапсуляция веб-компонентов (не связан с React)

#### 2. React Fiber
- **Reconciliation** — процесс сравнения (Diffing, O(n))
- **Fiber** — разбивка рендеринга на части (chunks)
- Асинхронный рендеринг, не блокирует Event Loop

#### 3. Компоненты высшего порядка (HOC)
- Паттерн Декоратор
- `withAuth(Dashboard)` — обёртка для логики

#### 4. React Portals
- Рендер вне иерархии родителя
- `ReactDOM.createPortal(child, container)`
- Для модалок, тултипов, меню

#### 5. Производительность
- **Lazy Loading + Suspense:**
```javascript
const HeavyComponent = React.lazy(() => import('./HeavyComponent'));
<Suspense fallback={<Spinner />}><HeavyComponent /></Suspense>
```
- **Shimmer UI** — скелетные экраны

---

## 🔄 Next.js App Router & React 19

### Из файла [[03_React_Frontend/03_NextJS_App_Router_React19]]

#### 1. 4 уровня кэширования

| Уровень | Где | Срок жизни |
|---------|-----|------------|
| **Request Memoization** | Сервер, один запрос | Обработка запроса |
| **Data Cache** | Сервер, между запросами | Персистентный |
| **Full Route Cache** | Сервер, SSG | До деплоя |
| **Router Cache** | Браузер | Посещение + ссылки |

#### 2. Инвалидация кэша
- **Time-based:** `next: { revalidate: 3600 }`
- **On-demand:**
  - `revalidatePath('/blog')`
  - `revalidateTag('posts')`

#### 3. Server Actions
- Избавляют от API routes
- **Progressive Enhancement** — работают без JS

#### 4. Хуки React 19
- **`useActionState`** — состояние серверного экшена
- **`useFormStatus`** — статус отправки формы (внутри `<form>`)
- **`useOptimistic`** — оптимистичный UI
- **`use()`** — чтение Promises/Context в условиях и циклах

#### 5. Оптимизация списков
- **Виртуализация (Windowing):** `react-window`, `@tanstack/react-virtual`
- **React.memo** — кэширование при неизменных пропсах

---

## 💧 Гидратация и RSC

### Из файла [[03_React_Frontend/04_NextJS_RSC]]

#### 1. RSC vs SSR
- **SSR:** HTML + JS бандл → гидратация
- **RSC:** Серверный код не попадает к клиенту → RSC Payload

#### 2. Гидратация
- **Hydration Mismatch:**
  - Причина: `Math.random()`, `Date.now()` в разметке
  - Решение: `useEffect` для клиентской динамики

---

## 🔗 Связанные темы

- [[05_Infrastructure/03_TypeScript_Security_Testing]] — типизация компонентов
- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Zustand, Atomic Design
- [[04_Databases/02_Database_Optimization_N1]] — LCP, INP, CLS

---

*Категория: React & Frontend*
