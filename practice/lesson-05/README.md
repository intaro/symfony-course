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


## Применение миграций

```bash
docker-compose exec php bin/console doctrine:database:create
make migration
make migrate
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

## Проверка api

```bash
curl -X POST -H "Content-Type: application/json" http://billing.study-on.local:82/api/auth -d '{"username":"user@intaro.ru","password":"mypass"}'
```

Должен вернуться ответ вида
```bash
{"token": "eyJ0eXAiOiJKV1QiLCJh..."}
```

Расширенный ответ
```bash
{"token": "eyJ0eXAiOiJKV1QiLCJh...", "roles": ["ROLE_USER", "ROLE_SUPER_ADMIN"]}
```

## Установка бандлов для регистрации

```bash
docker-compose exec php composer require jms/serializer-bundle symfony/validator
```


## DTO

Разбор и валидация запроса в экшне контроллера упрощенно должны выглядеть следующим образом:

```bash
$userDto = $this->deserialize($request, \App\Model\Request\User::class);
$errors = $validator->validate($userDto);
```

Для создания объекта сущности из DTO нужно создать соответствующую статическую функцию, которая будет формировать новый объект сущности из DTO-объекта:

```bash
$user = \App\Entity\User::fromDto($userDto);
```

## Документирование

Установите необходимые бандлы
```bash
docker-compose exec php composer require nelmio/api-doc-bundle twig asset
```

## Тестирование 

Установите необходимые бандлы
```bash
docker-compose exec php composer require symfony/dom-crawler symfony/browser-kit --dev
```