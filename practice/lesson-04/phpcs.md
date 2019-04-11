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

[[https://github.com/intaro/symfony-course/blob/master/practice/lesson-4/img/phpcs_settings.png]]

Смените уровень предупреждений в Inspections и укажите путь к файлу с настройками:

[[https://github.com/intaro/symfony-course/blob/master/practice/lesson-4/img/phpcs_settings.png]]