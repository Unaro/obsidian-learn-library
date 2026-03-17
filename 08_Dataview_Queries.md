# 📊 Dataview запросы для трекинга подготовки

> **Назначение:** Готовые Dataview запросы (DQL и DataviewJS) для отслеживания прогресса, повторения и планирования изучения тем.

---

## 📋 Доступные поля Dataview

### Поля файлов (Frontmatter)

```yaml
created: YYYY-MM-DD          # Дата создания
updated: YYYY-MM-DD          # Дата обновления
category: string             # Категория
type: string                 # Тип (topic, index, cheat-sheet, guide, dashboard)
priority: string             # Приоритет (high, medium, low)
status: string               # Статус (pending, in_progress, completed, review)
completed_date: YYYY-MM-DD   # Дата завершения
tags: [tag1, tag2, ...]      # Теги
difficulty: string           # Сложность (beginner, intermediate, advanced)
estimated_hours: number      # Примерное время на изучение
```

---

## 🔍 DQL запросы (Dataview Query Language)

### 1. Все темы по категориям

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    status as "Статус",
    priority as "Приоритет",
    difficulty as "Сложность"
FROM "Собеседование"
WHERE type = "topic"
SORT category ASC, priority ASC
```

---

### 2. Темы для изучения (не завершённые)

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

### 3. Завершённые темы (для повторения)

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    completed_date as "Завершено",
    date(now) - date(completed_date) as "Дней назад"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
SORT completed_date DESC
LIMIT 10
```

---

### 4. Темы для повторения (прошло > 14 дней)

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    completed_date as "Завершено",
    date(now) - date(completed_date) as "Дней назад"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
WHERE date(now) - date(completed_date) > 14
SORT completed_date ASC
```

---

### 5. Прогресс по категориям

```dataview
TABLE 
    category as "Категория",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "completed")) as "Завершено",
    round(100 * length(filter(rows, (r) => r.status = "completed")) / length(rows), 1) + "%" as "Прогресс"
FROM "Собеседование"
WHERE type = "topic"
GROUP BY category
```

---

### 6. Темы по приоритету

```dataview
TABLE 
    priority as "Приоритет",
    length(rows) as "Количество",
    length(filter(rows, (r) => r.status = "completed")) as "Завершено"
FROM "Собеседование"
WHERE type = "topic" AND priority != null
GROUP BY priority
SORT priority ASC
```

---

### 7. Темы по сложности

```dataview
TABLE 
    difficulty as "Сложность",
    length(rows) as "Количество",
    length(filter(rows, (r) => r.status = "completed")) as "Завершено",
    round(100 * length(filter(rows, (r) => r.status = "completed")) / length(rows), 1) + "%" as "Прогресс"
FROM "Собеседование"
WHERE type = "topic" AND difficulty != null
GROUP BY difficulty
SORT difficulty ASC
```

---

### 8. Последние обновлённые темы

```dataview
TABLE 
    file.link as "Тема",
    category as "Категория",
    updated as "Обновлено",
    status as "Статус"
FROM "Собеседование"
WHERE updated != null AND type = "topic"
SORT updated DESC
LIMIT 10
```

---

### 9. Темы без статуса

```dataview
TABLE
    file.link as "Тема",
    category as "Категория",
    file.folder as "Папка"
FROM "Собеседование"
WHERE status = null AND type = "topic"
SORT file.folder ASC
```
---

## 📊 DataviewJS запросы

### Прогресс бар по категориям

```dataviewjs
const categories = dv.pages('"Собеседование"')
    .where(p => p.type === 'topic')
    .groupBy(p => p.category);

dv.table(
    ["Категория", "Всего", "✅ Завершено", "Прогресс"],
    categories.map(g => [
        g.key,
        g.rows.length,
        g.rows.where(r => r.status === 'completed').length,
        Math.round(100 * g.rows.where(r => r.status === 'completed').length / g.rows.length) + '%'
    ])
);
```

---

### Список тем для изучения сегодня

```dataviewjs
const today = dv.date('now');

const topics = dv.pages('"Собеседование"')
    .where(p => p.status === 'in_progress')
    .sort(p => p.priority, 'asc');

if (topics.length > 0) {
    dv.table(
        ["Тема", "Категория", "Приоритет", "Часов"],
        topics.map(p => [
            p.file.link,
            p.category,
            p.priority,
            p.estimated_hours
        ])
    );
} else {
    dv.paragraph("Нет активных тем для изучения!");
}
```

---

### Карточки для повторения (Spaced Repetition)

```dataviewjs
const completed = dv.pages('"Собеседование"')
    .where(p => p.status === 'completed' && p.completed_date != null);

const forReview = completed.where(p => {
    const daysAgo = dv.date('now') - p.completed_date;
    return [7, 14, 30, 60].includes(daysAgo.days);
});

if (forReview.length > 0) {
    dv.table(
        ["Тема", "Категория", "Последнее повторение", "Дней назад"],
        forReview.sort(p => p.completed_date, 'asc').map(p => [
            p.file.link,
            p.category,
            p.completed_date,
            (dv.date('now') - p.completed_date).days
        ])
    );
} else {
    dv.paragraph("Нет тем для повторения!");
}
```

---

### Визуализация прогресса

```dataviewjs
const categories = dv.pages('"Собеседование"')
    .where(p => p.type === 'topic')
    .groupBy(p => p.category);

for (let group of categories) {
    const total = group.rows.length;
    const completed = group.rows.where(r => r.status === 'completed').length;
    const percent = Math.round(100 * completed / total);
    
    dv.header(3, group.key);
    dv.paragraph(`**Прогресс:** ${completed}/${total} (${percent}%)`);
    
    // Progress bar
    const bar = '█'.repeat(Math.floor(percent / 5)) + '░'.repeat(20 - Math.floor(percent / 5));
    dv.paragraph(`\`${bar}\` ${percent}%`);
}
```

---

## 🎯 Примеры использования

### Daily Note шаблон с Dataview

```markdown
## 📚 Изучение на сегодня
```

```dataview
TABLE file.link as "Тема", priority as "Приоритет", estimated_hours as "Часов"
FROM "Собеседование"
WHERE status = "in_progress"
LIMIT 3
```

## 🔄 Для повторения

```dataview
TABLE file.link as "Тема", date(now) - date(completed_date) as "Дней назад"
FROM "Собеседование"
WHERE status = "completed" AND completed_date != null
WHERE date(now) - date(completed_date) >= 7 AND date(now) - date(completed_date) <= 14
```

## ✅ Сегодня завершено

```dataview
TABLE file.link as "Тема", category as "Категория"
FROM "Собеседование"
WHERE completed_date = date(now)
```
---

*Руководство обновлено: 17 марта 2026*
