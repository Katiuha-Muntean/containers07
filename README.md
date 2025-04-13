# Лабораторная работа №7: Создание многоконтейнерного приложения

## Задание: Создать php приложение на базе трех контейнеров: nginx, php-fpm, mariadb, используя docker-compose

### Описание выполнения работы с ответами на вопросы

1. Создаем репозиторий ```containers07``` и копируем его на компьютер.
2. В директории ```containers07``` создаем директорию ```mounts/site```. В данную директорию переписываем сайт на php.
3. Создаем файл ```.gitignore``` в корне проекта и добавляем в него строки:

   ```git
   # Ignore files and directories
   mounts/site/*
   ```

4. Создаем в директории ```containers07``` файл ```nginx/default.conf``` со следующим содержимым:

    ```nginx
    server {
        listen 80;
        server_name _;
        root /var/www/html;
        index index.php;
        location / {
            try_files $uri $uri/ /index.php?$args;
        }
        location ~ \.php$ {
            fastcgi_pass backend:9000;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }

     }
    ```

5. Создаем в директории ```containers07``` файл ```docker-compose.yml``` со следующим содержимым:

     ```yml
     version: '3.9'

    services:
    frontend:
        image: nginx:1.19
        volumes:
        - ./mounts/site:/var/www/html
        - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
        ports:
        - "80:80"
        networks:
        - internal
    backend:
        image: php:7.4-fpm
        volumes:
        - ./mounts/site:/var/www/html
        networks:
        - internal
        env_file:
        - mysql.env
    database:
        image: mysql:8.0
        env_file:
        - mysql.env
        networks:
        - internal
        volumes:
        - db_data:/var/lib/mysql

    networks:
    internal: {}

    volumes:
    db_data: {}
    ```

6. Создаем файл ```mysql.env``` в корне проекта и добавляем в него строки:

   ```env
    MYSQL_ROOT_PASSWORD=secret
    MYSQL_DATABASE=app
    MYSQL_USER=user
    MYSQL_PASSWORD=secret
   ```

7. Запускаем контейнеры командой: ```docker-compose up -d```, открываем сайт по адресу ```http://localhost```:
![Img-1](https://imgur.com/TMpQ50m.jpg)
![Img-2](https://imgur.com/wePxqir.jpg)
8. Ответы на вопросы:

- В каком порядке запускаются контейнеры?

Контейнеры запускаются в определённом порядке,  в ```docker-compose.yml```, но он не гарантирует строгой последовательности. Поэтому:

- ```database``` запускается первым, так как ```backend``` зависит от него (подключается к БД).

- ```backend``` запускается после ```database```, так как использует MySQL.

- ```frontend (nginx)``` запускается последним, так как ему нужен ```backend```.

В общем, при первом запуске ```backend``` может попытаться подключиться к БД до её полной готовности. Можно добавить ```depends_on``` в ```docker-compose.yml```, но это не гарантирует, что MySQL уже принял подключения — лучше использовать ```healthcheck```.

- Где хранятся данные базы данных?
  
Данные хранятся в ```Docker volume db_data```, который монтируется в ```/var/lib/mysql``` контейнера ```database```. Это определено в ```docker-compose.yml```:

```yml
volumes:
  - db_data:/var/lib/mysql
```

```Volume db_data``` создаётся автоматически и хранится в Docker.

- Как называются контейнеры проекта?

Контейнеры получают имена по шаблону ```{имя_проекта}_{имя_сервиса}_1```. Если проект запущен из папки containers07, то:

```shell
containers07-frontend-1 (nginx)

containers07-backend-1 (php-fpm)

containers07-database-1 (mysql)

```

![Img-3](https://imgur.com/3xOm0Lt.jpg)

- Как добавить файл ```app.env``` с переменной окружения ```APP_VERSION``` для ```backend``` и ```frontend```?

  - Создаем ```app.env``` в корне проекта с содержимым:

   ```env
   APP_VERSION=1.0.0
   ```

  - Добавляем этот файл в ```docker-compose.yml``` для ```frontend``` и ```backend```:

   ```yml
   env_file:
  - app.env
  ```
![Img-4](https://imgur.com/9uF3PH8.jpg)
