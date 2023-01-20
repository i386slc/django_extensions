# django-parler

> «Легко перевести «сырный омлет» на «омлет из сыра».

**django-parler** обеспечивает перевод модели Django без неприятных хаков.

Особенности:

* Хорошая интеграция с админкой.
* Доступ к переведенным атрибутам, как к обычным атрибутам.
* Автоматический возврат к другим языкам по умолчанию.
* Отдельная таблица для переведенных полей, совместимая с [django-hvad](https://github.com/kristianoellegaard/django-hvad).
* Хорошо работает с другими, совместим с [django-polymorphic](https://github.com/django-polymorphic/django-polymorphic), [django-mptt](https://github.com/django-mptt/django-mptt) и т. д.:
  * Никаких взломов ORM-запросов.
  * Легко сочетается с пользовательскими классами **Manager** или **QuerySet**.
  * Легко построить модель переводов вручную, когда это необходимо.

## Приступаем к работе

* Краткое руководство пользователя
  * Установка **django-parler**
  * Краткий обзор
  * Конфигурация
* Варианты конфигурации
  * PARLER\_DEFAULT\_LANGUAGE\_CODE
  * PARLER\_LANGUAGES
  * PARLER\_ENABLE\_CACHING
  * PARLER\_CACHE\_PREFIX
  * PARLER\_SHOW\_EXCLUDED\_LANGUAGE\_TABS
  * PARLER\_DEFAULT\_ACTIVATE
* Теги шаблона
  * Получение переведенного URL
  * Изменение языка объекта
  * Примечания по безопасности потоков
* Рекомендации по производительности
  * Кеширование
  * Предварительная выборка запроса

## Углубленные темы

* Расширенные шаблоны использования
  * Переводы без резервных языков
  * Использование переведенных слагов в представлениях
  * Делаем существующие поля переводимыми
  * Добавление переведенных полей в существующую модель
  * Интеграция с **django-mptt**
  * Интеграция с **django-polymorphic**
  * Интеграция с **django-guardian**
  * Интеграция с **django-rest-framework**
  * Поддержка нескольких сайтов
  * Отключение кэширования
  * Парлер на большем количестве сайтов с таким же кешем
  * Построение модели переводов вручную
  * Настройка языковых параметров
* Совместимость с Django
  * Использование нескольких вызовов `filter()`
  * Метаполе сортировки **ordering**
  * Использование **search\_fields** в админке
  * Использование **prepopulated\_fields** в админке
* Задний план
  * Краткая история
  * Презентации
  * Схема базы данных
  * Именование пакета

## Документация API

* API
  * [parler package](https://django-parler.readthedocs.io/en/stable/api/parler.html)
  * [parler.admin module](https://django-parler.readthedocs.io/en/stable/api/parler.admin.html)
  * [parler.cache module](https://django-parler.readthedocs.io/en/stable/api/parler.cache.html)
  * [parler.fields module](https://django-parler.readthedocs.io/en/stable/api/parler.fields.html)
  * [parler.forms module](https://django-parler.readthedocs.io/en/stable/api/parler.forms.html)
  * [parler.managers module](https://django-parler.readthedocs.io/en/stable/api/parler.managers.html)
  * [parler.models module](https://django-parler.readthedocs.io/en/stable/api/parler.models.html)
  * [parler.signals module](https://django-parler.readthedocs.io/en/stable/api/parler.signals.html)
  * [parler.utils package](https://django-parler.readthedocs.io/en/stable/api/parler.utils.html)
  * [parler.views module](https://django-parler.readthedocs.io/en/stable/api/parler.views.html)
  * [parler.widgets module](https://django-parler.readthedocs.io/en/stable/api/parler.widgets.html)
* Список изменений
  * [Changes in 2.3 (2021-11-18)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-3-2021-11-18)
  * [Changes in 2.2.1 (2021-10-18)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-2-1-2021-10-18)
  * [Changes in 2.2 (2020-09-06)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-2-2020-09-06)
  * [Changes in 2.1 (2020-08-05)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-1-2020-08-05)
  * [Changes in 2.0.1 (2020-01-02)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-0-1-2020-01-02)
  * [Changes in 2.0 (2019-07-26)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-2-0-2019-07-26)
  * [Changes in 1.9.2 (2018-02-12)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-9-2-2018-02-12)
  * [Changes in 1.9.1 (2017-12-06)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-9-1-2017-12-06)
  * [Changes in 1.9 (2017-12-04)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-9-2017-12-04)
  * [Changes in 1.8.1 (2017-11-20)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-8-1-2017-11-20)
  * [Changes in 1.8 (2017-06-20)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-8-2017-06-20)
  * [Changes in 1.7 (2016-11-29)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-7-2016-11-29)
  * [Changes in 1.6.5 (2016-07-11)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-6-5-2016-07-11)
  * [Changes in 1.6.4 (2016-06-14)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-6-4-2016-06-14)
  * [Changes in 1.6.3 (2016-05-05)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-6-3-2016-05-05)
  * [Changes in 1.6.2 (2016-03-08)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-1-6-2-2016-03-08)
  * [Changes in version 1.6.1 (2016-02-11)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-6-1-2016-02-11)
  * [Changes in version 1.6 (2015-12-29)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-6-2015-12-29)
  * [Changes in version 1.5.1 (2015-10-01)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-5-1-2015-10-01)
  * [Changes in version 1.5 (2015-06-30)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-5-2015-06-30)
  * [Changes in version 1.4 (2015-04-13)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-4-2015-04-13)
  * [Changes in version 1.3 (2015-03-13)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-3-2015-03-13)
  * [Changes in version 1.2.1 (2014-10-31)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-2-1-2014-10-31)
  * [Changes in version 1.2 (2014-10-30)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-2-2014-10-30)
  * [Changes in version 1.1.1 (2014-10-14)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-1-1-2014-10-14)
  * [Changes in version 1.1 (2014-09-29)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-1-2014-09-29)
  * [Changes in version 1.0 (2014-07-07)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-1-0-2014-07-07)
  * [Changes in version 0.9.4 (beta)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-0-9-4-beta)
  * [Changes in version 0.9.3 (beta)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-0-9-3-beta)
  * [Changes in version 0.9.2 (beta)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-0-9-2-beta)
  * [Changes in version 0.9.1 (beta)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-0-9-1-beta)
  * [Changes in version 0.9 (beta)](https://django-parler.readthedocs.io/en/stable/changelog.html#changes-in-version-0-9-beta)

## Дорожная карта

Следующие функции находятся на радаре для будущих выпусков:

* Поддержка многоуровневого наследования моделей
* Улучшение использование запросов, например, путем добавления объектов «Предварительная выборка».

Пожалуйста, вносите свои улучшения или работайте над этими областями!

## Индексы и таблицы

* [Index](https://django-parler.readthedocs.io/en/stable/genindex.html)
* [Module Index](https://django-parler.readthedocs.io/en/stable/py-modindex.html)
* [Search Page](https://django-parler.readthedocs.io/en/stable/search.html)
