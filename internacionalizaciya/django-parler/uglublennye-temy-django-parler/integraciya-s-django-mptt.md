# Интеграция с django-mptt

Когда вам нужно объединить **TranslatableModel** с **MPTTModel**, вы должны убедиться, что менеджеры моделей обоих классов также объединены.

Это можно сделать, расширив классы **Manager** и **QuerySet**.

{% hint style="info" %}
Этот пример написан для [django-mptt](https://github.com/django-mptt/django-mptt) >= **0.7.0**, что также требует объединения классов набора запросов.
{% endhint %}

Рабочий пример см. в разделе [django-categories-i18n](https://github.com/edoburu/django-categories-i18n).

## Объединение TranslatableModel с MPTTModel

Скажем, у нас есть базовая модель **Category**, которую необходимо перевести:

```python
from django.db import models
from django.utils.encoding import force_text
from parler.models import TranslatableModel, TranslatedFields
from parler.managers import TranslatableManager
from mptt.models import MPTTModel
from .managers import CategoryManager


class Category(MPTTModel, TranslatableModel):
    # Общая базовая модель. Либо поместите переведенные поля здесь,
    # либо поместите их в подклассы (см. примечание ниже).
    parent = models.ForeignKey('self', related_name='children')

    translations = TranslatedFields(
        name=models.CharField(blank=False, default='', max_length=128),
        slug=models.SlugField(blank=False, default='', max_length=128)
    )

    objects = CategoryManager()

    def __str__(self):
        return self.safe_translation_getter('name', any_language=True)
```

## Объединение менеджеров

Менеджеры могут быть объединены путем наследования:

```python
from parler.managers import TranslatableManager, TranslatableQuerySet
from mptt.managers import TreeManager
from mptt.querysets import TreeQuerySet

class CategoryQuerySet(TranslatableQuerySet, TreeQuerySet):

    def as_manager(cls):
        # убедитесь, что создание менеджеров из наборов запросов работает.
        manager = CategoryManager.from_queryset(cls)()
        manager._built_with_as_manager = True
        return manager
    as_manager.queryset_only = True
    as_manager = classmethod(as_manager)


class CategoryManager(TreeManager, TranslatableManager):
    _queryset_class = CategoryQuerySet
```

Назначьте менеджера атрибуту **objects** модели.

## Реализация админки

Благодаря объединению базовых классов интерфейс администратора поддерживает переводимые модели **MPTT**:

```python
from django.contrib import admin
from parler.admin import TranslatableAdmin, TranslatableModelForm
from mptt.admin import MPTTModelAdmin
from mptt.forms import MPTTAdminForm
from .models import Category

class CategoryAdminForm(MPTTAdminForm, TranslatableModelForm):
    pass

class CategoryAdmin(TranslatableAdmin, MPTTModelAdmin):
    form = CategoryAdminForm

    def get_prepopulated_fields(self, request, obj=None):
        return {'slug': ('title',)}  # нужно для переведенных полей

admin.site.register(Category, CategoryAdmin)
```
