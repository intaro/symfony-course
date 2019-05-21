# Lesson 6

## Настройка JWTRefreshTokenBundle в study-on.billing

Из директории проекта study-on.billing выполните:
```bash
$ docker-compose exec php composer require "gesdinet/jwt-refresh-token-bundle"
```

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