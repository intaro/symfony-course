# Lesson 1

## Создание скелетонов проектов

Создаем папку под проекты
```bash
cd ~ && mkdir sf-lessons```

Создаем проект StudyOn из веб-скелетона (не забываем подставить свое имя в адресе git-репозитория).
```bash
cd ~/sf-lessons
composer create-project symfony/website-skeleton study-on
cd study-on
git init .
git remote add origin https://github.com/sfuser/study-on.git```

Создаем проект StudyOn.Billing из обычного скелетона. Также сразу добавим PHPUnit для тестов (в website-skeleton он устанавливается сразу).
```bash
cd ~/sf-lessons
composer create-project symfony/skeleton study-on.billing
cd  study-on.billing
composer req symfony/phpunit-bridge
git init .
git remote add origin https://github.com/sfuser/study-on.billing.git```