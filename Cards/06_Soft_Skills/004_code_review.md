---
created: 2026-03-17
category: SoftSkills
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [soft-skills, code-review, flashcard/softskills]
related: [[06_Soft_Skills/03_Team_Leadership#1-Code-Review]]
---

# Code Review best practices

## Вопрос
Как проводить эффективный Code Review?
?
## Ответ

**Чек-лист ревьюера:**

**Перед review:**
- □ Понимаю контекст (ticket, description)
- □ Проверяю размер PR (< 400 строк)
- □ Выделяю 30-60 минут без прерываний

**Технический чек-лист:**
- □ Код решает задачу
- □ Нет дублирования (DRY)
- □ Логирование на месте
- □ Обработка ошибок
- □ Тесты добавлены
- □ Документация обновлена

**Читаемость:**
- □ Имена понятны
- □ Функции делают одно дело
- □ Комментарии объясняют "почему"

**SLA для Code Review:**
- Первый ответ: 24 часа
- Финал: 48 часов

**Правила комментариев:**
- ✅ "Что думал об использовании...?"
- ✅ "Есть альтернативный подход..."
- ❌ "Почему ты сделал..."
- ❌ "Это неправильно"

**Блокирующие vs Неблокирующие:**
- 🔴 BLOCKING: баги, security, архитектура
- 🟡 NON-BLOCKING: нейминг, рефакторинг

---

#flashcard #soft-skills #code-review #best-practices
