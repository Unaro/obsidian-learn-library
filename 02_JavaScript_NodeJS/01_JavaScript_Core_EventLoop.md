# JavaScript Core & Event Loop: Полное руководство

> **Назначение:** Глубокое понимание JavaScript: движок, Event Loop, замыкания, прототипы, асинхронность. Включает сравнение с Python и частые вопросы на собеседованиях.

---

## 1. Движок JavaScript и среда выполнения

### 1.1. Архитектура JavaScript Engine

```
┌─────────────────────────────────────────────────────────┐
│            JavaScript Engine (V8, SpiderMonkey)         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌──────────────┐    ┌──────────────┐                  │
│  │  Call Stack  │    │     Heap     │                  │
│  │  (LIFO)      │    │  (Память)    │                  │
│  └──────────────┘    └──────────────┘                  │
│                                                         │
│  ┌──────────────────────────────────────────┐           │
│  │         Web APIs / Node APIs             │           │
│  │  (setTimeout, DOM, fetch, fs, net)       │           │
│  └──────────────────────────────────────────┘           │
│                                                         │
│  ┌──────────────┐    ┌──────────────┐                  │
│  │ Microtask    │    │ Macrotask    │                  │
│  │   Queue      │    │    Queue     │                  │
│  └──────────────┘    └──────────────┘                  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Call Stack (Стек вызовов):**
- **LIFO** (Last In, First Out)
- Хранит кадры стека (stack frames)
- Каждый вызов функции добавляет кадр
- При возврате — кадр удаляется

```javascript
function first() {
    console.log("First");
    second();
}

function second() {
    console.log("Second");
    third();
}

function third() {
    console.log("Third");
}

first();
// Stack:
// third()
// second()
// first()
// global
```

---

### 1.2. Event Loop в браузере

**Алгоритм работы:**
```
1. Выполнить весь синхронный код (очистить Call Stack)
2. Выполнить ВСЕ микрозадачи из Microtask Queue
3. Выполнить ОДНУ макрозадачу из Macrotask Queue
4. Вернуться к шагу 1
5. Отрисовать UI (после каждой макрозадачи)
```

**Что куда попадает:**

| Microtask Queue | Macrotask Queue |
|----------------|-----------------|
| `Promise.then/catch/finally` | `setTimeout` |
| `queueMicrotask()` | `setInterval` |
| `MutationObserver` | DOM события (клик, скролл) |
| — | Сетевые запросы (fetch) |
| — | `requestAnimationFrame` |

**Пример с приоритетами:**
```javascript
console.log('1. Script start');

setTimeout(() => {
    console.log('2. setTimeout');
}, 0);

Promise.resolve()
    .then(() => console.log('3. Promise 1'))
    .then(() => console.log('4. Promise 2'));

queueMicrotask(() => {
    console.log('5. queueMicrotask');
});

console.log('6. Script end');

// Вывод:
// 1. Script start
// 6. Script end
// 3. Promise 1
// 4. Promise 2
// 5. queueMicrotask
// 2. setTimeout
```

**⚠️ Бесконечная микрозадача (зависание UI):**
```javascript
// БЛОКИРОВКА Event Loop!
Promise.resolve().then(function recurse() {
    Promise.resolve().then(recurse);
});
// UI никогда не обновится, браузер зависнет
```

---

### 1.3. Event Loop в Node.js (libuv)

**Фазы Event Loop:**
```
   ┌───────────────────────────┐
┌─>│           Timers          │
│  │  setTimeout, setInterval  │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │     Pending Callbacks     │
│  │  I/O (TCP, UDP errors)    │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │      Idle, Prepare        │
│  │    (внутренние нужды)     │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │           Poll            │ ◄── Самая важная фаза
│  │  I/O callbacks, fs, net   │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │           Check           │
│  │      setImmediate()       │
│  └─────────────┬─────────────┘
│                │
│  ┌─────────────▼─────────────┐
│  │     Close Callbacks       │
│  │  socket.on('close', ...)  │
│  └───────────────────────────┘
```

**Микрозадачи между фазами:**
```javascript
const fs = require('fs');

