---
created: 2026-03-17
category: JavaScript
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review:
tags:
  - javascript
  - event-loop
  - async
  - "#flashcard/javascript"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#1-2-Event-Loop-в-браузере
---

# Как работает Event Loop?

## Вопрос
Как работает Event Loop в JavaScript? В чём разница между microtask и macrotask?
?
## Ответ

**Event Loop** — механизм, координирующий выполнение синхронного кода и асинхронных задач.

**Алгоритм:**
1. Выполняет синхронный код в **Call Stack**
2. Когда стек пуст, берёт задачи из **Microtask Queue** (Promise, queueMicrotask)
3. Затем берёт одну задачу из **Macrotask Queue** (setTimeout, DOM события)
4. Повторяет цикл

**Microtask Queue (высший приоритет):**
- Promise коллбэки (`.then`, `.catch`, `.finally`)
- `queueMicrotask()`
- Выполняются все перед переходом к macrotask

**Macrotask Queue (низший приоритет):**
- `setTimeout`, `setInterval`
- DOM события (клик, скролл)
- Сетевые запросы

**Пример:**
```javascript
console.log('1. Start');

setTimeout(() => console.log('2. Timeout'), 0);

Promise.resolve()
    .then(() => console.log('3. Promise'));

console.log('4. End');

// Вывод: 1, 4, 3, 2
```

---

#flashcard #javascript #event-loop #async
