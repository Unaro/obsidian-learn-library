# ⚡ JavaScript & Node.js

> **Назначение:** Навигация по темам JavaScript, Node.js и асинхронного программирования

---

## 📁 Файлы категории

| № | Файл | Описание |
|---|------|----------|
| 01 | [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] | Event Loop, контекст, замыкания, стейт-менеджеры |
| 02 | [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding]] | Live-coding задачи, Promise, события, память |
| 03 | [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]] | Streams, Event Loop фазы, Worker Threads |
| 04 | [[02_JavaScript_NodeJS/04_NodeJS_Threads_Queues]] | Потоки, воркеры, очереди, микросервисы |

---

## 🧠 JavaScript Core

### Из файла [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]]

#### 1. Event Loop в браузере
- **Call Stack** (LIFO)
- **Web APIs** (setTimeout, DOM, fetch)
- **Microtask Queue** (наивысший приоритет):
  - Promise коллбэки (`.then`, `.catch`, `.finally`)
  - `queueMicrotask()`
- **Macrotask Queue** (низший приоритет):
  - `setTimeout`, `setInterval`
  - DOM события
  - Сетевые запросы

#### 2. Event Loop в Node.js (libuv)
- **Фазы:**
  1. Timers
  2. Pending Callbacks
  3. Idle, Prepare
  4. **Poll** (I/O события)
  5. Check (`setImmediate`)
  6. Close Callbacks
- **Микрозадачи** выполняются между фазами
- `process.nextTick()` — абсолютный приоритет
- **Пул потоков** (4 потока по умолчанию)

#### 3. Ключевые концепции
- **Контекст выполнения** и `this`
- **Стрелочные функции** (лексический `this`)
- **Замыкания** (closure)
- **Hoisting** (всплытие var, function, TDZ для let/const)

#### 4. Архитектурные паттерны
- **Redux Toolkit** — единый store, actions, reducers
- **Context API** — для редко меняющихся данных
- **Zustand** — селективная подписка, нет boilerplate
- **Atomic Design:** Атомы → Молекулы → Организмы → Шаблоны → Страницы

---

## 💻 Продвинутый JavaScript

### Из файла [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding]]

#### 1. Live-Coding функции

**Debounce:**
```javascript
function debounce(fn, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

**Throttle:**
```javascript
function throttle(fn, delay) {
  let shouldWait = false;
  return function(...args) {
    if (shouldWait) return;
    fn.apply(this, args);
    shouldWait = true;
    setTimeout(() => shouldWait = false, delay);
  };
}
```

**Currying:**
```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...nextArgs) {
      return curried.apply(this, args.concat(nextArgs));
    };
  };
}
```

#### 2. Глубокое копирование
- **Shallow:** `Object.assign()`, spread `{...obj}`
- **Deep:**
  - `JSON.parse(JSON.stringify(obj))` — теряет методы, Date, undefined
  - `structuredClone(obj)` — современный стандарт
  - Рекурсивный полифил

#### 3. Promise методы
- `Promise.all()` — ждёт все, падает при первой ошибке
- `Promise.allSettled()` — ждёт все независимо от результата
- `Promise.race()` — первый завершённый (любой)
- `Promise.any()` — первый успешный

#### 4. События в браузере
- **3 фазы:** Capturing → Target → Bubbling
- `event.target` vs `event.currentTarget`
- **Делегирование событий**

#### 5. Управление памятью
- **Garbage Collector** (Mark-and-Sweep)
- **WeakMap / WeakSet** — не предотвращают сборку мусора
- **Ограничение объектов:**
  - `Object.preventExtensions()` — нет новых свойств
  - `Object.seal()` — нет добавления/удаления
  - `Object.freeze()` — полная заморозка

---

## 🔄 Node.js Streams & Multithreading

### Из файла [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]]

#### 1. Streams (Потоки)
- **Проблема:** чтение больших файлов в память
- **Типы:**
  - **Readable** — источник (чтение файла, HTTP-запрос)
  - **Writable** — приёмник (запись, HTTP-ответ)
  - **Duplex** — чтение и запись (TCP-сокеты)
  - **Transform** — модификация данных (zlib, парсинг)
- **Backpressure** — механизм регулировки скорости
- **`.pipe()`** — связывание потоков

#### 2. Worker Threads
- Для **CPU-bound** задач
- Изолированный Event Loop
- `SharedArrayBuffer` для общей памяти
- `postMessage` для коммуникации

#### 3. Real-Time протоколы
- **WebSockets** — двунаправленное (Full-Duplex)
- **SSE** — однонаправленное (Server → Client)
- **Long Polling** — fallback для сокетов

---

## 📬 Очереди и микросервисы

### Из файла [[02_JavaScript_NodeJS/04_NodeJS_Threads_Queues]]

#### 1. Очереди сообщений
- **Producer-Consumer паттерн**
- **Инструменты:**
  - Redis + BullMQ / Bull — in-memory, быстрые
  - RabbitMQ / Kafka — брокеры для микросервисов

#### 2. Event Emitter
- Паттерн **Наблюдатель (Observer)**
- Множество слушателей на одно событие

#### 3. Микросервисная архитектура
- **Монолит** vs **Микросервисы**
- Плюсы: независимое масштабирование, отказоустойчивость
- Минусы: сложность инфраструктуры, распределённые транзакции

---

## 🔗 Связанные темы

- [[05_Infrastructure/03_TypeScript_Security_Testing]] — типизация, OWASP
- [[03_React_Frontend/02_React_Internals_Optimization]] — Virtual DOM, Fiber
- [[03_React_Frontend/01_React_Architecture_FSD]] — RSC, FSD
- [[04_Databases/02_Database_Optimization_N1]] — DataLoader, метрики

---

*Категория: JavaScript & Node.js*
