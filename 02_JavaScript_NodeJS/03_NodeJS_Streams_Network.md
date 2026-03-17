---
created: 2026-03-17
updated: 2026-03-17
category: JavaScript
type: topic
priority: high
status: completed
tags: [nodejs, streams, worker-threads, websockets, backpressure]
difficulty: advanced
estimated_hours: 8
---

# Node.js: Streams, Worker Threads & Network

> **Назначение:** Продвинутые возможности Node.js: потоки данных, многопоточность, сетевые протоколы. Включает практические примеры и паттерны архитектуры.

---

## 1. Streams (Потоки данных)

### 1.1. Проблема работы с большими файлами

```javascript
// ❌ ПЛОХО: Чтение всего файла в память
const fs = require('fs');

function processFileBad() {
    const data = fs.readFileSync('large-file.csv', 'utf8');
    // Файл 2 ГБ = 2 ГБ в памяти!
    // V8 Heap Limit ~1.4 ГБ → Out of Memory
    
    const lines = data.split('\n');
    // Обработка...
}

// ❌ Асинхронно, но всё равно в память
fs.readFile('large-file.csv', 'utf8', (err, data) => {
    if (err) throw err;
    // Всё ещё 2 ГБ в памяти
});
```

**Проблема:**
- Лимит памяти V8: ~1.4 ГБ (по умолчанию)
- Файл 2+ ГБ → **Out of Memory**
- Блокировка Event Loop при синхронном чтении

---

### 1.2. Решение: Streams

```javascript
// ✅ ХОРОШО: Чтение по частям (chunks)
const fs = require('fs');

function processFileStream() {
    const readStream = fs.createReadStream('large-file.csv', {
        highWaterMark: 64 * 1024,  // 64 KB чанки
        encoding: 'utf8',
        start: 0,
        end: 1000000  // Можно читать часть файла
    });
    
    readStream.on('data', (chunk) => {
        console.log('Received chunk:', chunk.length, 'bytes');
        // Обрабатываем чанк сразу
    });
    
    readStream.on('end', () => {
        console.log('File fully read');
    });
    
    readStream.on('error', (err) => {
        console.error('Stream error:', err);
    });
}

// Память: ~64 KB независимо от размера файла!
```

---

### 1.3. Типы потоков

**Readable (для чтения):**
```javascript
const { Readable } = require('stream');

// Создание Readable потока
const readable = new Readable({
    read(size) {
        if (this.currentChunk >= 10) {
            this.push(null);  // Конец потока
            return;
        }
        
        this.push(`Chunk ${this.currentChunk++}\n`);
    }
});

readable.currentChunk = 0;

readable.on('data', (chunk) => {
    console.log(chunk.toString());
});
```

**Writable (для записи):**
```javascript
const { Writable } = require('stream');
const fs = require('fs');

const writeStream = fs.createWriteStream('output.txt', {
    flags: 'a',  // append
    encoding: 'utf8'
});

writeStream.write('First line\n');
writeStream.write('Second line\n');

writeStream.end();  // Завершение

writeStream.on('finish', () => {
    console.log('Writing completed');
});

writeStream.on('error', (err) => {
    console.error('Write error:', err);
});
```

**Duplex (двунаправленные):**
```javascript
const { Duplex } = require('stream');
const net = require('net');

// TCP сокет — Duplex поток
const socket = net.connect({ port: 8080 });

socket.on('data', (data) => {
    // Чтение
    console.log('Received:', data.toString());
});

socket.write('Hello server');  // Запись
```

