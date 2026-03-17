---
created: 2026-03-17
category: JavaScript
type: flashcard
difficulty: beginner
frequency: medium
status: new
next_review:
tags:
  - javascript
  - hoisting
  - flashcard
  - "#flashcard/javascript"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#2-3-Hoisting-Всплытие
---

# Что такое hoisting?

## Вопрос
Что такое hoisting (всплытие) в JavaScript?
?
## Ответ

**Hoisting** — механизм, при котором объявления переменных и функций "поднимаются" в верх области видимости перед выполнением кода.

**Объявления функций (полностью всплывают):**
```javascript
sayHello();  // 'Hello!'
function sayHello() {
    console.log('Hello!');
}
```

**var (всплывает объявление, не значение):**
```javascript
console.log(x);  // undefined
var x = 5;
console.log(x);  // 5

// Эквивалентно:
var x;
console.log(x);
x = 5;
```

**let и const (всплывают, но TDZ):**
```javascript
console.log(y);  // ReferenceError!
let y = 10;

// Попадают в Temporal Dead Zone
```

**Порядок всплытия:**
1. Объявления функций
2. Объявления переменных (var)
3. let, const (не всплывают, TDZ)

---

#flashcard #javascript #hoisting
