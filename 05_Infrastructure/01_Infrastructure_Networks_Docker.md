# Инфраструктура, Сети и DevOps

> **Назначение:** Docker, контейнеризация, Git, HTTP/REST, JWT, архитектурные принципы. Включает практические примеры деплоя и лучшие практики.

---

## 1. Docker и контейнеризация

### 1.1. Архитектура Docker

```
┌─────────────────────────────────────────────────────────┐
│              Docker Architecture                        │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌─────────────────────────────────────────────────┐   │
│  │  Docker Daemon (dockerd)                        │   │
│  │  ┌───────────┐  ┌───────────┐  ┌───────────┐   │   │
│  │  │  Images   │  │Containers │  │ Networks  │   │   │
│  │  │           │  │           │  │           │   │   │
│  │  │  my-app   │  │ app-1     │  │ bridge    │   │   │
│  │  │  postgres │  │ db-1      │  │ host      │   │   │
│  │  │  redis    │  │ redis-1   │  │ none      │   │   │
│  │  └───────────┘  └───────────┘  └───────────┘   │   │
│  └─────────────────────────────────────────────────┘   │
│                         ↑                               │
│  ┌──────────────────────┴──────────────────────┐       │
│  │         Docker CLI (docker command)         │       │
│  └─────────────────────────────────────────────┘       │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

**Image vs Container:**
```
Image (Образ)              Container (Контейнер)
─────────────────          ─────────────────────
Статический шаблон         Запущенный экземпляр
Неизменяемый               Изменяемый (слой записи)
Как класс в ООП            Как объект класса
Хранится в registry        Запускается на хосте

docker build -t app        docker run -d --name app-1 app
docker pull postgres       docker exec -it app-1 bash
```

---

### 1.2. Dockerfile: Лучшие практики

**❌ ПЛОХОЙ Dockerfile:**
```dockerfile
FROM node:18

# Установка всех зависимостей системы
RUN apt-get update && apt-get install -y \
    vim \
    curl \
    git \
    build-essential

WORKDIR /app

# Копирование ВСЕХ файлов (включая node_modules)
COPY . .

# Установка зависимостей каждый раз при изменении кода
RUN npm install

# Запуск через npm (лишний процесс)
CMD npm run start
```

**✅ ХОРОШИЙ Dockerfile:**
```dockerfile
# Multi-stage build для минимального размера
FROM node:18-alpine AS builder

WORKDIR /app

# Копирование package файлов первыми (кэширование слоёв)
COPY package*.json ./
COPY pnpm-lock.yaml ./

# Установка зависимостей (кэшируется)
RUN npm ci --only=production

# Копирование исходного кода
COPY src/ ./src/
COPY tsconfig.json ./

# Сборка приложения
RUN npm run build

# Финальный образ
FROM node:18-alpine

# Создание non-root пользователя для безопасности
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Копирование только нужных файлов
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./

# Переключение на non-root пользователя
USER nodejs

# Переменные окружения
ENV NODE_ENV=production
ENV PORT=3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD node -e "require('http').get('http://localhost:3000/health', r => process.exit(r.statusCode === 200 ? 0 : 1))"

# Запуск приложения
CMD ["node", "dist/main.js"]
```

---

### 1.3. Docker Compose для разработки

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Основное приложение
  app:
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    volumes:
      - .:/app  # Hot reload
      - /app/node_modules  # Сохранение node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://user:pass@db:5432/myapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    command: npm run dev

  # PostgreSQL
  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    environment:
      - POSTGRES_USER=user
      - POSTGRES_PASSWORD=pass
      - POSTGRES_DB=myapp
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Redis для кэша
  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

  # pgAdmin для администрирования
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      - PGADMIN_DEFAULT_EMAIL=admin@example.com
      - PGADMIN_DEFAULT_PASSWORD=admin
    ports:
      - "5050:80"
    depends_on:
      - db

volumes:
  postgres_data:
  redis_data:
```

---

### 1.4. Docker сети

```yaml
# docker-compose.yml с кастомными сетями
version: '3.8'

services:
  frontend:
    build: ./frontend
    networks:
      - public  # Доступ извне
      - internal  # Доступ к backend

  backend:
    build: ./backend
    networks:
      - internal  # Доступ от frontend
      - database  # Доступ к БД
    environment:
      - DB_HOST=db  # Имя сервиса как хост

  db:
    image: postgres:15
    networks:
      - database  # Только backend имеет доступ

networks:
  public:
    driver: bridge
  internal:
    driver: bridge
    internal: true  # Нет доступа извне
  database:
    driver: bridge
    internal: true
```

