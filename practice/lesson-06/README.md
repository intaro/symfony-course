# Lesson 6

## Просмотр списка сетей докера

```$ docker network ls```


## Проверка сети между контейнерами

Из директории проекта studyOn выполните:
```bash
$ docker-compose exec php curl billing.study-on.local
```
## Пример авторизации

В методе authenticate(Request $request) нужно использовать функцию UserBadge() для загрузки пользователя через billing и функцию CustomCredentials() для проверки.
```php
public function authenticate(Request $request): PassportInterface
{
    //...
    return new Passport(
            new UserBadge($credentials, $loadUser),
            new CustomCredentials($checkUser, $credentials['email']),
            [
                new CsrfTokenBadge('authenticate', $request->get('_csrf_token')),
            ]
        );
}
```
Где $loadUser анонимная функция в которой происходит обращение к billing. В эту функцию строкой передается $credentials. Если нет надобности использовать CustomCredentials(), то нужно заменить new Passport() на new SelfValidatingPassport().

Еще пример упрощенной авторизации в BillingAuthenticator.php

```php
public function authenticate(Request $request): PassportInterface
{
    //...
    $request->getSession()->set(Security::LAST_USERNAME, $email);

    $userLoader = function ($token): UserInterface {
        return User::fromDto($this->billingClient->getCurrentUser($token))->setApiToken($token);
    };

    return new SelfValidatingPassport(
        new UserBadge($token, $userLoader),
        [
            new CsrfTokenBadge('authenticate', $request->get('_csrf_token'))
        ]
    );
}
```

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
