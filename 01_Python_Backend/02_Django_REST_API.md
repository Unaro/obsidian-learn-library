# Django & REST API: Полное руководство

> **Назначение:** Архитектура Django, ORM, оптимизация запросов, Django REST Framework. Включает продвинутые техники и лучшие практики.

---

## 1. Архитектура Django: Паттерн MVT

### 1.1. MVT vs MVC

```
┌─────────────────────────────────────────────────┐
│              Django MVT Pattern                 │
├─────────────────────────────────────────────────┤
│                                                 │
│  Client → URLs → View → Model → Database        │
│              ↓                                  │
│          Template → HTML → Client               │
│                                                 │
└─────────────────────────────────────────────────┘

MVC (классический):
- Model — данные
- View — отображение (UI)
- Controller — логика

Django MVT:
- Model — данные
- View — логика (аналог Controller)
- Template — отображение (аналог View)
```

### 1.2. Жизненный цикл запроса в Django

```
1. HTTP-запрос → Web Server (Nginx/Apache)
2. WSGI/ASGI → Gunicorn/uWSGI
3. Django Middleware (обработка запроса)
4. URL Router (urls.py)
5. View (обработка логики)
6. Model (работа с БД)
7. Template (рендеринг HTML)
8. Middleware (обработка ответа)
9. HTTP-ответ клиенту
```

---

## 2. Django ORM и оптимизация запросов

### 2.1. Типы связей (Relationships)

**OneToOneField:**
```python
class User(models.Model):
    username = models.CharField(max_length=100)

class Profile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    bio = models.TextField()
    avatar = models.ImageField(upload_to='avatars/')

# Использование
profile = Profile.objects.get(user_id=1)
print(profile.user.username)  # Доступ к пользователю

user = User.objects.get(id=1)
print(user.profile.bio)  # Обратная связь
```

**ForeignKey (один ко многим):**
```python
class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')
    
    class Meta:
        indexes = [
            models.Index(fields=['author', '-published_date']),
        ]

# Использование
book = Book.objects.get(id=1)
print(book.author.name)  # Прямая связь

author = Author.objects.get(id=1)
print(author.books.all())  # Обратная связь через related_name
```

**ManyToManyField (многие ко многим):**
```python
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField('Course', related_name='students')

class Course(models.Model):
    title = models.CharField(max_length=200)
    credits = models.IntegerField()

# Промежуточная таблица (для дополнительных полей)
class Enrollment(models.Model):
    student = models.ForeignKey(Student, on_delete=models.CASCADE)
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
    enrolled_date = models.DateField(auto_now_add=True)
    grade = models.CharField(max_length=2, blank=True)
    
    class Meta:
        unique_together = ['student', 'course']
```

---

### 2.2. Проблема N+1 и оптимизация

**❌ Проблема N+1:**
```python
# Запрос 1: Получаем все книги
books = Book.objects.all()

# Запросы N: Для каждой книги получаем автора
for book in books:
    print(book.author.name)  # N дополнительных запросов!

# Итого: 1 + N запросов к БД
```

**✅ Решение: select_related()**
```python
# Для ForeignKey и OneToOneField
books = Book.objects.select_related('author').all()

for book in books:
    print(book.author.name)  # Без дополнительных запросов!

# Итого: 1 запрос с JOIN
# SQL: SELECT * FROM book INNER JOIN author ON book.author_id = author.id
```

**✅ Решение: prefetch_related()**
```python
# Для ManyToManyField и обратных ForeignKey
authors = Author.objects.prefetch_related('books').all()

for author in authors:
    for book in author.books.all():  # Без дополнительных запросов!
        print(book.title)

# Итого: 2 запроса (отдельно к каждой таблице)
# QuerySet prefetch_related() выполняет отдельный запрос и связывает в Python
```

**Сравнение:**

| Метод | Тип связи | SQL | Когда использовать |
|-------|-----------|-----|-------------------|
| `select_related()` | ForeignKey, OneToOne | JOIN | Связь "вперёд" |
| `prefetch_related()` | ManyToMany, обратная ForeignKey | Отдельные запросы | Связь "назад" или M2M |

