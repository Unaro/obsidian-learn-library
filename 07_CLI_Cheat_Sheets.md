---
created: 2026-03-17
updated: 2026-03-17
category: Reference
type: cheat-sheet
priority: high
status: completed
tags: [cli, linux, git, docker, kubectl, nodejs, commands]
difficulty: beginner
estimated_hours: 2
---

# CLI Cheat Sheets

> **Назначение:** Быстрые шпаргалки по командам Linux, Git, Docker, kubectl для повседневного использования.

---

## 1. Linux Commands

### Навигация и файлы
```bash
# Навигация
pwd                     # Текущая директория
cd /path/to/dir         # Перейти в директорию
cd ..                   # На уровень вверх
cd ~                    # Домой
ls -la                  # Список файлов (все + детали)
tree -L 2               # Дерево файлов (2 уровня)

# Файлы
touch file.txt          # Создать файл
cat file.txt            # Показать содержимое
less file.txt           # Показать с постраничной навигацией
head -n 10 file.txt     # Первые 10 строк
tail -n 10 file.txt     # Последние 10 строк
tail -f file.txt        # Следить за изменениями (log)

# Копирование/Перемещение
cp file.txt backup.txt  # Копировать файл
cp -r dir1 dir2         # Копировать директорию
mv file.txt /path/      # Переместить
mv old.txt new.txt      # Переименовать
rm file.txt             # Удалить файл
rm -rf dir/             # Удалить директорию (ОСТОРОЖНО!)

# Поиск
find . -name "*.txt"    # Найти файлы по имени
find . -type f -size +100M  # Файлы > 100MB
grep "pattern" file.txt # Найти в файле
grep -r "pattern" .     # Рекурсивный поиск
grep -i "pattern"       # Без учёта регистра
grep -A 3 "pattern"     # + 3 строки после
grep -B 3 "pattern"     # + 3 строки до
```

### Процессы
```bash
ps aux                  # Все процессы
top                     # Интерактивный мониторинг
htop                    # Улучшенный top
kill PID                # Убить процесс
kill -9 PID             # Принудительно
pkill process_name      # Убить по имени
pgrep process_name      # Найти PID по имени

# Фоновые задачи
command &               # Запустить в фоне
Ctrl+Z                  # Приостановить
bg                      # Продолжить в фоне
fg                      # Вернуть на передний план
jobs                    # Список фоновых задач
```

### Сеть
```bash
curl https://api.com            # GET запрос
curl -X POST -d "data" url      # POST запрос
curl -H "Authorization: Bearer" # С заголовком

wget https://file.com/file.zip  # Скачать файл

ssh user@host                   # SSH подключение
ssh -i key.pem user@host        # С ключом
scp file.txt user@host:/path/   # Копировать на сервер
scp user@host:/path/file.txt .  # Копировать с сервера

netstat -tulpn                  # Открытые порты
ss -tulpn                       # Альтернатива netstat

ping google.com                 # Проверка доступности
traceroute google.com           # Трассировка маршрута

dig domain.com                  # DNS запрос
nslookup domain.com             # DNS запрос (альтернатива)
```

### Диски и память
```bash
df -h                   # Свободное место на дисках
du -sh folder/          # Размер папки
du -h --max-depth=1     # Размер папок в текущей

free -h                 # Оперативная память
vmstat 1                # Статистика (обновление 1с)

iostat                  # Статистика дисков
iotop                   # Процессы с I/O (как top для диска)
```

### Права доступа
```bash
chmod 755 file          # Изменить права (rwxr-xr-x)
chmod +x script.sh      # Сделать исполняемым
chown user:group file   # Сменить владельца
chgrp group file        # Сменить группу

# Числовые значения:
# 4 = read, 2 = write, 1 = execute
# 755 = rwxr-xr-x, 644 = rw-r--r--
```

### Архивы
```bash
tar -czvf archive.tar.gz folder/    # Создать .tar.gz
tar -xzvf archive.tar.gz            # Распаковать .tar.gz
zip -r archive.zip folder/          # Создать .zip
unzip archive.zip                   # Распаковать .zip
```

### Системная информация
```bash
uname -a                # Информация о ядре
hostname                # Имя хоста
whoami                  # Текущий пользователь
id                      # ID пользователя и группы
uptime                  # Время работы и нагрузка
w                       # Кто залогинен и что делает
last                    # История входов
dmesg | tail            # Сообщения ядра
```

---

## 2. Git Commands

