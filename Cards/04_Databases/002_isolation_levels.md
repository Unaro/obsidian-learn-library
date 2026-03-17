---
created: 2026-03-17
category: Databases
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [sql, isolation-levels, transactions, flashcard/databases]
related: [[04_Databases/01_Databases_ORM_SQL#2-Уровни-изоляции-транзакций]]
---

# Уровни изоляции транзакций

## Вопрос
Какие существуют уровни изоляции транзакций в SQL?
?
## Ответ

**4 уровня изоляции (по возрастанию):**

**1. READ UNCOMMITTED:**
- Можно читать незафиксированные данные других транзакций
- **Грязное чтение** возможно
- Максимальная производительность

**2. READ COMMITTED (по умолчанию в PostgreSQL):**
- Чтение только зафиксированных данных
- **Грязное чтение** предотвращено
- **Non-repeatable read** возможно

**3. REPEATABLE READ:**
- Данные не меняются в процессе транзакции
- **Non-repeatable read** предотвращено
- **Phantom read** возможно

**4. SERIALIZABLE:**
- Полная эмуляция последовательного выполнения
- Все аномалии предотвращены
- Минимальная производительность

**Аномалии:**
| Уровень | Dirty Read | Non-Repeatable | Phantom |
|---------|-----------|----------------|---------|
| READ UNCOMMITTED | ✅ | ✅ | ✅ |
| READ COMMITTED | ❌ | ✅ | ✅ |
| REPEATABLE READ | ❌ | ❌ | ✅ |
| SERIALIZABLE | ❌ | ❌ | ❌ |

---

#flashcard #sql #isolation #transactions
