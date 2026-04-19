# Lesson 6

## Просмотр списка сетей докера

```bash
docker network ls
```


## Проверка сети между контейнерами

Из директории проекта `study-on` выполните:
```bash
docker compose exec php curl -sS http://billing.study-on.local
```

Если не работает, проверьте:
- контейнер nginx в billing имеет имя `billing.study-on.local`;
- проект `study-on` подключен к внешней сети billing;
- имя сети в `study-on/docker-compose.yml` совпадает с фактическим из `docker network ls`.

## Пример авторизации

В актуальном Symfony удобно использовать `SelfValidatingPassport` и загружать пользователя через `UserBadge` callback.

Идея реализации:
- взять `email` и `password` из формы;
- сохранить `LAST_USERNAME` в сессию;
- обратиться в billing (`/api/v1/auth`) и получить JWT-токен;
- по токену запросить текущего пользователя (`/api/v1/users/current`);
- собрать `User` (email, roles, apiToken);
- вернуть `SelfValidatingPassport` с `CsrfTokenBadge` и `RememberMeBadge`;
- при ошибке авторизации бросать `CustomUserMessageAuthenticationException`;
- при недоступности billing показать пользователю сообщение о временной недоступности сервиса.

```php
public function authenticate(Request $request): PassportInterface
{
    // 1) Получите email/password из request.
    // 2) Вызовите billingClient->auth(...), обработайте ошибку/отсутствие token.
    // 3) Вызовите billingClient->getCurrentUser($token), получите username/roles.
    // 4) Соберите App\Security\User и положите туда apiToken.
    // 5) Верните SelfValidatingPassport(new UserBadge(...), [CsrfTokenBadge, RememberMeBadge]).
}
```

Важно для `security.yaml`:
- используйте `custom_authenticators` (массив), а не устаревший `custom_authenticator`.

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

Создайте mock-класс для BillingClient:

```php
namespace App\Tests\Mock;

use App\Service\BillingClient;

final class BillingClientMock extends BillingClient
{
    //...
}
```

Рекомендуемый способ для актуального Symfony: подменять сервис на уровне `config/services_test.yaml`, а не через `$container->set()` в каждом тесте.

```yaml
# config/services_test.yaml
services:
    App\Tests\Mock\BillingClientMock:
        public: true

    App\Service\BillingClient:
        alias: App\Tests\Mock\BillingClientMock
        public: true
```

В тестах:
- используйте обычный `createClient()`;
- если тест делает несколько последовательных запросов, добавляйте `$client->disableReboot();`.

Если нужен вариант с ручной подменой в тесте:

```php
$client = static::createClient();

// запрещаем перезагрузку ядра, чтобы не сбросилась подмена сервиса при запросе
$client->disableReboot();

$client->getContainer()->set(
    App\Service\BillingClient::class,
    new BillingClientMock()
);

$client->request(...);
```
