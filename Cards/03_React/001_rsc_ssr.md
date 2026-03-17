---
created: 2026-03-17
category: React
type: flashcard
difficulty: advanced
frequency: high
status: new
next_review:
tags:
  - react
  - rsc
  - ssr
  - "#flashcard/reactJS"
related:
  - - 03_React_Frontend/04_NextJS_RSC#1-RSC-vs-SSR-Архитектурные-различия
---

# RSC vs SSR

## Вопрос
В чём разница между React Server Components (RSC) и SSR?
?
## Ответ

**SSR (Server-Side Rendering):**
- Сервер генерирует HTML и отправляет клиенту
- JavaScript бандл загружается и выполняется гидратация
- Весь код компонентов попадает в бандл
- Используется в Next.js Pages Router

**RSC (React Server Components):**
- Компоненты выполняются только на сервере
- Код и зависимости **не попадают** в клиентский бандл
- Передаётся RSC Payload (бинарный формат)
- Используется в Next.js App Router

**Сравнение:**
| Характеристика | SSR | RSC |
|---------------|-----|-----|
| JS в бандле | Весь компонент | Только Client Components |
| Доступ к БД | Через API | Прямой |
| Хуки | Все | Только в Client Components |
| Размер бандла | Большой | Минимальный |

**Пример:**
```tsx
// RSC — только на сервере
export default async function Page() {
    const data = await db.query('SELECT * FROM...');
    return <DataDisplay data={data} />;
}
```

---

#flashcard #react #rsc #ssr #nextjs
