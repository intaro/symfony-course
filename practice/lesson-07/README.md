# Lesson 7

## Настройка JWTRefreshTokenBundle в StudyOn.Billing

Из директории проекта StudyOn.Billing выполните команды:
```bash
docker compose exec php composer require "gesdinet/jwt-refresh-token-bundle"
docker compose exec php bin/console doctrine:migrations:diff
docker compose exec php bin/console doctrine:migrations:migrate -n
```

Будет создана таблица refresh_tokens

Команда для подключения к БД через консоль:
```bash
docker compose exec postgres psql -U pguser -d study_on_billing
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
            pattern: ^/api/v1/token/refresh
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

В `security.yaml` firewall для refresh-токена должен находиться перед `api` (или следуйте инструкции в [README бандла](https://github.com/markitosgv/JWTRefreshTokenBundle/blob/master/README.md).

```yaml
firewalls:
    api_token_refresh:
        pattern: ^/api/v1/token/refresh
        stateless: true
        refresh_jwt:
            check_path: /api/v1/token/refresh
    api:
        ...

access_control:
    - { path: ^/api/v1/token/refresh, roles: PUBLIC_ACCESS }
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
            30 * 24 * 3600 // TTL в секундах (30 дней)
        );
        $refreshTokenManager->save($refreshToken);      
       
       ...
   }
}

```

Для проверки запросами с хоста: 
```bash
curl -X GET -H "Authorization: Bearer PLACE_HERE_TOKEN" 'http://billing.study-on.local:82/api/v1/users/current'
curl -X POST -d "refresh_token=PLACE_HERE_REFRESH_TOKEN" 'http://billing.study-on.local:82/api/v1/token/refresh'
```

Для проверки из контейнера `php` в StudyOn.Billing:
```bash
docker compose exec php curl -sS -X POST http://nginx/api/v1/token/refresh \
  -d "refresh_token=PLACE_HERE_REFRESH_TOKEN"
```
