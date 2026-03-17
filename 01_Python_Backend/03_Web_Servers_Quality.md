---
created: 2026-03-17
updated: 2026-03-17
category: Python
type: topic
priority: medium
status: completed
tags: [django, wsgi, asgi, caching, websockets, tdd]
difficulty: advanced
estimated_hours: 6
---

# Web-серверы, Real-time и Качество кода

> **Назначение:** WSGI/ASGI, кэширование, WebSockets, Django Channels, методологии разработки. Включает продвинутые техники деплоя и оптимизации.

---

## 1. Кэширование в Django

### 1.1. Архитектура кэширования

```
┌─────────────────────────────────────────────────────────┐
│              Django Cache Framework                     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  View → Cache Middleware → Cache Backend → Storage      │
│           ↓                            ↑                │
│      Template Cache ← Low-level API ←──┘                │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### 1.2. Бэкенды кэширования

**Memcached (рекомендуемый):**
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',  # Или список серверов
        # 'LOCATION': [
        #     '192.168.1.10:11211',
        #     '192.168.1.11:11211',
        # ]
        'TIMEOUT': 300,  # 5 минут по умолчанию
        'OPTIONS': {
            'no_delay': True,
            'ignore_exc': True,
        }
    }
}

# pip install pymemcache
```

**Redis (универсальный):**
```python
# settings.py
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.redis.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        # 'redis://user:password@host:port/db'
        'TIMEOUT': 300,
    }
}

# pip install redis
```

**Сравнение бэкендов:**

| Бэкенд | Производительность | Персистентность | Масштабирование | Когда использовать |
|--------|-------------------|-----------------|-----------------|-------------------|
| **Memcached** | ⭐⭐⭐⭐⭐ | ❌ Нет | ⭐⭐⭐⭐⭐ | Простое кэширование |
| **Redis** | ⭐⭐⭐⭐ | ✅ Да | ⭐⭐⭐⭐ | Сложные структуры, очереди |
| **Database** | ⭐⭐ | ✅ Да | ⭐⭐ | Нет другого варианта |
| **Filesystem** | ⭐⭐ | ✅ Да | ⭐ | Разработка, тесты |
| **Local-memory** | ⭐⭐⭐ | ❌ Нет | ❌ | Разработка, однопоточный |

---

### 1.3. Уровни кэширования

**1. Кэширование всего сайта (Per-site cache):**
```python
# settings.py
MIDDLEWARE = [
    'django.middleware.cache.UpdateCacheMiddleware',  # ПЕРВЫМ
    'django.middleware.common.CommonMiddleware',
    'django.middleware.cache.FetchFromCacheMiddleware',  # ПОСЛЕДНИМ
]

CACHE_MIDDLEWARE_SECONDS = 600  # 10 минут
CACHE_MIDDLEWARE_KEY_PREFIX = 'mysite'
CACHE_MIDDLEWARE_ALIAS = 'default'
```

**2. Кэширование представлений (View cache):**
```python
from django.views.decorators.cache import cache_page

@cache_page(60 * 15)  # 15 минут
def my_view(request):
    # ...
    return response

# В URLconf
urlpatterns = [
    path('articles/<int:code>/', cache_page(60 * 15)(my_view)),
]
```

**3. Кэширование шаблонов (Template cache):**
```django
{% load cache %}

{% cache 500 sidebar request.user.username %}
    {% include "sidebar.html" %}
{% endcache %}

{% cache 600 article_block article.id language %}
    <h2>{{ article.title }}</h2>
    {{ article.content|truncatewords:100 }}
{% endcache %}
```

**4. Низкоуровневое API (Low-level cache API):**
```python
from django.core.cache import cache

# Установка
cache.set('my_key', 'hello, world!', 30)  # 30 секунд
cache.set_many({
    'a': 1,
    'b': 2,
    'c': 3
}, timeout=60)

# Получение
value = cache.get('my_key')
values = cache.get_many(['a', 'b', 'c'])

# Удаление
cache.delete('my_key')
cache.delete_many(['a', 'b', 'c'])

# Очистка всего кэша
cache.clear()

# Атомарные операции (только некоторые бэкенды)
cache.add('my_key', 'value', timeout=30)  # Только если ключа нет
cache.incr('counter')  # Увеличить на 1
cache.decr('counter')  # Уменьшить на 1
```

---

### 1.4. Кэширование ORM запросов

