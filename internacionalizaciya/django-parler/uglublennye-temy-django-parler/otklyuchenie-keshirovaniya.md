# Отключение кэширования

При желании кеширование переведенных полей можно отключить, добавив в настройки [PARLER\_ENABLE\_CACHING=False](../varianty-konfiguracii-django-parler.md#parler\_enable\_caching).

## Parler на большем количестве сайтов с таким же кешем

Если Parler работает на нескольких сайтах, которые используют один и тот же кеш, необходимо установить разные префиксы для каждого сайта, добавив в настройки [PARLER\_CACHE\_PREFIX = ‘mysite’](../varianty-konfiguracii-django-parler.md#parler\_cache\_prefix) .
