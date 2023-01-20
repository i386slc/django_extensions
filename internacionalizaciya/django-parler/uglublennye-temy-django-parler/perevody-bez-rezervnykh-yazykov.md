# Переводы без резервных языков

Когда перевод отсутствует, используются резервные языки. Однако если у объекта нет резервных языков, это все равно не удается.

Есть несколько решений этой проблемы:

1. Явно объявите переведенный атрибут с помощью `any_language=True`:

```python
from parler.models import TranslatableModel
from parler.fields import TranslatedField

class MyModel(TranslatableModel):
    title = TranslatedField(any_language=True)
```

Теперь заголовок попытается получить один из существующих языков из базы данных.

2\. Используйте `safe_translation_getter()` для атрибутов, у которых нет настройки `any_language=True`. Например:

```python
model.safe_translation_getter("fieldname", any_language=True)
```

3\. Перехватите исключение **TranslationDoesNotExist**. Например:

```python
try:
    return object.title
except TranslationDoesNotExist:
    return ''
```

Поскольку это исключение наследуется от **AttributeError**, шаблоны по умолчанию уже отображают пустые значения.

4\. Избегайте получения непереведенных объектов с помощью методов набора запросов **queryset**. Например:

```python
queryset.active_translations()
```

Что почти идентично:

```python
codes = get_active_language_choices()
queryset.filter(translations__language_code__in=codes).distinct()
```

Обратите внимание, что здесь применяются те же [ограничения ORM](https://docs.djangoproject.com/en/dev/topics/db/queries/#spanning-multi-valued-relationships).
