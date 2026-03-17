---
created: 2026-03-17
category: TypeScript
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [typescript, utility-types, flashcard/typescript]
related: [[05_Infrastructure/03_TypeScript_Security_Testing#1-3-Utility-Types-Утилитарные-типы]]
---

# Utility Types в TypeScript

## Вопрос
Что такое Utility Types в TypeScript? Приведите примеры использования.
?
## Ответ

**Utility Types** — встроенные типы для трансформации других типов.

**`Partial<T>`** — все свойства опциональны:
```typescript
interface User { id: number; name: string; }
type PartialUser = Partial<User>;  // { id?: number; name?: string; }
```

**`Required<T>`** — все свойства обязательны:
```typescript
type RequiredUser = Required<PartialUser>;  // { id: number; name: string; }
```

**`Pick<T, K>`** — выбрать свойства:
```typescript
type UserId = Pick<User, 'id'>;  // { id: number; }
```

**`Omit<T, K>`** — исключить свойства:
```typescript
type UserWithoutId = Omit<User, 'id'>;  // { name: string; }
```

**`Record<K, T>`** — словарь:
```typescript
type UserMap = Record<string, User>;  // { [key: string]: User; }
```

**`Exclude<T, U>`** — исключить из union:
```typescript
type Status = 'pending' | 'success' | 'error';
type NonError = Exclude<Status, 'error'>;  // 'pending' | 'success'
```

---

#flashcard #typescript #utility-types
