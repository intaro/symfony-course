# PHP CS

Установите codesniffer в дирректории проекта

```
composer require squizlabs/php_codesniffer
```

Скопируйте создавшийся файл phpcs.xml.dist в phpcs.xml, он содержит настройки phpcs, в частности, указание стандарта PCR-2

```
cp phpcs.xml.dist phpcs.xml
```

Проверьте, что в шторме корректно установился code sniffer в настройках:

![Quality Tools](https://raw.githubusercontent.com/intaro/symfony-course/master/practice/lesson-04/img/phpcs_settings.png "Quality tools")

Смените уровень предупреждений в Inspections и укажите путь к файлу с настройками:

![Inspections](https://raw.githubusercontent.com/intaro/symfony-course/master/practice/lesson-04/img/phpcs_inspections.png "Inspections")
