# Лабораторная работа №6: Взаимодействие контейнеров

## Цель работы
Целью данной работы является создание двух контейнеров с использованием Docker, которые обеспечат работу веб-приложения с сервером Nginx и PHP. Это включает в себя создание контейнеров для сервера PHP (backend) и для веб-сервера Nginx (frontend), а также настройку взаимодействия между ними в рамках сети Docker.

## Задание
- Создать php приложение на базе двух контейнеров: nginx, php-fpm.

## Описание выполнения работы

1. **Создание структуры проекта и файлов**
   В директории `containers06` были созданы необходимые папки для монтирования файлов:
   - `mounts/site` для размещения сайта.
   ![mounts/site](/Images/2.png)
   - `index.php` основа сайта
   ![PHP](/Images/1.png)
   - `nginx` для хранения конфигурационного файла Nginx.
   ![nginx](/Images/3.png)
   Также был создан `.gitignore`, чтобы игнорировать файлы в папке `mounts/site`.
    ![.gitignore](/Images/gitignore.png)
    - Разметка папки
    ![list](/Images/list.png)
2. **Создание контейнеров**
   Контейнеры были настроены следующим образом:
   - **Backend**: Контейнер на основе образа `php:7.4-fpm`, с монтированием каталога сайта в директорию `/var/www/html`. Он подключен к сети `internal`.
   - **Frontend**: Контейнер на основе образа `nginx:1.23-alpine`, с монтированием каталога сайта и конфигурации Nginx. Также он подключен к сети `internal`, а порт 80 проброшен на порт 80 хоста.
    ![internal](/Images/4.png)
    ![BACK-FRONT](/Images/5.png)
3. **Конфигурация Nginx**
   Для правильной работы Nginx, его конфигурация была переопределена в файле `nginx/default.conf`, чтобы настроить обработку PHP-запросов через backend контейнер. Конфигурация использует `fastcgi_pass backend:9000` для проксирования запросов на контейнер backend, работающий на порту 9000.
    ![default](/Images/default.png)
4. **Запуск контейнеров**
   Сначала была создана сеть `internal`, после чего были запущены оба контейнера с нужными параметрами.

5. **Проверка работоспособности**
   После запуска контейнеров, сайт был доступен через `http://localhost`. Все компоненты взаимодействуют корректно: Nginx проксирует PHP-запросы на контейнер backend.
![Site](/Images/site.png)
## Ответы на вопросы

### Каким образом в данном примере контейнеры могут взаимодействовать друг с другом?

Контейнеры могут взаимодействовать друг с другом через **сеть Docker**. В данном примере была создана пользовательская сеть `internal`, к которой подключены оба контейнера. Это позволяет контейнерам видеть друг друга по имени контейнера, например, `backend`. В рамках этой сети они могут обмениваться данными через IP-адреса или имена контейнеров, что позволяет фронтенду (Nginx) обращаться к бэкенду (PHP) по имени контейнера `backend` на порту 9000.

### Как видят контейнеры друг друга в рамках сети internal?

Контейнеры в рамках сети Docker могут обращаться друг к другу по имени контейнера, так как Docker автоматически добавляет записи в DNS-сервер для контейнеров в сети. В этом примере контейнер `frontend` (Nginx) может обращаться к контейнеру `backend` (PHP) по имени `backend`, и Nginx будет отправлять запросы на адрес `backend:9000`. Это работает благодаря тому, что оба контейнера находятся в одной сети (`internal`), и Docker разрешает имена контейнеров в IP-адреса.

### Почему необходимо было переопределять конфигурацию nginx?

Конфигурацию Nginx нужно было переопределить, чтобы настроить правильную обработку PHP-запросов. Стандартная конфигурация Nginx не обрабатывает PHP-файлы, поэтому мы изменили конфигурацию с помощью файла `default.conf`, чтобы:
- Установить правильный путь к корню сайта.
- Настроить обработку PHP-файлов с помощью FastCGI и передать запросы на контейнер backend, который использует PHP-FPM для обработки этих файлов.
- Указать, что для обработки PHP-запросов используется контейнер `backend`, работающий на порту 9000.

Без этого конфигурация Nginx не могла бы правильно передавать PHP-запросы на сервер PHP.

## Выводы
В ходе выполнения лабораторной работы был успешно развернут веб-сайт с использованием контейнеров Docker для серверов Nginx и PHP. Все компоненты работают корректно, взаимодействуя друг с другом через пользовательскую сеть Docker. Были изучены основные принципы работы с контейнерами Docker, монтирование директорий, настройка сети и конфигурация Nginx для работы с PHP.
