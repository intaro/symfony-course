# Lesson 2

## Пример настройки подключения к БД

```
DATABASE_URL=pgsql://pguser:pguser@postgres:5432/study_on

# pguser - логин и пароль основного пользователя в PostgreSQL, которые заданы в контейнере postgres
# postgres - название контейнера, в котором работает PostgreSQL
# 5432 - стандартный порт, который слушает PostgreSQL
# study_on - название БД (для StudyOn.Billing будет соответственно study_on_billing)
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
        server_version: '11'
        url: '%env(resolve:DATABASE_URL)%'
        connections:
          default:
            use_savepoints: true
    orm:
        auto_generate_proxy_classes: true
        naming_strategy: doctrine.orm.naming_strategy.underscore
        auto_mapping: true
        mappings:
            App:
                is_bundle: false
                type: annotation
                dir: '%kernel.project_dir%/src/Entity'
                prefix: 'App\Entity'
                alias: App
```

### Полезные ссылки для изучения и настройки фикстур 
https://symfony.com/doc/current/testing.html#load-dummy-data-fixtures

https://symfony.com/doc/current/bundles/DoctrineFixturesBundle/index.html#writing-fixtures

Установка конкретной версии:
```bash
docker-compose exec php composer require --with-all-dependencies doctrine/doctrine-fixtures-bundle:^4.0
```

## Часто используемые команды Doctrine

```bash
# Create the database
php bin/console doctrine:database:create
 
# The same in docker container
docker-compose exec php bin/console doctrine:database:create

# Execute a sql
php bin/console doctrine:query:sql "select 1"
# The same in short syntax
php bin/console do:query:sql "select 1"
# The same in docker container
docker-compose exec php bin/console doctrine:query:sql "select 1"

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