setTimeout(() => {
    console.log('timeout');
}, 0);

setImmediate(() => {
    console.log('immediate');
});

// В Node.js: зависит от контекста
// В callback I/O: сначала immediate
// В главном контексте: неопределено (50/50)
```

**`process.nextTick()` — абсолютный приоритет:**
```javascript
Promise.resolve().then(() => {
    console.log('Promise 1');
    process.nextTick(() => console.log('nextTick 2'));
});

process.nextTick(() => {
    console.log('nextTick 1');
    process.nextTick(() => console.log('nextTick 3'));
});

// Вывод:
// nextTick 1
// nextTick 2
// nextTick 3
// Promise 1
```

---

### 1.4. Пул потоков (Thread Pool)

**libuv Thread Pool (4 потока по умолчанию):**

| Операция | Использует Thread Pool |
|----------|----------------------|
| `fs` (файловая система) | ✅ Да |
| `crypto` (криптография) | ✅ Да |
| `zlib` (сжатие) | ✅ Да |
| `dns` (DNS-запросы) | ✅ Да |
| `http/https` (сеть) | ❌ Нет (асинхронно) |
| `setTimeout` | ❌ Нет (таймеры) |

**Увеличение пула:**
```bash
# Перед запуском Node.js
export UV_THREADPOOL_SIZE=8
node app.js

# Или в коде (до любых async операций)
process.env.UV_THREADPOOL_SIZE = 8;
```

---

## 2. Ключевые концепции JavaScript

### 2.1. Контекст выполнения и `this`

**Execution Context:**
```javascript
// Глобальный контекст
var globalVar = 'global';

function outer() {
    var outerVar = 'outer';
    
    function inner() {
        var innerVar = 'inner';
        console.log(globalVar, outerVar, innerVar);
    }
    
    inner();
}

outer();
```

**`this` в разных контекстах:**
```javascript
// 1. Глобальный контекст
console.log(this);  // Window (в браузере), global (в Node.js)

// 2. Метод объекта
const obj = {
    name: 'Object',
    getName() {
        return this.name;
    }
};
obj.getName();  // 'Object'

// 3. Обычная функция
function regular() {
    console.log(this);
}
regular();  // Window (в strict mode: undefined)

// 4. Стрелочная функция (лексический this)
const arrow = () => {
    console.log(this);  // Берётся из родительской области
};

// 5. Конструктор
function Person(name) {
    this.name = name;
}
const person = new Person('John');

// 6. call, apply, bind
function greet(greeting) {
    return `${greeting}, ${this.name}`;
}
greet.call(person, 'Hello');     // 'Hello, John'
greet.apply(person, ['Hi']);     // 'Hi, John'
const boundGreet = greet.bind(person);
boundGreet('Hey');               // 'Hey, John'
```

**Стрелочные функции — особенности:**
```javascript
const obj = {
    name: 'Object',
    
    // Обычная функция — свой this
    regularMethod() {
        const inner = function() {
            console.log(this.name);  // undefined (в strict mode)
        };
        inner();
    },
    
    // Стрелочная — лексический this
    arrowMethod() {
        const inner = () => {
            console.log(this.name);  // 'Object'
        };
        inner();
    }
};

obj.regularMethod();
obj.arrowMethod();
```

---

### 2.2. Замыкания (Closures)

**Определение:**
> Замыкание — это способность функции запоминать и получать доступ к своей лексической области видимости, даже когда функция выполняется вне этой области.

**Примеры:**

```javascript
// 1. Базовое замыкание
function outer() {
    let count = 0;
    
    return function inner() {
        count++;
        return count;
    };
}

const counter = outer();
console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3

