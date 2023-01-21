# Совместимость с Django

Этот пакет был протестирован с:

* Джанго версий от 2.2 до 4.0
* Python версии 3.6 и выше

См. [файл tox.ini](https://github.com/django-parler/django-parler/blob/master/tox.ini) для матрицы тестирования совместимости.

## Использование нескольких вызовов `filter()`

Поскольку переведенные поля находятся в отдельной модели, их можно фильтровать, как и любое обычное отношение:

```python
object = MyObject.objects.filter(translations__title='cheese omelet')

translation1 = myobject.translations.all()[0]
```

Однако если вам нужно запросить язык или переведенный атрибут, это должно произойти в одном запросе. Это может быть один вызов `filter()`, `translated()` или `active_translations()`):

```python
from parler.utils import get_active_language_choices

MyObject.objects.filter(
    translations__language_code__in=get_active_language_choices(),
    translations__slug='omelette'
)
```

Запросы к переведенным полям, даже просто `.translated()`, охватывают отношения. Следовательно, их нельзя комбинировать с другими фильтрами для переведенных полей, так как это приводит к двойному соединению в таблице переводов. См. [документацию ORM](https://docs.djangoproject.com/en/dev/topics/db/queries/#spanning-multi-valued-relationships) для более подробной информации.

## Метаполе сортировки **ordering**

По умолчанию невозможно упорядочить переведенные поля. Django не позволит следующее:

```python
from django.db import models
from parler.models import TranslatableModel, TranslatedFields

class MyModel(TranslatableModel):
    translations = TranslatedFields(
        title = models.CharField(max_length=100),
    )

    class Meta:
        ordering = ('title',)  # NOT ALLOWED

    def __unicode__(self):
        return self.title
```

Однако вы можете выполнить упорядочение в наборе запросов:

```python
MyModel.objects.translated('en').order_by('translations__title')
```

Вы также можете использовать предоставленные классы для выполнения сортировки в коде Python.

* Для **list\_filter** в админке используйте: **SortedRelatedFieldListFilter**
* Для виджетов форм используйте: **SortedSelect**, **SortedSelectMultiple**, **SortedCheckboxSelectMultiple**

## Использование **search\_fields** в админке

Когда переведенные поля включаются в **search\_fields**, они должны быть включены с их полным путем ORM. Например:

```python
from parler.admin import TranslatableAdmin

class MyModelAdmin(TranslatableAdmin):
    search_fields = ('translations__title',)
```

## Использование **prepopulated\_fields** в админке

Использование **prepopulated\_fields** пока не работает, так как админка будет жаловаться, что поля не существует. Используйте `get_prepopulated_fields()` в качестве обходного пути:

```python
from parler.admin import TranslatableAdmin

class MyModelAdmin(TranslatableAdmin):

    def get_prepopulated_fields(self, request, obj=None):
        # нельзя использовать `prepopulated_fields = ..`, потому что это
        # нарушает проверку в админке переведенных полей.
        # Это официальный обходной путь django-parler.
        return {
            'slug': ('title',)
        }
```
