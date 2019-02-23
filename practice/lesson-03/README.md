# Lesson 3

## Установка Encore через контейнер

```bash
make require encore
docker-compose run node yarn install
docker-compose run node yarn add @symfony/webpack-encore --dev
```

## Make-команды для сборки фронтенда

```
encore_dev:
	@${COMPOSE} run node yarn encore dev

encore_prod:
	@${COMPOSE} run node yarn encore production
```
