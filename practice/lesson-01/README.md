# Lesson 1

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
cd study-on.billing
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

## Регистрация доменов сервисов в локальных хостах

Чтобы обращаться к проектам по доменам, необходимо зарегистрировать их в локальном hosts-файле:
```bash
$ sudo cat >> /etc/hosts

# symfony lessons
127.0.0.1 study-on.local billing.study-on.local
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

Запуск команды внутри запущенного контейнера
```bash
$ docker-compose exec php bin/console list
```

## Работа с make

Пояснения и синтаксис запуска make-команд из [Makefile](Makefile).

Запустить docker-compose
```bash
$ make up
```

Остановить docker-compose
```bash
$ make down
```

Отчистить кеш Symfony
```bash
$ make clear
```

Создать миграцию
```bash
$ make migration
```

Применить миграции
```bash
$ make migrate
```

Загрузить фикстуры
```bash
$ make fixtload
```

Установить пакет Composer
```bash
$ make require some/package
```