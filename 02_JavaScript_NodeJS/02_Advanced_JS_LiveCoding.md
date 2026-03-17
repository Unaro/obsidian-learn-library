# Advanced JavaScript & Live-Coding

> **Назначение:** Продвинутые концепции JavaScript с практическими задачами. Включает live-coding паттерны, глубокое копирование, Promise и работу с памятью.

---

## 1. Live-Coding: Функции высшего порядка

### 1.1. Debounce (Устранение дребезга)

**Задача:** Отложить вызов функции до тех пор, пока не пройдёт заданное время с момента последнего вызова.

**Применение:** Поиск с автодополнением, валидация форм, сохранение черновиков.

```javascript
/**
 * @param {Function} fn - Функция для вызова
 * @param {number} delay - Задержка в мс
 * @returns {Function} - Debounced функция
 */
function debounce(fn, delay) {
    let timeoutId = null;
    
    return function(...args) {
        // Сохраняем контекст this
        const context = this;
        
        // Очищаем предыдущий таймер
        clearTimeout(timeoutId);
        
        // Устанавливаем новый таймер
        timeoutId = setTimeout(() => {
            fn.apply(context, args);
        }, delay);
    };
}

// Использование
const searchInput = document.querySelector('#search');
const handleSearch = (e) => {
    console.log('Searching for:', e.target.value);
    // API запрос...
};

searchInput.addEventListener('input', debounce(handleSearch, 300));
```

**Тестирование:**
```javascript
// Проверка работы
let callCount = 0;
const fn = () => callCount++;

const debounced = debounce(fn, 100);

// Быстрые вызовы
debounced();
debounced();
debounced();

setTimeout(() => {
    console.log(callCount);  // 1 (только один вызов)
}, 150);
```

**Debounce с немедленным вызовом (leading edge):**
```javascript
function debounce(fn, delay, immediate = false) {
    let timeoutId = null;
    
    return function(...args) {
        const context = this;
        const callNow = immediate && !timeoutId;
        
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            timeoutId = null;
            if (!immediate) {
                fn.apply(context, args);
            }
        }, delay);
        
        if (callNow) {
            fn.apply(context, args);
        }
    };
}
```

---

### 1.2. Throttle (Троттлинг)

**Задача:** Гарантировать, что функция вызывается не чаще одного раза в заданный интервал.

**Применение:** Скролл, ресайз окна, drag-and-drop.

```javascript
/**
 * @param {Function} fn - Функция для вызова
 * @param {number} delay - Минимальный интервал между вызовами
 * @returns {Function} - Throttled функция
 */
function throttle(fn, delay) {
    let shouldWait = false;
    let waitingArgs = null;
    
    return function(...args) {
        const context = this;
        
        if (shouldWait) {
            // Если ждём — сохраняем аргументы
            waitingArgs = args;
            return;
        }
        
        // Вызываем функцию
        fn.apply(context, args);
        shouldWait = true;
        
        // После задержки — разрешаем следующий вызов
        setTimeout(() => {
            shouldWait = false;
            
            // Если были сохранённые аргументы — вызываем снова
            if (waitingArgs) {
                const argsToCall = waitingArgs;
                waitingArgs = null;
                fn.apply(context, argsToCall);
            }
        }, delay);
    };
}

// Использование
const handleScroll = () => {
    console.log('Scroll position:', window.scrollY);
};

window.addEventListener('scroll', throttle(handleScroll, 100));
```

**Throttle с setTimeout:**
```javascript
// Более простая версия
function throttle(fn, delay) {
    let lastCall = 0;
    
    return function(...args) {
        const now = Date.now();
        const context = this;
        
        if (now - lastCall >= delay) {
            lastCall = now;
            fn.apply(context, args);
        }
    };
}
```

---

### 1.3. Currying (Каррирование)

**Задача:** Преобразовать функцию с множеством аргументов в цепочку функций с одним аргументом.

```javascript
/**
 * @param {Function} fn - Функция для каррирования
 * @returns {Function} - Каррированная функция
 */
function curry(fn) {
    return function curried(...args) {
        // Если аргументов достаточно — вызываем функцию
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        
        // Иначе возвращаем функцию для следующего аргумента
        return function(...nextArgs) {
            return curried.apply(this, args.concat(nextArgs));
        };
    };
}

// Использование
function sum(a, b, c) {
    return a + b + c;
}

const curriedSum = curry(sum);

console.log(curriedSum(1, 2, 3));     // 6
console.log(curriedSum(1)(2, 3));     // 6
console.log(curriedSum(1)(2)(3));     // 6
console.log(curriedSum(1, 2)(3));     // 6

// Частичное применение
const add5 = curriedSum(5);
const add5And10 = add5(10);
console.log(add5And10(3));  // 18
```

