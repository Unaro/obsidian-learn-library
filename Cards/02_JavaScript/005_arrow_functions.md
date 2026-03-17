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
  - this
  - arrow-functions
  - "#flashcard/javascript"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#2-1-Контекст-выполнения-и-this
---

# Стрелочные функции vs обычные

## Вопрос
В чём разница между стрелочными и обычными функциями в JavaScript?
?
## Ответ

**Стрелочные функции:**
- Нет своего `this` (берётся из родительской области)
- Нельзя использовать как конструктор
- Нет `arguments`
- Компактный синтаксис

```javascript
const obj = {
    name: 'Object',
    regularMethod() {
        const inner = function() {
            console.log(this.name);  // undefined
        };
        inner();
    },
    arrowMethod() {
        const inner = () => {
            console.log(this.name);  // 'Object'
        };
        inner();
    }
};
```

**Обычные функции:**
- Свой `this` (определяется контекстом вызова)
- Можно использовать как конструктор
- Есть `arguments`
- Полный синтаксис

**Когда что использовать:**
- **Стрелочные** — колбэки, сохранение `this`, короткие функции
- **Обычные** — методы объектов, конструкторы, динамический `this`

---

#flashcard #javascript #arrow-functions #this