```python
from django.core.cache import cache
from django.db.models import QuerySet

def get_expensive_query():
    # Проверка кэша
    cache_key = 'expensive_query_result'
    result = cache.get(cache_key)
    
    if result is None:
        # Выполнение запроса
        result = list(
            Book.objects
            .select_related('author')
            .prefetch_related('reviews')
            .filter(status='published')
            .values('id', 'title', 'author__name')
        )
        # Сохранение в кэш
        cache.set(cache_key, result, 60 * 10)  # 10 минут
    
    return result

# Декоратор для кэширования
from functools import wraps

def cache_result(timeout=300):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{args}:{kwargs}"
            result = cache.get(cache_key)
            
            if result is None:
                result = func(*args, **kwargs)
                cache.set(cache_key, result, timeout)
            
            return result
        return wrapper
    return decorator

@cache_result(timeout=600)
def get_top_books(limit=10):
    return list(Book.objects.order_by('-sales')[:limit])
```

---

### 1.5. Инвалидация кэша

```python
from django.core.cache import cache
from django.db.models.signals import post_save, post_delete
from django.dispatch import receiver

# Версионирование ключей
def make_cache_key(base_key, version=None):
    if version is None:
        version = cache.get(f'{base_key}:version', 1)
    return f'{base_key}:v{version}'

def invalidate_cache(base_key):
    version = cache.get(f'{base_key}:version', 1)
    cache.set(f'{base_key}:version', version + 1)

# Автоматическая инвалидация при изменении модели
@receiver(post_save, sender=Book)
@receiver(post_delete, sender=Book)
def invalidate_book_cache(sender, **kwargs):
    cache.delete('top_books')
    cache.delete('new_books')
    invalidate_cache('book_list')

# Кэширование с зависимостями
class CacheWithDependencies:
    def __init__(self):
        self.cache = cache
    
    def get(self, key, default=None):
        return self.cache.get(key, default)
    
    def set(self, key, value, timeout=None, dependencies=None):
        if dependencies:
            for dep_key in dependencies:
                if self.cache.get(dep_key) is None:
                    return  # Зависимость устарела
        self.cache.set(key, value, timeout)
```

---

## 2. WSGI vs ASGI

### 2.1. WSGI (Web Server Gateway Interface)

**Архитектура:**
```
Client → Nginx → Gunicorn/uWSGI → Django (WSGI) → Response

Синхронная модель:
Request 1 → [==== Processing ====] → Response 1
Request 2 →          [wait]         → Response 2
Request 3 →            [wait]       → Response 3
```

**Конфигурация Gunicorn:**
```python
# gunicorn.conf.py
bind = '0.0.0.0:8000'
workers = 4  # (2 x CPU) + 1
worker_class = 'sync'  # или 'gevent', 'eventlet'
worker_connections = 1000
timeout = 30
keepalive = 2
accesslog = '/var/log/gunicorn/access.log'
errorlog = '/var/log/gunicorn/error.log'
loglevel = 'info'
```

**Запуск:**
```bash
gunicorn myproject.wsgi:application \
    --bind 0.0.0.0:8000 \
    --workers 4 \
    --worker-class sync \
    --timeout 30 \
    --access-logfile /var/log/gunicorn/access.log \
    --error-logfile /var/log/gunicorn/error.log
```

---

### 2.2. ASGI (Asynchronous Server Gateway Interface)

**Архитектура:**
```
Client → Nginx → Daphne/Uvicorn → Django (ASGI) → Response

Асинхронная модель:
Request 1 → [== Processing ==]  → Response 1
Request 2 → [==== Processing ====] → Response 2
Request 3 → [ Processing ] → Response 3
            (параллельно в одном потоке)
```

**asgi.py:**
```python
import os
from django.core.asgi import get_asgi_application
from channels.routing import ProtocolTypeRouter, URLRouter
from channels.auth import AuthMiddlewareStack
from channels.security.websocket import AllowedHostsOriginValidator

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'myproject.settings')

django_asgi_app = get_asgi_application()

from myapp.routing import websocket_urlpatterns

application = ProtocolTypeRouter({
    "http": django_asgi_app,
    "websocket": AllowedHostsOriginValidator(
        AuthMiddlewareStack(
            URLRouter(websocket_urlpatterns)
        )
    ),
})
```

**Запуск Uvicorn:**
```bash
uvicorn myproject.asgi:application \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --loop uvloop \
    --http h11
```

---

### 2.3. Сравнение WSGI и ASGI

