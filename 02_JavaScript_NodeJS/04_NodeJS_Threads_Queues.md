---
created: 2026-03-17
updated: 2026-03-17
category: JavaScript
type: topic
priority: medium
status: completed
tags: [nodejs, queues, bullmq, microservices, event-driven, saga]
difficulty: advanced
estimated_hours: 10
---

# Node.js: Очереди, Микросервисы и Распределённые системы

> **Назначение:** Продвинутая архитектура Node.js: очереди сообщений, микросервисы, Event-Driven архитектура. Включает паттерны распределённых систем и лучшие практики.

---

## 1. Очереди сообщений (Message Queues)

### 1.1. Зачем нужны очереди

**Проблемы без очередей:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │───>│   Server    │───>│  Database   │
│             │    │             │    │             │
│ 1000 req/s  │    │ Обработка   │    │ Запись      │
│             │    │ 200 req/s   │    │ 500 req/s   │
└─────────────┘    └─────────────┘    └─────────────┘
                          │
                    ❌ Перегрузка!
                    ❌ Потеря данных
                    ❌ Таймауты
```

**Решение с очередью:**
```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Client    │───>│   Queue     │<───│   Worker    │
│             │    │             │    │             │
│ 1000 req/s  │    │ Буфер       │    │ 200 req/s   │
│             │    │ (сглаживание)│   │ (стабильно) │
└─────────────┘    └─────────────┘    └─────────────┘
```

---

### 1.2. BullMQ: Продвинутые паттерны

**Flows (цепочки задач):**
```javascript
const { FlowProducer } = require('bullmq');

const flow = new FlowProducer({
    connection: { host: 'localhost', port: 6379 }
});

async function createFlow() {
    await flow.add({
        name: 'order-processing',
        data: { orderId: '123' },
        queueName: 'orders',
        children: [
            {
                name: 'payment',
                data: { amount: 100 },
                queueName: 'payments'
            },
            {
                name: 'inventory',
                data: { productId: '456' },
                queueName: 'inventory',
                children: [
                    {
                        name: 'reserve',
                        data: { quantity: 2 },
                        queueName: 'reservations'
                    }
                ]
            }
        ]
    });
}
```

**Priorities (приоритеты):**
```javascript
const queue = new Queue('tasks');

// Высокий приоритет (1 = высший)
await queue.add('critical', { data }, { priority: 1 });
await queue.add('normal', { data }, { priority: 5 });
await queue.add('low', { data }, { priority: 10 });

// Обработка по приоритету
const worker = new Worker('tasks', handleJob, {
    connection: redisConnection
});
```

**Rate Limiting (ограничение скорости):**
```javascript
const queue = new Queue('api-calls');

// Максимум 10 задач в секунду
await queue.add('api-call', { url }, {
    limiter: {
        max: 10,
        duration: 1000
    }
});

// Глобальный лимит для очереди
const queue = new Queue('api-calls', {
    limiter: {
        max: 100,
        duration: 60000,  // 100 в минуту
        bounceBack: false  // Не возвращать задачу при превышении
    }
});
```

**Repeatable Jobs (повторяющиеся задачи):**
```javascript
const queue = new Queue('reports');

// Каждое утро в 9:00
await queue.add('daily-report', {}, {
    repeat: {
        pattern: '0 9 * * *'  // Cron表达式
    }
});

// Каждые 5 минут
await queue.add('health-check', {}, {
    repeat: {
        every: 300000  // 5 минут
    }
});
```

**Delayed Jobs (отложенные задачи):**
```javascript
// Через 1 час
await queue.add('reminder', { userId: 123 }, {
    delay: 3600000
});

// С экспоненциальным backoff при ошибке
await queue.add('flaky-task', { data }, {
    attempts: 5,
    backoff: {
        type: 'exponential',
        delay: 1000  // 1s, 2s, 4s, 8s, 16s
    }
});
```

---

### 1.3. Dead Letter Queue (DLQ)

```javascript
const { Queue, Worker } = require('bullmq');

