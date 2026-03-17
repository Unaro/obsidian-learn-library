---
created: 2026-03-17
category: JavaScript
type: flashcard
difficulty: beginner
frequency: high
status: new
next_review:
tags:
  - javascript
  - var
  - let
  - const
  - "#flashcard/javascript"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#2-3-Hoisting-Всплытие
---

# var vs let vs const

## Вопрос
В чём разница между `var`, `let` и `const` в JavaScript?
?
## Ответ

**`var`:**
- Функциональная область видимости
- Всплывает (hoisting), инициализируется `undefined`
- Можно переназначать
- Доступно до объявления (возвращает undefined)

```javascript
console.log(x);  // undefined
var x = 5;
```

**`let`:**
- Блочная область видимости (`{...}`)
- Всплывает, но в TDZ (Temporal Dead Zone)
- Можно переназначать
- ReferenceError до объявления

```javascript
console.log(x);  // ReferenceError
let x = 5;
```

**`const`:**
- Блочная область видимости
- Нельзя переназначить
- Обязательна инициализация
- Для объектов можно менять свойства

```javascript
const obj = { a: 1 };
obj.a = 2;  // OK
obj = {};   // TypeError
```

**Когда что использовать:**
- `const` — по умолчанию (80-90% случаев)
- `let` — когда нужно переназначать
- `var` — никогда (только в легаси)

---

#flashcard #javascript #var #let #const
