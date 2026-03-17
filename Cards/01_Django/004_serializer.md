---
created: 2026-03-17
category: Django
type: flashcard
difficulty: intermediate
frequency: medium
status: new
next_review: 
tags: [django, rest, serializer, flashcard/django]
related: [[01_Python_Backend/02_Django_REST_API#5-Django-Rest-Framework-DRF]]
---

# Что такое Serializer в DRF?

## Вопрос
Что такое Serializer в Django REST Framework? Для чего используется?
?
## Ответ

**Serializer** — класс для преобразования сложных типов данных (моделей Django, QuerySet) в форматы типа JSON и обратно.

**Основные функции:**
1. **Сериализация** — модель → JSON
2. **Десериализация** — JSON → модель
3. **Валидация** — проверка данных

**Пример:**
```python
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'price']
        read_only_fields = ['id']
    
    # Кастомная валидация
    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price cannot be negative")
        return value
```

**Использование:**
```python
# Сериализация
book = Book.objects.get(id=1)
serializer = BookSerializer(book)
json_data = serializer.data  # {'id': 1, 'title': '...'}

# Десериализация
serializer = BookSerializer(data=request.data)
if serializer.is_valid():
    serializer.save()  # Сохранение в БД
```

---

#flashcard #django #rest #serializer
