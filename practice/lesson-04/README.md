# Lesson 4

## Пример настройки подключения к тестовой БД

```
DATABASE_URL=pgsql://pguser:pguser@postgres:5432/study_on_test
```

Для установки версии и ее зависимостей для php8.4
```bash
composer require --with-all-dependencies doctrine/doctrine-fixtures-bundle:^4.0
```

## Создание и настройка тестовой БД

```bash
docker-compose exec php bin/console doctrine:database:create --env=test
docker-compose exec php bin/console doctrine:migrations:migrate --env=test
docker-compose exec php bin/console do:fi:load --env=test
```

## Make-команды для запуска тестов

```
phpunit:
	@${PHP} bin/phpunit
```

Запуск тестов:
```bash
make phpunit
```

## Генерация заготовок классов тестов

```
docker-compose exec php bin/console make:functional-test
```

## Создание теста

Класс с тестами в идеале должен наследовать от WebTestCase или PantherTestCase.
Достаточно WebTestCase

После каждого теста нужно вернуть состояние БД в исходное состояние, это можно сделать с помощью https://symfony.com/doc/current/testing.html#resetting-the-database-automatically-before-each-test 
Более подробная информация -https://github.com/dmaicher/doctrine-test-bundle

```php
public function testSomething(): void
    {
        $client = static::createClient();
        
        $url = '/course/';

        $crawler = $client->request('GET', $url);
        
        $link = $crawler->selectLink('Подробнее')->link();
        $crawler = $client->click($link);
	
        $this->assertResponseOk();
    }
```
