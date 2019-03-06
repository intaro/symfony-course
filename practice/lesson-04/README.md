# Lesson 4

## Пример настройки подключения к тестовой БД

```
DATABASE_URL=pgsql://pguser:pguser@postgres:5432/study_on_test
```

## Создание тестовой БД

```
docker-compose exec php bin/console doctrine:database:create --env=test
docker-compose exec php bin/console doctrine:migrations:migrate --env=test
```
