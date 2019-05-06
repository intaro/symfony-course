# Lesson 6

## Проверка сети между контейнерами

Из директории проекта studyOn выполните:
```bash
$ docker-compose exec php curl billing.study-on.local
```

## Тестирование сервисов

Создайте mock-класс для BillingClient
```php
namespace App\Tests\Mock;

use App\Service\BillingClient;

class BillingClientMock extends BillingClient
{
    //...
}
```

Подмените оригинальный сервис в тестах
```php
$client = static::createClient();

// запрещаем перезагрузку ядра, чтобы не сбросилась подмена сервиса при запросе
$client->disableReboot();

$client->getContainer()->set(
    'App\Service\BillingClient', 
    new BillingClientMock('')
);

$client->request(...);
```