// 2. Замыкание в цикле (классическая ошибка)
for (var i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 3, 3, 3 (одна переменная i)
    }, 100);
}

// Решение с let
for (let i = 0; i < 3; i++) {
    setTimeout(() => {
        console.log(i);  // 0, 1, 2 (новая переменная для каждой итерации)
    }, 100);
}

// Решение с IIFE
for (var i = 0; i < 3; i++) {
    ((j) => {
        setTimeout(() => {
            console.log(j);  // 0, 1, 2
        }, 100);
    })(i);
}

// 3. Фабрика функций
function multiply(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiply(2);
const triple = multiply(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

**Практическое применение:**

```javascript
// 4. Инкапсуляция (модульный паттерн)
const Module = (function() {
    let privateVar = 0;
    
    function privateMethod() {
        return privateVar * 2;
    }
    
    return {
        increment() {
            privateVar++;
            return this;
        },
        getValue() {
            return privateVar;
        },
        getDoubled() {
            return privateMethod();
        }
    };
})();

Module.increment().increment();
console.log(Module.getValue());     // 2
console.log(Module.getDoubled());   // 4
```

---

### 2.3. Hoisting (Всплытие)

**Что всплывает:**
```javascript
// Объявления функций (полностью)
sayHello();  // 'Hello!'
function sayHello() {
    console.log('Hello!');
}

// Объявления var (только объявление, не значение)
console.log(x);  // undefined
var x = 5;
console.log(x);  // 5

// let и const (в TDZ — Temporal Dead Zone)
console.log(y);  // ReferenceError!
let y = 10;

console.log(z);  // ReferenceError!
const z = 15;
```

**Порядок всплытия:**
```javascript
// 1. Объявления функций
// 2. Объявления переменных (var)
// 3. let, const (не всплывают, TDZ)

function test() {
    console.log(a);  // undefined
    console.log(b);  // ReferenceError
    console.log(c);  // ReferenceError
    
    var a = 1;
    let b = 2;
    const c = 3;
    
    // Эквивалентно:
    // var a;
    // console.log(a);
    // console.log(b);
    // console.log(c);
    // a = 1;
    // let b = 2;
    // const c = 3;
}
```

---

## 3. Прототипы и классы

### 3.1. Прототипное наследование

```javascript
// Конструктор
function Person(name, age) {
    this.name = name;
    this.age = age;
}

// Метод в прототипе
Person.prototype.sayHello = function() {
    return `Hello, I'm ${this.name}`;
};

// Наследование
function Employee(name, age, position) {
    Person.call(this, name, age);  // Вызов родительского конструктора
    this.position = position;
}

Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;

Employee.prototype.work = function() {
    return `${this.name} is working as ${this.position}`;
};

const emp = new Employee('John', 30, 'Developer');
console.log(emp.sayHello());  // 'Hello, I'm John'
console.log(emp.work());      // 'John is working as Developer'
```

**Цепочка прототипов:**
```
emp → Employee.prototype → Person.prototype → Object.prototype → null
```

---

### 3.2. Классы ES6

```javascript
// Синтаксический сахар над прототипами
class Person {
    // Конструктор
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    // Метод экземпляра
    sayHello() {
        return `Hello, I'm ${this.name}`;
    }
    
    // Геттер
    get info() {
        return `${this.name}, ${this.age} years old`;
    }
    
    // Сеттер
    set age(value) {
        if (value < 0) throw new Error('Age cannot be negative');
        this._age = value;
    }
    
    // Статический метод
    static compareAges(p1, p2) {
        return p1.age - p2.age;
    }
}

class Employee extends Person {
    constructor(name, age, position) {
        super(name, age);  // Вызов родительского конструктора
        this.position = position;
    }
    
    // Переопределение метода
    sayHello() {
        return `${super.sayHello()} and I'm a ${this.position}`;
    }
    
    work() {
        return `${this.name} is working as ${this.position}`;
    }
}

const emp = new Employee('John', 30, 'Developer');
console.log(emp.sayHello());  // 'Hello, I'm John and I'm a Developer'
```

---

## 4. Паттерны управления состоянием

### 4.1. Redux Toolkit vs Context API vs Zustand

**Context API:**
```javascript
// ✅ Хорошо для редко меняющихся данных
const ThemeContext = React.createContext('light');

function App() {
    const [theme, setTheme] = React.useState('light');
    
    return (
        <ThemeContext.Provider value={theme}>
            <Toolbar />
        </ThemeContext.Provider>
    );
}

// ❌ Проблема: все потребители перерисовываются
function ThemedButton() {
    const theme = React.useContext(ThemeContext);
    return <button className={theme}>Click</button>;
}
```

**Redux Toolkit:**
```javascript
import { createSlice, configureStore } from '@reduxjs/toolkit';

const counterSlice = createSlice({
    name: 'counter',
    initialState: { value: 0 },
    reducers: {
        increment(state) { state.value += 1; },
        decrement(state) { state.value -= 1; },
        incrementByAmount(state, action) { state.value += action.payload; }
    }
});

const store = configureStore({
    reducer: { counter: counterSlice.reducer }
});

// В компоненте
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
    const count = useSelector(state => state.counter.value);
    const dispatch = useDispatch();
    
    return (
        <div>
            <span>{count}</span>
            <button onClick={() => dispatch(increment())}>+</button>
        </div>
    );
}
```

**Zustand:**
```javascript
import { create } from 'zustand';