| Характеристика | WSGI | ASGI |
|---------------|------|------|
| **Модель** | Синхронная | Асинхронная |
| **Протоколы** | HTTP/1.1 | HTTP/1.1, HTTP/2, WebSocket |
| **Производительность** | Хорошая для CPU | Лучшая для I/O |
| **WebSockets** | ❌ Нет | ✅ Да |
| **Server-Sent Events** | ❌ Нет | ✅ Да |
| **Серверы** | Gunicorn, uWSGI | Daphne, Uvicorn, Hypercorn |
| **Когда использовать** | Классические приложения | Real-time, long-polling |

---

## 3. WebSockets и Django Channels

### 3.1. Установка и настройка

```bash
pip install channels channels-redis daphne
```

**settings.py:**
```python
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'channels',
    'myapp',
]

ASGI_APPLICATION = 'myproject.asgi.application'

CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            'hosts': [('127.0.0.1', 6379)],
            'capacity': 1500,  # Максимум сообщений в канале
            'expiry': 10,  # Время жизни сообщения (сек)
        },
    },
}
```

---

### 3.2. Создание WebSocket потребителя

**consumers.py:**
```python
import json
from channels.generic.websocket import AsyncWebsocketConsumer
from channels.db import database_sync_to_async
from django.contrib.auth.models import User

class ChatConsumer(AsyncWebsocketConsumer):
    async def connect(self):
        self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_group_name = f'chat_{self.room_name}'
        
        # Присоединение к группе
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )
        
        await self.accept()
        
        # Отправка приветствия
        await self.send(text_data=json.dumps({
            'type': 'connection',
            'message': f'Connected to {self.room_name}'
        }))
    
    async def disconnect(self, close_code):
        # Отключение от группы
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
    
    async def receive(self, text_data):
        # Получение сообщения от клиента
        data = json.loads(text_data)
        message = data.get('message')
        
        # Сохранение в БД
        await self.save_message(message)
        
        # Отправка сообщения группе
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
                'user': self.scope['user'].username
            }
        )
    
    async def chat_message(self, event):
        # Получение сообщения от группы
        await self.send(text_data=json.dumps({
            'type': 'chat_message',
            'message': event['message'],
            'user': event['user']
        }))
    
    @database_sync_to_async
    def save_message(self, message):
        from myapp.models import Message
        Message.objects.create(
            room=self.room_name,
            user=self.scope['user'],
            content=message
        )
```

**routing.py:**
```python
from django.urls import re_path
from . import consumers

websocket_urlpatterns = [
    re_path(r'ws/chat/(?P<room_name>\w+)/$', consumers.ChatConsumer.as_asgi()),
]
```

**HTML клиент:**
```html
<script>
const roomName = "{{ room_name }}";
const chatSocket = new WebSocket(
    'ws://' + window.location.host + '/ws/chat/' + roomName + '/'
);

chatSocket.onopen = function(e) {
    console.log("Connected");
};

chatSocket.onmessage = function(e) {
    const data = JSON.parse(e.data);
    if (data.type === 'chat_message') {
        document.querySelector('#chat-log').innerHTML += (
            '<strong>' + data.user + ':</strong> ' + data.message + '<br>'
        );
    }
};

chatSocket.onclose = function(e) {
    console.log("Disconnected");
};

function sendMessage() {
    const messageInput = document.querySelector('#chat-message-input');
    chatSocket.send(JSON.stringify({
        'message': messageInput.value
    }));
    messageInput.value = '';
}
</script>
```

---

### 3.3. Фоновые задачи с Channels

**tasks.py:**
```python
from channels.layers import get_channel_layer
from asgiref.sync import async_to_sync

def send_notification(user_id, message):
    """Отправка уведомления пользователю"""
    channel_layer = get_channel_layer()
    
    async_to_sync(channel_layer.group_send)(
        f'user_{user_id}',
        {
            'type': 'notification',
            'message': message
        }
    )

def broadcast_update(event_type, data):
    """Рассылка обновлений всем подключенным клиентам"""
    channel_layer = get_channel_layer()
    
    async_to_sync(channel_layer.group_send)(
        'updates',
        {
            'type': 'update_event',
            'event_type': event_type,
            'data': data
        }
    )
```

---

## 4. Качество кода и методологии

### 4.1. PEP 8 — основные правила