**Transform (модификация данных):**
```javascript
const { Transform } = require('stream');
const zlib = require('zlib');

// Сжатие данных
const gzip = zlib.createGzip();

// Парсинг CSV на лету
class CSVParseStream extends Transform {
    constructor(options) {
        super({ ...options, objectMode: true });
        this.buffer = '';
    }
    
    _transform(chunk, encoding, callback) {
        this.buffer += chunk.toString();
        
        const lines = this.buffer.split('\n');
        this.buffer = lines.pop();  // Последний чанк может быть неполным
        
        for (const line of lines) {
            const fields = line.split(',');
            this.push({ fields });  // objectMode
        }
        
        callback();
    }
    
    _flush(callback) {
        if (this.buffer) {
            const fields = this.buffer.split(',');
            this.push({ fields });
        }
        callback();
    }
}
```

---

### 1.4. Piping и Backpressure

**Pipe (автоматическое управление):**
```javascript
const fs = require('fs');
const zlib = require('zlib');

// Чтение → Сжатие → Запись
const readStream = fs.createReadStream('input.txt');
const gzipStream = zlib.createGzip();
const writeStream = fs.createWriteStream('input.txt.gz');

readStream
    .pipe(gzipStream)
    .pipe(writeStream)
    .on('finish', () => {
        console.log('File compressed');
    });
```

**Backpressure (обратное давление):**
```javascript
// ❌ Без обработки backpressure
source.on('data', (chunk) => {
    dest.write(chunk);  // Может переполнить!
});

// ✅ С обработкой backpressure
source.on('data', (chunk) => {
    const canContinue = dest.write(chunk);
    
    if (!canContinue) {
        source.pause();  // Приостановить чтение
    }
});

dest.on('drain', () => {
    source.resume();  // Возобновить чтение
});

// ✅ Или просто pipe (автоматически)
source.pipe(dest);
```

**Как работает Backpressure:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Readable   │───>│  Writable   │───>│   Disk/Net  │
│   (Fast)    │    │   (Slow)    │     │             │
└─────────────┘    └─────────────┘    └─────────────┘
       │                  │
       │   .pause()       │
       │<─────────────────┤
       │                  │
       │   .resume()      │
       │─────────────────>│
       │   on('drain')    │
```

---

### 1.5. Практические примеры

**1. Сжатие файла:**
```javascript
const fs = require('fs');
const zlib = require('zlib');

fs.createReadStream('input.txt')
    .pipe(zlib.createGzip())
    .pipe(fs.createWriteStream('input.txt.gz'))
    .on('finish', () => console.log('Done'));
```

**2. Загрузка файла на сервер:**
```javascript
const express = require('express');
const busboy = require('busboy');
const app = express();

app.post('/upload', (req, res) => {
    const bb = busboy({ headers: req.headers });
    
    bb.on('file', (name, file, info) => {
        const saveTo = `uploads/${info.filename}`;
        file.pipe(fs.createWriteStream(saveTo));
    });
    
    bb.on('close', () => {
        res.send('Upload complete');
    });
    
    req.pipe(bb);
});
```

**3. Чтение большого JSON:**
```javascript
const JSONStream = require('JSONStream');
const es = require('event-stream');

fs.createReadStream('large-data.json')
    .pipe(JSONStream.parse('rows.*'))
    .pipe(es.mapSync((row) => {
        // Обработка каждой записи
        console.log(row);
    }));
```

---

## 2. Worker Threads (Многопоточность)

### 2.1. Проблема CPU-bound задач

```javascript
// ❌ Блокировка Event Loop
app.get('/heavy-calculation', (req, res) => {
    let result = 0;
    for (let i = 0; i < 1e9; i++) {
        result += Math.sqrt(i);
    }
    // Event Loop заблокирован на ~5 секунд!
    // Все остальные запросы ждут
    res.json({ result });
});
```

---

### 2.2. Решение с Worker Threads

```javascript
// main.js
const { Worker } = require('worker_threads');
const express = require('express');
const path = require('path');

const app = express();

app.get('/heavy-calculation', (req, res) => {
    const worker = new Worker(path.join(__dirname, 'worker.js'));
    
    worker.on('message', (result) => {
        res.json({ result });
    });
    
    worker.on('error', (err) => {
        res.status(500).json({ error: err.message });
    });
    
    worker.postMessage({ limit: 1e9 });
});

