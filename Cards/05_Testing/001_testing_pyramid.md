---
created: 2026-03-17
category: Testing
type: flashcard
difficulty: beginner
frequency: high
status: new
next_review: 
tags: [testing, pyramid, unit, integration, e2e, flashcard/testing]
related: [[05_Infrastructure/03_TypeScript_Security_Testing#3-Пирамида-тестирования]]
---

# Пирамида тестирования

## Вопрос
Что такое пирамида тестирования? Какие уровни тестов существуют?
?
## Ответ

**Пирамида тестирования** — модель распределения тестов по уровням:

```
       /‾‾‾‾‾\
      /  E2E   \   — 10% (медленные, дорогие)
     /__________\
    / Integration \ — 20%
   /_______________\
  /  Unit Tests     \ — 70% (быстрые, дешёвые)
 /___________________\
```

**1. Unit тесты (основание):**
- Тестирование изолированных функций
- Самые быстрые и массовые
- Инструменты: Jest, Vitest, PyTest

**2. Integration тесты:**
- Взаимодействие между модулями
- Компонент + стейт-менеджер
- API контроллер + БД

**3. E2E тесты (вершина):**
- Сквозное тестирование в браузере
- Полный пользовательский сценарий
- Самые медленные и дорогие
- Инструменты: Cypress, Playwright

**Почему пирамида:**
- Unit: быстро дёшево, много
- E2E: медленно, дорого, мало

**Распределение:**
- 70% Unit
- 20% Integration
- 10% E2E

---

#flashcard #testing #pyramid #unit #integration #e2e