### Базовые операции
```bash
# Инициализация и клонирование
git init                        # Инициализировать репозиторий
git clone <url>                 # Клонировать репозиторий
git clone <url> --depth 1       # Клонировать без истории (быстрее)

# Статус и просмотр
git status                      # Статус изменений
git diff                        # Изменения в working directory
git diff --staged               # Изменения в staged
git diff HEAD~1                 # Разница с предыдущим коммитом
git log --oneline -10           # Последние 10 коммитов
git log --graph --oneline       # Граф коммитов
git show <commit>               # Показать коммит
git blame file.txt              # Кто изменил каждую строку

# Добавление и коммит
git add file.txt                # Добавить файл
git add .                       # Добавить все
git add -p                      # Добавить по частям (интерактивно)
git commit -m "Message"         # Коммит с сообщением
git commit --amend              # Исправить последний коммит
git commit --amend --no-edit    # Исправить без изменения сообщения
```

### Ветвление
```bash
# Ветви
git branch                      # Список веток
git branch -a                   # Все ветки (локальные + remote)
git branch feature              # Создать ветку
git checkout feature            # Переключиться на ветку
git checkout -b feature         # Создать и переключиться
git branch -d feature           # Удалить ветку
git branch -D feature           # Удалить принудительно

# Слияние
git merge feature               # Влить ветку feature
git merge --no-ff feature       # Влить с сохранением истории
git merge --abort               # Прервать слияние при конфликте

# Rebase
git rebase main                 # Перебазировать на main
git rebase -i HEAD~3            # Интерактивный rebase (3 коммита)
git rebase --abort              # Прервать rebase
git rebase --continue           # Продолжить после разрешения конфликтов
```

### Remote
```bash
git remote -v                   # Список remote репозиториев
git remote add origin <url>     # Добавить remote
git fetch origin                # Получить изменения (не мержить)
git pull origin main            # Получить и влить (fetch + merge)
git pull --rebase origin main   # Получить и перебазировать
git push origin main            # Отправить ветку
git push -u origin main         # Отправить и установить upstream
git push --force                # Принудительный push (ОСТОРОЖНО!)
git push --force-with-lease     # Безопасный force push
```

### Отмена изменений
```bash
git restore file.txt            # Отменить изменения в файле
git restore --staged file.txt   # Убрать из staged
git reset HEAD~1                # Отменить последний коммит (сохранить изменения)
git reset --hard HEAD~1         # Отменить коммит и изменения
git revert <commit>             # Отменить коммит новым коммитом
git clean -fd                   # Удалить неотслеживаемые файлы
```

### Stash
```bash
git stash                       # Спрятать изменения
git stash list                  # Список stash
git stash pop                   # Достать и удалить stash
git stash apply                 # Достать (сохранить в stash)
git stash drop                  # Удалить stash
git stash show -p               # Показать содержимое stash
```

### Продвинутые
```bash
# Cherry-pick
git cherry-pick <commit>        # Взять конкретный коммит
git cherry-pick -n <commit>     # Без коммита (только изменения)

# Bisect (поиск баг-коммита)
git bisect start                # Начать поиск
git bisect bad                  # Текущий коммит плохой
git bisect good <commit>        # Этот коммит хороший
git bisect reset                # Завершить поиск

# Submodule
git submodule init              # Инициализировать submodules
git submodule update            # Обновить submodules
```

### Конфликты
```bash
# При конфликте слияния:
git status                      # Показать конфликтующие файлы
# Редактировать файлы, разрешить конфликты
git add <resolved_files>        # Пометить разрешёнными
git commit                      # Завершить слияние

# Инструменты
git mergetool                   # GUI для разрешения конфликтов
git diff --name-only --diff-filter=U  # Список конфликтующих файлов
```

---

## 3. Docker Commands

### Контейнеры
```bash
# Запуск и управление
docker run -d --name app nginx          # Запустить контейнер
docker run -it ubuntu bash              # Интерактивный запуск
docker run -p 8080:80 nginx             # Проброс портов
docker run -v /data:/app/data nginx     # Монтирование тома
docker run -e ENV=prod app              # Переменная окружения
docker run --rm app                     # Удалить после остановки

# Просмотр
docker ps                               # Запущенные контейнеры
docker ps -a                            # Все контейнеры
docker logs <container>                 # Логи контейнера
docker logs -f <container>              # Следить за логами
docker logs --tail 100 <container>      # Последние 100 строк
docker inspect <container>              # Детальная информация
docker top <container>                  # Процессы в контейнере
docker stats                            # Статистика использования

# Остановка и удаление
docker stop <container>                 # Остановить
docker start <container>                # Запустить
docker restart <container>              # Перезапустить
docker kill <container>                 # Принудительно остановить
docker rm <container>                   # Удалить контейнер
docker rm -f <container>                # Удалить запущенный
docker rm $(docker ps -aq)              # Удалить все контейнеры

# Копирование и exec
docker cp file.txt container:/path/     # Копировать в контейнер
docker cp container:/path/file.txt .    # Копировать из контейнера
docker exec -it container bash          # Выполнить команду
docker exec -it container sh            # Если нет bash
```

