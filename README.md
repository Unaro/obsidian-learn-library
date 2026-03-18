# 📚 Библиотека для подготовки к собеседованиям

> **Назначение:** Полное руководство для подготовки к собеседованиям на Fullstack разработчика (Python/JavaScript)

---

## 📊 О проекте

**Библиотека содержит:**
- **49 карточек** для интервального повторения
- **20 тем** для углублённого изучения
- **50+ вопросов** из реальной практики собеседований
- **Dataview запросы** для отслеживания прогресса
- **CLI шпаргалки** по Linux, Git, Docker

**Категории:**
- 🐍 Python & Django
- ⚡ JavaScript & Node.js
- ⚛️ React & Next.js
- 🗄️ Базы Данных & SQL
- 🌐 Инфраструктура & DevOps
- 🤝 Soft Skills

---

## 🚀 Быстрый старт

### 1. Откройте Dashboard

Начните с [[00_Dashboard]] — центральный экран с прогрессом и навигацией.

### 2. Изучайте темы

Используйте [[00_КАТАЛОГ_ТЕМ]] для навигации по темам.

### 3. Повторяйте по карточкам

Откройте [[101_Cards_Index]] — база карточек для интервального повторения.

---

## 📁 Структура

```
obsidian-learn-library/
│
├── 📄 00_Dashboard.md              # Главный экран
├── 📄 00_КАТАЛОГ_ТЕМ.md            # Каталог тем
├── 📄 101_Cards_Index.md           # База карточек
├── 📄 07_CLI_Cheat_Sheets.md       # CLI шпаргалки
├── 📄 08_Dataview_Queries.md       # Dataview запросы
│
├── 📁 Cards/                       # Карточки (49)
│   ├── 01_Python/                 # Python (6)
│   ├── 01_Django/                 # Django (5)
│   ├── 02_JavaScript/             # JavaScript (5)
│   ├── 02_NodeJS/                 # Node.js (4)
│   ├── 03_React/                  # React (6)
│   ├── 03_NextJS/                 # Next.js (2)
│   ├── 04_TypeScript/             # TypeScript (3)
│   ├── 04_Databases/              # Базы Данных (6)
│   ├── 05_Infrastructure/         # Инфраструктура (7)
│   ├── 05_Testing/                # Тестирование (1)
│   ├── 05_SystemDesign/           # System Design (2)
│   └── 06_Soft_Skills/            # Soft Skills (5)
│
├── 📁 01_Python_Backend/          # Темы Python
├── 📁 02_JavaScript_NodeJS/       # Темы JavaScript
├── 📁 03_React_Frontend/          # Темы React
├── 📁 04_Databases/               # Темы БД
├── 📁 05_Infrastructure/          # Темы Инфраструктура
├── 📁 06_Soft_Skills/             # Темы Soft Skills
│
└── 📁 Templates/                  # Шаблоны
    └── Flashcard_Template.md      # Шаблон карточки
```

---

## 🎴 Система карточек

### Формат карточки

```yaml
---
created: 2026-03-17
category: Python
type: flashcard
difficulty: beginner
frequency: high
status: new
next_review: 
tags: [python, types, flashcard/python]
---

# Заголовок

## Вопрос
Текст вопроса?
## Ответ

Текст ответа
```

### Интервалы повторения

| Повторение | Интервал |
|------------|----------|
| 1 | 7 дней |
| 2 | 14 дней |
| 3 | 30 дней |
| 4 | 60 дней |
| 5 | 90 дней |
| 6 | 180 дней |

### Использование с плагином Spaced Repetition

1. Установите плагин **Spaced Repetition**
2. Откройте [[101_Cards_Index]]
3. Выберите карточки для повторения
4. Отмечайте сложность (Easy/Medium/Hard)
5. Плагин автоматически назначит следующее повторение

---

## 📈 Отслеживание прогресса

### Dataview запросы

Используйте [[08_Dataview_Queries]] для создания отчётов:

```dataview
TABLE 
    category as "Категория",
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "learned")) as "✅"
FROM "obsidian-learn-library/Cards"
GROUP BY category
```

### Dashboard

[[00_Dashboard]] автоматически показывает:
- Общий прогресс
- Прогресс по категориям
- Карточки для повторения сегодня
- Последние завершённые темы

---

## 🔧 Настройка Obsidian

### Рекомендуемые плагины

**Обязательные:**
- **Dataview** — запросы и отчёты
- **Spaced Repetition** — интервальное повторение
- **Templater** — шаблоны

**Опциональные:**
- **Kanban** — доски для планирования
- **Calendar** — календарь для ежедневных заметок
- **Excalidraw** — рисование диаграмм

### Настройка плагинов

