# Lesson 5

## Установка бандлов 

```bash
docker compose exec php composer require symfony/security-bundle
docker compose exec php composer require --dev doctrine/doctrine-fixtures-bundle symfony/maker-bundle
```

## Генерация пользователя

```bash
docker compose exec php php bin/console make:user
```

В качестве уникального идентификатора выберите email

Укажите в entity пользователя кастомное название таблицы (user - ключевое слово в postgres, с ним возникают проблемы при выполнении запросов):

```php
#[ORM\Entity(repositoryClass: UserRepository::class)]
#[ORM\Table(name: 'billing_user')]
```

## Настройка JWT

Установите бандл

```bash
docker compose exec php composer require lexik/jwt-authentication-bundle
```

Сгенерируйте RSA-ключи для подписи JWT

```bash
mkdir -p config/jwt 
openssl genrsa -out config/jwt/private.pem -aes256 4096
openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem
```

Перенесите JWT-параметры в `.env.local` и `.env.test.local`. В `JWT_PASSPHRASE` укажите passphrase, который вы задали при генерации ключа.
```yaml
###> lexik/jwt-authentication-bundle ###
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=your-passphrase
###< lexik/jwt-authentication-bundle ###
```

## Установка бандлов для регистрации

```bash
docker compose exec php composer require jms/serializer-bundle symfony/validator
```

До реализации основного функционала создания пользователей, можно добавить нескольких с помощью фикстур. Можно использовать UserPasswordHasherInterface

Подключение к БД в контейнере через консоль:
```bash
docker exec -it study-onbilling_postgres_1 psql -U pguser -d study_on_billing
# study_on_billing/study-on - название БД
```

## Получение системного UserPasswordHasher

```php
/** @var UserPasswordHasherInterface $hasher */
$hasher = static::$container->get('security.user_password_hasher');
```

## DTO
Для регистрации удобно использовать отдельный DTO-класс с полями `$email` и `$password`.

Для валидации полей нужно использовать:

```php
namespace App\Dto;

use Symfony\Component\Validator\Constraints as Assert;

final class RegisterUserDto
{
    #[Assert\NotBlank(message: 'Email should not be blank.')]
    #[Assert\Email(message: 'Invalid email address.')]
    public string $email;

    #[Assert\NotBlank(message: 'Password should not be blank.')]
    #[Assert\Length(min: 6, minMessage: 'Password should be at least {{ limit }} characters long.')]
    public string $password;
}
```

Разбор и валидация запроса в экшне контроллера упрощенно должны выглядеть следующим образом:

```php
/** @var RegisterUserDto $userDto */
$userDto = $serializer->deserialize($request->getContent(), RegisterUserDto::class, 'json');
$errors = $validator->validate($userDto);
```

Отдельная статическая `fromDto()` не обязательна: можно собрать `User` прямо в контроллере или в отдельном сервисе.

```php
$user = new \App\Entity\User();
$user->setEmail($userDto->email);
// ...
```

## Получение текущего пользователя
Чтобы получить текущего пользователя в billing API, достаточно настроить `jwt` firewall и использовать `$this->getUser()`. Если нужно декодировать текущий токен вручную, можно использовать:
```php
  $decodedJwt = $this->jwtManager->decode($this->tokenStorageInterface->getToken());
```

## Проверка API

```bash
curl -X POST -H "Content-Type: application/json" http://billing.study-on.local:82/api/v1/auth -d '{"username":"user@intaro.ru","password":"mypass"}'
```

Должен вернуться ответ вида
```bash
{"token": "eyJ0eXAiOiJKV1QiLCJh..."}
```


## Документирование

Установите необходимые бандлы и сконфигурируйте nelmio_api_doc.yaml
```bash
docker compose exec php composer require nelmio/api-doc-bundle twig asset zircote/swagger-php
```
Кнопка `Authorize` сохраняет токен. Для методов, требующих JWT, добавьте описание Bearer security scheme в `nelmio_api_doc.yaml` и укажите security requirement у защищённых endpoint'ов.

```php
#[OA\Get(
    // ...
    security: [['Bearer' => []]],
)]
```

## Тестирование 

Установите необходимые бандлы
```bash
docker compose exec php composer require --dev phpunit/phpunit symfony/browser-kit symfony/css-selector dama/doctrine-test-bundle
```

## Передача заголовка авторизации в тестах

Чтобы при отправке запроса к api добавить заголовок "Authorization: Bearer {token}", нужно указать его следующим образом

```php
<?php
namespace App\Tests\Functional;

class BillingUserControllerTest extends WebTestCase
{
    public function testCurrentUser()
    {
        //...
        $this->client->request('GET', '/api/v1/users/current', [], [], ['HTTP_AUTHORIZATION' => 'Bearer '. $token]);
        //....
    }
    
 }
```