**Каррирование с фиксацией аргументов:**
```javascript
function partial(fn, ...fixedArgs) {
    return function(...args) {
        return fn.apply(this, fixedArgs.concat(args));
    };
}

function greet(greeting, name, punctuation) {
    return `${greeting}, ${name}${punctuation}`;
}

const hello = partial(greet, 'Hello');
const helloJohn = hello('John');

console.log(helloJohn('!'));  // 'Hello, John!'
```

---

### 1.4. Compose и Pipe

**Compose (справа налево):**
```javascript
function compose(...functions) {
    return function(initialValue) {
        return functions.reduceRight((acc, fn) => fn(acc), initialValue);
    };
}

// Использование
const double = x => x * 2;
const increment = x => x + 1;
const square = x => x * x;

const transform = compose(square, increment, double);
console.log(transform(3));  // ((3 * 2) + 1)² = 49
```

**Pipe (слева направо):**
```javascript
function pipe(...functions) {
    return function(initialValue) {
        return functions.reduce((acc, fn) => fn(acc), initialValue);
    };
}

const transform = pipe(double, increment, square);
console.log(transform(3));  // ((3 * 2) + 1)² = 49
```

---

## 2. Глубокое копирование объектов

### 2.1. Поверхностное копирование (Shallow Copy)

```javascript
const original = {
    name: 'Test',
    nested: { value: 1 },
    array: [1, 2, 3]
};

// Spread оператор
const copy1 = { ...original };

// Object.assign
const copy2 = Object.assign({}, original);

// Изменение вложенного объекта влияет на оригинал
copy1.nested.value = 999;
console.log(original.nested.value);  // 999 (изменился!)
```

---

### 2.2. Глубокое копирование (Deep Copy)

**Способ 1: JSON (ограниченный):**
```javascript
function deepCopyJSON(obj) {
    return JSON.parse(JSON.stringify(obj));
}

const original = { a: 1, b: { c: 2 } };
const copy = deepCopyJSON(original);

copy.b.c = 999;
console.log(original.b.c);  // 2 (не изменился)

// ❌ Проблемы:
// - Теряются функции
// - Теряются undefined
// - Date превращается в строку
// - Не работает с циклическими ссылками
// - Symbol теряется
```

**Способ 2: structuredClone (современный):**
```javascript
const original = {
    a: 1,
    b: { c: 2 },
    date: new Date(),
    map: new Map([[1, 2]]),
    set: new Set([1, 2, 3])
};

const copy = structuredClone(original);

// ✅ Работает с Date, Map, Set, Array
// ✅ Работает с циклическими ссылками
// ❌ Не работает с функциями, Symbol

copy.map.set(3, 4);
console.log(original.map.has(3));  // false
```

**Способ 3: Рекурсивное копирование (полифил):**
```javascript
function deepCopy(value, hash = new WeakMap()) {
    // Примитивы
    if (value === null || typeof value !== 'object') {
        return value;
    }
    
    // Date
    if (value instanceof Date) {
        return new Date(value.getTime());
    }
    
    // Array
    if (Array.isArray(value)) {
        return value.map(item => deepCopy(item, hash));
    }
    
    // Циклические ссылки
    if (hash.has(value)) {
        return hash.get(value);
    }
    
    // Object
    const copy = Object.create(Object.getPrototypeOf(value));
    hash.set(value, copy);
    
    for (const key in value) {
        if (Object.hasOwnProperty.call(value, key)) {
            copy[key] = deepCopy(value[key], hash);
        }
    }
    
    return copy;
}

// Тест с циклической ссылкой
const obj = { a: 1 };
obj.self = obj;

const copy = deepCopy(obj);
console.log(copy.self === copy);  // true
console.log(copy.self === obj);   // false
```

---

## 3. Продвинутая асинхронность: Promise

### 3.1. Методы Promise

