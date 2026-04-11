# Lesson 2

## Подготовка проектов к работе с Doctrine

В `study-on` после установки `symfony/webapp-pack` пакеты Doctrine уже присутствуют в проекте.

В `study-on.billing` после первого урока используется только `symfony/skeleton`, поэтому перед настройкой `DATABASE_URL` и `config/packages/doctrine.yaml` нужно отдельно установить Doctrine:

```bash
docker compose exec php composer require symfony/orm-pack doctrine/doctrine-migrations-bundle
```

После этого в проекте появится файл `config/packages/doctrine.yaml`, и можно продолжать настройку подключения к PostgreSQL.

## Пример настройки подключения к БД

```
DATABASE_URL="postgresql://pguser:pguser@postgres:5432/study_on?serverVersion=18&charset=utf8"

# pguser - логин и пароль основного пользователя в PostgreSQL, которые заданы в контейнере postgres
# postgres - название контейнера, в котором работает PostgreSQL
# 5432 - стандартный порт, который слушает PostgreSQL
# study_on - название БД (для StudyOn.Billing после установки Doctrine будет соответственно study_on_billing)
# serverVersion=18 - версия PostgreSQL из docker-образа postgres:18-alpine
```

## Пример настроек Doctrine в связке с PostgreSQL

```yaml
parameters:
    # Adds a fallback DATABASE_URL if the env var is not set.
    # This allows you to run cache:warmup even if your
    # environment variables are not available yet.
    # You should not need to change this value.
    env(DATABASE_URL): ''

doctrine:
    dbal:
        # configure these for your database server
        driver: 'pdo_pgsql'
        server_version: '18'
        url: '%env(resolve:DATABASE_URL)%'
        profiling_collect_backtrace: '%kernel.debug%'
    orm:
        validate_xml_mapping: true
        naming_strategy: doctrine.orm.naming_strategy.underscore
        identity_generation_preferences:
            Doctrine\DBAL\Platforms\PostgreSQLPlatform: identity
        auto_mapping: true
        mappings:
            App:
                is_bundle: false
                type: attribute
                dir: '%kernel.project_dir%/src/Entity'
                prefix: 'App\Entity'
                alias: App
```

### Полезные ссылки для изучения и настройки фикстур 
https://symfony.com/doc/current/testing.html#load-dummy-data-fixtures

https://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html#writing-fixtures

Установка библиотеки для работы с фикстурами:
```bash
docker compose exec php composer require --with-all-dependencies doctrine/doctrine-fixtures-bundle
```

## Часто используемые команды Doctrine

```bash
# Create the database
php bin/console doctrine:database:create
 
# The same in docker container
docker compose exec php bin/console doctrine:database:create

# Execute a sql
php bin/console doctrine:query:sql "select 1"
# The same in short syntax
php bin/console do:query:sql "select 1"
# The same in docker container
docker compose exec php bin/console doctrine:query:sql "select 1"

# Create a empty migration
php bin/console make:migration

# Create migration after you created entities
php bin/console do:mi:diff

# Apply the migrations
php bin/console doctrine:migrations:migrate

# Revert last migration
php bin/console doctrine:migrations:migrate prev

# Revert the migration
bin/console doctrine:migrations:execute --down 'DoctrineMigrations\Version20220403203744'
```