app.listen(3000);
```

```javascript
// worker.js
const { parentPort } = require('worker_threads');

parentPort.on('message', (data) => {
    let result = 0;
    
    for (let i = 0; i < data.limit; i++) {
        result += Math.sqrt(i);
    }
    
    // Отправка результата в главный поток
    parentPort.postMessage(result);
});
```

---

### 2.3. SharedArrayBuffer (общая память)

```javascript
// main.js
const { Worker, isMainThread, SharedArrayBuffer } = require('worker_threads');

if (isMainThread) {
    // Создаём буфер на 1000 integers (4 bytes каждый)
    const buffer = new SharedArrayBuffer(1000 * 4);
    const array = new Int32Array(buffer);
    
    // Инициализация
    for (let i = 0; i < 1000; i++) {
        array[i] = i;
    }
    
    const worker = new Worker(__filename, {
        workerData: { buffer }
    });
    
    worker.on('message', () => {
        console.log('Sum:', array[0]);  // Результат в общей памяти
    });
    
} else {
    const { workerData } = require('worker_threads');
    const array = new Int32Array(workerData.buffer);
    
    // Вычисление суммы
    let sum = 0;
    for (let i = 1; i < 1000; i++) {
        sum += array[i];
    }
    
    // Запись результата в общую память
    array[0] = sum;
    
    parentPort.postMessage('done');
}
```

---

### 2.4. Парсинг AST с Worker Threads

```javascript
// parser-worker.js
const { parentPort, workerData } = require('worker_threads');
const parser = require('@babel/parser');

parentPort.on('message', (code) => {
    try {
        // Парсинг AST (CPU-intensive)
        const ast = parser.parse(code, {
            sourceType: 'module',
            plugins: ['jsx', 'typescript']
        });
        
        parentPort.postMessage({ success: true, ast });
    } catch (error) {
        parentPort.postMessage({ success: false, error: error.message });
    }
});
```

```javascript
// main.js
const { Worker } = require('worker_threads');

function parseCode(code) {
    return new Promise((resolve, reject) => {
        const worker = new Worker('./parser-worker.js');
        
        worker.on('message', (result) => {
            if (result.success) {
                resolve(result.ast);
            } else {
                reject(new Error(result.error));
            }
        });
        
        worker.postMessage(code);
    });
}

// Использование
parseCode('const x = 1 + 2;')
    .then(ast => console.log('Parsed:', ast));
```

---

## 3. Real-Time протоколы связи

### 3.1. WebSockets

**Сервер (ws):**
```javascript
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
    console.log('Client connected');
    
    ws.on('message', (message) => {
        console.log('Received:', message.toString());
        
        // Рассылка всем подключенным
        wss.clients.forEach((client) => {
            if (client !== ws && client.readyState === WebSocket.OPEN) {
                client.send(message);
            }
        });
    });
    
    ws.on('close', () => {
        console.log('Client disconnected');
    });
    
    ws.on('error', (error) => {
        console.error('WebSocket error:', error);
    });
    
    // Heartbeat
    const interval = setInterval(() => {
        if (ws.readyState === WebSocket.OPEN) {
            ws.ping();
        } else {
            clearInterval(interval);
        }
    }, 30000);
});
```

**Клиент:**
```javascript
const ws = new WebSocket('ws://localhost:8080');

ws.on('open', () => {
    ws.send(JSON.stringify({ type: 'message', data: 'Hello' }));
});

ws.on('message', (data) => {
    console.log('Received:', data.toString());
});

ws.on('close', () => {
    console.log('Disconnected');
});
```

---

### 3.2. Server-Sent Events (SSE)

**Сервер (Express):**
```javascript
const express = require('express');
const app = express();