### Образы
```bash
# Поиск и загрузка
docker images                           # Список образов
docker pull nginx                       # Скачать образ
docker pull nginx:1.21                  # Конкретная версия
docker search nginx                     # Поиск образов

# Сборка и удаление
docker build -t myapp .                 # Собрать образ
docker build -t myapp:1.0 .             # С тегом
docker rmi myapp                        # Удалить образ
docker rmi -f myapp                     # Принудительно удалить
docker images -q | xargs docker rmi -f  # Удалить все образы

# Экспорт/Импорт
docker save -o myapp.tar myapp          # Сохранить в tar
docker load -i myapp.tar                # Загрузить из tar
```

### Docker Compose
```bash
docker-compose up                       # Запустить сервисы
docker-compose up -d                    # В фоне (detached)
docker-compose up --build               # Пересобрать образы
docker-compose down                     # Остановить и удалить
docker-compose down -v                  # + удалить volumes
docker-compose ps                       # Статус сервисов
docker-compose logs                     # Логи всех сервисов
docker-compose logs -f app              # Логи конкретного сервиса
docker-compose logs -f --tail 100 app   # Последние 100 строк
docker-compose exec app bash            # Выполнить в контейнере
docker-compose restart app              # Перезапустить сервис
docker-compose stop app                 # Остановить сервис
docker-compose start app                # Запустить сервис
docker-compose build app                # Пересобрать образ
docker-compose pull app                 # Скачать образы
```

### Сети и тома
```bash
# Сети
docker network ls                       # Список сетей
docker network create mynet             # Создать сеть
docker network inspect bridge           # Информация о сети
docker network connect mynet container  # Подключить контейнер
docker network disconnect mynet container # Отключить

# Тома
docker volume ls                        # Список томов
docker volume create mydata             # Создать том
docker volume inspect mydata            # Информация о томе
docker volume rm mydata                 # Удалить том
docker volume prune                     # Удалить неиспользуемые
```

### Очистка
```bash
docker system prune                     # Удалить неиспользуемое
docker system prune -a                  # + все образы без контейнеров
docker system prune --volumes           # + тома
df -sh /var/lib/docker                  # Размер Docker
```

---

## 4. kubectl Commands (Kubernetes)

### Ресурсы
```bash
# Получение информации
kubectl get pods                        # Список подов
kubectl get pods -n namespace           # В конкретном namespace
kubectl get pods -o wide                # Подробная информация
kubectl get deployments                 # Список deployment
kubectl get services                    # Список сервисов
kubectl get all                         # Все ресурсы
kubectl get events --sort-by='.lastTimestamp'  # События

# Детали
kubectl describe pod <pod>              # Детали пода
kubectl describe deployment <deploy>    # Детали deployment
kubectl explain pod                     # Документация по ресурсу
kubectl explain pod.spec.containers     # Документация по полю

# Логи
kubectl logs <pod>                      # Логи пода
kubectl logs -f <pod>                   # Следить за логами
kubectl logs <pod> -c container         # Логи конкретного контейнера
kubectl logs <pod> --previous           # Логи предыдущего экземпляра
```

### Управление
```bash
# Создание и удаление
kubectl apply -f manifest.yaml          # Применить манифест
kubectl apply -f folder/                # Применить все в папке
kubectl create deployment app --image=nginx  # Создать deployment
kubectl delete pod <pod>                # Удалить под
kubectl delete -f manifest.yaml         # Удалить по манифесту

# Редактирование
kubectl edit deployment <deploy>        # Редактировать deployment
kubectl edit configmap <config>         # Редактировать configmap

# Масштабирование
kubectl scale deployment app --replicas=3    # Изменить реплики
kubectl autoscale deployment app --min=2 --max=10 --cpu-percent=80  # Autoscaling
```

