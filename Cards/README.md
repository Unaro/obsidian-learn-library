---
created: 2026-03-17
category: README
type: flashcard
difficulty: beginner
frequency: low
status: _
next_review:
tags:
  - readme
---

# 🎴 Система карточек для собеседований

> **Назначение:** Единая система вопросов и карточек для подготовки к собеседованиям с интервальным повторением.

---

## 📁 Структура

```
Cards/
├── README.md                    # Это руководство
├── 01_Python/                   # Карточки по Python
├── 02_JavaScript/               # Карточки по JavaScript
├── 03_React/                    # Карточки по React
├── 04_Databases/                # Карточки по БД
├── 05_Infrastructure/           # Карточки по инфраструктуре
└── 06_Soft_Skills/              # Карточки по soft skills
```

---

## 📝 Формат карточки

Каждая карточка — это вопрос с развёрнутым ответом:

```yaml
---
created: 2026-03-17
category: Python
type: flashcard
difficulty: beginner     # beginner, intermediate, advanced
frequency: high          # high, medium, low
status: new              # new, learning, learned
next_review: 2026-03-24  # Дата следующего повторения
tags: [python, flashcard]
related: [[связанный_файл]]
---

# Название вопроса

## Вопрос

Текст вопроса

---

## Ответ

Развёрнутый ответ с примерами кода и пояснениями.

**Важно для собеседования:**
- Ключевой момент 1
- Ключевой момент 2

---

#flashcard #тег
```

---

## 🔄 Рабочий процесс

### 1. Создание карточки

1. Используйте шаблон [[Templates/Flashcard_Template]]
2. Заполните вопрос и развёрнутый ответ
3. Укажите категорию, сложность, частоту
4. Добавьте теги и связи с другими темами

### 2. Изучение

1. Откройте [[101_Cards_Index]]
2. Найдите карточки со статусом `new`
3. Изучите вопрос и ответ
4. Измените статус на `learning`
5. Установите `next_review: 2026-03-24` (через 7 дней)

### 3. Повторение

1. Откройте раздел "Для повторения сегодня"
2. Повторите карточку
3. Если помните хорошо:
   - Увеличьте интервал (14 → 30 → 60 → 90 → 180 дней)
   - Обновите `next_review`
   - При достижении 180 дней измените статус на `learned`
4. Если забыли:
   - Сбросьте интервал на 7 дней
   - Оставьте статус `learning`

---

## 📊 Статусы карточек

| Статус | Значение | Интервалы |
|--------|----------|-----------|
| `new` | Новая карточка, ещё не изучалась | — |
| `learning` | Изучается, требует повторения | 7, 14, 30, 60, 90 дней |
| `learned` | Выучена, долгосрочное хранение | 180+ дней |

---

## 🏷️ Интервалы повторения

### Стандартная схема

| Повторение | Интервал | Статус |
|------------|----------|--------|
| 1 | 7 дней | learning |
| 2 | 14 дней | learning |
| 3 | 30 дней | learning |
| 4 | 60 дней | learning |
| 5 | 90 дней | learning |
| 6 | 180 дней | learned |

### Настройка интервалов

```yaml
# Лёгкие карточки (frequency: low)
next_review: date(now) + 14 days

# Средние карточки (frequency: medium)
next_review: date(now) + 7 days

# Частые карточки (frequency: high)
next_review: date(now) + 3 days
```

---

## 🔍 Поиск карточек

### Через Obsidian Search

```
# Новые карточки
status: new

# Для повторения сегодня
next_review <= date(now)

# Карточки по Python
category: Python

# Сложные карточки
difficulty: advanced

# Частые вопросы
frequency: high
```

### Через Dataview

**Новые карточки:**
```dataview
TABLE file.link, difficulty, frequency
FROM "Собеседование/Cards"
WHERE status = "new"
SORT frequency ASC
LIMIT 10
```

**Для повторения:**
```dataview
TABLE file.link, date(now) - date(next_review) as "Просрочено дней"
FROM "Собеседование/Cards"
WHERE status = "learning" AND next_review <= date(now)
SORT next_review ASC
```

**Выученные:**
```dataview
TABLE file.link, next_review, difficulty
FROM "Собеседование/Cards"
WHERE status = "learned"
SORT next_review ASC
LIMIT 10
```

---

## 📈 Прогресс по категориям

```dataview
TABLE 
    length(rows) as "Всего",
    length(filter(rows, (r) => r.status = "learned")) as "✅",
    length(filter(rows, (r) => r.status = "learning")) as "🔄",
    length(filter(rows, (r) => r.status = "new")) as "🆕"
FROM "Собеседование/Cards"
GROUP BY category
```

---

## 💡 Советы

1. **Создавайте 5-10 карточек в день** — не больше, чтобы не перегружаться
2. **Повторяйте ежедневно** — лучше по 15 минут каждый день, чем 2 часа раз в неделю
3. **Добавляйте примеры** — чем конкретнее пример, тем лучше запоминается
4. **Связывайте карточки** — добавляйте ссылки на связанные темы
5. **Используйте теги** — для быстрого поиска по темам
6. **Указывайте частоту** — приоритизируйте важные вопросы

---

## 🔗 Интеграция с плагинами

### Spaced Repetition плагин

Карточки совместимы с плагином **Spaced Repetition**:
- Тег `#flashcard` распознаётся плагином
- Формат вопроса и ответа через `---`
- Статусы синхронизируются

### Anki

Для экспорта в Anki:
1. Установите плагин **Obsidian_to_Anki**
2. Настройте формат экспорта
3. Экспортируйте карточки по тегу `#flashcard`

---

## 🔗 Связанные файлы

- [[README]] — 📚 Главная библиотека
- [[101_Cards_Index]] — Главная страница карточек
- [[Templates/Flashcard_Template]] — Шаблон карточки
- [[07_CLI_Cheat_Sheets]] — Шпаргалки CLI
- [[08_Dataview_Queries]] — Dataview запросы
- [[00_Dashboard]] — Dashboard подготовки
- [[00_КАТАЛОГ_ТЕМ]] — Каталог тем

---

*Руководство обновлено: 17 марта 2026*