app.get('/events', (req, res) => {
    // SSE заголовки
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');
    res.setHeader('X-Accel-Buffering', 'no');  // Для nginx
    
    // Отправка событий
    const sendEvent = (data, event = 'message') => {
        res.write(`event: ${event}\n`);
        res.write(`data: ${JSON.stringify(data)}\n\n`);
    };
    
    sendEvent({ status: 'connected' });
    
    // Периодическая отправка
    const interval = setInterval(() => {
        sendEvent({ time: Date.now(), type: 'heartbeat' });
    }, 5000);
    
    // Очистка при отключении
    req.on('close', () => {
        clearInterval(interval);
    });
});

app.listen(3000);
```

**Клиент:**
```javascript
const eventSource = new EventSource('http://localhost:3000/events');

eventSource.onopen = () => {
    console.log('SSE connected');
};

eventSource.addEventListener('message', (e) => {
    console.log('Message:', JSON.parse(e.data));
});

eventSource.addEventListener('heartbeat', (e) => {
    console.log('Heartbeat:', JSON.parse(e.data));
});

eventSource.onerror = (e) => {
    console.error('SSE error:', e);
    eventSource.close();
};
```

---

### 3.3. Long Polling

**Сервер:**
```javascript
const express = require('express');
const app = express();

const pendingRequests = [];
const messages = [];

// Отправка сообщения
app.post('/message', (req, res) => {
    const { message } = req.body;
    messages.push(message);
    
    // Уведомление ожидающих клиентов
    while (pendingRequests.length > 0 && messages.length > 0) {
        const clientRes = pendingRequests.shift();
        const msg = messages.shift();
        clientRes.json({ message: msg });
    }
    
    res.json({ status: 'sent' });
});

// Получение сообщений (Long Polling)
app.get('/messages', (req, res) => {
    if (messages.length > 0) {
        // Есть сообщения — отправляем сразу
        res.json({ message: messages.shift() });
    } else {
        // Нет сообщений — ждём
        pendingRequests.push(res);
        
        // Таймаут 30 секунд
        setTimeout(() => {
            const index = pendingRequests.indexOf(res);
            if (index !== -1) {
                pendingRequests.splice(index, 1);
                res.json({ message: null });
            }
        }, 30000);
    }
});

app.listen(3000);
```

---

### 3.4. Сравнение протоколов

| Протокол | Направление | Производительность | Когда использовать |
|----------|-------------|-------------------|-------------------|
| **WebSockets** | Двунаправленное | ⭐⭐⭐⭐⭐ | Чаты, игры, collaboration |
| **SSE** | Сервер → Клиент | ⭐⭐⭐⭐ | Уведомления, котировки, статус |
| **Long Polling** | Двунаправленное | ⭐⭐⭐ | Fallback для старых браузеров |
| **HTTP Polling** | Клиент → Сервер | ⭐⭐ | Редкие обновления |

---

## 4. Очереди сообщений и фоновые задачи

### 4.1. BullMQ с Redis

**Установка:**
```bash
npm install bullmq ioredis
```

**Producer (создание задач):**
```javascript
const { Queue } = require('bullmq');

const videoQueue = new Queue('video-processing', {
    connection: {
        host: 'localhost',
        port: 6379
    }
});

// Добавление задачи
async function processVideo(videoId) {
    await videoQueue.add('transcode', {
        videoId,
        format: 'mp4',
        quality: '1080p'
    }, {
        attempts: 3,  // Попытки
        backoff: {
            type: 'exponential',
            delay: 1000
        },
        timeout: 300000  // 5 минут
    });
}

// Отложенная задача
await videoQueue.add('notify', { userId: 123 }, {
    delay: 60000  // Через 1 минуту
});

// Приоритет
await videoQueue.add('urgent', { data }, { priority: 1 });  // 1 = высший
await videoQueue.add('normal', { data }, { priority: 10 });
```

**Consumer (обработка задач):**
```javascript
const { Worker } = require('bullmq');

