---
created: 2026-03-17
category: Django
type: flashcard
difficulty: intermediate
frequency: high
status: new
next_review: 
tags: [django, orm, n-plus-one, optimization, flashcard/django]
related: [[01_Python_Backend/02_Django_REST_API#2-2-Оптимизация-Проблема-N1]]
---

# Как оптимизировать N+1 запрос в Django?

## Вопрос
Что такое проблема N+1 в Django ORM? Как её решить?
?
## Ответ

**Проблема N+1:**
- 1 запрос на получение списка объектов
- N запросов в цикле для связанных данных
- Итого: N+1 запрос вместо 1

**❌ Пример проблемы:**
```python
books = Book.objects.all()  # 1 запрос
for book in books:
    print(book.author.name)  # N запросов!
```

**✅ Решение 1: select_related()**
- Для ForeignKey и OneToOneField
- Делает JOIN на уровне SQL
```python
books = Book.objects.select_related('author').all()
# 1 запрос с JOIN
```

**✅ Решение 2: prefetch_related()**
- Для ManyToManyField и обратных ForeignKey
- Делает раздельные запросы, связывает в Python
```python
authors = Author.objects.prefetch_related('books').all()
# 2 запроса (к authors и books)
```

**Сравнение:**
| Метод | Тип связи | SQL | Когда |
|-------|-----------|-----|-------|
| select_related | ForeignKey, OneToOne | JOIN | Связь "вперёд" |
| prefetch_related | ManyToMany, обратная | Отдельные | Связь "назад" |

---

#flashcard #django #orm #optimization #n-plus-one
