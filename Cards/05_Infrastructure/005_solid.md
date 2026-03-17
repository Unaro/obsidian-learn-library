---
created: 2026-03-17
category: Infrastructure
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [solid, oop, principles, flashcard/infrastructure]
related: [[05_Infrastructure/01_Infrastructure_Networks_Docker#6-SOLID-Принципы]]
---

# SOLID принципы

## Вопрос
Что такое SOLID принципы в ООП?
?
## Ответ

**SOLID** — 5 принципов объектно-ориентированного программирования:

**S — Single Responsibility Principle:**
- Класс имеет только одну причину для изменения
- Одна зона ответственности

**O — Open/Closed Principle:**
- Открыт для расширения (наследование)
- Закрыт для модификации (не меняем исходный код)

**L — Liskov Substitution Principle:**
- Подклассы могут заменять суперклассы
- Без поломки логики

**I — Interface Segregation Principle:**
- Много узких интерфейсов лучше одного большого
- Не заставлять реализовывать неиспользуемые методы

**D — Dependency Inversion Principle:**
- Зависимость от абстракций (интерфейсов)
- Не от конкретных реализаций

**Пример (SRP):**
```python
# ❌ Нарушение
class UserService:
    def create(self): ...      # Создание
    def send_email(self): ...  # Email
    def log(self): ...         # Логирование

# ✅ Применение
class UserService:
    def __init__(self, email_service, logger): ...
    def create(self): ...  # Только создание
```

---

#flashcard #solid #oop #principles #architecture