### Отладка
```bash
# Exec и копирование
kubectl exec -it <pod> -- bash          # Выполнить в поде
kubectl exec -it <pod> -c container -- sh  # В конкретном контейнере
kubectl cp file.txt <pod>:/path/        # Копировать в под
kubectl cp <pod>:/path/file.txt .       # Копировать из пода

# Port forwarding
kubectl port-forward <pod> 8080:80      # Проброс порта
kubectl port-forward svc/app 8080:80    # Проброс сервиса

# Запуск временного пода
kubectl run temp --image=busybox --rm -it -- bash  # Временный под
```

### Конфигурация
```bash
# ConfigMap и Secrets
kubectl create configmap myconfig --from-file=config.yaml
kubectl create secret generic mysecret --from-literal=key=value
kubectl get configmap
kubectl get secrets
kubectl describe configmap myconfig
kubectl describe secret mysecret

# Применение конфигурации
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml
```

### Контекст и namespace
```bash
kubectl config get-contexts             # Список контекстов
kubectl config use-context <context>    # Переключить контекст
kubectl config current-context          # Текущий контекст

kubectl get namespaces                  # Список namespace
kubectl create namespace myns           # Создать namespace
kubectl config set-context --current --namespace=myns  # Установить namespace
```

### Метрики и мониторинг
```bash
kubectl top pods                        # Использование ресурсов подами
kubectl top nodes                       # Использование ресурсов нодами
kubectl get hpa                         # Horizontal Pod Autoscaler
```

---

## 5. Node.js Commands

### npm
```bash
npm init -y                     # Инициализировать package.json
npm install package             # Установить пакет
npm install package@version     # Конкретная версия
npm install -g package          # Глобально
npm install --save-dev package  # В devDependencies
npm uninstall package           # Удалить пакет

npm list                        # Список установленных пакетов
npm list --depth=0              # Только зависимости верхнего уровня
npm outdated                    # Устаревшие пакеты
npm update                      # Обновить пакеты

npm run script                  # Запустить скрипт
npm run build                   # Сборка
npm run test                    # Тесты
npm run start                   # Запуск

npm audit                       # Проверка уязвимостей
npm audit fix                   # Исправить уязвимости
npm cache clean --force         # Очистить кэш

npx package                     # Запустить без установки
npx create-react-app myapp      # Создать React приложение
```

### yarn
```bash
yarn init -y                    # Инициализировать
yarn add package                # Установить пакет
yarn add -D package             # В devDependencies
yarn remove package             # Удалить пакет

yarn install                    # Установить зависимости
yarn upgrade                    # Обновить пакеты
yarn outdated                   # Устаревшие пакеты

yarn run script                 # Запустить скрипт
yarn build                      # Сборка
yarn test                       # Тесты
yarn start                      # Запуск
```

### pnpm
```bash
pnpm init                       # Инициализировать
pnpm add package                # Установить пакет
pnpm add -D package             # В devDependencies
pnpm remove package             # Удалить пакет

pnpm install                    # Установить зависимости
pnpm up                         # Обновить пакеты

pnpm run script                 # Запустить скрипт
```

---

## 6. Полезные утилиты

### jq (JSON процессор)
```bash
cat file.json | jq '.key'               # Получить поле
cat file.json | jq '.[] | select(.age > 25)'  # Фильтрация
cat file.json | jq 'map(.name)'         # Трансформация массива
curl api.com/data | jq '.data[].id'     # Из API
```

### grep, awk, sed
```bash
# grep
grep -r "pattern" .                     # Рекурсивный поиск
grep -l "pattern" *                     # Только имена файлов
grep -c "pattern" file                  # Количество совпадений
grep -v "pattern" file                  # Исключить pattern
grep -E "pattern1|pattern2" file        # Regex

# awk
awk '{print $1}' file                   # Первое поле
awk -F: '{print $1}' /etc/passwd        # С разделителем
awk '$3 > 100 {print $1}' file          # С условием

# sed
sed 's/old/new/g' file                  # Замена
sed -i 's/old/new/g' file               # Замена в файле
sed -n '5,10p' file                     # Строки 5-10
sed '/pattern/d' file                   # Удалить строки с pattern
```

### watch, xargs, tee
```bash
watch -n 1 'docker ps'                  # Обновлять каждую секунду
watch -n 5 'kubectl get pods'           # Обновлять каждые 5 секунд

echo "file1 file2" | xargs rm           # Передать аргументы команде
find . -name "*.log" | xargs rm         # Удалить все .log файлы

command | tee output.txt                # Сохранить вывод в файл
command | tee -a output.txt             # Добавить в файл
```

---

*Файл обновлён: 17 марта 2026*
