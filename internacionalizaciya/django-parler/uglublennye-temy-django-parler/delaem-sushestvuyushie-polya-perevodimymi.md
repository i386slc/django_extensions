# Делаем существующие поля переводимыми

В следующем руководстве объясняется, как сделать существующие поля переводимыми и перенести данные из старых полей в переведенные поля.

**django-parler** хранит переведенные поля в отдельной модели, поэтому он может хранить несколько версий (переводов) одного и того же поля. Чтобы сделать существующие поля переводимыми, необходимо выполнить 3 шага миграции:

1. Создайте таблицу перевода, сохраните существующие столбцы
2. Скопируйте данные из исходной таблицы в таблицу перевода.
3. Удалите поля из исходной модели.

В следующих разделах это объясняется подробно:

## Шаг 1: Создайте таблицу перевода

Скажем, у нас есть следующая модель:

```python
class MyModel(models.Model):
    name = models.CharField(max_length=123)
```

Сначала создайте переводимые поля:

```python
class MyModel(TranslatableModel):
    name = models.CharField(max_length=123)

    translations = TranslatedFields(
          name=models.CharField(max_length=123),
    )
```

Теперь создайте миграцию:

```bash
manage.py makemigrations myapp --name "add_translation_model"
```

## Шаг 2: Скопируйте данные

В рамках переноса данных скопируйте существующие данные:

Создайте пустую миграцию:

```python
manage.py makemigrations --empty myapp --name "migrate_translatable_fields"
```

И используйте его для перемещения данных:

```python
from django.db import migrations
from django.conf import settings
from django.core.exceptions import ObjectDoesNotExist

def forwards_func(apps, schema_editor):
    MyModel = apps.get_model('myapp', 'MyModel')
    MyModelTranslation = apps.get_model('myapp', 'MyModelTranslation')

    for object in MyModel.objects.all():
        MyModelTranslation.objects.create(
            master_id=object.pk,
            language_code=settings.LANGUAGE_CODE,
            name=object.name
        )

def backwards_func(apps, schema_editor):
    MyModel = apps.get_model('myapp', 'MyModel')
    MyModelTranslation = apps.get_model('myapp', 'MyModelTranslation')

    for object in MyModel.objects.all():
        translation = _get_translation(object, MyModelTranslation)
        object.name = translation.name
        object.save()   # Обратите внимание, что это только вызовы Model.save()

def _get_translation(object, MyModelTranslation):
    translations = MyModelTranslation.objects.filter(master_id=object.pk)
    try:
        # Пробуем перевод по умолчанию
        return translations.get(language_code=settings.LANGUAGE_CODE)
    except ObjectDoesNotExist:
        try:
            # Пробуем язык по умолчанию
            return translations.get(
                language_code=settings.PARLER_DEFAULT_LANGUAGE_CODE
            )
        except ObjectDoesNotExist:
            # Может быть, объект был перевод только на определенный язык?
            # Надеюсь, что есть один перевод
            return translations.get()

class Migration(migrations.Migration):

    dependencies = [
        ('yourappname', '0001_initial'),
    ]

    operations = [
        migrations.RunPython(forwards_func, backwards_func),
    ]
```

{% hint style="info" %}
Будьте осторожны, какой язык используется для переноса существующих данных. В этом примере логика `reverses_func()` чрезвычайно защитная, чтобы не потерять переведенные данные.
{% endhint %}

## Шаг 3. Удалите старые поля.

Удалите старое поле из исходной модели. Примерная модель теперь выглядит так:

```python
class MyModel(TranslatableModel):
    translations = TranslatedFields(
        name=models.CharField(max_length=123),
    )
```

Создайте миграцию базы данных, она просто удалит исходное поле:

```bash
manage.py makemigrations myapp --name "remove_untranslated_fields"
```

## Обновление кода

Код проекта должен быть обновлен. Например:

* Замените `filter(field_name)` на `.translated(field_name)` или `filter(translations__field_name)`.
* Убедитесь, что в переведенных полях есть один фильтр, см. [Использование нескольких вызовов filter()](../sovmestimost-s-django.md#ispolzovanie-neskolkikh-vyzovov-filter).
* Обновите код `ordering` и `order_by()`. См. [Метаполе ordering](../sovmestimost-s-django.md#metapole-sortirovki-ordering).
* Обновите admin `search_fields` и `prepopulated_fields`. См. [Использование search\_fields в админке](../sovmestimost-s-django.md#ispolzovanie-search\_fields-v-adminke).

## Развертывание

Для беспрепятственного развертывания рекомендуется запускать только первые две миграции, которые создают столбцы и перемещают данные. Удаление старых полей следует выполнять после перезагрузки экземпляра WSGI.
