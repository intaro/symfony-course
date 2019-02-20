# Lesson 2

## Пример настройки подключения к БД

```
DATABASE_URL=pgsql://study_on:study_on@127.0.0.1:5432/study_on
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
        server_version: '9.6'
        url: '%env(resolve:DATABASE_URL)%'
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

## Часто используемые команды Doctrine

```bash
# Execute a sql
bin/console doctrine:query:sql "select 1"
# The same in short syntax
bin/console do:query:sql "select 1"
# The same in docker container
docker-compose exec php bin/console doctrine:query:sql "select 1"

# Create a migration
bin/console make:migration

# Apply the migrations
bin/console doctrine:migrations:migrate

# Revert the migration
bin/console doctrine:migrations:execute --down 20190203151634
```