# Lesson 6

## Просмотр списка сетей докера

```$ docker network ls```


## Проверка сети между контейнерами

Из директории проекта studyOn выполните:
```bash
$ docker-compose exec php curl billing.study-on.local
```
## Пример аутентификации

В методе authenticate(Request $request) нужно использовать функцию CustomCredentials() для аутентификаци пользователя через billing:
```php
public function authenticate(Request $request): PassportInterface
{
    //...
    return new Passport(
            new UserBadge($email),
            new CustomCredentials($checkUser, $credentials),
            [
                new CsrfTokenBadge('authenticate', $request->get('_csrf_token')),
            ]
        );
}
```
Где $checkUser анонимная функция в которой происходит обращение к billing. В эту функцию передается два параментра: массив данных пользователей $credetioals и сущность User.

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
