# Lesson 7

## Настройка JWTRefreshTokenBundle в StudyOn.Billing

Из директории проекта StudyOn.Billing выполните:
```bash
$ docker-compose exec php composer require "gesdinet/jwt-refresh-token-bundle"
```

Сгенерируйте и примените миграции.

### Настройка метода обновления токена
```
use Gesdinet\JWTRefreshTokenBundle\Model\RefreshTokenManagerInterface;
...
       
class SomeController extends AbstractController
{
    
    ...
    
    /**
     * @Route("/api/v1/token/refresh", name="refresh", methods={"POST"})
     * ...
     */
    public function refresh(Request $request, RefreshToken $refreshService)
    {
        return $refreshService->refresh($request);
    }
}    
```

В config/services.yaml добавьте алиас для сервиса JWTRefreshTokenBundle
```
    Gesdinet\JWTRefreshTokenBundle\Service\RefreshToken:
        alias: "gesdinet.jwtrefreshtoken"
```

Разрешите анонимный доступ к методу обновления токена в config/packages/security.yaml

```
    firewalls:
    # ...
        refresh:
            pattern:  ^/api/v1/token/refresh
            stateless: true
            anonymous: true
    # ...

    access_control:
        # ...
        - { path: ^/api/v1/token/refresh, roles: IS_AUTHENTICATED_ANONYMOUSLY }
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
        RefreshTokenManagerInterface $refreshTokenManager
    )
    {
        ...
        
        $refreshToken = $refreshTokenManager->create();
        $refreshToken->setUsername($user->getEmail());
        $refreshToken->setRefreshToken();
        $refreshToken->setValid((new \DateTime())->modify('+1 month'));
        $refreshTokenManager->save($refreshToken);
       //в $refreshToken->getRefreshToken() строка токена
       
       
       ...
   }
}
```
В config/services.yaml добавьте алиас для сервиса RefreshTokenManager
    
```
    Gesdinet\JWTRefreshTokenBundle\Model\RefreshTokenManagerInterface:
        alias: "gesdinet.jwtrefreshtoken.refresh_token_manager"
```