const mainQueue = new Queue('main');
const dlq = new Queue('dlq');

const worker = new Worker('main', async (job) => {
    try {
        await processJob(job);
    } catch (error) {
        // Логирование ошибки
        console.error(`Job ${job.id} failed:`, error);
        
        // Перемещение в DLQ после всех попыток
        if (job.attemptsMade >= job.opts.attempts) {
            await dlq.add('failed-job', {
                originalJobId: job.id,
                error: error.message,
                data: job.data,
                timestamp: Date.now()
            });
        }
        
        throw error;  // Для backoff
    }
}, {
    connection: redisConnection,
    attempts: 3
});

// Обработчик DLQ
const dlqWorker = new Worker('dlq', async (job) => {
    // Уведомление администратора
    await notifyAdmin(job.data);
    
    // Или повторная попытка вручную
    // await mainQueue.add('retry', job.data);
});
```

---

### 1.4. RabbitMQ: Продвинутая маршрутизация

**Установка:**
```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

**Exchange Types:**
```javascript
const amqp = require('amqplib');

async function setupExchanges() {
    const connection = await amqp.connect('amqp://localhost');
    const channel = await connection.createChannel();
    
    // Direct Exchange (точная маршрутизация по routing key)
    await channel.assertExchange('direct-logs', 'direct', { durable: true });
    await channel.assertQueue('error-logs');
    await channel.bindQueue('error-logs', 'direct-logs', 'error');
    
    // Fanout Exchange (рассылка всем)
    await channel.assertExchange('notifications', 'fanout', { durable: true });
    
    // Topic Exchange (паттерны)
    await channel.assertExchange('app-events', 'topic', { durable: true });
    await channel.assertQueue('analytics');
    await channel.bindQueue('analytics', 'app-events', 'user.*');  // user.created, user.deleted
    await channel.bindQueue('analytics', 'app-events', '*.created');  // user.created, order.created
    
    // Headers Exchange (маршрутизация по заголовкам)
    await channel.assertExchange('headers-exchange', 'headers', { durable: true });
    await channel.assertQueue('priority-queue');
    await channel.bindQueue('priority-queue', 'headers-exchange', '', {
        priority: 10,
        'x-match': 'all'
    });
}
```

**Publisher Confirmations:**
```javascript
channel.publish(
    'app-events',
    'user.created',
    Buffer.from(JSON.stringify({ userId: 123 })),
    { persistent: true }  // Сохранить на диск
);

channel.waitForConfirms();  // Подтверждение доставки
```

---

### 1.5. Apache Kafka: Event Streaming

**Установка:**
```bash
docker-compose up -d zookeeper kafka
```

**Producer:**
```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
    clientId: 'my-app',
    brokers: ['localhost:9092']
});

const producer = kafka.producer();

async function sendMessage() {
    await producer.connect();
    
    await producer.send({
        topic: 'user-events',
        messages: [
            {
                key: 'user-123',  // Для партиционирования
                value: JSON.stringify({
                    event: 'user.created',
                    userId: 123,
                    timestamp: Date.now()
                }),
                headers: {
                    'correlation-id': 'abc-123'
                }
            }
        ]
    });
}
```

**Consumer:**
```javascript
const consumer = kafka.consumer({ groupId: 'analytics-group' });

async function startConsumer() {
    await consumer.connect();
    await consumer.subscribe({ topic: 'user-events', fromBeginning: true });
    
    await consumer.run({
        eachMessage: async ({ topic, partition, message }) => {
            console.log({
                topic,
                partition,
                offset: message.offset,
                value: message.value.toString()
            });
            
            // Обработка с гарантией (после успешной обработки — commit offset)
            await processEvent(message);
        }
    });
}
```

---

## 2. Event-Driven Архитектура

### 2.1. Event Emitter паттерны

