---
created: 2026-03-17
category: Node.js
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review:
tags:
  - nodejs
  - worker-threads
  - cpu-bound
  - "#flashcard/nodeJS"
related:
  - - 02_JavaScript_NodeJS/03_NodeJS_Streams_Network#2-Worker-Threads-Многопоточность
---

# Для чего нужны Worker Threads?

## Вопрос
Для чего нужны Worker Threads в Node.js? Когда использовать?
?
## Ответ

**Worker Threads** — модуль для запуска JavaScript-кода в параллельных системных потоках.

**Проблема:**
- Node.js однопоточный
- **CPU-bound задачи** блокируют Event Loop
- Пока идёт вычисление, сервер не обрабатывает другие запросы

**CPU-bound задачи:**
- Парсинг AST-деревьев
- Сложные математические вычисления
- Криптография
- Обработка изображений

**Решение:**
```javascript
const { Worker } = require('worker_threads');

const worker = new Worker('./worker.js');

worker.on('message', (result) => {
    console.log('Result:', result);
});

worker.postMessage({ data: 'heavy computation' });
```

**Особенности:**
- У воркера свой Event Loop и V8
- Могут делить память через `SharedArrayBuffer`
- Коммуникация через `postMessage`

**Когда использовать:**
- ✅ CPU-bound задачи
- ❌ I/O-bound задачи (для них асинхронность Node.js)

---

#flashcard #nodejs #worker-threads #cpu-bound
