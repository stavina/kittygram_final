# Kittygram - социальная сеть для публикации котиков.
Пользователи могут регистрироваться, делиться фотографиями котиков и смотреть публикации других.
## Функционал проекта
- Зарегистрироваться.
- Добавить имя и фото кота.
- Убрать (своего) кота.
- Подобрать цвет, на который его окрас больше всего походит.
- Рассказть всем о достижениях вашего кота.
- Указать возраст любимца.
## Технологии
- Python 3.9
- Django==3.2.3
- djangorestframework==3.12.4
- Docker

## Запуск на локальном сервере
### Запуск backend
-   Создайте и активируйте виртуальное окружение
-   Установите зависимости из файла requirements.txt

```
pip install -r requirements.txt
```
Создайте файл .env и заполните его своими данными:

    ```bash
    POSTGRES_DB=...
    POSTGRES_USER=...
    POSTGRES_PASSWORD=...
    DB_HOST=...
    DB_PORT=...
    SECRET_KEY=...
    ALLOWED_HOST=...
    ```
-   В папке с файлом manage.py выполните команду:

```
python3 manage.py runserver
```
### Запуск frontend
Установите пакетный менеджер npm

Установите зависимости для фронтенд-приложения в директории frontend/ 
```
npm i
```
Запуск frontend осуществляется командой 
```
npm run start
```
## Создание Docker-образов

-  Для создания Docker-образов последовательно выполните следующие команды, замените username на ваш логин на DockerHub:

    ```bash
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

-  Загрузите образы на DockerHub:

    ```bash
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

-  Подключитесь к удаленному серверу

    ```bash
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_с_SSH_ключом имя_пользователя@ip_адрес_сервера 
    ```

-  Создайте на сервере директорию kittygram

    ```bash
    mkdir kittygram
    ```

-  Для установки docker compose на сервер, поочередно выполните следующие команды:

    ```bash
    sudo apt update
    sudo apt install curl
    curl -fSL https://get.docker.com -o get-docker.sh
    sudo sh ./get-docker.sh
    sudo apt-get install docker-compose-plugin
    ```

-  Скопируйте на сервер в директорию kittygram/ файлы docker-compose.production.yml и .env:

    ```bash
    scp -i path_to_SSH/SSH_name docker-compose.production.yml username@server_ip:/home/username/kittygram/docker-compose.production.yml
    ```

-  Переходите в директорию kittygram/ и запустите docker compose в режиме демона:

    ```bash
    sudo docker compose -f docker-compose.production.yml up -d
    ```

-  Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:

    ```bash
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
    sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

-  На сервере в редакторе nano откройте конфиг Nginx:

    ```bash
    nano /etc/nginx/sites-enabled/default
    ```

-  Измените настройки location в секции server:

    ```bash
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
    ```

-  Проверьте работоспособность конфига и перезапустите Nginx:

    ```bash
    sudo nginx -t 
    sudo service nginx reload
    ```

### Настройка CI/CD

Файл workflow писан в директории

    ```bash
    kittygram/.github/workflows/main.yml
    ```
