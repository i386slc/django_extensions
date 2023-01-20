# Построение модели переводов вручную

Также возможно создать модель переведенных полей вручную:

```python
from django.db import models
from parler.models import TranslatableModel, TranslatedFieldsModel
from parler.fields import TranslatedField

class MyModel(TranslatableModel):
    title = TranslatedField()  # Необязательно, явно укажите поле

    class Meta:
        verbose_name = _("MyModel")

    def __unicode__(self):
        return self.title

class MyModelTranslation(TranslatedFieldsModel):
    master = models.ForeignKey(MyModel, related_name='translations', null=True)
    title = models.CharField(_("Title"), max_length=200)

    class Meta:
        unique_together = ('language_code', 'master')
        verbose_name = _("MyModel translation")
```

Это имеет тот же эффект, но также позволяет переопределить метод `save()` или добавить новые методы самостоятельно.
