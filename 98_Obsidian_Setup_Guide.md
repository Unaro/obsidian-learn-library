---
created: 2026-03-17
updated: 2026-03-17
category: Reference
type: guide
priority: high
status: completed
tags: [obsidian, setup, plugins, dataview, workflow]
difficulty: beginner
estimated_hours: 2
---

# 🎨 Настройка Obsidian для максимальной продуктивности

> **Назначение:** Полное руководство по настройке Obsidian для работы с библиотекой подготовки к собеседованиям.

---

## 1. Рекомендуемые плагины

### Обязательные плагины (Community Plugins)

**1. Dataview** — запросы и динамические списки
```yaml
Установка: Settings → Community plugins → Browse → "Dataview"
Использование: Создание динамических списков тем, отслеживание прогресса
```

**Пример Dataview запроса для отслеживания изучения:**
```dataview
TABLE status, priority, completed_date
FROM "Собеседование"
WHERE contains(tags, "#тема")
SORT priority ASC
```

---

**2. Templater** — шаблоны для заметок
```yaml
Установка: Settings → Community plugins → Browse → "Templater"
Настройка: Создать папку Templates, указать в настройках
```

**Шаблон для изучения темы:**
```markdown
---
created: <% tp.date.now("YYYY-MM-DD") %>
status: in_progress
priority: high
completed_date:
tags: [тема, собеседование]
---

# <% tp.file.title %>

## 📚 Конспект

## 🔑 Ключевые моменты

## ❓ Вопросы для самопроверки

## 🔗 Связанные темы

## 📝 Заметки
```

---

**3. Advanced Tables** — улучшенные таблицы
```yaml
Установка: Settings → Community plugins → Browse → "Advanced Tables"
Функции: Авто-форматирование, навигация по таблицам
```

---

**4. Calendar** — календарь для ежедневных заметок
```yaml
Установка: Settings → Community plugins → Browse → "Calendar"
Использование: Трекинг изучения, планирование
```

---

**5. Kanban** — доски для управления прогрессом
```yaml
Установка: Settings → Community plugins → Browse → "Kanban"
Создание доски:
```

**Пример Kanban доски для подготовки:**
```markdown
---
kanban-plugin: basic
---

## 📚 К изучению
- [ ] Python Core
- [ ] Django ORM
- [ ] Event Loop

## 🔄 В процессе
- [ ] React Server Components

## ✅ Изучено
- [x] JavaScript Basics
- [x] Git Commands

## 🎯 Повторение
- [ ] SQL Индексы (через 1 неделю)
- [ ] Docker (через 2 недели)
```

---

**6. Spaced Repetition** — интервальное повторение
```yaml
Установка: Settings → Community plugins → Browse → "Spaced Repetition"
Использование: Карточки для запоминания концепций
```

**Пример карточек:**
```markdown
## Flashcards

Что такое GIL?
?
Global Interpreter Lock — мьютекс в CPython

Какие есть уровни изоляции в SQL?
?
READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, SERIALIZABLE
```

---

**7. Excalidraw** — диаграммы от руки
```yaml
Установка: Settings → Community plugins → Browse → "Excalidraw"
Использование: Рисование архитектурных схем
```

---

**8. QuickAdd** — быстрое добавление заметок
```yaml
Установка: Settings → Community plugins → Browse → "QuickAdd"
Настройка: Горячие клавиши для быстрого создания заметок
```

---

## 2. Темы оформления

### Рекомендуемые темы

**1. Minimal** (самая популярная)
```yaml
Установка: Settings → Appearance → Themes → Manage → Minimal
Особенности: Чистый дизайн, кастомизация через CSS
```

**2. Things**
```yaml
Установка: Settings → Appearance → Themes → Manage → Things
Особенности: Стиль Apple Notes, фокус на контенте
```

**3. Blue Topaz**
```yaml
Установка: Settings → Appearance → Themes → Manage → Blue Topaz
Особенности: Тёмная тема, много настроек
```

---

## 3. CSS сниппеты для кастомизации

### Расположение CSS сниппетов
```
Vault/.obsidian/snippets/custom.css
```

**Сниппет для цветных заголовков:**
```css
/* Цветные заголовки по уровням */
.markdown-preview-view h1 {
    color: #e91e63;
    border-bottom: 2px solid #e91e63;
}

.markdown-preview-view h2 {
    color: #9c27b0;
    border-bottom: 1px solid #9c27b0;
}

.markdown-preview-view h3 {
    color: #673ab7;
}

.markdown-preview-view h4 {
    color: #3f51b5;
}
```

---

**Сниппет для callout блоков:**
```css
/* Кастомные callout блоки */
.callout[data-callout="important"] {
    --callout-color: 255, 152, 0;
    --callout-icon: lucide-alert-circle;
}

.callout[data-callout="tip"] {
    --callout-color: 76, 175, 80;
    --callout-icon: lucide-lightbulb;
}

.callout[data-callout="warning"] {
    --callout-color: 244, 67, 54;
    --callout-icon: lucide-triangle-alert;
}
```

**Использование в заметках:**
```markdown
> [!important] Важно
> Это критичная информация

> [!tip] Совет
> Используйте этот подход

> [!warning] Предупреждение
> Опасность производительности
```

---

**Сниппет для улучшения таблиц:**
```css
/* Чередование цветов в таблицах */
.markdown-preview-view table tr:nth-child(even) {
    background-color: rgba(0, 0, 0, 0.05);
}

.markdown-preview-view table tr:hover {
    background-color: rgba(0, 0, 0, 0.1);
}

/* Фиксированная ширина колонок */
.markdown-preview-view table {
    table-layout: auto;
}
```

---

