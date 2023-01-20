# Добавление переведенных полей в существующую модель

Создайте прокси-класс:

```python
from django.contrib.sites.models import Site
from parler.models import TranslatableModel, TranslatedFields

class TranslatableSite(TranslatableModel, Site):
    class Meta:
        proxy = True

    translations = TranslatedFields()
```

И обновите админку:

```python
from django.contrib.sites.admin import SiteAdmin
from django.contrib.sites.models import Site
from parler.admin import TranslatableAdmin, TranslatableStackedInline

class NewSiteAdmin(TranslatableAdmin, SiteAdmin):
    pass

admin.site.unregister(Site)
admin.site.register(TranslatableSite, NewSiteAdmin)
```

## Перезапись существующих непереведенных полей

Обратите внимание, что невозможно добавить переводы в прокси-класс с тем же именем, что и поля в родительской модели. Это еще не будет отображаться как ошибка, но произойдет сбой при извлечении объектов из базы данных. Вместо этого выберите чтение [Как сделать существующие поля переводимыми](delaem-sushestvuyushie-polya-perevodimymi.md).
