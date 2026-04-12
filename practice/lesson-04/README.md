# Lesson 4

## Пример настройки подключения к тестовой БД

Если в проекте для окружения `test` уже используется `doctrine.dbal.dbname_suffix`, не добавляйте суффикс `_test` в `DATABASE_URL` вручную.

```
DATABASE_URL=pgsql://pguser:pguser@postgres:5432/study_on
```

Для установки фикстур:
```bash
composer require --dev doctrine/doctrine-fixtures-bundle
```

Для изоляции тестов по БД:

```bash
composer require --dev dama/doctrine-test-bundle
```

После установки `DAMADoctrineTestBundle` проверьте:

- что bundle подключен в `config/bundles.php` для окружения `test`;
- что PHPUnit extension подключен в `phpunit.dist.xml`;
- что используется совместимая версия `phpunit/phpunit`.

## Создание и настройка тестовой БД

```bash
docker compose exec php php bin/console doctrine:database:create --env=test
docker compose exec php php bin/console doctrine:migrations:migrate --env=test --no-interaction
docker compose exec php php bin/console doctrine:fixtures:load --env=test --no-interaction
```

## Make-команды для запуска тестов

```
phpunit:
	@${PHP} php bin/phpunit
```

Запуск тестов:
```bash
make phpunit
```

При корректной настройке тестового окружения и при отсутствии тестов команда должна успешно запускать PHPUnit и сообщать, что тесты ещё не найдены.

## Генерация заготовок классов тестов

```
docker compose exec php php bin/console make:functional-test
```

## Создание теста

Класс с тестами в идеале должен наследовать от WebTestCase или PantherTestCase.
Достаточно WebTestCase

После каждого теста состояние БД должно возвращаться к исходному.
Для этого в задании используется `DAMADoctrineTestBundle`.

Тесты должны быть детерминированными:

- не зависеть от вручную подготовленного состояния БД между запусками;
- работать с тестовыми фикстурами;
- проверять пользовательские сценарии через переходы по страницам, ссылки, кнопки и формы.

```php
public function testSomething(): void
{
    $client = static::createClient();

    $crawler = $client->request('GET', '/courses');

    $link = $crawler->selectLink('Подробнее')->link();
    $client->click($link);

    $this->assertResponseIsSuccessful();
}
```
