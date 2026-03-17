---
created: 2026-03-17
category: Django
type: flashcard
difficulty: beginner
frequency: medium
status: new
next_review: 
tags: [django, middleware, flashcard/django]
related: [[01_Python_Backend/02_Django_REST_API#3-Middleware-и-Безопасность]]
---

# Что такое Middleware в Django?

## Вопрос
Что такое Middleware в Django? Для чего используется?
?
## Ответ

**Middleware** — промежуточное ПО, которое перехватывает и обрабатывает:
- HTTP-запросы **до** попадания во View
- HTTP-ответы **после** выполнения View

**Примеры использования:**
- Аутентификация (AuthenticationMiddleware)
- Сессии (SessionMiddleware)
- CSRF защита (CsrfViewMiddleware)
- Логирование
- Кэширование

**Создание middleware:**
```python
class SimpleMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Код ДО view
        print("Before view")
        
        response = self.get_response(request)
        
        # Код ПОСЛЕ view
        print("After view")
        
        return response
```

**Порядок выполнения:**
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
]
# Выполняются сверху вниз для запроса
# Снизу вверх для ответа
```

---

#flashcard #django #middleware