**Глубокая оптимизация:**
```python
# Цепочка связей
books = Book.objects.select_related(
    'author',
    'publisher'
).prefetch_related(
    'categories',
    'reviews__user'  # Глубокая prefetch
).all()

# Только нужные поля (экономия памяти)
books = Book.objects.select_related('author').only(
    'title', 
    'author__name',
    'price'
)
```

---

### 2.3. Q-объекты и сложные запросы

```python
from django.db.models import Q, F, Count, Avg

# Логические операторы
Book.objects.filter(
    Q(price__lt=100) | Q(discount__gt=0)  # ИЛИ
)

Book.objects.filter(
    Q(price__lt=100) & Q(author__name='John')  # И
)

Book.objects.filter(
    ~Q(status='draft')  # НЕ
)

# F-объекты (ссылка на поле)
Book.objects.filter(
    price__gt=F('discount')  # price > discount
)

# Агрегация и аннотация
from django.db.models import Sum, Avg, Count

# Аннотация (добавляет поле к каждому объекту)
authors = Author.objects.annotate(
    book_count=Count('book'),
    avg_price=Avg('book__price')
).filter(book_count__gt=5)

# Агрегация (возвращает одно значение)
result = Book.objects.aggregate(
    total_sales=Sum('sales'),
    avg_rating=Avg('rating')
)
```

---

### 2.4. Менеджеры и QuerySet

**Кастомный менеджер:**
```python
class BookManager(models.Manager):
    def published(self):
        return self.filter(status='published')
    
    def available(self):
        return self.filter(stock__gt=0, status='published')
    
    def bestsellers(self, min_sales=1000):
        return self.filter(sales__gte=min_sales)

class Book(models.Model):
    title = models.CharField(max_length=200)
    status = models.CharField(max_length=20)
    stock = models.IntegerField(default=0)
    sales = models.IntegerField(default=0)
    
    objects = BookManager()  # Кастомный менеджер

# Использование
Book.objects.published()
Book.objects.available()
Book.objects.bestsellers()
```

---

## 3. Middleware и безопасность

### 3.1. Создание middleware

```python
# middleware.py
import time
from django.utils.deprecation import MiddlewareMixin

class TimingMiddleware(MiddlewareMixin):
    def process_request(self, request):
        request.start_time = time.time()
    
    def process_response(self, request, response):
        elapsed = time.time() - request.start_time
        response['X-Response-Time'] = f"{elapsed:.3f}s"
        return response
    
    def process_exception(self, request, exception):
        # Вызывается при ошибке в view
        print(f"Error: {exception}")
        return None  # Продолжить обработку ошибки

class AuthenticationMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        token = request.headers.get('Authorization')
        if token and self.validate_token(token):
            request.user = self.get_user_from_token(token)
        else:
            request.user = None
        
        response = self.get_response(request)
        return response
    
    def validate_token(self, token):
        # Логика валидации
        return True
    
    def get_user_from_token(self, token):
        # Получение пользователя
        return None
```

**Порядок выполнения middleware:**
```python
# settings.py
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'myapp.middleware.TimingMiddleware',  # Выполняется первым для request
]
```

---

### 3.2. CSRF защита

**Как работает:**
```
1. Сервер генерирует CSRF-токен
2. Токен сохраняется в cookie и сессии
3. Токен добавляется в формы:
   <form method="post">
     {% csrf_token %}
     <input type="text" name="username">
   </form>
4. При POST запросе middleware сравнивает токены
5. Если не совпадают → 403 Forbidden
```

