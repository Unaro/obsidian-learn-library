---
created: 2026-03-17
updated: 2026-03-17
category: Index
type: dashboard
priority: high
status: completed
tags: [dashboard, navigation, dataview, progress]
---

# 🎯 Dashboard подготовки к собеседованию

> **Назначение:** Центральный экран для отслеживания прогресса, навигации и планирования изучения тем.

---

## 📊 Общий прогресс

```dataviewjs
const topics = dv.pages('"Собеседование"').where(p => p.type === 'topic');
const completed = topics.where(p => p.status === 'completed').length;
const inProgress = topics.where(p => p.status === 'in_progress').length;
const pending = topics.where(p => p.status === 'pending').length;

dv.table(
    ["📚 Всего тем", "✅ Завершено", "🔄 В процессе", "⏳ Ожидает"],
    [[topics.length, completed, inProgress, pending]]
);
```

---

## 📈 Прогресс по категориям

```dataview
TABLE 
    category as "Категория",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "completed")) as "✅",
    round(100 * length(filter(rows, (r) => r.status = "completed")) / length(rows), 1) + "%" as "Прогресс"
FROM "Собеседование"
WHERE type = "topic"
GROUP BY category
```

---

## 🎯 Темы для изучения

### 🔥 Высокий приоритет

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    difficulty as "Сложность",
    estimated_hours as "Часов"
FROM "Собеседование"
WHERE priority = "high" AND status != "completed"
SORT difficulty ASC
LIMIT 5
```

### 📋 Все незавершённые

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    priority as "Приоритет",
    estimated_hours as "Часов"
FROM "Собеседование"
WHERE status != "completed" AND type = "topic"
SORT priority ASC
```

---

## 🔄 Для повторения

### Прошло 7-14 дней

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    completed_date as "Завершено",
    date(now) - date(completed_date) as "Дней назад"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
WHERE date(now) - date(completed_date) >= 7 AND date(now) - date(completed_date) <= 14
SORT completed_date ASC
```

### Прошло > 30 дней

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    completed_date as "Завершено",
    date(now) - date(completed_date) as "Дней назад"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
WHERE date(now) - date(completed_date) > 30
SORT completed_date ASC
```

---

## 📅 Последние завершённые

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    completed_date as "Дата",
    difficulty as "Сложность"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
SORT completed_date DESC
LIMIT 5
```

---

## 📊 Статистика

### По сложности

```dataview
TABLE 
    difficulty as "Сложность",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "completed")) as "Завершено",
    round(100 * length(filter(rows, (r) => r.status = "completed")) / length(rows), 1) + "%" as "Прогресс"
FROM "Собеседование"
WHERE type = "topic" AND difficulty != null
GROUP BY difficulty
SORT difficulty ASC
```

### По времени (DataviewJS)

```dataviewjs
const topics = dv.pages('"Собеседование"')
    .where(p => p.type === 'topic' && p.estimated_hours != null);

const totalHours = topics.sum(p => p.estimated_hours);
const completedHours = topics.where(p => p.status === 'completed').sum(p => p.estimated_hours);
const remainingHours = totalHours - completedHours;

dv.table(
    ["⏱️ Всего часов", "✅ Изучено", "📚 Осталось"],
    [[totalHours, completedHours, remainingHours]]
);

// Прогресс бар
const percent = Math.round(100 * completedHours / totalHours);
dv.paragraph(`**Прогресс:** ${percent}%`);
const bar = '█'.repeat(Math.floor(percent / 5)) + '░'.repeat(20 - Math.floor(percent / 5));
dv.paragraph(`\`${bar}\` ${percent}%`);
```

---

## 🗂️ Быстрая навигация

### Темы для изучения

| Категория | Темы |
|-----------|------|
| 🐍 Python | [[01_Python_Backend/01_Python_Core\|Core]], [[01_Python_Backend/02_Django_REST_API\|Django]], [[01_Python_Backend/03_Web_Servers_Quality\|Web]] |
| ⚡ JavaScript | [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop\|Core]], [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding\|Advanced]], [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network\|Node.js]], [[02_JavaScript_NodeJS/04_NodeJS_Threads_Queues\|Queues]] |
| ⚛️ React | [[03_React_Frontend/01_React_Architecture_FSD\|Architecture]], [[03_React_Frontend/02_React_Internals_Optimization\|Internals]], [[03_React_Frontend/03_NextJS_App_Router_React19\|Next.js]], [[03_React_Frontend/04_NextJS_RSC\|RSC]] |
| 🗄️ Базы Данных | [[04_Databases/01_Databases_ORM_SQL\|SQL]], [[04_Databases/02_Database_Optimization_N1\|Optimization]], [[04_Databases/03_Advanced_Database_Offline\|Advanced]] |
| 🌐 Инфраструктура | [[05_Infrastructure/01_Infrastructure_Networks_Docker\|Docker]], [[05_Infrastructure/02_System_Analysis_BPMN_UML\|System Analysis]], [[05_Infrastructure/03_TypeScript_Security_Testing\|TypeScript]] |
| 🤝 Soft Skills | [[06_Soft_Skills/01_Behavioral_Interviews\|Interviews]], [[06_Soft_Skills/02_Time_Management_Productivity\|Productivity]], [[06_Soft_Skills/03_Team_Leadership\|Leadership]] |

### Карточки для повторения

| Категория | Карточки |
|-----------|----------|
| 🐍 Python | [[Cards/01_Python\|Перейти]] |
| ⚡ JavaScript | [[Cards/02_JavaScript\|Перейти]] |
| ⚛️ React | [[Cards/03_React\|Перейти]] |
| 🗄️ Базы Данных | [[Cards/04_Databases\|Перейти]] |
| 🌐 Инфраструктура | [[Cards/05_Infrastructure\|Перейти]] |
| 🤝 Soft Skills | [[Cards/06_Soft_Skills\|Перейти]] |

### Справочные материалы

| Файл | Описание |
|------|----------|
| 📄 [[07_CLI_Cheat_Sheets]] | Шпаргалки по Linux, Git, Docker, kubectl |
| 📄 [[08_Dataview_Queries]] | Dataview запросы |
| 📄 [[98_Obsidian_Setup_Guide]] | Настройка Obsidian |
| 📄 [[101_Cards_Index]] | База карточек (50+ вопросов) |

---

## 📝 План на сегодня

```markdown
## Цели на день
- [ ] Изучить тему: 
- [ ] Повторить карточки: 
- [ ] Прорешать задач: 

## Заметки
```

---

*Последнее обновление: `=date(now)`*
