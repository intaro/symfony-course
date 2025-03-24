# Lesson 3

## Установка Encore через контейнер

```bash
docker-compose run node yarn install
docker-compose run node yarn add @symfony/webpack-encore --dev
```
Установите все необходимые зависимости
Пример package.json для Symfony 7.2
```
{
  "devDependencies": {
    "@babel/core": "^7.17.0",
    "@babel/preset-env": "^7.16.0",
    "@hotwired/stimulus": "^3.0.0",
    "@hotwired/turbo": "^7.1.1 || ^8.0",
    "@popperjs/core": "^2.11.7",
    "@symfony/stimulus-bridge": "^3.2.0",
    "@symfony/ux-turbo": "file:vendor/symfony/ux-turbo/assets",
    "@symfony/webpack-encore": "^5.1.0",
    "bootstrap": "^5.2.3",
    "core-js": "^3.23.0",
    "jquery": "^3.6.4",
    "regenerator-runtime": "^0.13.9",
    "sass": "^1.62.0",
    "sass-loader": "^16.0.5",
    "webpack": "^5.74.0",
    "webpack-cli": "^6.0.1"
  },
  "dependencies": {
    "popper.js": "^1.16.1",
    "stimulus": "^3.2.1"
  }
}

```

В репозитории есть пример [webpack.config.js](webpack.config.js)

## Make-команды для сборки фронтенда

Добавьте для удобства в Makefile следующие команды:

```
encore_dev:
	@${COMPOSE} run node yarn encore dev

encore_prod:
	@${COMPOSE} run node yarn encore production
```