**Сниппет для прогресс-баров:**
```css
/* Прогресс бары для трекинга */
.progress-bar {
    background: linear-gradient(90deg, 
        #4caf50 0%, 
        #4caf50 var(--progress), 
        #e0e0e0 var(--progress), 
        #e0e0e0 100%);
    height: 8px;
    border-radius: 4px;
    margin: 10px 0;
}
```

---

## 4. Структура Vault

### Рекомендуемая структура
```
Собеседование/
├── 📁 00_Index/                    # Навигация
│   ├── 00_КАТАЛОГ_ТЕМ.md
│   └── Progress_Tracker.md
│
├── 📁 01_Python_Backend/
├── 📁 02_JavaScript_NodeJS/
├── 📁 03_React_Frontend/
├── 📁 04_Databases/
├── 📁 05_Infrastructure/
├── 📁 06_Soft_Skills/
│
├── 📁 07_Additional/
│   ├── 07_CLI_Cheat_Sheets.md
│   ├── Practice_Tasks.md
│   └── Self_Check_Questions.md
│
├── 📁 Templates/                   # Шаблоны
│   ├── Topic_Template.md
│   ├── Daily_Note.md
│   └── Review_Template.md
│
├── 📁 Daily_Notes/                 # Ежедневные заметки
│   ├── 2026-03.md
│   └── 2026-03-17.md
│
├── 📁 Kanban/                      # Доски
│   └── Preparation_Board.md
│
└── 📁 Attachments/                 # Вложения
    ├── Images/
    └── Diagrams/
```

---

## 5. Горячие клавиши

### Настройка горячих клавиш
```
Settings → Hotkeys
```

**Рекомендуемые настройки:**
```
Ctrl/Cmd + N          → Создать новую заметку
Ctrl/Cmd + O          → Быстрое открытие
Ctrl/Cmd + P          → Command palette
Ctrl/Cmd + E          → Переключение Edit/Preview
Ctrl/Cmd + K          → Вставить ссылку
Ctrl/Cmd + Shift + T  → Вставить шаблон
Ctrl/Cmd + Shift + K  → Создать карточку
Ctrl/Cmd + Shift + D  → Открыть Daily note
```

---

## 6. Workflow для подготовки

### Daily Routine
```
┌─────────────────────────────────────────────────────────┐
│  Утро                                                   │
├─────────────────────────────────────────────────────────┤
│  1. Открыть Daily Note (Ctrl+Shift+D)                  │
│  2. Записать план на день                               │
│  3. Выбрать тему для изучения из Kanban                 │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Во время изучения                                      │
├─────────────────────────────────────────────────────────┤
│  1. Открыть тему                                        │
│  2. Использовать шаблон Topic_Template                  │
│  3. Делать заметки в разделе 📝                         │
│  4. Создавать связи с другими темами                    │
│  5. Отмечать прогресс в Kanban                          │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│  Вечер                                                  │
├─────────────────────────────────────────────────────────┤
│  1. Обновить статус темы                                │
│  2. Создать flashcards для повторения                   │
│  3. Записать что узнал в Daily Note                     │
│  4. Запланировать повторение                            │
└─────────────────────────────────────────────────────────┘
```

---

## 7. Graph View настройки

### Оптимизация графа связей
```
Settings → Graph View
```

**Рекомендуемые настройки:**
```
• Local Backlinks: 2
• Local Forelinks: 2
• Depth: 1
• Show Arrows: true
• Show Attachments: false
```

**Фильтры для графа:**
```
path:Собеседование/01_Python_Backend
tag:#тема
-status:completed
```

---

## 8. Sync и Backup

### Obsidian Sync (платный)
```
• Официальный sync от Obsidian
• Шифрование end-to-end
• Версионирование (30 дней)
• Цена: $8-10/месяц
```

### Альтернативы

**Git (бесплатно):**
```bash
# Инициализация Git в vault
cd Vault
git init
git add .
git commit -m "Initial commit"

# Автоматический commit через плагин
Установить плагин: "Obsidian Git"
Настроить авто-commit каждые 10 минут
```

**iCloud/Google Drive/Dropbox:**
```
• Автоматическая синхронизация
• Нет версионирования (кроме iCloud)
• Бесплатно (с лимитами)
```

---

## 9. Мобильное приложение

### Настройка для iOS/Android
```
1. Установить Obsidian app
2. Подключить vault через sync
3. Включить offline access
4. Настроить quick capture
```

**Mobile плагины:**
```
• Obsidian Mobile (официальный)
• GoodNotes интеграция (iOS)
• Quick Add для мобильного захвата
```

---

## 10. Примеры использования

### Tracker прогресса подготовки

```dataviewjs
const topics = dv.pages('"Собеседование"')
    .where(p => p.tags && p.tags.includes('#тема'));

const priorityOrder = { 'high': 1, 'medium': 2, 'low': 3 };

dv.table(
    ["Тема", "Статус", "Приоритет", "Завершено"],
    topics.sort(p => priorityOrder[p.priority] || 999, 'asc').map(p => [
        p.file.link,
        p.status,
        p.priority,
        p.completed_date
    ])
);
```

### Календарь повторений

```dataview
CALENDAR completed_date
FROM "Собеседование"
WHERE status = "completed"
```

### Список тем для повторения

```dataview
LIST
FROM "Собеседование"
WHERE status = "completed" AND file.day < date(now) - dur("14 days")
SORT file.day ASC
```

---

## 🔗 Полезные ресурсы

- [Obsidian Help](https://help.obsidian.md/)
- [Obsidian Forum](https://forum.obsidian.md/)
- [Obsidian Showcase](https://forum.obsidian.md/c/showcase/9)
- [Awesome Obsidian](https://github.com/obsidianmd/awesome-obsidian)

---

*Руководство обновлено: 17 марта 2026*
