# Интеграция с django-polymorphic

Когда вам нужно объединить **TranslatableModel** с **PolymorphicModel**, вы должны убедиться, что менеджеры моделей обоих классов также объединены.

Это можно сделать либо перезаписав [default\_manager](https://docs.djangoproject.com/en/dev/topics/db/managers/#custom-managers-and-inheritance), либо расширив классы **Manager** и **QuerySet**.

## Объединение TranslatableModel с PolymorphicModel

Скажем, у нас есть базовый **Product** с двумя конкретными продуктами, **Book** с двумя переводимыми полями, **name** и **slug**, и **Pen** с одним переводимым поля **identificator**. Тогда для полиморфной модели Django работает следующий шаблон:

```python
from django.db import models
from django.utils.encoding import force_text
from parler.models import TranslatableModel, TranslatedFields
from parler.managers import TranslatableManager
from polymorphic import PolymorphicModel
from .managers import BookManager

class Product(PolymorphicModel):
    # Общая базовая модель. Либо поместите переведенные поля здесь,
    # либо поместите их в подклассы (см. примечание ниже).
    code = models.CharField(blank=False, default='', max_length=16)
    price = models.DecimalField(max_digits=10, decimal_places=2, default=0.00)

class Book(Product, TranslatableModel):
    # Решение 1: используйте собственный менеджер, который сочетает в себе оба.
    objects = BookManager()

    translations = TranslatedFields(
        name=models.CharField(blank=False, default='', max_length=128),
        slug=models.SlugField(blank=False, default='', max_length=128)
    )

    def __str__(self):
        return force_text(self.code)

class Pen(Product, TranslatableModel):
    # Решение 2: переопределите диспетчер по умолчанию.
    default_manager = TranslatableManager()

    translations = TranslatedFields(
        identifier=models.CharField(blank=False, default='', max_length=255)
    )

    def __str__(self):
        return force_text(self.identifier)
```

Единственная предосторожность, которую необходимо предпринять, — переопределить диспетчер по умолчанию в каждом из классов, содержащих переводимые поля. Это показано в примере выше.

Начиная с **django-parler 1.2** можно иметь переводы как для базовой, так и для производной модели. Убедитесь, что имя поля (в данном случае **translations**) отличается в обеих моделях, так как это имя используется как **related\_name** для модели переведенных полей.

## Объединение менеджеров

Менеджеры можно комбинировать, унаследовав их и указав атрибут **queryset\_class** с использованием как **django-parler**, так и [django-polymorphic](https://github.com/django-polymorphic/django-polymorphic).

```python
from parler.managers import TranslatableManager, TranslatableQuerySet
from polymorphic import PolymorphicManager
from polymorphic.query import PolymorphicQuerySet

class BookQuerySet(TranslatableQuerySet, PolymorphicQuerySet):
    pass

class BookManager(PolymorphicManager, TranslatableManager):
    queryset_class = BookQuerySet
```

Назначьте менеджера атрибуту **objects** модели.

## Реализация админки

Вполне возможно зарегистрировать отдельные полиморфные модели в административном интерфейсе Django. Однако для использования этих моделей в едином связном интерфейсе доступны некоторые дополнительные базовые классы.

Этот интерфейс администратора добавляет переводимые поля в полиморфную модель:

```python
from django.contrib import admin
from parler.admin import TranslatableAdmin, TranslatableModelForm
from polymorphic.admin import PolymorphicParentModelAdmin, PolymorphicChildModelAdmin
from .models import BaseProduct, Book, Pen

class BookAdmin(TranslatableAdmin, PolymorphicChildModelAdmin):
    base_form = TranslatableModelForm
    base_model = BaseProduct
    base_fields = ('code', 'price', 'name', 'slug')

class PenAdmin(TranslatableAdmin, PolymorphicChildModelAdmin):
    base_form = TranslatableModelForm
    base_model = BaseProduct
    base_fields = ('code', 'price', 'identifier',)

class BaseProductAdmin(PolymorphicParentModelAdmin):
    base_model = BaseProduct
    child_models = ((Book, BookAdmin), (Pen, PenAdmin),)
    list_display = ('code', 'price',)

admin.site.register(BaseProduct, BaseProductAdmin)
```
