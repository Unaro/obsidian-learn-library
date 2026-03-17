---
created: 2026-03-17
category: Next.js
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [nextjs, server-actions, flashcard]
related: [[03_React_Frontend/03_NextJS_App_Router_React19#3-Server-Actions-Серверные-экшены]]
---

# Что такое Server Actions?

## Вопрос
Что такое Server Actions в Next.js? Когда использовать?
?
## Ответ

**Server Actions** — асинхронные функции, которые выполняются на сервере, но могут вызываться из клиентских компонентов.

**Преимущества:**
- Не нужно писать API routes
- Прямой доступ к БД на сервере
- Строгая типизация между клиентом и сервером
- **Progressive Enhancement** — работает без JS

**Пример:**
```tsx
// app/actions.ts
'use server';

export async function createPost(formData: FormData) {
    const title = formData.get('title');
    await db.posts.create({ title });
    revalidatePath('/blog');
}

// app/page.tsx
import { createPost } from './actions';

export default function Page() {
    return (
        <form action={createPost}>
            <input name="title" />
            <button type="submit">Create</button>
        </form>
    );
}
```

**Когда использовать:**
- ✅ Мутации данных (create, update, delete)
- ✅ Обработка форм
- ❌ Получение данных (используйте Server Components)

---

#flashcard #nextjs #server-actions #forms
