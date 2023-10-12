# Kittygram — сервис для обмена фотографиями котиков.

### Описание проекта
Пользователи могут регистрироваться, загружать фотографии с описанием котов и смотреть котиков других пользователей.

### Установка

 Склонируйте репозиторий на свой компьютер:
    ```
    git clone git@github.com:edgar1148/kittygram_final.git
    ```


### Создание Docker-образов

1. Замените `username` на свой логин на DockerHub:

    ```
    cd frontend
    docker build -t username/kittygram_frontend .
    cd ../backend
    docker build -t username/kittygram_backend .
    cd ../nginx
    docker build -t username/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```
    docker push username/kittygram_frontend
    docker push username/kittygram_backend
    docker push username/kittygram_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```
    ssh -i путь_до_файла_с_SSH_ключом/название_файла_закрытого_SSH-ключа login@ip 
    ```

2. Создайте на сервере директорию `kittygram`:

    ```
    mkdir kittygram
    ```

3. Установите Docker Compose на сервер:

    ```
    sudo apt update
    sudo apt install curl
    curl -fsSL https://get.docker.com -o get-docker.sh
    sudo sh get-docker.sh
    sudo apt install docker-compose
    ```

4. Скопируйте файлы `docker-compose.production.yml` и `.env` в директорию `kittygram/` на сервере:

    ```
    scp -i путь_до_файла_с_SSH_ключом/название_файла_закрытого_SSH-ключа docker-compose.production.yml login@ip:/home/username/kittygram/docker-compose.production.yml
    ```


5. Запустите Docker Compose:

    ```
    sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml up -d
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в `/backend_static/static/`:

    ```
    sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend python manage.py migrate
    sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker-compose -f /home/username/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
    ```

7. Откройте конфигурационный файл Nginx в редакторе nano:

    ```
    sudo nano /etc/nginx/sites-enabled/default
    ```

8. Измените настройки `location` в секции `server`:

    ```
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
    ```

9. Проверьте правильность конфигурации Nginx:

    ```
    sudo nginx -t
    ```

    Если вы получаете следующий ответ, значит, ошибок нет:

    ```
    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful
    ```

10. Перезапустите Nginx:

    ```
    sudo service nginx reload
    ```

### Настройка CI/CD

1. Файл workflow уже написан и находится в директории:

    ```
    kittygram/.github/workflows/main.yml
    ```

2. Добавьте секреты в GitHub Actions:

    ```
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # IP-адрес сервера
    USER                           # имя пользователя
    SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/id_rsa)
    SSH_PASSPHRASE                 # пароль для SSH-ключа

    TELEGRAM_TO                    # ID вашего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен вашего бота (получить токен можно у @BotFather, команда /token, имя бота)
    ```

### Технологии
Python 3.9.10,
Django 4.2.5,
djangorestframework==3.12.4, 
PostgreSQL 13.10

### Автор
[Евгений Екишев - edgar1148](https://github.com/edgar1148)