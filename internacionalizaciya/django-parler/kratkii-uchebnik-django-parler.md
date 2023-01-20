# Краткий учебник django-parler

## Установка django-parler

Пакет можно установить с помощью:

```bash
pip install django-parler
```

Добавьте следующие настройки:

```python
INSTALLED_APPS += (
    'parler',
)
```

## Краткий обзор

### Создание моделей

С помощью оболочки **TranslatedFields** поля модели можно пометить как переводимые:

```python
from django.db import models
from parler.models import TranslatableModel, TranslatedFields

class MyModel(TranslatableModel):
    translations = TranslatedFields(
        title = models.CharField(_("Title"), max_length=200)
    )

    def __unicode__(self):
        return self.title
```

### Доступ к полям

Переводимые поля можно использовать как обычные поля:

```python
>>> object = MyModel.objects.all()[0]
>>> object.get_current_language()
'en'
>>> object.title
u'cheese omelet'

>>> object.set_current_language('fr')       # Только переключение
>>> object.title = "omelette du fromage"    # Перевод создается по запросу.
>>> object.save()
```

Внутри **django-parler** хранит переведенные поля в отдельной модели, по одной строке на каждый язык.

### Фильтрация переводов

Чтобы запросить переведенные поля, используйте метод `translated()`:

```python
MyObject.objects.translated(title='cheese omelet')
```

Чтобы получить доступ к объектам как на текущем, так и на настроенном резервном языке, используйте:

```python
MyObject.objects.active_translations(title='cheese omelet')
```

Это возвращает объекты на языках, которые считаются «активными», а именно:

* Текущий язык
* Резервные языки, когда `hide_untranslated=False` в настройке **PARLER\_LANGUAGES**.

{% hint style="info" %}
Из-за [ограничений ORM](sovmestimost-s-django.md#ispolzovanie-neskolkikh-vyzovov-filter) запрос должен выполняться в одном вызове `translated()` или `active_translations()`.

Метод `active_translations()` обычно должен включать вызов `distinct()`, чтобы избежать дублирования результатов одного и того же объекта.
{% endhint %}

### Изменение языка

Набору запросов можно указать возвращать объекты на определенном языке:

```python
>>> objects = MyModel.objects.language('fr').all()
>>> objects[0].title
u'omelette du fromage'
```

Это только устанавливает язык объекта. По умолчанию используется текущий язык Django.

Используйте `get_current_language()` и `set_current_language()` для изменения языка отдельных объектов. Для этого есть контекстный менеджер:

```python
from parler.utils.context import switch_language

with switch_language(model, 'fr'):
    print model.title
```

И функция для запроса только определенного поля:

```python
model.safe_translation_getter('title', language_code='fr')
```

## Конфигурирование

По умолчанию резервные языки такие же, как: `[LANGUAGE_CODE]`. Резервный язык можно изменить в настройках:

```python
PARLER_DEFAULT_LANGUAGE_CODE = 'en'
```

При желании вкладки администратора также могут быть настроены:

```python
PARLER_LANGUAGES = {
    None: (
        {'code': 'en',},
        {'code': 'en-us',},
        {'code': 'it',},
        {'code': 'nl',},
    ),
    'default': {
        # по умолчанию PARLER_DEFAULT_LANGUAGE_CODE
        'fallbacks': ['en'],
        # по умолчанию; пусть .active_translations() также возвращает запасные варианты.
        'hide_untranslated': False,
    }
}
```

Замените `None` на **SITE\_ID** при запуске проекта с несколькими сайтами с помощью платформы сайтов. Каждый **SITE\_ID** может быть добавлен как дополнительная запись в словарь.
