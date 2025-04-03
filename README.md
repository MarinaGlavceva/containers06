# Лабораторная работа №6. Взаимодействие контейнеров

##  Цель работы

Научиться организовывать взаимодействие нескольких Docker-контейнеров на примере веб-приложения на PHP, работающее через Nginx и PHP-FPM.

---

##  Задание

1. Создать PHP-приложение на базе двух контейнеров: `nginx` и `php-fpm`.
2. Реализовать взаимодействие контейнеров в одной внутренней сети `internal`.
3. Использовать собственный PHP-файл `index3.php` в качестве приложения.
4. Настроить проброс портов, монтирование директорий и конфигурацию веб-сервера.
5. Протестировать работу сайта на `localhost`.

---

##  Выполнение

###  Шаг 1: Создание структуры проекта

Я создала новую папку с именем `containers06` в своей пользовательской директории. Внутри неё создала подкаталоги `mounts/site` и `nginx`, чтобы в них разместить PHP-сайт и конфигурацию Nginx.

```bash
mkdir containers06
cd containers06
mkdir mounts/site
mkdir nginx
```

Затем я скопировала файл `index3.php`, ранее сделанный в курсе по PHP, в папку `mounts/site`. Я использовала Проводник Windows, чтобы найти `index3.php` по пути `C:/xampp/htdocs/01_Introduction/index3.php` и вручную вставила его в `C:/Users/marin/containers06/mounts/site/`.

###  Шаг 2: Создание .gitignore

Чтобы избежать загрузки локальных файлов сайта в репозиторий, я создала файл `.gitignore` и добавила в него:

```
mounts/site/*
```

Я сделала это в командной строке так:

```cmd
cd C:\Users\marin\containers06
echo mounts/site/* > .gitignore
```

###  Шаг 3: Создание конфигурации Nginx

Я создала файл `default.conf` в папке `nginx` с конфигурацией веб-сервера. Мне было важно, чтобы Nginx мог обрабатывать запросы к файлу `index3.php`:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/html;
    index index3.php;

    location / {
        try_files $uri $uri/ /index3.php?$args;
    }

    location ~ \.php$ {
        fastcgi_pass backend:9000;
        fastcgi_index index3.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```


###  Шаг 4: Создание сети internal

Чтобы контейнеры могли "видеть" друг друга по имени, я создала сеть `internal` с помощью:

```bash
docker network create internal
```

###  Шаг 5: Запуск контейнера backend

Это контейнер с `php-fpm`, к которому я подключила папку `site`:

```bash
docker run -d --name backend --network internal -v C:/Users/marin/containers06/mounts/site:/var/www/html php:7.4-fpm
```

###  Шаг 6: Запуск контейнера frontend

После нескольких попыток и правок я успешно запустила контейнер с Nginx:

```bash
docker rm -f frontend
docker run -d --name frontend --network internal -v C:/Users/marin/containers06/mounts/site:/var/www/html -v C:/Users/marin/containers06/nginx/default.conf:/etc/nginx/conf.d/default.conf -p 8080:80 nginx:1.23-alpine
```

Я выбрала порт `8080`, потому что `80` у меня был занят XAMPP.

###  Шаг 7: Проверка работы сайта

Открыв браузер по адресу `http://localhost:8080`, я увидела результат работы моего PHP-файла. После того как я изменила конфигурацию Nginx, чтобы использовать `index3.php`, всё заработало корректно.

![Image](https://github.com/user-attachments/assets/615fb28c-9a04-4fe1-a6e8-852160e69f2b)
---

##  Ответы на вопросы

**1. Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?**  
Контейнеры подключены к одной пользовательской сети Docker (`internal`). Благодаря этому, они могут обращаться друг к другу по имени. В моём случае Nginx передаёт PHP-запросы контейнеру `backend` по адресу `backend:9000`.

**2. Как видят контейнеры друг друга в рамках сети internal?**  
Docker внутри сети сам настраивает DNS: каждый контейнер виден другим по имени. Это позволяет использовать в конфиге `fastcgi_pass backend:9000` без IP-адреса.

**3. Почему необходимо было переопределять конфигурацию nginx?**  
По умолчанию Nginx не умеет обрабатывать PHP. Чтобы он мог направлять `.php`-запросы в PHP-FPM, я вручную добавила нужные директивы (`fastcgi_pass`, `SCRIPT_FILENAME`, и т.д.)

---


##  Выводы

В ходе лабораторной работы я настроила связку `nginx + php-fpm` в отдельных Docker-контейнерах. Создала и применила собственную конфигурацию сервера, обеспечила взаимодействие через пользовательскую сеть и вывела страницу PHP в браузере. Получила полезный опыт настройки контейнеров, конфигурации и устранения ошибок в процессе.