См. раздел [[README#Настройка-плагинов|Настройка]] ниже.

---

## 📚 Категории

### 🐍 Python & Backend

**Темы:**
- [[01_Python_Backend/01_Python_Core|Python Core]] — типы данных, ООП, GIL
- [[01_Python_Backend/02_Django_REST_API|Django & DRF]] — ORM, сериализаторы
- [[01_Python_Backend/03_Web_Servers_Quality|Web-серверы]] — WSGI/ASGI, кэширование

**Карточки:** 11 (6 Python + 5 Django)

---

### ⚡ JavaScript & Node.js

**Темы:**
- [[02_JavaScript_NodeJS/01_JavaScript_Core_EventLoop|JavaScript Core]] — Event Loop, замыкания
- [[02_JavaScript_NodeJS/02_Advanced_JS_LiveCoding|Advanced JS]] — debounce, throttle, Promise
- [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network|Node.js Streams]] — потоки, WebSockets
- [[02_JavaScript_NodeJS/04_NodeJS_Threads_Queues|Queues & Microservices]] — BullMQ, Kafka

**Карточки:** 9 (5 JavaScript + 4 Node.js)

---

### ⚛️ React & Frontend

**Темы:**
- [[03_React_Frontend/01_React_Architecture_FSD|React Architecture]] — FSD, RSC, Zustand
- [[03_React_Frontend/02_React_Internals_Optimization|React Internals]] — Fiber, Virtual DOM
- [[03_React_Frontend/03_NextJS_App_Router_React19|Next.js]] — кэширование, Server Actions
- [[03_React_Frontend/04_NextJS_RSC|RSC vs SSR]] — гидратация, паттерны

**Карточки:** 8 (6 React + 2 Next.js)

---

### 🗄️ Базы Данных

**Темы:**
- [[04_Databases/01_Databases_ORM_SQL|SQL & ORM]] — ACID, индексы, JOIN
- [[04_Databases/02_Database_Optimization_N1|Optimization]] — N+1, B-Tree
- [[04_Databases/03_Advanced_Database_Offline|Advanced]] — JSONB, CTE, оффлайн

**Карточки:** 6

---

### 🌐 Инфраструктура

**Темы:**
- [[05_Infrastructure/01_Infrastructure_Networks_Docker|Docker & Networks]] — контейнеры, HTTP
- [[05_Infrastructure/02_System_Analysis_BPMN_UML|System Analysis]] — BPMN, IDEF0, UML
- [[05_Infrastructure/03_TypeScript_Security_Testing|TypeScript & Security]] — OWASP, тесты

**Карточки:** 10 (7 Infrastructure + 1 Testing + 2 SystemDesign)

---

### 🤝 Soft Skills

**Темы:**
- [[06_Soft_Skills/01_Behavioral_Interviews|Behavioral Interviews]] — STAR метод
- [[06_Soft_Skills/02_Time_Management_Productivity|Productivity]] — тайм-менеджмент
- [[06_Soft_Skills/03_Team_Leadership|Leadership]] — code review, конфликты

**Карточки:** 5

---

## 📝 Планы подготовки

### Неделя 1-2: Python Core
- [ ] [[Cards/01_Python/001_mutable_immutable|Мутабельность типов]]
- [ ] [[Cards/01_Python/002_list_tuple|List vs Tuple]]
- [ ] [[Cards/01_Python/003_gil|GIL]]
- [ ] [[Cards/01_Python/004_decorators|Декораторы]]
- [ ] [[Cards/01_Python/005_args_kwargs|*args/**kwargs]]
- [ ] [[Cards/01_Python/006_is_vs_equals|is vs ==]]

### Неделя 3-4: JavaScript
- [ ] [[Cards/02_JavaScript/001_event_loop|Event Loop]]
- [ ] [[Cards/02_JavaScript/002_closure|Замыкания]]
- [ ] [[Cards/02_JavaScript/003_var_let_const|var/let/const]]
- [ ] [[Cards/02_JavaScript/004_hoisting|Hoisting]]
- [ ] [[Cards/02_JavaScript/005_arrow_functions|Стрелочные функции]]

### Неделя 5-6: React
- [ ] [[Cards/03_React/001_rsc_ssr|RSC vs SSR]]
- [ ] [[Cards/03_React/002_virtual_vs_shadow_dom|Virtual vs Shadow DOM]]
- [ ] [[Cards/03_React/003_memo_hooks|useMemo vs useCallback]]
- [ ] [[Cards/03_React/004_controlled_uncontrolled|Controlled vs Uncontrolled]]
- [ ] [[Cards/03_React/005_keys|Keys в списках]]

---

## 🔗 Полезные ссылки

- [Obsidian Help](https://help.obsidian.md/)
- [Dataview Documentation](https://blacksmithgu.github.io/obsidian-dataview/)
- [Spaced Repetition Plugin](https://github.com/st3v3nmw/obsidian-spaced-repetition)
- [Templater Plugin](https://silentvoid13.github.io/Templater/)

---

*Последнее обновление: 17 марта 2026*
