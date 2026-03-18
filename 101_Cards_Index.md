---
created: 2026-03-17
updated: 2026-03-17
category: Reference
type: flashcards-db
priority: high
status: completed
tags: [interview, flashcards, questions, database]
---

# 🎴 База карточек для собеседований

> **Назначение:** Единая база вопросов и карточек для подготовки к собеседованиям.
>
> **Использование:** Изучение, повторение, отслеживание прогресса.

---

## 📊 Статистика базы

```dataviewjs
const cards = dv.pages('"obsidian-learn-library/Cards"');
const byCategory = cards.groupBy(c => c.category);
const byDifficulty = cards.groupBy(c => c.difficulty);
const byStatus = cards.groupBy(c => c.status);

dv.table(
    ["Категория", "Всего", "Изучено", "В процессе", "Новых"],
    byCategory.map(g => [
        g.key,
        g.rows.length,
        g.rows.where(c => c.status === "learned").length,
        g.rows.where(c => c.status === "learning").length,
        g.rows.where(c => c.status === "new").length
    ])
);

dv.paragraph(`**Всего карточек:** ${cards.length}`);
dv.paragraph(`**Изучено:** ${cards.where(c => c.status === "learned").length}`);
dv.paragraph(`**В процессе:** ${cards.where(c => c.status === "learning").length}`);
dv.paragraph(`**Новых:** ${cards.where(c => c.status === "new").length}`);
```

---

## 🗂️ Категории

| Категория         | Карточки                             | Изучено                                                                                          | Прогресс                                                                                                                                                                                          |
| ----------------- | ------------------------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🐍 Python         | [[Cards/01_Python\|Перейти]]         | `=length(filter(file.link, (f) => contains(f, "Cards/01_Python") and status="learned"))`         | `=round(length(filter(file.link, (f) => contains(f, "Cards/01_Python") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/01_Python"))),1) * 100)`%                  |
| ⚡ JavaScript      | [[Cards/02_JavaScript\|Перейти]]     | `=length(filter(file.link, (f) => contains(f, "Cards/02_JavaScript") and status="learned"))`     | `=round(length(filter(file.link, (f) => contains(f, "Cards/02_JavaScript") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/02_JavaScript"))), 1) * 100)`%         |
| ⚛️ React          | [[Cards/03_React\|Перейти]]          | `=length(filter(file.link, (f) => contains(f, "Cards/03_React") and status="learned"))`          | `=round(length(filter(file.link, (f) => contains(f, "Cards/03_React") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/03_React"))), 1) * 100)`%                   |
| 🗄️ Базы Данных   | [[Cards/04_Databases\|Перейти]]      | `=length(filter(file.link, (f) => contains(f, "Cards/04_Databases") and status="learned"))`      | `=round(length(filter(file.link, (f) => contains(f, "Cards/04_Databases") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/04_Databases"))), 1) * 100)`%           |
| 🌐 Инфраструктура | [[Cards/05_Infrastructure\|Перейти]] | `=length(filter(file.link, (f) => contains(f, "Cards/05_Infrastructure") and status="learned"))` | `=round(length(filter(file.link, (f) => contains(f, "Cards/05_Infrastructure") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/05_Infrastructure"))), 1) * 100)`% |
| 🤝 Soft Skills    | [[Cards/06_Soft_Skills\|Перейти]]    | `=length(filter(file.link, (f) => contains(f, "Cards/06_Soft_Skills") and status="learned"))`    | `=round(length(filter(file.link, (f) => contains(f, "Cards/06_Soft_Skills") and status="learned")) / max(length(filter(file.link, (f) => contains(f, "Cards/06_Soft_Skills"))), 1) * 100)`%       |


---

## 🔄 Для повторения сегодня

```dataview
TABLE without id
    file.link as "Карточка",
    category as "Категория",
    difficulty as "Сложность",
    date(now) - date(next_review) as "Просрочено дней"
FROM "obsidian-learn-library/Cards"
WHERE status = "learning" AND next_review <= date(now)
SORT next_review ASC
LIMIT 20
```

---

## 📝 Новые карточки для изучения

```dataview
TABLE without id
    file.link as "Карточка",
    category as "Категория",
    difficulty as "Сложность",
    frequency as "Частота"
FROM "obsidian-learn-library/Cards"
WHERE status = "new"
SORT frequency ASC, difficulty ASC
LIMIT 10
```

---

## 🔥 Топ частых вопросов

### Python
- [[Cards/01_Python/001_mutable_immutable\|Мутабельность типов]]
- [[Cards/01_Python/002_list_tuple\|List vs Tuple]]

### JavaScript
- [[Cards/02_JavaScript/001_event_loop\|Event Loop]]
- [[Cards/02_JavaScript/002_closure\|Замыкания]]

### React
- [[Cards/03_React/001_rsc_ssr\|RSC vs SSR]]

---

## 📚 Списки для изучения

### [[Cards/List_Beginner\|Для начинающих]]
### [[Cards/List_Intermediate\|Продолжающие]]
### [[Cards/List_Advanced\|Экспертные]]

---

## 📊 Прогресс по сложности

```dataview
TABLE 
    difficulty as "Сложность",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "learned")) as "✅",
    round(100 * length(filter(rows, (r) => r.status = "learned")) / length(rows), 1) + "%" as "Прогресс"
FROM "obsidian-learn-library/Cards"
WHERE difficulty != null
GROUP BY difficulty
SORT difficulty ASC
```

---

## 📊 Прогресс по частоте

```dataview
TABLE 
    frequency as "Частота",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "learned")) as "✅",
    round(100 * length(filter(rows, (r) => r.status = "learned")) / length(rows), 1) + "%" as "Прогресс"
FROM "obsidian-learn-library/Cards"
WHERE frequency != null
GROUP BY frequency
SORT frequency ASC
```

---

## 🔗 Связанные файлы

- [[README]] — 📚 Главная библиотека
- [[00_Dashboard]] — Главный экран подготовки
- [[00_КАТАЛОГ_ТЕМ]] — Каталог тем
- [[Cards/README]] — Руководство по карточкам
- [[07_CLI_Cheat_Sheets]] — Шпаргалки CLI
- [[08_Dataview_Queries]] — Dataview запросы

---

*База обновлена: `=date(now)`*