**AJAX запросы:**
```javascript
// JavaScript
function getCookie(name) {
    let cookieValue = null;
    if (document.cookie) {
        const cookies = document.cookie.split(';');
        for (let cookie of cookies) {
            cookie = cookie.trim();
            if (cookie.startsWith(name + '=')) {
                cookieValue = decodeURIComponent(cookie.substring(name.length + 1));
                break;
            }
        }
    }
    return cookieValue;
}

const csrftoken = getCookie('csrftoken');

fetch('/api/endpoint/', {
    method: 'POST',
    headers: {
        'X-CSRFToken': csrftoken,
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({data: 'value'})
});
```

---

## 4. Представления (Views)

### 4.1. FBV vs CBV

**Function-Based Views:**
```python
from django.http import JsonResponse
from django.views.decorators.http import require_http_methods

@require_http_methods(["GET", "POST"])
def book_list(request):
    if request.method == 'GET':
        books = Book.objects.all()
        data = [{'id': b.id, 'title': b.title} for b in books]
        return JsonResponse({'books': data})
    
    elif request.method == 'POST':
        title = request.POST.get('title')
        book = Book.objects.create(title=title)
        return JsonResponse({'id': book.id}, status=201)
```

**Class-Based Views:**
```python
from django.views import View
from django.utils.decorators import method_decorator
from django.views.decorators.csrf import csrf_exempt

@method_decorator(csrf_exempt, name='dispatch')
class BookView(View):
    def get(self, request):
        books = Book.objects.all()
        data = [{'id': b.id, 'title': b.title} for b in books]
        return JsonResponse({'books': data})
    
    def post(self, request):
        title = request.POST.get('title')
        book = Book.objects.create(title=title)
        return JsonResponse({'id': book.id}, status=201)
    
    def put(self, request, pk):
        book = Book.objects.get(pk=pk)
        book.title = request.POST.get('title')
        book.save()
        return JsonResponse({'status': 'updated'})
```

**Generic Class-Based Views:**
```python
from django.views.generic import ListView, DetailView, CreateView, UpdateView
from django.urls import reverse_lazy

class BookListView(ListView):
    model = Book
    template_name = 'books/book_list.html'
    context_object_name = 'books'
    paginate_by = 10
    
    def get_queryset(self):
        return Book.objects.select_related('author').filter(status='published')

class BookCreateView(CreateView):
    model = Book
    fields = ['title', 'author', 'price']
    template_name = 'books/book_form.html'
    success_url = reverse_lazy('book_list')
    
    def form_valid(self, form):
        form.instance.created_by = self.request.user
        return super().form_valid(form)
```

---

## 5. Django REST Framework (DRF)

### 5.1. Сериализаторы

**Базовый сериализатор:**
```python
from rest_framework import serializers
from .models import Book, Author

class AuthorSerializer(serializers.ModelSerializer):
    class Meta:
        model = Author
        fields = ['id', 'name', 'email']

class BookSerializer(serializers.ModelSerializer):
    author = AuthorSerializer(read_only=True)
    author_id = serializers.IntegerField(write_only=True)
    
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'author_id', 'price', 'created_at']
        read_only_fields = ['created_at']
    
    # Кастомная валидация
    def validate_price(self, value):
        if value < 0:
            raise serializers.ValidationError("Price cannot be negative")
        if value > 10000:
            raise serializers.ValidationError("Price is too high")
        return value
    
    def validate(self, data):
        # Валидация нескольких полей
        if data.get('price') and data.get('discount'):
            if data['price'] < data['discount']:
                raise serializers.ValidationError(
                    "Discount cannot be greater than price"
                )
        return data
```

**Вложенные сериализаторы:**
```python
class ReviewSerializer(serializers.ModelSerializer):
    user_name = serializers.CharField(source='user.username', read_only=True)
    
    class Meta:
        model = Review
        fields = ['id', 'user_name', 'rating', 'comment']

class BookWithReviewsSerializer(BookSerializer):
    reviews = ReviewSerializer(many=True, read_only=True)
    
    class Meta(BookSerializer.Meta):
        fields = BookSerializer.Meta.fields + ['reviews']
```

---

### 5.2. Views в DRF

**APIView:**
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

