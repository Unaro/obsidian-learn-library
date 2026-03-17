---
created: 2026-03-17
category: Django
type: flashcard
difficulty: beginner
frequency: low
status: new
next_review: 
tags: [django, fbv, cbv, views, flashcard/django]
related: [[01_Python_Backend/02_Django_REST_API#4-Представления-Views]]
---

# FBV vs CBV в Django

## Вопрос
В чём разница между Function-Based Views и Class-Based Views в Django?
?
## Ответ

**FBV (Function-Based Views):**
- Обычные функции Python
- Принимают `request`, возвращают `HttpResponse`
- Простые для понимания
- Легко читать линейный код

```python
def book_list(request):
    books = Book.objects.all()
    return render(request, 'books.html', {'books': books})
```

**CBV (Class-Based Views):**
- Классы с методами
- Используют ООП (наследование, миксины)
- Методы `get()`, `post()` автоматически маршрутизируются
- Переиспользование кода через наследование

```python
from django.views import View

class BookView(View):
    def get(self, request):
        books = Book.objects.all()
        return render(request, 'books.html', {'books': books})
    
    def post(self, request):
        # Обработка POST
        pass
```

**Когда что использовать:**
- **FBV** — простая логика, уникальное поведение
- **CBV** — CRUD операции, переиспользование логики, Generic Views

---

#flashcard #django #fbv #cbv #views