**Domain Events:**
```javascript
const EventEmitter = require('events');

class DomainEvents extends EventEmitter {
    constructor() {
        super();
        this.setMaxListeners(0);  // Без лимита слушателей
    }
    
    // Гарантированная доставка
    async emitAsync(event, data) {
        const listeners = this.listeners(event);
        
        const promises = listeners.map(async (listener) => {
            try {
                await listener(data);
            } catch (error) {
                console.error(`Listener error for ${event}:`, error);
                // Не прерываем другие слушатели
            }
        });
        
        await Promise.all(promises);
    }
}

const domainEvents = new DomainEvents();

// Использование
domainEvents.on('order.created', async (order) => {
    await sendConfirmationEmail(order);
});

domainEvents.on('order.created', async (order) => {
    await updateInventory(order);
});

domainEvents.on('order.created', async (order) => {
    await logAnalytics(order);
});

// Генерация события
await domainEvents.emitAsync('order.created', orderData);
```

---

### 2.2. Event Sourcing

```javascript
class EventStore {
    constructor() {
        this.events = [];
        this.snapshots = new Map();
    }
    
    // Сохранение события
    async appendEvent(streamId, event) {
        const eventWithMetadata = {
            ...event,
            streamId,
            timestamp: Date.now(),
            version: this.getStreamVersion(streamId) + 1
        };
        
        this.events.push(eventWithMetadata);
        
        // Создаём snapshot каждые 100 событий
        if (eventWithMetadata.version % 100 === 0) {
            await this.createSnapshot(streamId);
        }
        
        // Публикация для подписчиков
        domainEvents.emit('event.appended', eventWithMetadata);
    }
    
    // Получение истории
    async getStreamHistory(streamId, fromVersion = 0) {
        const snapshot = this.snapshots.get(streamId);
        const baseVersion = snapshot ? snapshot.version : 0;
        const baseState = snapshot ? snapshot.state : {};
        
        const events = this.events.filter(
            e => e.streamId === streamId && e.version > Math.max(fromVersion, baseVersion)
        );
        
        // Применение событий к состоянию
        return events.reduce((state, event) => {
            return this.applyEvent(state, event);
        }, baseState);
    }
    
    applyEvent(state, event) {
        switch (event.type) {
            case 'order.created':
                return { ...state, status: 'created', items: event.items };
            case 'order.paid':
                return { ...state, status: 'paid', paymentId: event.paymentId };
            case 'order.shipped':
                return { ...state, status: 'shipped', trackingId: event.trackingId };
            default:
                return state;
        }
    }
}
```

---

## 3. Микросервисная архитектура

### 3.1. Монолит vs Микросервисы

**Монолит:**
```
┌─────────────────────────────────────────┐
│           Monolithic Application        │
├─────────────────────────────────────────┤
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │  Auth   │  │ Orders  │  │ Payments│ │
│  │ Module  │  │ Module  │  │ Module  │ │
│  └─────────┘  └─────────┘  └─────────┘ │
│                                         │
│         Единая база данных              │
└─────────────────────────────────────────┘

Плюсы:
✅ Простая разработка (один репозиторий)
✅ Легко тестировать (in-memory DB)
✅ Простой деплой (один сервер)

Минусы:
❌ Единая точка отказа
❌ Сложно масштабировать
❌ Технологическая锁定
```

**Микросервисы:**
```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│   Auth      │   │   Orders    │   │  Payments   │
│   Service   │   │   Service   │   │   Service   │
│  (Node.js)  │   │  (Python)   │   │    (Go)     │
└──────┬──────┘   └──────┬──────┘   └──────┬──────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
              ┌──────────▼──────────┐
              │   API Gateway       │
              │   (Kong, Traefik)   │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │   Load Balancer     │
              └──────────┬──────────┘
```

---

### 3.2. Межсервисная коммуникация

**Синхронная (HTTP/gRPC):**
```javascript
// API Gateway → Order Service
const orderService = {
    async getOrder(orderId) {
        const response = await fetch(`http://order-service:3000/orders/${orderId}`);
        return response.json();
    }
};