class BookListCreateView(APIView):
    def get(self, request):
        books = Book.objects.all()
        serializer = BookSerializer(books, many=True)
        return Response(serializer.data)
    
    def post(self, request):
        serializer = BookSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)
```

**GenericAPIView + Mixins:**
```python
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin

class BookListCreateView(GenericAPIView, ListModelMixin, CreateModelMixin):
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    
    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)
```

**ViewSet:**
```python
from rest_framework import viewsets
from rest_framework.decorators import action
from rest_framework.response import Response

class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.select_related('author').all()
    serializer_class = BookSerializer
    filterset_fields = ['author', 'status']
    search_fields = ['title', 'description']
    ordering_fields = ['price', 'created_at']
    
    @action(detail=True, methods=['post'])
    def publish(self, request, pk=None):
        book = self.get_object()
        book.status = 'published'
        book.save()
        return Response({'status': 'published'})
    
    @action(detail=False)
    def bestsellers(self, request):
        books = self.queryset.filter(sales__gte=1000)
        serializer = self.get_serializer(books, many=True)
        return Response(serializer.data)
```

**Router:**
```python
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'books', BookViewSet, basename='book')
router.register(r'authors', AuthorViewSet, basename='author')

urlpatterns = router.urls
```

---

### 5.3. Аутентификация в DRF

**Token Authentication:**
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.TokenAuthentication',
        'rest_framework.authentication.SessionAuthentication',
    ],
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticated',
    ]
}

# views.py
from rest_framework.authtoken.views import ObtainAuthToken
from rest_framework.authtoken.models import Token

class CustomAuthToken(ObtainAuthToken):
    def post(self, request, *args, **kwargs):
        serializer = self.serializer_class(data=request.data,
                                           context={'request': request})
        serializer.is_valid(raise_exception=True)
        user = serializer.validated_data['user']
        token, created = Token.objects.get_or_create(user=user)
        return Response({
            'token': token.key,
            'user_id': user.pk,
            'email': user.email
        })
```

**JWT Authentication:**
```python
# pip install djangorestframework-simplejwt

# settings.py
from datetime import timedelta

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}

from rest_framework_simplejwt.settings import api_settings

SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=15),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=30),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
}

# urls.py
from rest_framework_simplejwt.views import (
    TokenObtainPairView,
    TokenRefreshView,
)

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view()),
    path('api/token/refresh/', TokenRefreshView.as_view()),
]
```

---

### 5.4. Пагинация

**Настройка:**
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}

# Кастомная пагинация
class CustomPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100
    
    def get_paginated_response(self, data):
        return Response({
            'count': self.page.paginator.count,
            'total_pages': self.page.paginator.num_pages,
            'current_page': self.page.number,
            'next': self.get_next_link(),
            'previous': self.get_previous_link(),
            'results': data
        })
```

---

## 6. Продвинутые темы

### 6.1. Транзакции

```python
from django.db import transaction

@transaction.atomic
def transfer_money(from_account, to_account, amount):
    if from_account.balance < amount:
        raise ValueError("Insufficient funds")
    
    from_account.balance -= amount
    from_account.save()
    
    to_account.balance += amount
    to_account.save()
    
    # При ошибке всё откатится

# Вложенные транзакции
@transaction.atomic
def outer():
    with transaction.atomic():  # Savepoint
        inner()

def inner():
    # Если ошибка — откат до savepoint
    pass
```

---

### 6.2. Сигналы

```python
from django.db.models.signals import post_save, pre_delete
from django.dispatch import receiver

@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        Profile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()

# Отключение сигналов (для тестов)
from django.db.models import signals

signals.post_save.disconnect(create_user_profile, sender=User)
```

---

## 🔗 Связанные темы

- [[04_Databases/01_Databases_ORM_SQL]] — SQL основы
- [[04_Databases/02_Database_Optimization_N1]] — Оптимизация N+1
- [[01_Python_Backend/03_Web_Servers_Quality]] — WSGI/ASGI

---

*Файл обновлён: 17 марта 2026*
