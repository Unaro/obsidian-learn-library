---
created: 2026-03-17
category: TypeScript
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [typescript, interface, type, flashcard/typescript]
related: [[05_Infrastructure/03_TypeScript_Security_Testing#1-1-interface-vs-type]]
---

# interface vs type в TypeScript

## Вопрос
В чём разница между `interface` и `type` в TypeScript?
?
## Ответ

**`interface`:**
- Для описания формы **объектов**
- Поддерживает **слияние** (declaration merging)
- Поддерживает `extends`
- Лучше для публичных API

```typescript
interface User {
    id: number;
    name: string;
}

interface User {
    email: string;  // Сливается с предыдущим
}
```

**`type` (Type Alias):**
- Универсален (примитивы, union, intersection)
- Не поддерживает слияние
- Поддерживает вычисляемые свойства

```typescript
type ID = string | number;  // Union
type Point = [number, number];  // Tuple
type User = { id: number } & { name: string };  // Intersection
type Mapped<T> = { [K in keyof T]: T[K] | null };
```

**Когда что использовать:**
- **interface** — объекты, классы, расширение
- **type** — union, примитивы, mapped types

---

#flashcard #typescript #interface #type