**Promise.all — ждём все (падает при первой ошибке):**
```javascript
const promise1 = Promise.resolve(1);
const promise2 = Promise.resolve(2);
const promise3 = new Promise((resolve, reject) => {
    setTimeout(resolve, 1000, 3);
});

Promise.all([promise1, promise2, promise3])
    .then(values => {
        console.log(values);  // [1, 2, 3]
    })
    .catch(error => {
        console.error('Любая ошибка:', error);
    });

// Если один упадёт:
Promise.all([
    Promise.resolve(1),
    Promise.reject(new Error('Failed')),
    Promise.resolve(3)
])
.catch(error => {
    console.error(error.message);  // 'Failed'
});
```

**Promise.allSettled — ждём все (независимо от результата):**
```javascript
Promise.allSettled([
    Promise.resolve(1),
    Promise.reject(new Error('Failed')),
    Promise.resolve(3)
])
.then(results => {
    console.log(results);
    // [
    //   { status: 'fulfilled', value: 1 },
    //   { status: 'rejected', reason: Error: Failed },
    //   { status: 'fulfilled', value: 3 }
    // ]
    
    const success = results
        .filter(r => r.status === 'fulfilled')
        .map(r => r.value);
    
    const errors = results
        .filter(r => r.status === 'rejected')
        .map(r => r.reason);
});
```

**Promise.race — первый завершённый (любой):**
```javascript
Promise.race([
    new Promise(resolve => setTimeout(resolve, 500, 'slow')),
    new Promise(resolve => setTimeout(resolve, 100, 'fast')),
    Promise.reject(new Error('error'))
])
.then(result => {
    console.log(result);  // 'fast'
})
.catch(error => {
    // Если ошибка будет быстрее
});

// Таймаут для запроса
function withTimeout(promise, ms) {
    const timeout = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('Timeout')), ms);
    });
    
    return Promise.race([promise, timeout]);
}

fetch('/api/data')
    .then(withTimeout(5000))
    .catch(console.error);
```

**Promise.any — первый успешный:**
```javascript
Promise.any([
    Promise.reject(new Error('Error 1')),
    Promise.reject(new Error('Error 2')),
    Promise.resolve('Success!')
])
.then(result => {
    console.log(result);  // 'Success!'
})
.catch(error => {
    // AggregateError, если все упали
    console.error(error.errors);  // [Error1, Error2]
});
```

---

### 3.2. Реализация Promise.all с нуля

```javascript
function promiseAll(promises) {
    return new Promise((resolve, reject) => {
        const results = [];
        let completed = 0;
        
        if (promises.length === 0) {
            resolve([]);
            return;
        }
        
        promises.forEach((promise, index) => {
            Promise.resolve(promise)
                .then(value => {
                    results[index] = value;
                    completed++;
                    
                    if (completed === promises.length) {
                        resolve(results);
                    }
                })
                .catch(reject);
        });
    });
}

// Тест
promiseAll([
    Promise.resolve(1),
    Promise.resolve(2),
    new Promise(resolve => setTimeout(resolve, 100, 3))
])
.then(console.log);  // [1, 2, 3]
```

---

### 3.3. Async/Await паттерны

**Параллельное выполнение:**
```javascript
// ❌ Последовательно (медленно)
async function getData() {
    const users = await fetch('/api/users');
    const posts = await fetch('/api/posts');
    const comments = await fetch('/api/comments');
    return { users, posts, comments };
}

// ✅ Параллельно (быстро)
async function getData() {
    const [users, posts, comments] = await Promise.all([
        fetch('/api/users'),
        fetch('/api/posts'),
        fetch('/api/comments')
    ]);
    return { users, posts, comments };
}
```

**Обработка ошибок:**
```javascript
// try-catch для всех
async function fetchAll() {
    try {
        const [users, posts] = await Promise.all([
            fetch('/api/users'),
            fetch('/api/posts')
        ]);
        return { users, posts };
    } catch (error) {
        console.error('Любая ошибка:', error);
        throw error;
    }
}

// Индивидуальная обработка
async function fetchAllSafe() {
    const results = await Promise.allSettled([
        fetch('/api/users'),
        fetch('/api/posts')
    ]);
    
    return {
        users: results[0].status === 'fulfilled' ? results[0].value : null,
        posts: results[1].status === 'fulfilled' ? results[1].value : null,
        errors: results
            .filter(r => r.status === 'rejected')
            .map(r => r.reason)
    };
}
```

---

## 4. События в браузере

### 4.1. Три фазы события

