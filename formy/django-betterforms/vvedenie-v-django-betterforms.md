# Введение в django-betterforms

**django-betterforms** предоставляет набор инструментов, упрощающих работу с формами в Django.

## Установка

1. Установите пакет:

```bash
$ pip install django-betterforms
```

Или вы можете установить его из исходников:

```bash
$ pip install -e git://github.com/fusionbox/django-betterforms@master#egg=django-betterforms-dev
```

2\. Добавьте **betterforms** в свои **INSTALLED\_APPS**.

## Быстрый старт

Начать работу с **betterforms** очень просто. Если вы используете встроенные базовые классы формы, предоставляемые django, это так же просто, как переключиться на базовые классы формы, предоставляемые **betterforms**.

```python
from betterforms.forms import BaseForm

class RegistrationForm(BaseForm):
    ...
    class Meta:
        fieldsets = (
            ('info', {'fields': ('username', 'email')}),
            ('location', {'fields': ('address', ('city', 'state', 'zip'))}),
            ('password', {'password1', 'password2'}),
        )
```

А потом в своем шаблоне.

```django
<form method="post">
     {% raw %}
{% include 'betterforms/form_as_fieldsets.html' %}
{% endraw %}
</form>
```

Который отобразит следующее.

```html
<fieldset class="formFieldset info">
        <div class="required username formField">
                <label for="id_username">Username</label>
                <input id="id_username" name="username" type="text" />
        </div>
        <div class="required email formField">
                <label for="id_email">Email</label>
                <input id="id_email" name="email" type="text" />
        </div>
</fieldset>
<fieldset class="formFieldset location">
        <div class="required address formField">
                <label for="id_address">Address</label>
                <input id="id_address" name="address" type="text" />
        </div>
        <fieldset class="formFieldset location_1">
                <div class="required city formField">
                        <label for="id_city">City</label>
                        <input id="id_city" name="city" type="text" />
                </div>
                <div class="required state formField">
                        <label for="id_state">State</label>
                        <input id="id_state" name="state" type="text" />
                </div>
                <div class="required zip formField">
                        <label for="id_zip">Zip</label>
                        <input id="id_zip" name="zip" type="text" />
                </div>
        </fieldset>
</fieldset>
<fieldset class="formFieldset password">
        <div class="required password1 formField">
                <label for="id_password1">Password</label>
                <input id="id_password1" name="password1" type="password" />
        </div>
        <div class="required password2 formField">
                <label for="id_password2">Confirm Password</label>
                <input id="id_password2" name="password2" type="password" />
        </div>
</fieldset>
```
