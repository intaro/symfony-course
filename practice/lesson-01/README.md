# Lesson 1

## Создание пользователей и БД в PostgreSQL

Заводим пользователя `study_on`.
```bash
$ createuser -P -e study_on
Enter password for new role: study_on
Enter it again: study_on
CREATE ROLE study_on PASSWORD 'md582280c92998413dc4bd3ae4e33835049' NOSUPERUSER NOCREATEDB NOCREATEROLE INHERIT LOGIN;
```

Cоздаем 2 базы данных, владельцем которых будет данный пользователь.
```bash
$ createdb -O study_on -e study_on
CREATE DATABASE study_on OWNER study_on;
$ createdb -O study_on -e study_on_billing
CREATE DATABASE study_on_billing OWNER study_on;
```

## Создание скелетонов проектов

Создаем папку под проекты.
```bash
cd ~ && mkdir sf-lessons
```

Создаем проект StudyOn из веб-скелетона (не забываем подставить свое имя в адресе git-репозитория).
```bash
cd ~/sf-lessons
composer create-project symfony/website-skeleton study-on
cd study-on
git init .
git remote add origin https://github.com/sfuser/study-on.git
```

Создаем проект StudyOn.Billing из обычного скелетона. Также сразу добавим PHPUnit для тестов (в website-skeleton он устанавливается сразу).
```bash
cd ~/sf-lessons
composer create-project symfony/skeleton study-on.billing
cd  study-on.billing
composer req symfony/phpunit-bridge
git init .
git remote add origin https://github.com/sfuser/study-on.billing.git
```

## Указание порта, на котором будет работать Nginx

Для запуска проекта, необходимо указать порт Nginx, по которому вы будете обращаться к проекту:
```bash
$ cat >> .env

# external port for nginx
NGINX_PORT=84
^C
```

## Работа с Docker Compose

Ниже показаны примеры типовых операций с docker-compose. Все операции выполняются в папке, где расположен `docker-compose.yml`.

Запуск сборки
```bash
$ docker-compose up -d
```

Остановка работы сборки
```bash
$ docker-compose down
```

