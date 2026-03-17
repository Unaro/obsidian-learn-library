---
created: 2026-03-17
category: Next.js
type: flashcard
difficulty: advanced
frequency: high
status: new
next_review:
tags:
  - nextjs
  - caching
  - revalidation
  - "#flashcard/nextJS"
related:
  - - 03_React_Frontend/03_NextJS_App_Router_React19#1-1-Архитектура-кэширования
---

# Уровни кэширования в Next.js

## Вопрос
Какие 4 уровня кэширования в Next.js App Router?
?
## Ответ

**1. Request Memoization:**
- На сервере в рамках **одного запроса**
- Если 5 раз вызвать `fetch` с одинаковым URL → 1 реальный запрос
- Срок жизни: время обработки запроса

**2. Data Cache:**
- На сервере **между запросами**
- Персистентный кэш fetch-запросов
- Инвалидируется через `revalidatePath` / `revalidateTag`

**3. Full Route Cache:**
- На сервере, относится к **SSG**
- HTML и RSC Payload кэшируются на этапе build
- До деплоя или инвалидации

**4. Router Cache:**
- **В браузере** (клиентский кэш)
- RSC Payload посещённых страниц
- Делает навигацию мгновенной

**Пример:**
```javascript
// Data Cache с revalidate
const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 3600 }  // 1 час
});

// Инвалидация
revalidatePath('/blog');
revalidateTag('posts');
```

---

#flashcard #nextjs #caching #revalidation
