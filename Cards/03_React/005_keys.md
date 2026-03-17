---
created: 2026-03-17
category: React
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review:
tags:
  - react
  - keys
  - lists
  - "#flashcard/reactJS"
related:
  - - 03_React_Frontend/02_React_Internals_Optimization#2-React-Fiber-и-Алгоритм-Согласования-Reconciliation
---

# Зачем нужен key в React?

## Вопрос
Для чего используется prop `key` в React списках? Почему не использовать index?
?
## Ответ

**`key`** — уникальный идентификатор элемента списка, помогающий React понять какие элементы изменились.

**Зачем нужен:**
- React использует key для оптимизации Reconciliation
- Без key React пересоздаёт все элементы
- С key React точечно обновляет только изменённые

**❌ Плохо (index как key):**
```javascript
{items.map((item, index) => (
    <ListItem key={index} data={item} />
))}
// При изменении порядка — React думает что элементы те же
```

**✅ Хорошо (уникальный ID):**
```javascript
{items.map((item) => (
    <ListItem key={item.id} data={item} />
))}
// React точно знает какой элемент изменился
```

**Когда index допустим:**
- Список не меняется (нет сортировки, фильтрации)
- Элементы не имеют внутреннего состояния

**Правила key:**
- Уникальный средиsiblings
- Стабильный (не меняется со временем)
- Не использовать index (если список динамический)

---

#flashcard #react #keys #lists #optimization
