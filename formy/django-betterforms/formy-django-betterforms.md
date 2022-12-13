# Формы django-betterforms

**django-betterforms** предоставляет два новых базовых класса для использования вместо [django.forms.Form](https://docs.djangoproject.com/en/1.5/ref/forms/api/#django.forms.Form) и [django.forms.ModelForm](https://docs.djangoproject.com/en/1.5/topics/forms/modelforms/#django.forms.ModelForm).

### _class_ betterforms.forms.BetterForm

Базовый класс формы для немодельных форм ([исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/forms.html#BetterForm)).

### _class_ betterforms.forms.BetterModelForm

Базовый класс формы для форм модели ([исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/forms.html#BetterModelForm)).

## Ошибки (errors)

Добавить ошибки в **betterforms** легко:

```python
>>> form = BlogEntryForm(request.POST)
>>> form.is_valid()
True
>>> form.field_error('title', 'This title is already taken')
>>> form.is_valid()
False
>>> form.errors
{'title': ['This title is already taken']}
```

Вы также можете добавить глобальные ошибки:

```python
>>> form = BlogEntryForm(request.POST)
>>> form.form_error('Not accepting new entries at this time')
>>> form.is_valid()
False
>>> form.errors
{'__all__': ['Not accepting new entries at this time']}
```

**form\_error** — это просто оболочка для **field\_error**, которая использует ключ **\_\_all\_\_** для имени поля.

## Наборы форм (fieldsets)