---

## 2. Git: Продвинутые техники

### 2.1. Merge vs Rebase

**Git Merge:**
```
Before merge:
      A---B---C  (feature)
     /
D---E---F---G  (main)

Command: git merge feature

After merge:
      A---B---C
     /         \
D---E---F---G---M  (main)
                ↑
            Merge commit
```

**Git Rebase:**
```
Before rebase:
      A---B---C  (feature)
     /
D---E---F---G  (main)

Command: git rebase main

After rebase:
              A'--B'--C'  (feature)
             /
D---E---F---G  (main)

Command: git checkout main && git merge feature

Result:
D---E---F---G---A'--B'--C'  (main, feature)
```

**Когда что использовать:**
```
✅ Merge:
- Публичные ветки
- Сохранение истории слияния
- Feature flags

✅ Rebase:
- Локальные ветки перед merge
- Очистка истории коммитов
- Обновление ветки от main
```

**⚠️ Никогда не делайте rebase:**
```bash
# Если ветка уже запушена и используется другими
git push origin feature  # Ветка опубликована

# НЕЛЬЗЯ делать rebase после этого!
git rebase main  # ❌ Сломает историю другим разработчикам
```

---

### 2.2. Git Flow

```
main (production)
│
├───release/v1.0────────────────────► main
│     │
develop (integration)               │
│     │                             │
├───feature/auth───────────►│       │
│     │                     │       │
├───feature/cart────────────┼───────►│
│     │                     │       │
├───hotfix/bug-fix──────────┴───────►│
│                                   │
main: v1.0 released ◄───────────────┘
```

**Команды:**
```bash
# Новая фича
git checkout develop
git checkout -b feature/authentication

# Завершение фичи
git checkout develop
git merge --no-ff feature/authentication
git branch -d feature/authentication

# Подготовка релиза
git checkout -b release/v1.0
# Фикс багов, версионирование
git checkout main
git merge --no-ff release/v1.0
git tag -a v1.0 -m "Release v1.0"

# Hotfix для продакшена
git checkout -b hotfix/critical-bug main
# Исправление
git checkout main
git merge --no-ff hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix"
git checkout develop
git merge hotfix/critical-bug
```

---

### 2.3. Interactive Rebase

```bash
# Редактирование последних 3 коммитов
git rebase -i HEAD~3

# Откроется редактор:
pick abc123 Add feature
pick def456 Fix typo
pick ghi789 Update docs

# Можно изменить на:
reword abc123  # Изменить сообщение коммита
edit def456    # Остановиться для редактирования
squash ghi789  # Объединить с предыдущим
drop xyz999    # Удалить коммит
```

---

## 3. HTTP и REST API

### 3.1. HTTP методы и идемпотентность

```
┌─────────────────────────────────────────────────────────┐
│              HTTP Methods Idempotency                   │
├──────────┬───────────────┬──────────────────────────────┤
│  Метод   │  Идемпотентный│  Пример                     │
├──────────┼───────────────┼──────────────────────────────┤
│ GET      │ ✅ Да         │ Получить пользователя       │
│ HEAD     │ ✅ Да         │ Получить заголовки          │
│ PUT      │ ✅ Да         │ Заменить ресурс целиком     │
│ DELETE   │ ✅ Да         │ Удалить ресурс              │
│ OPTIONS  │ ✅ Да         │ Узнать доступные методы     │
├──────────┼───────────────┼──────────────────────────────┤
│ POST     │ ❌ Нет        │ Создать новый ресурс        │
│ PATCH    │ ❌ Нет        │ Частичное обновление        │
└──────────┴───────────────┴──────────────────────────────┘

Идемпотентность: повторный запрос не меняет состояние
```

**Пример:**
```javascript
// PUT (идемпотентный)
PUT /users/123
{ "name": "John" }

// Один вызов: пользователь обновлён
// Десять вызовов: пользователь обновлён (тот же результат)

// POST (не идемпотентный)
POST /users
{ "name": "John" }

// Один вызов: создан 1 пользователь
// Десять вызовов: создано 10 пользователей!
```

---

### 3.2. REST принципы

