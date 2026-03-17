---
created: 2026-03-17
category: TypeScript
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [typescript, generics, flashcard/typescript]
related: [[05_Infrastructure/03_TypeScript_Security_Testing#1-2-Generics-Дженерики]]
---

# Что такое Generics в TypeScript?

## Вопрос
Что такое Generics (дженерики) в TypeScript? Когда использовать?
?
## Ответ

**Generics** — переменные для типов (`<T>`), позволяющие создавать переиспользуемые компоненты с сохранением типизации.

**Базовый пример:**
```typescript
function identity<T>(arg: T): T {
    return arg;
}

identity<string>('Hello');  // Явное указание
identity(42);  // Вывод типа (number)
```

**Ограничения (constraints):**
```typescript
function getLength<T extends { length: number }>(arg: T): number {
    return arg.length;
}

getLength('Hello');  // OK (string имеет length)
getLength(42);  // Error (number не имеет length)
```

**Generics в React:**
```typescript
interface ListProps<T> {
    items: T[];
    renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
    return <ul>{items.map(renderItem)}</ul>;
}
```

**Когда использовать:**
- Переиспользуемые компоненты
- Библиотеки и утилиты
- Типизация API ответов

---

#flashcard #typescript #generics
