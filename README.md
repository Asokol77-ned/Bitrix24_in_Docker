# Bitrix Docker Development Setup

Проект для локальной разработки Bitrix CMS в Docker с возможностью деплоя на тестовый сервер.

## Структура проекта

```
bitrix-docker/
├── docker-compose.yml          # Конфигурация Docker сервисов
├── .env                        # Переменные окружения (создать из .env.example)
├── .env.example                # Шаблон переменных окружения
├── .gitignore                  # Правила исключения файлов из Git
├── nginx.conf                  # Конфигурация Nginx
├── php/
│   ├── Dockerfile             # Образ PHP с расширениями
│   └── conf.d/
│       └── bitrix.ini         # Настройки PHP для Bitrix
├── bitrix/                    # Директория Bitrix
└── deploy/
    └── deploy.sh              # Скрипт деплоя на тестовый сервер
```

## Требования

- Docker и Docker Compose установлены на системе
- Git для версионирования кода

## Установка и запуск

### 1. Клонирование репозитория

```bash
git clone <your-repo-url>
cd bitrix-docker
```

### 2. Настройка переменных окружения

Скопируйте шаблон и заполните значения:

```bash
cp .env.example .env
```

Отредактируйте `.env` файл и установите нужные пароли и настройки базы данных.

### 3. Запуск проекта

```bash
# Запуск всех сервисов
docker compose up -d

# Проверка статуса
docker compose ps
```

### 4. Установка Bitrix

1. Скачайте установочный скрипт BitrixSetup:

```bash
wget https://www.1c-bitrix.ru/download/scripts/bitrixsetup.php -O ./bitrix/bitrixsetup.php
```

2. Установите права доступа:

```bash
chmod -R 777 ./bitrix
```

3. Откройте в браузере: `http://localhost/bitrixsetup.php`

4. При настройке подключения к базе данных используйте:
   - **Хост БД**: `db` (имя сервиса из docker-compose.yml)
   - **Имя БД**: значение из `.env` (по умолчанию `bitrix`)
   - **Пользователь БД**: значение из `.env` (по умолчанию `bitrix`)
   - **Пароль БД**: значение из `.env`

## Работа с Git

Проект использует два репозитория:
- **origin** (Bitrix24_in_Docker) - конфигурация Docker и инфраструктура
- **deploy** (r4s.git) - код Bitrix портала для деплоя на тестовый сервер

### Что исключено из репозитория разработки

- `.env` файлы с паролями
- `bitrix/upload/` - загруженные пользователями файлы
- `bitrix/bitrixcache/` - кеш Bitrix
- `bitrix/managed_cache/` - управляемый кеш
- Базы данных и временные файлы

### Что включено в репозиторий разработки

- Конфигурационные файлы Docker
- Настройки Nginx и PHP
- Структура проекта

### Отправка изменений Bitrix в репозиторий деплоя

После разработки изменений внутри Docker контейнера, для отправки кода Bitrix в репозиторий деплоя используйте:

```bash
# Убедитесь, что все изменения закоммичены в основной репозиторий
git add .
git commit -m "Описание изменений"

# Отправка только содержимого bitrix/ в репозиторий деплоя
./deploy/push-to-deploy.sh "Описание изменений для деплоя"
```

Скрипт автоматически отправит только содержимое папки `bitrix/` в репозиторий `r4s.git`, который используется для деплоя на тестовый сервер.

## Деплой на тестовый сервер

### Подготовка сервера

1. Установите Docker и Docker Compose на тестовом сервере
2. Клонируйте репозиторий на сервер
3. Создайте `.env` файл с параметрами для тестовой среды

### Процесс деплоя

#### Вариант 1: Использование скрипта деплоя

```bash
chmod +x deploy/deploy.sh
./deploy/deploy.sh
```

#### Вариант 2: Ручной деплой

```bash
# Подключитесь к серверу по SSH
ssh user@test-server

# Перейдите в директорию проекта
cd /path/to/bitrix-docker

# Обновите код из Git
git pull origin main

# Перезапустите контейнеры
docker compose down
docker compose up -d
```

## Полезные команды

### Просмотр логов

```bash
# Все сервисы
docker compose logs -f

# Конкретный сервис
docker compose logs -f php
docker compose logs -f nginx
docker compose logs -f db
```

### Остановка проекта

```bash
docker compose down
```

### Остановка с удалением volumes (удалит базу данных!)

```bash
docker compose down -v
```

### Пересборка PHP образа

```bash
docker compose build --no-cache php
docker compose up -d
```

### Доступ к контейнерам

```bash
# PHP контейнер
docker compose exec php bash

# База данных
docker compose exec db mysql -u bitrix -p bitrix
```

## Порты

- **80** - Nginx веб-сервер
- **3306** - MariaDB (для внешнего доступа, если нужно)

## Структура базы данных

База данных хранится в Docker volume `db_data` и сохраняется между перезапусками контейнеров.

## Решение проблем

### Ошибка "Permission denied" при установке Bitrix

```bash
chmod -R 777 ./bitrix
```

### Контейнеры не запускаются

Проверьте логи:
```bash
docker compose logs
```

Убедитесь, что порт 80 свободен:
```bash
sudo lsof -i :80
```

### База данных недоступна

Проверьте, что контейнер `db` запущен:
```bash
docker compose ps
```

Убедитесь, что в `.env` указаны правильные параметры подключения.

## Дополнительная информация

- [Официальная документация Bitrix](https://dev.1c-bitrix.ru/)
- [Docker документация](https://docs.docker.com/)
- [Docker Compose документация](https://docs.docker.com/compose/)
