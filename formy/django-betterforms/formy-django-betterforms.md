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

Одной из самых мощных функций улучшенных форм является возможность объявлять группы полей. И [BetterForm](formy-django-betterforms.md#class-betterforms.forms.betterform), и [BetterModelForm](formy-django-betterforms.md#class-betterforms.forms.bettermodelform) предоставляют общий интерфейс для работы с наборами полей.

Наборы полей могут быть объявлены в любом из трех форматов или в любом сочетании этих трех форматов.

* Как два кортежа

Подобно набору [полей админки](https://docs.djangoproject.com/en/1.5/topics/auth/default/#built-in-auth-views), в виде списка из двух кортежей. Два кортежа должны быть в формате `(name, fieldset_options)`, где **name** — это строка, представляющая заголовок набора полей, а **fieldset\_options** — это словарь, который будет передан как **kwargs** конструктору набора полей.

```python
from betterforms.forms import BetterForm

class RegistrationForm(BetterForm):
    ...
    class Meta:
        fieldsets = (
            ('info', {'fields': ('username', 'email')}),
            ('location', {'fields': ('address', ('city', 'state', 'zip'))}),
            ('password', {'password1', 'password2'}),
        )
```

* В виде списка имен полей

Наборы полей могут быть объявлены как список имен полей.

```python
from betterforms.forms import BetterForm

class RegistrationForm(BetterForm):
    ...
    class Meta:
        fieldsets = (
            ('username', 'email'),
            ('address', ('city', 'state', 'zip')),
            ('password1', 'password2'),
        )
```

* Как созданные экземпляры **Fieldset** или подклассы **Fieldset**.

Наборы полей могут быть объявлены как список имен полей.

```python
from betterforms.forms import BetterForm, Fieldset

class RegistrationForm(BetterForm):
    ...
    class Meta:
        fieldsets = (
            Fieldset('info', fields=('username', 'email')),
            Fieldset('location', ('address', ('city', 'state', 'zip'))),
            Fieldset('password', ('password1', 'password2')),
        )
```

Все три из этих примеров будут иметь примерно одинаковый результат. Все эти форматы можно смешивать, сопоставлять и вкладывать друг в друга. И, в отличие от **django-admin**, вы можете вкладывать наборы полей так глубоко, как вам хочется.

**Fieldset** также может быть опционально объявлен с легендой **kwarg**, которая затем будет доступна как свойство для связанного **BoundFieldset**.

```python
Fieldset(
    'location',
    ('address', ('city', 'state', 'zip')),
    legend='Place of Residence'
)
```

Если вы решите визуализировать форму с использованием шаблонов улучшенной формы, подробно описанных ниже, каждый набор полей с легендой будет отображаться с добавленным тегом легенды в шаблоне.

## Рендеринг

Чтобы отобразить форму, используйте предоставленный частичный шаблон.

```django
<form method="post">
     {% raw %}
{% include 'betterforms/form_as_fieldsets.html' %}
{% endraw %}
</form>
```

Этот партиал предполагает, что в его контексте доступна переменная **form**. Этот шаблон делает следующие вещи.

* выводит **csrf\_token**
* выводит скрытое поле с именем **next**, если в контексте шаблона доступно значение **next**
* выводит **media**
*   проходит цикл на **form.fieldsets**

    * для каждого набора полей отображает набор полей с использованием шаблона `betterforms/fieldset_as_div.html`
      * для каждого элемента в наборе полей, если это набор полей, он отображается с использованием одного и того же шаблона, а если это поле, он отображается с использованием шаблона поля
    * для каждого поля отображает поле с использованием шаблона `betterforms/field_as_div.html`

    Если вы хотите вывести форму без токена **CSRF** (например, в форме GET), вы можете сделать это, передав переменную **csrf\_exempt**.

```django
<form method="post">
     {% raw %}
{% include 'betterforms/form_as_fieldsets.html' csrf_exempt=True %}
{% endraw %}
</form>
```

Если вы хотите переопределить суффикс метки, **django-betterforms** предоставляет удобный атрибут класса для классов [BetterForm](formy-django-betterforms.md#class-betterforms.forms.betterform) и [BetterModelForm](formy-django-betterforms.md#class-betterforms.forms.bettermodelform).

```python
class MyForm(forms.BetterForm):
    # ... fields

    label_suffix = '->'
```

{% hint style="warning" %}
Из-за ошибки в работе с суффиксом метки в Django < 1.6, **label\_suffix** не будет отображаться ни в каких формах, отображаемых с использованием шаблонов **betterforms**. Для получения дополнительной информации обратитесь к [ошибке Django № 18134](https://code.djangoproject.com/ticket/18134).
{% endhint %}