```
┌─────────────────────────────────────────────────────────┐
│              REST Architectural Constraints             │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Client-Server                                       │
│     Разделение ответственности между клиентом и сервером│
│                                                         │
│  2. Stateless                                           │
│     Сервер не хранит состояние клиента между запросами │
│     (аутентификация через JWT, сессии в БД)            │
│                                                         │
│  3. Cacheable                                           │
│     Ответы должны быть помечены как кэшируемые или нет │
│                                                         │
│  4. Uniform Interface                                     │
│     - Resource identification (URI)                     │
│     - Resource manipulation through representations     │
│     - Self-descriptive messages                         │
│     - HATEOAS (Hypermedia as the Engine of Application  │
│       State)                                            │
│                                                         │
│  5. Layered System                                      │
│     Клиент не знает, подключён ли он напрямую к серверу│
│     (прокси, балансировщики, кэш)                       │
│                                                         │
│  6. Code on Demand (опционально)                        │
│     Сервер может отправлять исполняемый код (JS, WASM) │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 3.3. REST API дизайн

**Хорошие URI:**
```
✅ /users              # Коллекция
✅ /users/123          # Конкретный ресурс
✅ /users/123/posts    # Вложенная коллекция
✅ /users?role=admin   # Фильтрация
✅ /users?page=2&limit=20  # Пагинация
✅ /users?sort=-created_at # Сортировка

❌ /getUsers           # Глаголы не нужны
❌ /users/list         # Избыточно
❌ /get_user_by_id?id=123  # Не RESTful
```

**HTTP статус коды:**
```
2xx Success:
- 200 OK — успешный запрос
- 201 Created — ресурс создан
- 204 No Content — успешно, без тела ответа

3xx Redirection:
- 301 Moved Permanently
- 304 Not Modified (кэш валиден)

4xx Client Error:
- 400 Bad Request — невалидные данные
- 401 Unauthorized — нужна аутентификация
- 403 Forbidden — нет прав доступа
- 404 Not Found — ресурс не найден
- 409 Conflict — конфликт (дубликат)
- 422 Unprocessable Entity — валидация не прошла
- 429 Too Many Requests — rate limit

5xx Server Error:
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
```

---

## 4. JWT Authentication

### 4.1. Структура токена

```
JWT = Header.Payload.Signature

Header (алгоритм и тип):
{
  "alg": "HS256",
  "typ": "JWT"
}

Payload (данные):
{
  "sub": "1234567890",
  "username": "john",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516242622
}

Signature (подпись):
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

**Пример токена:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

### 4.2. Реализация на Node.js

```typescript
// auth/jwt.ts
import jwt from 'jsonwebtoken';
import bcrypt from 'bcrypt';

const JWT_SECRET = process.env.JWT_SECRET!;
const ACCESS_TOKEN_EXPIRY = '15m';
const REFRESH_TOKEN_EXPIRY = '7d';

export async function hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 12);
}

export async function verifyPassword(
    password: string,
    hashed: string
): Promise<boolean> {
    return bcrypt.compare(password, hashed);
}

export function generateAccessToken(payload: { userId: string; role: string }): string {
    return jwt.sign(payload, JWT_SECRET, {
        expiresIn: ACCESS_TOKEN_EXPIRY
    });
}

export function generateRefreshToken(payload: { userId: string }): string {
    return jwt.sign(payload, JWT_SECRET, {
        expiresIn: REFRESH_TOKEN_EXPIRY
    });
}

export function verifyToken(token: string): jwt.JwtPayload {
    return jwt.verify(token, JWT_SECRET) as jwt.JwtPayload;
}

// Middleware для защиты роутов
export function authMiddleware(req: Request, res: Response, next: NextFunction) {
    const authHeader = req.headers.authorization;
    
    if (!authHeader?.startsWith('Bearer ')) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    const token = authHeader.substring(7);
    
    try {
        const payload = verifyToken(token);
        req.user = payload;
        next();
    } catch (error) {
        return res.status(401).json({ error: 'Invalid token' });
    }
}
```

---

### 4.3. Refresh Token Flow

```typescript
// auth/routes.ts
import { Router } from 'express';
import { generateAccessToken, generateRefreshToken, verifyToken } from './jwt';

const router = Router();

// Login
router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    // Проверка credentials
    const user = await db.users.findUnique({ where: { email } });
    if (!user || !await verifyPassword(password, user.password)) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // Генерация токенов
    const accessToken = generateAccessToken({
        userId: user.id,
        role: user.role
    });
    
    const refreshToken = generateRefreshToken({ userId: user.id });
    
    // Сохранение refresh токена в БД
    await db.refreshTokens.create({
        data: {
            userId: user.id,
            token: refreshToken,
            expiresAt: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000)
        }
    });
    
    res.json({ accessToken, refreshToken });
});

// Refresh token
router.post('/refresh', async (req, res) => {
    const { refreshToken } = req.body;
    
    try {
        const payload = verifyToken(refreshToken);
        
        // Проверка токена в БД
        const tokenRecord = await db.refreshTokens.findUnique({
            where: { token: refreshToken }
        });
        
        if (!tokenRecord || tokenRecord.expiresAt < new Date()) {
            return res.status(401).json({ error: 'Invalid refresh token' });
        }
        
        // Генерация нового access токена
        const accessToken = generateAccessToken({
            userId: payload.userId,
            role: tokenRecord.user.role
        });
        
        res.json({ accessToken });
        
    } catch (error) {
        res.status(401).json({ error: 'Invalid refresh token' });
    }
});

// Logout
router.post('/logout', async (req, res) => {
    const { refreshToken } = req.body;
    
    await db.refreshTokens.delete({ where: { token: refreshToken } });
    
    res.json({ message: 'Logged out' });
});
```

