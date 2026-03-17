---
created: 2026-03-17
category: React
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review:
tags:
  - react
  - virtual-dom
  - shadow-dom
  - "#flashcard/reactJS"
related:
  - - 03_React_Frontend/02_React_Internals_Optimization#1-Virtual-DOM-vs-Shadow-DOM
---

# Virtual DOM vs Shadow DOM

## Вопрос
В чём разница между Virtual DOM и Shadow DOM?
?
## Ответ

**Virtual DOM (React):**
- Легковесная JavaScript-репрезентация реального DOM
- React создаёт новое V-DOM дерево при изменении состояния
- Сравнивает со старым (Diffing алгоритм)
- Точечно обновляет только изменённые узлы
- **Никак не связан с Shadow DOM**

**Shadow DOM (браузерная технология):**
- Механизм инкапсуляции веб-компонентов
- Изоляция HTML и CSS от внешнего мира
- Часть спецификации Web Components
- **Никак не связан с React**

**Сравнение:**
| Характеристика | Virtual DOM | Shadow DOM |
|---------------|-------------|------------|
| **Что это** | JS библиотека | Браузерный API |
| **Зачем** | Оптимизация рендеринга | Инкапсуляция стилей |
| **Кто использует** | React, Vue, Preact | Web Components |
| **Изоляция** | Нет | Да (HTML + CSS) |

**Почему путают:**
- Похожее название
- Оба связаны с DOM
- Оба используются в современных фреймворках

---

#flashcard #react #virtual-dom #shadow-dom
