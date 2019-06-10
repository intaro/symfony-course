# Lesson 8

## Настройка почтовых уведомлений в StudyOn.Billing

Для symfony версии 4.4 и выше установите пакет symfony/mailer

```bash
$ docker-compose exec php composer require symfony/mailer
```

В env.local укажите параметр MAILER_DSN
```
MAILER_DSN=smtp://mailhog:1025
```

Для версий 4.3 и ниже используйте SwiftMailer

```bash
$ docker-compose exec php composer require symfony/swiftmailer-bundle
```
В env.local укажите параметр MAILER_URL

```
MAILER_URL=smtp://mailhog:1025
```


## Настройка MailHog

Добавьте в [docker-compose.yml](study-on.billing/docker-compose.yml) строки

```
  mailhog:
      image: mailhog/mailhog
      container_name: 'mailhog'
      ports:
        - "1025:1025"
        - "8025:8025"
```

Перезапустите контейнер.   
Интерфейс mailhog должен открыться по адресу http://localhost:8025/

## Рендеринг twig-шаблонов внутри комманда

Разместите в проекте сервис [Twig](study-on.billing/src/Service/Twig.php) из материалов урока

Используйте его в вашем комманде следующим образом 

```
<?php

namespace App\Command;

use App\Service\Twig;
//...

class SomeCommand extends Command
{
    private $twig;
    //...

    public function __construct(Twig $twig)
    {
        $this->twig = $twig;
        parent::__construct();
    }
    //...
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        $html = $this->twig->render(
            'some.template.html.twig',
            ['data' => $data]
        );
        //your code
    }
}
```