```python
# Отступы: 4 пробела
def my_function():
    if True:
        print("Indented with 4 spaces")

# Длина строки: до 79 символов (до 99 для кода)
long_string = (
    "This is a very long string that exceeds the 79 character "
    "limit, so we break it into multiple lines"
)

# Пустые строки
# 2 пустые строки между функциями/классами
def function1():
    pass


def function2():
    pass


# 1 пустая строка между методами класса
class MyClass:
    def method1(self):
        pass
    
    def method2(self):
        pass

# Импорты в начале файла
import os
import sys
from datetime import datetime

from django.db import models
from rest_framework import serializers

from .models import MyModel
from .utils import helper_function

# Именование
variable_name = "snake_case"  # Переменные, функции
CONSTANT_NAME = "UPPER_CASE"  # Константы
ClassName = "PascalCase"      # Классы
_private_var = "leading underscore"  # Protected
__private = "name mangling"   # Private
```

---

### 4.2. Принципы чистого кода

**1.单一 ответственность (SRP):**
```python
# ❌ ПЛОХО
class UserService:
    def create_user(self, data):
        # Валидация
        # Создание пользователя
        # Отправка email
        # Логирование
        # Всё в одном методе!
        pass

# ✅ ХОРОШО
class UserService:
    def __init__(self, email_service, logger):
        self.email_service = email_service
        self.logger = logger
    
    def create_user(self, data):
        user = self._create_user_in_db(data)
        self.email_service.send_welcome_email(user)
        self.logger.log_user_creation(user)
        return user
    
    def _create_user_in_db(self, data):
        # Только создание в БД
        pass
```

**2. DRY (Don't Repeat Yourself):**
```python
# ❌ ПЛОХО
def get_active_users():
    return User.objects.filter(is_active=True)

def get_active_products():
    return Product.objects.filter(is_active=True)

def get_active_orders():
    return Order.objects.filter(is_active=True)

# ✅ ХОРОШО
class IsActiveManager(models.Manager):
    def get_queryset(self):
        return super().get_queryset().filter(is_active=True)

class User(models.Model):
    objects = IsActiveManager()
```

**3. KISS (Keep It Simple, Stupid):**
```python
# ❌ СЛИШКОМ СЛОЖНО
result = list(filter(lambda x: x > 0, map(lambda x: x * 2, numbers)))

# ✅ ПРОСТО И ПОНЯТНО
result = [n * 2 for n in numbers if n > 0]
```

---

### 4.3. TDD (Test-Driven Development)

**Цикл Red-Green-Refactor:**

```python
# 1. RED — Пишем тест (падает)
# tests.py
from django.test import TestCase
from myapp.models import Book

class BookTestCase(TestCase):
    def test_book_str(self):
        book = Book(title="Test Book", author="John Doe")
        self.assertEqual(str(book), "Test Book by John Doe")

# 2. GREEN — Пишем минимальный код (проходит)
# models.py
class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    
    def __str__(self):
        return f"{self.title} by {self.author}"

# 3. REFACTOR — Улучшаем код
class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    
    def __str__(self):
        return self.title
    
    @property
    def full_title(self):
        return f"{self.title} by {self.author.name}"
```

**Запуск тестов:**
```bash
# Запуск всех тестов
python manage.py test

# Запуск конкретного приложения
python manage.py test myapp

# Запуск конкретного теста
python manage.py test myapp.tests.BookTestCase.test_book_str

# С покрытием
coverage run --source='.' manage.py test
coverage report
coverage html  # Отчёт в HTML
```

---

### 4.4. Инструменты качества кода

**Black (автоформатирование):**
```bash
pip install black
black myproject/
```

**Flake8 (линтер):**
```bash
pip install flake8
flake8 myproject/

# .flake8
[flake8]
max-line-length = 99
exclude = .git,__pycache__,migrations
ignore = E203,W503
```

**MyPy (статическая типизация):**
```python
from typing import List, Optional

def greet(name: str, age: Optional[int] = None) -> str:
    if age is None:
        return f"Hello, {name}!"
    return f"Hello, {name}! You are {age} years old."

def process_items(items: List[str]) -> List[str]:
    return [item.upper() for item in items]
```

```bash
pip install mypy
mypy myproject/
```

**isort (сортировка импортов):**
```bash
pip install isort
isort myproject/
```

---

## 🔗 Связанные темы

- [[01_Python_Backend/02_Django_REST_API]] — Django ORM
- [[05_Infrastructure/01_Infrastructure_Networks_Docker]] — Docker, деплой
- [[02_JavaScript_NodeJS/03_NodeJS_Streams_Network]] — WebSockets для сравнения

---

*Файл обновлён: 17 марта 2026*
