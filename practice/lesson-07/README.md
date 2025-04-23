# Lesson 7

## Настройка JWTRefreshTokenBundle в StudyOn.Billing

Из директории проекта StudyOn.Billing выполните команды:
```bash
$ docker-compose exec php composer require "gesdinet/jwt-refresh-token-bundle"
$ docker-compose exec php bin/console do:mi:diff
$ docker-compose exec php bin/console do:mi:mi -n
```

Будет создана таблица refresh_tokens

Команда для подключения к бд через консоль: 
```bash
docker exec -it study-onbilling_postgres_1 psql -U pguser -d study_on_billing
# study_on_billing/study-on - название БД
```

Настройте параметры в config/packages/gesdinet_jwt_refresh_token.yaml

### Настройка роута для обновления токена
#### Symfony 4.4
```yaml
# config/routes.yaml
api_refresh_token:
    path:       /api/v1/token/refresh
    controller: gesdinet.jwtrefreshtoken::refresh
# ...
```
#### Symfony 5.4+
```yaml
# config/routes.yaml
api_refresh_token:
    path: /api/v1/token/refresh
# ...
```
#### Symfony 7.2
```yaml
# config/routes/gesdinet_jwt_refresh_token.yaml
gesdinet_jwt_refresh_token:
    path: /api/v1/token/refresh
    methods: POST
```

### Настройка security.yaml

#### Symfony 4.4
```yaml
# config/packages/security.yaml
security:
    firewalls:
        # put it before all your other firewall API entries
        refresh:
            pattern:  ^/api/token/refresh
            stateless: true
            anonymous: true
    # ...

    access_control:
        # ...
        - { path: ^/api/v1/token/refresh, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        # ...
# ...
```
#### Symfony 5.4+
```yaml
# config/packages/security.yaml
security:
    # this config is only required on Symfony 5.4, you can leave it out on Symfony 6
    enable_authenticator_manager: true

    firewalls:
        api_token_refresh:
            pattern: ^/api/v1/token/refresh
            stateless: true
            refresh_jwt:
                check_path: /api/v1/token/refresh # or, you may use the `api_refresh_token` route name
    # ...

    access_control:
        # ...
        - { path: ^/api/v1/token/refresh, roles: PUBLIC_ACCESS }
        # ...
# ...
```

#### Symfony 7.2

в security.yaml refresh_token файрвол должен находиться перед api (или следуйте инструкции в https://github.com/markitosgv/JWTRefreshTokenBundle/blob/master/README.md)

```yaml
firewalls:
    api_token_refresh:
        pattern: /token/refresh
        stateless: true
        provider: app_user_provider
        refresh_jwt:
            check_path: /token/refresh
        ...
    api: 
        ...
    access_control:
        ...
```

### Создание refreshToken при регистрации пользователя 

```php
use Gesdinet\JWTRefreshTokenBundle\Model\RefreshTokenManagerInterface;

...

class SomeController extends AbstractController
{

    public function register(
        ...
        RefreshTokenGeneratorInterface $refreshTokenGenerator,
        RefreshTokenManagerInterface $refreshTokenManager
    )
    {
        ...
        
        $refreshToken = $refreshTokenGenerator->createForUserWithTtl(
            $user,
            (new \DateTime())->modify('+1 month')->getTimestamp()
        );
        $refreshTokenManager->save($refreshToken);      
       
       ...
   }
}

```

Для проверки запросами с хоста: 
```bash
curl -X GET -H "Authorization: Bearer PLACE_HERE_TOKEN" 'http://billing.study-on.local:82/api/v1/users/current'
curl -X POST -d refresh_token="PLACE_HERE_TOKEN" 'http://billing.study-on.local:82/api/v1/users/current'
```