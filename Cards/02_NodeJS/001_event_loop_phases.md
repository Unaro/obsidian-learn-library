---
created: 2026-03-17
category: Node.js
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review:
tags:
  - nodejs
  - event-loop
  - phases
  - "#flashcard/nodeJS"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#1-3-Event-Loop-в-Node-js-libuv
---

# Фазы Event Loop в Node.js

## Вопрос
Какие фазы Event Loop в Node.js? В чём отличие от браузера?
?
## Ответ

**Фазы Event Loop в Node.js (libuv):**

1. **Timers** — `setTimeout()`, `setInterval()`
2. **Pending Callbacks** — отложенные I/O колбэки
3. **Idle, Prepare** — внутренние нужды Node.js
4. **Poll** — получение новых I/O событий (самая важная)
5. **Check** — `setImmediate()`
6. **Close Callbacks** — закрытие соединений

**Микрозадачи:**
- Выполняются **между каждой фазой**
- `process.nextTick()` — абсолютный приоритет
- Promise (`.then`, `.catch`) — после nextTick

**Отличие от браузера:**
| Браузер | Node.js |
|---------|---------|
| 2 очереди (micro, macro) | 6 фаз + microtask между фазами |
| Простой цикл | libuv Thread Pool |
| Нет `setImmediate` | Есть `setImmediate` в фазе Check |

**Пример:**
```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

// В callback I/O: сначала immediate
// В главном контексте: неопределено (50/50)
```

---

#flashcard #nodejs #event-loop #libuv