// gRPC (быстрее, типизированный)
const grpc = require('@grpc/grpc-js');

const client = new grpc.Client(
    'order-service:50051',
    grpc.credentials.createInsecure()
);

client.getOrder({ orderId: '123' }, (err, response) => {
    if (err) console.error(err);
    else console.log(response);
});
```

**Асинхронная (Message Queue):**
```javascript
// Order Service → Payment Service (через RabbitMQ)
const amqp = require('amqplib');

async function requestPayment(orderId, amount) {
    const connection = await amqp.connect('amqp://rabbitmq');
    const channel = await connection.createChannel();
    
    const correlationId = generateUUID();
    
    // Ответная очередь
    const replyQueue = await channel.assertQueue('', { exclusive: true });
    
    return new Promise((resolve, reject) => {
        channel.consume(replyQueue.queue, (msg) => {
            if (msg.properties.correlationId === correlationId) {
                resolve(JSON.parse(msg.content.toString()));
            }
        });
        
        // Запрос
        channel.sendToQueue('payment-requests', Buffer.from(JSON.stringify({
            orderId,
            amount
        })), {
            correlationId,
            replyTo: replyQueue.queue
        });
    });
}
```

---

### 3.3. Saga Pattern (распределённые транзакции)

```javascript
class OrderSaga {
    constructor() {
        this.steps = [
            { action: this.reserveInventory, compensate: this.releaseInventory },
            { action: this.processPayment, compensate: this.refundPayment },
            { action: this.createOrder, compensate: this.cancelOrder },
            { action: this.sendConfirmation, compensate: this.sendCancellation }
        ];
    }
    
    async execute(orderData) {
        const results = [];
        
        try {
            for (const step of this.steps) {
                const result = await step.action.call(this, orderData);
                results.push(result);
            }
            
            return { success: true, orderId: results[2].orderId };
            
        } catch (error) {
            // Компенсация (откат назад)
            for (let i = results.length - 1; i >= 0; i--) {
                try {
                    await this.steps[i].compensate.call(this, results[i]);
                } catch (compError) {
                    console.error('Compensation failed:', compError);
                    // Логирование для ручной обработки
                }
            }
            
            throw error;
        }
    }
    
    async reserveInventory(data) {
        // Резервирование товаров
        return { reserved: true };
    }
    
    async releaseInventory(data) {
        // Освобождение резерва
    }
    
    async processPayment(data) {
        // Обработка платежа
        return { paymentId: 'pay_123' };
    }
    
    async refundPayment(data) {
        // Возврат платежа
    }
    
    async createOrder(data) {
        // Создание заказа
        return { orderId: 'ord_123' };
    }
    
    async cancelOrder(data) {
        // Отмена заказа
    }
    
    async sendConfirmation(data) {
        // Отправка подтверждения
    }
    
    async sendCancellation(data) {
        // Отправка уведомления об отмене
    }
}
```

---

### 3.4. Circuit Breaker Pattern

```javascript
class CircuitBreaker {
    constructor(options = {}) {
        this.failureThreshold = options.failureThreshold || 5;
        this.resetTimeout = options.resetTimeout || 60000;
        
        this.state = 'CLOSED';  // CLOSED, OPEN, HALF_OPEN
        this.failures = 0;
        this.lastFailureTime = null;
    }
    
    async execute(fn) {
        if (this.state === 'OPEN') {
            if (Date.now() - this.lastFailureTime > this.resetTimeout) {
                this.state = 'HALF_OPEN';
                console.log('Circuit breaker: HALF_OPEN');
            } else {
                throw new Error('Circuit breaker is OPEN');
            }
        }
        
        try {
            const result = await fn();
            
            if (this.state === 'HALF_OPEN') {
                this.state = 'CLOSED';
                this.failures = 0;
                console.log('Circuit breaker: CLOSED');
            }
            
            return result;
            
        } catch (error) {
            this.failures++;
            this.lastFailureTime = Date.now();
            
            if (this.failures >= this.failureThreshold) {
                this.state = 'OPEN';
                console.log('Circuit breaker: OPEN');
            }
            
            throw error;
        }
    }
}

