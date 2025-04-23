# Lesson 5

## Установка бандлов 

```bash
docker-compose exec php composer require doctrine maker-bundle symfony/security-bundle doctrine/doctrine-fixtures-bundle
```

## Генерация пользователя

```bash
docker-compose exec php bin/console make:user
```

В качестве уникального идентификатора выберите email

Укажите в аннотациях модели пользователя кастомное название таблицы (user - ключевое слово в postgres, с ним возникают проблемы при выполнении запросов):

```php
/**
 * @ORM\Entity(repositoryClass="App\Repository\UserRepository")
 * @ORM\Table(name="billing_user")
 */
```

## Настройка JWT

Установите бандл

```bash
docker-compose exec php composer require lexik/jwt-authentication-bundle
```

Сгенерируйте ssh-ключи 

```bash
mkdir -p config/jwt 
openssl genrsa -out config/jwt/private.pem -aes256 4096
openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem
```

Убедитесь, что в .env стоит верное значение private key/passphrase
```yaml
###> lexik/jwt-authentication-bundle ###
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=your-passphrase
###< lexik/jwt-authentication-bundle ###
```

## Установка бандлов для регистрации

```bash
docker-compose exec php composer require jms/serializer-bundle symfony/validator
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
В структуру класса DTO входят переменные $username и $password.

Для валидации полей нужно использовать:

```php
namespace App\Dto;

use Symfony\Component\Validator\Constraints as Assert;

class UserDto
{
    /**
     * @Assert\NotBlank(message="Name is mandatory")
     * @Assert\Email( message="Invalid email address" )
     */
    public string $username;
    //...
}
```

Разбор и валидация запроса в экшне контроллера упрощенно должны выглядеть следующим образом:

```bash
$serializer = SerializerBuilder::create()->build();
$userDto = $serializer->deserialize($request->getContent(), UserDto::class, 'json');
$errors = $validator->validate($userDto);
```

Для создания объекта сущности из DTO нужно создать соответствующую статическую функцию, которая будет формировать новый объект сущности из DTO-объекта:

```bash
$user = \App\Entity\User::fromDto($userDto);
```

## Получение текущего пользователя
Чтобы получить текущего пользователя нужно получить токен и ваш публичный ключ. После чего нужно проверить токен и декодировать его, с помощью функции:
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
docker-compose exec php composer require nelmio/api-doc-bundle twig asset
```
Кнопка Authorize сохраняет токен. Для дальнейшего его использования в методе получения пользователя нужно добавить данную анотацию.

```bash
@Security(name="Bearer")
```

## Тестирование 

Установите необходимые бандлы
```bash
docker-compose exec php composer require symfony/dom-crawler symfony/browser-kit --dev
```

## Передача заголовка авторизации в тестах

Чтобы при отправке запроса к api добавить заголовок "Authorization: Bearer {token}", нужно указать его следующим образом

```php
<?php
namespace App\Tests;

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