const useStore = create((set) => ({
    count: 0,
    increment: () => set((state) => ({ count: state.count + 1 })),
    decrement: () => set((state) => ({ count: state.count - 1 })),
    
    // Асинхронные действия
    fetchCount: async () => {
        const response = await fetch('/api/count');
        const data = await response.json();
        set({ count: data.value });
    }
}));

// В компоненте — селективная подписка
function Counter() {
    const count = useStore((state) => state.count);
    const increment = useStore((state) => state.increment);
    
    return (
        <div>
            <span>{count}</span>
            <button onClick={increment}>+</button>
        </div>
    );
}

// Вне компонента
useStore.getState().increment();
```

**Сравнение:**

| Критерий | Context API | Redux Toolkit | Zustand |
|----------|-------------|---------------|---------|
| **Boilerplate** | Минимум | Много | Минимум |
| **Перформанс** | ❌ Все перерисовываются | ✅ Селекторы | ✅ Селективная подписка |
| **DevTools** | ❌ Нет | ✅ Да | ✅ Да |
| **Middleware** | ❌ Нет | ✅ Да | ✅ Да |
| **Размер** | 0 KB | ~15 KB | ~1 KB |
| **Когда** | Тема, локаль, auth | Enterprise | Универсально |

---

### 4.2. Atomic Design

```
atoms/
├── Button/
│   ├── Button.tsx
│   ├── Button.test.tsx
│   └── index.ts
├── Input/
│   ├── Input.tsx
│   └── index.ts
└── Label/
    └── Label.tsx

molecules/
├── SearchBar/
│   ├── SearchBar.tsx  (Input + Button)
│   └── index.ts
└── FormField/
    └── FormField.tsx  (Label + Input)

organisms/
├── Header/
│   ├── Header.tsx  (Logo + Navigation + SearchBar)
│   └── index.ts
└── ProductGrid/
    └── ProductGrid.tsx

templates/
├── PageTemplate/
│   └── PageTemplate.tsx

pages/
├── HomePage/
│   └── HomePage.tsx
└── ProductPage/
    └── ProductPage.tsx
```

---

## 🔗 Связанные темы

- [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding]] — Live-coding задачи
- [[03_React_Frontend/01_React_Architecture_FSD]] — FSD архитектура
- [[01_Python_Backend/01_Python_Core]] — Сравнение с Python GIL

---

*Файл обновлён: 17 марта 2026*
