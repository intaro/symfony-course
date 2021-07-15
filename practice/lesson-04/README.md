# Lesson 4

## Пример настройки подключения к тестовой БД

```
DATABASE_URL=pgsql://pguser:pguser@postgres:5432/study_on_test
```

## Создание тестовой БД

```bash
docker-compose exec php bin/console doctrine:database:create --env=test
docker-compose exec php bin/console doctrine:migrations:migrate --env=test
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

```php
public function testSomething(): void
    {
        $client = AbstractTest::getClient();
        $url = '/course/';

        $crawler = $client->request('GET', $url);
        
        $link = $crawler->selectLink('Подробнее')->link();
        $crawler = $client->click($link);
	
        $this->assertResponseOk();
    }
```