const worker = new Worker('video-processing', async (job) => {
    console.log(`Processing job ${job.id}:`, job.data);
    
    if (job.name === 'transcode') {
        // Транскодирование видео
        await transcodeVideo(job.data.videoId, job.data.quality);
        
        // Обновление прогресса
        await job.updateProgress(50);
        
        // Логирование
        await job.log('Transcoding completed');
    }
    
    return { success: true };
}, {
    connection: {
        host: 'localhost',
        port: 6379
    },
    concurrency: 5  // Параллельные задачи
});

worker.on('completed', (job) => {
    console.log(`Job ${job.id} completed`);
});

worker.on('failed', (job, err) => {
    console.error(`Job ${job.id} failed:`, err);
});

worker.on('error', (err) => {
    console.error('Worker error:', err);
});
```

---

### 4.2. Паттерн Producer-Consumer

```javascript
// API Server (Producer)
const express = require('express');
const { Queue } = require('bullmq');
const app = express();

const analysisQueue = new Queue('data-analysis');

app.post('/analyze', async (req, res) => {
    const { fileId, userId } = req.body;
    
    // Быстрое сохранение метаданных
    await db.save({ fileId, userId, status: 'pending' });
    
    // Добавление в очередь
    await analysisQueue.add('analyze-file', {
        fileId,
        userId,
        timestamp: Date.now()
    });
    
    // Мгновенный ответ
    res.status(202).json({
        status: 'accepted',
        jobId: fileId,
        message: 'Analysis in progress'
    });
});

app.listen(3000);
```

```javascript
// Worker (Consumer)
const { Worker } = require('bullmq');
const { WebSocketServer } = require('ws');

const wss = new WebSocketServer({ port: 8080 });
const clients = new Map();  // userId → WebSocket

const worker = new Worker('data-analysis', async (job) => {
    const { fileId, userId } = job.data;
    
    await job.updateProgress(10);
    
    // Тяжёлая обработка
    const result = await analyzeFile(fileId);
    
    await job.updateProgress(100);
    
    // Обновление БД
    await db.update(fileId, { status: 'completed', result });
    
    // Уведомление через WebSocket
    if (clients.has(userId)) {
        clients.get(userId).send(JSON.stringify({
            type: 'analysis-complete',
            fileId,
            result
        }));
    }
    
    return result;
});
```

---

### 4.3. Сравнение очередей

| Инструмент | Хранение | Производительность | Функции | Когда |
|-----------|----------|-------------------|---------|-------|
| **BullMQ** | Redis | ⭐⭐⭐⭐⭐ | Retries, delay, priority | Большинство случаев |
| **RabbitMQ** | In-memory + Disk | ⭐⭐⭐⭐ | Routing, topics, DLX | Микросервисы |
| **Kafka** | Disk | ⭐⭐⭐⭐⭐ | Stream processing, replay | Event sourcing, аналитика |
| **SQS** | AWS Cloud | ⭐⭐⭐⭐ | Managed, scalable | AWS инфраструктура |

---

## 5. Event Emitter

### 5.1. Базовое использование

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const emitter = new MyEmitter();

// Подписка
emitter.on('event', (arg) => {
    console.log('Listener 1:', arg);
});

emitter.once('event', (arg) => {
    console.log('Listener 2 (once):', arg);
});

// Генерация события
emitter.emit('event', 'Hello');
// Listener 1: Hello
// Listener 2 (once): Hello

emitter.emit('event', 'World');
// Listener 1: World
// (Listener 2 не вызван — сработал один раз)
```

---

### 5.2. Асинхронные обработчики

```javascript
emitter.on('async-event', async (data) => {
    await someAsyncOperation(data);
});

// Ожидание всех обработчиков
await emitter.emitAsync('async-event', data);
```

---

## 🔗 Связанные темы

- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop]] — Event Loop основы
- [[05_Infrastructure/01_Infrastructure_Networks_Docker]] — Docker деплой
- [[03_React_Frontend/01_React_Architecture_FSD]] — WebSocket в React

---

*Файл обновлён: 17 марта 2026*
