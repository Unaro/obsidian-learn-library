# 🐍 Python & Backend Development

> **Назначение:** Навигация по темам Python, Django и серверной разработки

---

## 📁 Файлы категории

| № | Файл | Описание |
|---|------|----------|
| 01 | [[01_Python_Backend/01_Python_Core]] | Базовые и продвинутые концепции Python |
| 02 | [[01_Python_Backend/02_Django_REST_API]] | Django, ORM, REST API, аутентификация |
| 03 | [[01_Python_Backend/03_Web_Servers_Quality]] | WSGI/ASGI, кэширование, WebSockets, TDD |

---

## 🔰 Python Core

### Из файла [[01_Python_Backend/01_Python_Core]]

#### 1. Типы данных и структуры
- **Мутабельность:**
  - Неизменяемые: `int`, `float`, `complex`, `bool`, `tuple`, `str`, `frozenset`
  - Изменяемые: `list`, `set`, `dict`
- **Словари:** O(1) доступ через хэш-таблицы
- **Копирование:** `copy()` vs `deepcopy()`

#### 2. Функции
- `*args`, `**kwargs`
- Lambda-функции
- Итераторы vs Генераторы (`yield`)
- Область видимости **LEGB**

#### 3. ООП
- `__new__` vs `__init__`
- Инкапсуляция (`_`, `__`)
- Декораторы: `@classmethod`, `@staticmethod`, `@property`
- Дандер-методы
- Абстрактные классы (`abc`, `@abstractmethod`)

#### 4. Память
- `is` vs `==`
- `id()`
- Передача по ссылке

#### 5. Конкурентность
- **GIL** (Global Interpreter Lock)
- `threading` (I/O-bound)
- `multiprocessing` (CPU-bound)

---

## 🌐 Django Framework

### Из файла [[01_Python_Backend/02_Django_REST_API]]

#### 1. Архитектура MVT
- **Model** — данные и бизнес-логика
- **View** — обработка запросов
- **Template** — отображение

#### 2. ORM и оптимизация
- Связи: `OneToOneField`, `ForeignKey`, `ManyToManyField`
- **Проблема N+1:**
  - `select_related()` — JOIN для ForeignKey
  - `prefetch_related()` — раздельные запросы для ManyToMany
- `Q` объекты для сложных запросов

#### 3. Middleware и безопасность
- Промежуточное ПО
- CSRF Middleware

#### 4. Views
- **FBV** (Function-Based Views)
- **CBV** (Class-Based Views)

#### 5. Django REST Framework
- **Serializer** — преобразование в JSON
- **Валидация:** `is_valid()`
- **Views:** `APIView`, `GenericAPIView`, `ViewSet`
- **Аутентификация:** Session, Token, JWT
- **Пагинация:** PageNumber, LimitOffset, Cursor

---

## ⚙️ Web-серверы и качество кода

### Из файла [[01_Python_Backend/03_Web_Servers_Quality]]

#### 1. Кэширование
- **Стратегии в Django:**
  - Memcached / Redis (в памяти)
  - Local-memory
  - FileSystem
  - Database

#### 2. WSGI / ASGI
- **WSGI:** синхронный стандарт (Gunicorn, uWSGI)
- **ASGI:** асинхронный стандарт (`asgi.py`)

#### 3. WebSockets
- **Django Channels** для real-time
- Каналы сообщений
- Асинхронные views

#### 4. Качество кода
- **PEP 8** — стиль кода
- **TDD:** Red → Green → Refactor

---

## 🔗 Связанные темы

- [[04_Databases/01_Databases_ORM_SQL]] — работа с БД
- [[04_Databases/02_Database_Optimization_N1]] — оптимизация запросов
- [[05_Infrastructure/01_Infrastructure_Networks_Docker]] — Docker, Git, сети
- [[05_Infrastructure/02_System_Analysis_BPMN_UML]] — BPMN, UML, SDLC

---

*Категория: Python & Backend*
