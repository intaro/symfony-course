# Lesson 3

## Установка Encore через контейнер

Если проект был создан на современном Symfony skeleton, в нём уже может быть настроен `AssetMapper`/`importmap`.
В этом задании фронтенд переводится на `Webpack Encore`, поэтому после его подключения проверьте, что в проекте не осталось двух конкурирующих способов подключения фронтенда одновременно.

```bash
docker compose run --rm node yarn install
docker compose run --rm php composer require symfony/webpack-encore-bundle
```

После установки `symfony/webpack-encore-bundle` Symfony recipe обычно уже создаёт:

- `package.json`
- `webpack.config.js`
- `config/packages/webpack_encore.yaml`

И может частично обновить `templates/base.html.twig`.

Дальше состав зависимостей и точный вид `package.json` лучше собрать самостоятельно, исходя из того, какой стек вы реально используете в проекте.
Ниже приведён минимальный ориентир для этого задания.

Минимальный пример `package.json` для задания:

```
{
  "devDependencies": {
    "@babel/core": "^7.17.0",
    "@babel/preset-env": "^7.16.0",
    "@popperjs/core": "^2.11.8",
    "@symfony/webpack-encore": "^6.0.0",
    "bootstrap": "^5.3.3",
    "core-js": "^3.38.0",
    "regenerator-runtime": "^0.13.9",
    "sass": "^1.93.0",
    "sass-loader": "^16.0.5",
    "webpack": "^5.72",
    "webpack-cli": "^6.0.0"
  },
  "license": "UNLICENSED",
  "private": true,
  "scripts": {
    "dev-server": "encore dev-server",
    "dev": "encore dev",
    "watch": "encore dev --watch",
    "build": "encore production --progress"
  }
}
```

Установить npm/yarn-зависимости:

```bash
docker compose run --rm node yarn install
```

В репозитории есть пример [webpack.config.js](webpack.config.js)

## Очистка от AssetMapper

Если проект изначально был создан с `AssetMapper`, после перехода на Encore полезно удалить старые артефакты:

- `symfony/asset-mapper` из `composer.json`
- `config/packages/asset_mapper.yaml`
- `importmap.php`
- `importmap:install` из `composer.json` scripts

Также обратите внимание на `templates/base.html.twig`: после переключения на Encore там не должно остаться старого способа подключения фронтенда.

## Make-команды для сборки фронтенда

Добавьте для удобства в Makefile следующие команды:

```
NODE=$(COMPOSE) run --rm node

encore_dev:
	@${NODE} yarn dev

encore_watch:
	@${NODE} yarn watch

encore_prod:
	@${NODE} yarn build
```

## Bootstrap и формы Symfony

После подключения Bootstrap проверьте, как в проекте рендерятся Symfony-формы.
Если вы хотите, чтобы `form_row()` сразу генерировал bootstrap-разметку, посмотрите в сторону bootstrap form theme из документации Symfony.

## Кастомные страницы ошибок

HTML-шаблоны ошибок в Symfony нужно размещать в:

```text
templates/bundles/TwigBundle/Exception/
```

Минимальный набор файлов для этого задания:

- `error403.html.twig`
- `error404.html.twig`
- `error.html.twig` для остальных ошибок, включая `5xx`

Важно: одного `cache:clear` в `prod` для проверки недостаточно.
Чтобы увидеть кастомные страницы ошибок в браузере, само HTTP-приложение тоже должно реально работать с `APP_ENV=prod`.
