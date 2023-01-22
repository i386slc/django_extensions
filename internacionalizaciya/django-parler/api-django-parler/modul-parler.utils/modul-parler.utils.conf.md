# Модуль parler.utils.conf

Оболочки конфигурации, которые используются для [PARLER\_LANGUAGES](../../varianty-konfiguracii-django-parler.md#parler\_languages).

## _class_ parler.utils.conf.LanguagesSetting

Базы (родители): **dict**

Это фактический тип объекта настройки [PARLER\_LANGUAGES](../../varianty-konfiguracii-django-parler.md#parler\_languages). Помимо обычного поведения словаря **dict**, он также добавляет некоторые дополнительные методы.

### get\_active\_choices (language\_code=None, site\_id=None)

Узнайте, какие переводы должны быть видны на сайте. Он возвращает список либо с одним выбором (текущий язык), либо список с текущим языком + резервным языком.

### get\_default\_language()

Вернуть язык по умолчанию.

### get\_fallback\_language (language\_code=None, site\_id=None)

Узнайте, какой резервный язык используется для выбранного языка.

_Устарело с версии 1.5_: вместо этого используйте `get_fallback_languages()`.

### get\_fallback\_languages (language\_code=None, site\_id=None)

Узнайте, какой резервный язык используется для выбранного языка.

### get\_first\_language (site\_id=None)

Вернуть первый язык для текущего сайта. Это можно использовать для пользовательских интерфейсов, где языки отображаются на вкладках.

### get\_language (language\_code, site\_id=None)

Вернуть настройки языка для текущего сайта.

Эту функцию можно использовать с другими переменными настроек для поддержки модулей, которые создают свои собственные варианты настройки **PARLER\_LANGUAGES**. Для примера см. `add_default_language_settings()`.

## parler.utils.conf.add\_default\_language\_settings(_languages\_list_, _var\_name='PARLER\_LANGUAGES'_, _\*\*extra\_defaults)_

Примените дополнительные значения по умолчанию к языковым настройкам. Эта функция также может использоваться другими пакетами для создания собственного варианта языка **PARLER\_LANGUAGES** с дополнительными полями. Например:

```python
from django.conf import settings
from parler import appsettings as parler_appsettings

# Создайте локальные имена, основанные на глобальных настройках парсера.
MYAPP_DEFAULT_LANGUAGE_CODE = getattr(
    settings,
    'MYAPP_DEFAULT_LANGUAGE_CODE',
    parler_appsettings.PARLER_DEFAULT_LANGUAGE_CODE
)
MYAPP_LANGUAGES = getattr(
    settings, 'MYAPP_LANGUAGES', parler_appsettings.PARLER_LANGUAGES
)

# Применить значения по умолчанию к языкам
MYAPP_LANGUAGES = parler_appsettings.add_default_language_settings(
    MYAPP_LANGUAGES, 'MYAPP_LANGUAGES',
    code=MYAPP_DEFAULT_LANGUAGE_CODE,
    fallback=MYAPP_DEFAULT_LANGUAGE_CODE,
    hide_untranslated=False
)
```

Возвращенный объект будет объектом **LanguagesSetting**, который добавляет дополнительные методы к объекту **dict**.

#### Параметры:

* **languages\_list** - Настройки в формате [PARLER\_LANGUAGES](../../varianty-konfiguracii-django-parler.md#parler\_languages).
* **var\_name** - Имя вашей переменной для отладки вывода.
* **extra\_defaults** - Любые значения по умолчанию для переопределения в разделе `languages_list['default']`, например, **code**, **fallback**, **hide\_untranslated**.

#### Возвращает:

Обновленный список **languages\_list** со всеми значениями по умолчанию, примененными ко всем разделам.

#### Возвращаемый тип:

[LanguagesSetting](modul-parler.utils.conf.md#class-parler.utils.conf.languagessetting)

## parler.utils.conf.get\_parler\_languages\_from\_django\_cms(_cms\_languages=None_)

Преобразует настройку Django CMS **CMS\_LANGUAGES** в **PARLER\_LANGUAGES**. Поскольку **CMS\_LANGUAGES** является строгой надстройкой **PARLER\_LANGUAGES**, мы проводим небольшую очистку, чтобы удалить ненужные элементы.