// Использование
const paymentBreaker = new CircuitBreaker({
    failureThreshold: 3,
    resetTimeout: 30000
});

async function processPayment(paymentData) {
    return paymentBreaker.execute(async () => {
        const response = await fetch('http://payment-service/process', {
            method: 'POST',
            body: JSON.stringify(paymentData)
        });
        
        if (!response.ok) throw new Error('Payment failed');
        return response.json();
    });
}
```

---

## 4. Распределённые системы: Паттерны

### 4.1. CQRS (Command Query Responsibility Segregation)

```javascript
// Command (запись)
class CreateOrderCommand {
    constructor(userId, items) {
        this.userId = userId;
        this.items = items;
    }
}

class OrderCommandHandler {
    constructor(writeDb, eventBus) {
        this.writeDb = writeDb;
        this.eventBus = eventBus;
    }
    
    async handle(command) {
        const order = {
            id: generateId(),
            userId: command.userId,
            items: command.items,
            status: 'created',
            createdAt: new Date()
        };
        
        await this.writeDb.orders.insert(order);
        
        await this.eventBus.publish('order.created', order);
        
        return order.id;
    }
}

// Query (чтение)
class OrderQueryHandler {
    constructor(readDb) {
        this.readDb = readDb;  // Оптимизировано для чтения
    }
    
    async getOrderById(orderId) {
        return this.readDb.orders.findOne({ id: orderId });
    }
    
    async getUserOrders(userId, filters) {
        return this.readDb.orders.find({
            userId,
            ...filters
        });
    }
}

// Projection (синхронизация write → read)
class OrderProjection {
    constructor(eventBus, readDb) {
        eventBus.subscribe('order.created', this.onOrderCreated.bind(this));
        eventBus.subscribe('order.updated', this.onOrderUpdated.bind(this));
        this.readDb = readDb;
    }
    
    async onOrderCreated(event) {
        await this.readDb.orders.insert(event.data);
    }
    
    async onOrderUpdated(event) {
        await this.readDb.orders.update(
            { id: event.data.orderId },
            event.data.changes
        );
    }
}
```

---

### 4.2. API Gateway Pattern

```javascript
const express = require('express');
const httpProxy = require('http-proxy-middleware');

const app = express();

// Аутентификация
app.use(async (req, res, next) => {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({ error: 'Unauthorized' });
    }
    
    try {
        req.user = await validateToken(token);
        next();
    } catch (error) {
        res.status(401).json({ error: 'Invalid token' });
    }
});

// Rate limiting
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 15 * 60 * 1000,  // 15 минут
    max: 100  // 100 запросов
});

app.use(limiter);

// Маршрутизация к микросервисам
app.use('/api/users', httpProxy.createProxyMiddleware({
    target: 'http://user-service:3000',
    changeOrigin: true
}));

app.use('/api/orders', httpProxy.createProxyMiddleware({
    target: 'http://order-service:3000',
    changeOrigin: true
}));

app.use('/api/payments', httpProxy.createProxyMiddleware({
    target: 'http://payment-service:3000',
    changeOrigin: true
}));

// Агрегация данных
app.get('/api/dashboard', async (req, res) => {
    const [orders, notifications, profile] = await Promise.all([
        fetch('http://order-service:3000/recent'),
        fetch('http://notification-service:3000/unread'),
        fetch('http://user-service:3000/profile')
    ]);
    
    res.json({
        orders: await orders.json(),
        notifications: await notifications.json(),
        profile: await profile.json()
    });
});

app.listen(8000);
```

---

## 🔗 Связанные темы

- [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]] — Streams, WebSockets
- [[05_Infrastructure/01_Infrastructure_Networks_Docker]] — Docker, деплой микросервисов
- [[04_Databases/01_Databases_ORM_SQL]] — Базы данных для микросервисов

---

*Файл обновлён: 17 марта 2026*
