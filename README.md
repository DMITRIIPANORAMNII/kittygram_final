# Kittygram — социальная сеть для обмена фотографиями любимых питомцев.

## О проекте

Проект реализует веб сайт kittigram.
Kittygram - это социальная сеть, для любителей кошек. Пользователи могут добавлять фотографии своих котов и смотреть на
котов, добавленных другими пользователями.
В проекте настроены Github Actions для автотестов и деплоя на сервер.


---
## В проекте были использованы технологии:
* #### Django REST
* #### Python 3.9
* #### Gunicorn
* #### Nginx
* #### JS
* #### Node.js
* #### PostgreSQL
* #### Docker
---

### Установка и развертвывание проекта
<i>Примечание: Все примеры указаны для Linux</i><br>
1. Склонируйте репозиторий на свой компьютер:
    ```
    git clone git@github.com:insaiderzog/kittygram_final.git
    ```
2. Создайте файл `.env` и заполните его своими данными. Все необходимые переменные перечислены в файле `.env.example`, находящемся в корневой директории проекта.

### Создание Docker-образов

1. Замените `YOUR_USERNAME` на свой логин на DockerHub:

    ```
    cd frontend
    docker build -t insaiderzog/kittygram_frontend .
    cd ../backend
    docker build -t insaiderzog/kittygram_backend .
    cd ../nginx
    docker build -t insaiderzog/kittygram_gateway . 
    ```

2. Загрузите образы на DockerHub:

    ```
    docker push insaiderzog/kittygram_frontend
    docker push insaiderzog/kittygram_backend
    docker push insaiderzog/kittygram_gateway
    docker push insaiderzog/taski_gateway
    ```

### Деплой на сервере

1. Подключитесь к удаленному серверу

    ```
    ssh -i PATH_TO_SSH_KEY/SSH_KEY_NAME YOUR_USERNAME@SERVER_IP_ADDRESS 
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
    scp -i PATH_TO_SSH_KEY/SSH_KEY_NAME docker-compose.production.yml YOUR_USERNAME@SERVER_IP_ADDRESS:/home/YOUR_USERNAME/kittygram/docker-compose.production.yml
    ```
    
    Где:
    - `PATH_TO_SSH_KEY` - путь к файлу с вашим SSH-ключом
    - `SSH_KEY_NAME` - имя файла с вашим SSH-ключом
    - `YOUR_USERNAME` - ваше имя пользователя на сервере
    - `SERVER_IP_ADDRESS` - IP-адрес вашего сервера

5. Запустите Docker Compose в режиме демона:

    ```
    вщслук 
    ```

6. Выполните миграции, соберите статические файлы бэкенда и скопируйте их в `/backend_static/static/`:

    ```
    sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend python manage.py migrate
    sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend python manage.py collectstatic
    sudo docker-compose -f /home/yc-user/kittygram/docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
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

2. Для адаптации его к вашему серверу добавьте секреты в GitHub Actions:

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
