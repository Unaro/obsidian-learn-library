---
created: 2026-03-17
category: Databases
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [sql, acid, transactions, flashcard/databases]
related: [[04_Databases/01_Databases_ORM_SQL#2-ACID-Свойства-транзакций]]
---

# Что такое ACID?

## Вопрос
Что означают свойства ACID в контексте транзакций БД?
?
## Ответ

**ACID** — свойства, гарантирующие надёжность транзакций в БД:

**A**tomicity (Атомарность):
- Транзакция неделима: либо все операции выполняются, либо ни одна
- Пример: перевод денег — списание и зачисление должны выполниться вместе

**C**onsistency (Согласованность):
- Транзакция переводит БД из одного валидного состояния в другое
- Все ограничения (constraints) сохраняются

**I**solation (Изолированность):
- Параллельные транзакции не влияют друг на друга
- Достигается уровнями изоляции (READ COMMITTED, SERIALIZABLE)

**D**urability (Устойчивость):
- Зафиксированные изменения сохраняются навсегда
- Даже при сбое питания (запись в WAL, checkpoint)

**Пример:**
```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- Или ROLLBACK при ошибке
```

---

#flashcard #sql #acid #transactions
