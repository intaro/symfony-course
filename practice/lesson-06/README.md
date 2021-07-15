# Lesson 6

## Просмотр списка сетей докера

```$ docker network ls```


## Проверка сети между контейнерами

Из директории проекта studyOn выполните:
```bash
$ docker-compose exec php curl billing.study-on.local
```
## Пример авторизации

В методе authenticate(Request $request) нужно использовать функцию CustomCredentials() для авторизации пользователя через billing:
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
Где $checkUser анонимная функция в которой происходит обращение к billing. В эту функцию передается два параментра: массив данных пользователей $credentials и сущность User.

## Пример регистрации

Для автоматической аутентификации пользователя после регистрации нужно добавить следующий код: 
```php
public function register(Request $request,  UserAuthenticatorInterface $authenticator,
                             BillingAuthenticator $formAuthenticator): Response
{
    //...
    if ($form->isSubmitted() && $form->isValid()) {
        //...
        return $authenticator->authenticateUser(
                $user,
                $formAuthenticator,
                $request);
        }
        //..
}
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
