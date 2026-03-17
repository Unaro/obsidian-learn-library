---
created: 2026-03-17
category: JavaScript
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review:
tags:
  - javascript
  - closure
  - scope
  - "#flashcard/javascript"
related:
  - - 02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop#2-2-Замыкания-Closures
---

# Что такое замыкание?

## Вопрос
Что такое замыкание в JavaScript? Приведите пример использования.
?
## Ответ

**Замыкание** — способность функции запоминать и получать доступ к своей лексической области видимости, даже когда функция выполняется вне этой области.

**Пример:**
```javascript
function outer() {
    let count = 0;
    
    return function inner() {
        count++;
        return count;
    };
}

const counter = outer();
console.log(counter());  // 1
console.log(counter());  // 2
console.log(counter());  // 3
```

**Применение:**
1. **Инкапсуляция** — скрытие приватных переменных
2. **Фабрики функций** — создание функций с параметрами
3. **Колбэки и обработчики** — доступ к внешним данным
4. **Модульный паттерн** — приватные методы и свойства

**Классическая ошибка:**
```javascript
// ❌ Все функции вернут 3
for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}

// ✅ let создаёт новую переменную для каждой итерации
for (let i = 0; i < 3; i++) {
    setTimeout(() => console.log(i), 100);
}
```

---

#flashcard #javascript #closure #scope
