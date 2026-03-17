---
created: 2026-03-17
category: Node.js
type: flashcard
difficulty: intermediate
frequency: low
status: new
next_review:
tags:
  - nodejs
  - process-nexttick
  - setimmediate
  - "#flashcard/nodeJS"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#1-3-Event-Loop-в-Node-js-libuv
---

# process.nextTick() vs setImmediate()

## Вопрос
В чём разница между `process.nextTick()` и `setImmediate()`?
?
## Ответ

**`process.nextTick()`:**
- Выполняется **до** следующей фазы Event Loop
- **Абсолютный приоритет** над любыми другими микрозадачами
- Выполняется сразу после текущей операции

```javascript
Promise.resolve().then(() => {
    console.log('Promise 1');
    process.nextTick(() => console.log('nextTick 2'));
});

process.nextTick(() => {
    console.log('nextTick 1');
});

// Вывод: nextTick 1, nextTick 2, Promise 1
```

**`setImmediate()`:**
- Выполняется в фазе **Check**
- После I/O событий фазы Poll

```javascript
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));

// В callback I/O: сначала immediate
```

**Когда что использовать:**
- `process.nextTick()` — когда нужно выполнить код немедленно, но не в текущем стеке
- `setImmediate()` — когда нужно выполнить код после завершения I/O

---

#flashcard #nodejs #process-nexttick #setimmediate
