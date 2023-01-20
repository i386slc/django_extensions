# Интеграция с django-rest-framework

Чтобы интегрировать переведенные поля в **django-rest-framework**, модуль [django-parler-rest](https://github.com/django-parler/django-parler-rest) предоставляет поля сериализатора. Эти поля можно использовать для интеграции переводов в выходные данные **REST**.

## Пример кода

Будет представлена следующая модель **Country**:

```python
from django.db import models
from parler.models import TranslatableModel, TranslatedFields

class Country(TranslatableModel):
    code = models.CharField(
        _("Country code"),
        max_length=2, unique=True, primary_key=True, db_index=True
    )

    translations = TranslatedFields(
        name = models.CharField(_("Name"), max_length=200, blank=True)
    )

    def __unicode__(self):
        self.name

    class Meta:
        verbose_name = _("Country")
        verbose_name_plural = _("Countries")
```

В сериализаторе используется следующий код:

```python
from parler_rest.serializers import TranslatableModelSerializer
from parler_rest.fields import TranslatedFieldsField
from myapp.models import Country

class CountrySerializer(TranslatableModelSerializer):
    translations = TranslatedFieldsField(shared_model=Country)

    class Meta:
        model = Country
        fields = ('code', 'translations')
```
