# Lesson 7

## Настройка JWTRefreshTokenBundle в StudyOn.Billing

Из директории проекта StudyOn.Billing выполните:
```bash
$ docker-compose exec php composer require "gesdinet/jwt-refresh-token-bundle"
```

Сгенерируйте и примените миграции.

### Настройка роута для обновления токена
#### Symfony 4.4
```
# config/routes.yaml
api_refresh_token:
    path:       /api/v1/token/refresh
    controller: gesdinet.jwtrefreshtoken::refresh
# ...
```
#### Symfony 5.4+
```
# config/routes.yaml
api_refresh_token:
    path: /api/v1/token/refresh
# ...
```

### Настройка security.yaml

#### Symfony 4.4
```
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
```
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

### Создание refreshToken при регистрации пользователя 

```
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