---

## 5. CORS

### 5.1. Как работает CORS

```
┌─────────────────────────────────────────────────────────┐
│              CORS Preflight Request                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Client (https://frontend.com)                          │
│         │                                               │
│         │ 1. OPTIONS /api/data                         │
│         │    Origin: https://frontend.com              │
│         │    Access-Control-Request-Method: POST       │
│         │    Access-Control-Request-Headers: Content-Type│
│         ▼                                               │
│  Server (https://api.backend.com)                       │
│         │                                               │
│         │ 2. Response                                   │
│         │    Access-Control-Allow-Origin:              │
│         │      https://frontend.com                    │
│         │    Access-Control-Allow-Methods: GET, POST   │
│         │    Access-Control-Allow-Headers: Content-Type│
│         │    Access-Control-Max-Age: 86400             │
│         ▼                                               │
│  Client (теперь может делать POST запросы)              │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

### 5.2. Настройка CORS в Express

```typescript
import cors from 'cors';
import express from 'express';

const app = express();

// ❌ Опасно: разрешает все домены
app.use(cors({
    origin: '*'
}));

// ✅ Безопасно: конкретные домены
app.use(cors({
    origin: [
        'https://myapp.com',
        'https://www.myapp.com',
        'https://admin.myapp.com'
    ],
    credentials: true,  // Разрешить cookies
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization'],
    maxAge: 86400  // Кэшировать preflight 24 часа
}));

// Для разных роутов разные настройки
app.use('/api/public', cors({ origin: '*' }));
app.use('/api/private', cors({
    origin: 'https://myapp.com',
    credentials: true
}));
```

---

## 6. SOLID Принципы

### 6.1. Примеры нарушения и применения

**Single Responsibility:**
```typescript
// ❌ Нарушение SRP
class UserService {
    async createUser(data: UserData) {
        // Валидация
        // Хеширование пароля
        // Сохранение в БД
        // Отправка email
        // Логирование
    }
}

// ✅ Применение SRP
class UserValidator { validate(data: UserData) { ... } }
class PasswordHasher { hash(password: string) { ... } }
class UserRepository { save(user: User) { ... } }
class EmailService { sendWelcome(user: User) { ... } }
class UserService {
    constructor(
        private validator: UserValidator,
        private hasher: PasswordHasher,
        private repo: UserRepository,
        private email: EmailService
    ) {}
    
    async createUser(data: UserData) {
        this.validator.validate(data);
        const user = new User(data, this.hasher.hash(data.password));
        await this.repo.save(user);
        await this.email.sendWelcome(user);
    }
}
```

**Open/Closed:**
```typescript
// ❌ Нарушение OCP
class PaymentProcessor {
    process(type: string, amount: number) {
        if (type === 'paypal') {
            // PayPal логика
        } else if (type === 'stripe') {
            // Stripe логика
        } else if (type === 'crypto') {
            // Crypto логика
        }
    }
}

// ✅ Применение OCP
interface PaymentProvider {
    process(amount: number): Promise<void>;
}

class PayPalProvider implements PaymentProvider {
    async process(amount: number) { /* ... */ }
}

class StripeProvider implements PaymentProvider {
    async process(amount: number) { /* ... */ }
}

class PaymentProcessor {
    constructor(private provider: PaymentProvider) {}
    
    async process(amount: number) {
        return this.provider.process(amount);
    }
}

// Новый провайдер — новый класс, без изменения существующих
```

---

## 🔗 Связанные темы

- [[05_Infrastructure/02_System_Analysis_BPMN_UML]] — Системный анализ
- [[05_Infrastructure/03_TypeScript_Security_Testing]] — Безопасность
- [[02_JavaScript_NodeJS/04_NodeJS_Threads_Queues]] — Микросервисы

---

*Файл обновлён: 17 марта 2026*
