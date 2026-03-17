---
created: 2026-03-17
category: Infrastructure
type: flashcard
difficulty: beginner
frequency: high
status: new
next_review: 
tags: [docker, container, image, flashcard/infrastructure]
related: [[05_Infrastructure/01_Infrastructure_Networks_Docker#1-1-Архитектура-Docker]]
---

# Образ vs Контейнер в Docker

## Вопрос
В чём разница между Docker Image и Docker Container?
?
## Ответ

**Docker Image (Образ):**
- Статический, неизменяемый шаблон
- "Инструкция" для создания контейнера
- Создаётся на основе Dockerfile
- Хранится в registry (Docker Hub)
- Как **класс** в ООП

**Docker Container (Контейнер):**
- Запущенный экземпляр образа
- Изменяемый (есть слой записи)
- Имеет своё пространство процессов и сеть
- Как **объект** класса в ООП

**Пример:**
```bash
# Образ (статичный)
docker pull nginx:latest
docker build -t myapp .

# Контейнер (запущенный)
docker run -d --name web nginx:latest
docker run -it --rm ubuntu bash
```

**Соотношение:**
- Один образ → много контейнеров
- Контейнер = Образ + тонкий слой записи

---

#flashcard #docker #container #image