```
1. Capturing (Погружение): Window → Document → HTML → Body → ... → Element
2. Target: Элемент-цель
3. Bubbling (Всплытие): Element → ... → Body → HTML → Document → Window
```

**Пример:**
```html
<div id="parent">
    <button id="child">Click</button>
</div>

<script>
const parent = document.getElementById('parent');
const child = document.getElementById('child');

// Capturing фаза (третий аргумент true)
parent.addEventListener('click', () => {
    console.log('Parent capturing');
}, true);

child.addEventListener('click', () => {
    console.log('Child capturing');
}, true);

// Bubbling фаза (по умолчанию)
child.addEventListener('click', () => {
    console.log('Child bubbling');
});

parent.addEventListener('click', () => {
    console.log('Parent bubbling');
});

// Порядок вывода при клике на кнопку:
// 1. Parent capturing
// 2. Child capturing
// 3. Child bubbling
// 4. Parent bubbling
</script>
```

---

### 4.2. event.target vs event.currentTarget

```javascript
// HTML: <ul id="list"><li><span>Text</span></li></ul>

document.getElementById('list').addEventListener('click', (e) => {
    console.log('target:', e.target);        // span (на что кликнули)
    console.log('currentTarget:', e.currentTarget);  // ul (где обработчик)
});
```

---

### 4.3. Делегирование событий

```javascript
// ❌ Плохо: 1000 обработчиков
document.querySelectorAll('li').forEach(li => {
    li.addEventListener('click', () => {
        console.log('Clicked:', li.textContent);
    });
});

// ✅ Хорошо: 1 обработчик на родителе
document.getElementById('list').addEventListener('click', (e) => {
    const li = e.target.closest('li');
    
    if (!li || !document.getElementById('list').contains(li)) {
        return;
    }
    
    console.log('Clicked:', li.textContent);
});

// Динамически добавленные элементы тоже работают
const newLi = document.createElement('li');
newLi.textContent = 'New item';
document.getElementById('list').appendChild(newLi);
```

---

## 5. Управление памятью

### 5.1. Garbage Collector (Mark-and-Sweep)

**Алгоритм:**
```
1. GC начинает от корней (Window, global)
2. Помечает все достижимые объекты
3. Все непомеченные объекты удаляются
```

**Утечки памяти:**
```javascript
// 1. Глобальные переменные
function leak() {
    leakedData = new Array(1000000);  // Глобальная!
}

// 2. Забытые таймеры
const interval = setInterval(() => {
    // Выполняется вечно
}, 1000);
// Нужно: clearInterval(interval);

// 3. Забытые event listeners
element.addEventListener('click', handler);
// Нужно: element.removeEventListener('click', handler);

// 4. Замыкания
function createLeak() {
    const largeData = new Array(1000000);
    
    return function() {
        console.log(largeData.length);  // largeData не удалится
    };
}
```

---

### 5.2. WeakMap и WeakSet

```javascript
// Map предотвращает сборку мусора
const map = new Map();
let obj = { data: 'value' };
map.set(obj, 'metadata');

obj = null;  // Объект НЕ удалится (ссылка в map)

// WeakMap позволяет сборку мусора
const weakMap = new WeakMap();
obj = { data: 'value' };
weakMap.set(obj, 'metadata');

obj = null;  // Объект будет удалён GC

// Применение: кэширование, приватные данные
const privateData = new WeakMap();

class MyClass {
    constructor(value) {
        privateData.set(this, { _value: value });
    }
    
    getValue() {
        return privateData.get(this)._value;
    }
}
```

---

### 5.3. Ограничение объектов

```javascript
const obj = { a: 1 };

// preventExtensions — запрет новых свойств
Object.preventExtensions(obj);
obj.b = 2;
console.log(obj.b);  // undefined (в strict mode: TypeError)

// seal — запрет добавления/удаления
Object.seal(obj);
obj.a = 999;  // OK
delete obj.a;  // TypeError

// freeze — полная заморозка
Object.freeze(obj);
obj.a = 111;  // TypeError (в strict mode)

// Проверка
Object.isExtensible(obj);  // false
Object.isSealed(obj);      // true/false
Object.isFrozen(obj);      // true/false
```

---

## 🔗 Связанные темы

- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Event Loop основы
- [[03_React_Frontend/02_React_Internals_Optimization]] — React оптимизация
- [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]] — Streams в Node.js

---

*Файл обновлён: 17 марта 2026*
