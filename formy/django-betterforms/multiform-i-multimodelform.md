# MultiForm и MultiModelForm

Контейнер, позволяющий обрабатывать несколько форм как одну форму. Это отлично подходит для использования более одной формы на странице с одной и той же кнопкой отправки. MultiForm имитирует **Form API**, поэтому никому другому (например, общим представлениям) не видно, что вы используете **MultiForm**.

Однако есть пара отличий. Один заключается в том, как вы инициализируете форму. См. этот пример:

```python
class UserProfileMultiForm(MultiForm):
    form_classes = {
        'user': UserForm,
        'profile': ProfileForm,
    }

UserProfileMultiForm(initial={
    'user': {
        # Исходные данные пользователя
    },
    'profile': {
        # Исходные данные профиля
    },
})
```

Аргумент **initial** должен быть вложенным словарем, чтобы мы могли связать правильные исходные данные с правильным классом формы.

Другое существенное отличие состоит в том, что нет прямого доступа к полям, потому что это может привести к конфликту пространств имен. Вы должны получить доступ к полям из их форм. Все формы доступны с использованием ключа, предоставленного в form\_classes:

```python
form = UserProfileMultiForm()
# получить объект Field
form['user'].fields['name']
# получиь объект BoundField
form['user']['name']
```

MultiForm, однако, делает все, чтобы перебирать все поля всех форм.

```django
{% raw %}
{% for field in form %}
  {{ field }}
{% endfor %}
{% endraw %}
```

Если вы полагаетесь на то, что поля будут выводиться в согласованном порядке, вам следует использовать **OrderedDict** для определения form\_classes.

```python
from collections import OrderedDict

class UserProfileMultiForm(MultiForm):
    form_classes = OrderedDict((
        ('user', UserForm),
        ('profile', ProfileForm),
    ))
```

## Работа с формами ModelForm

**MultiModelForm** добавляет поддержку **ModelForm** поверх **MultiForm**. Это просто означает, что он включает поддержку параметра экземпляра при инициализации и добавляет метод сохранения.

```python
class UserProfileMultiForm(MultiModelForm):
    form_classes = {
        'user': UserForm,
        'profile': ProfileForm,
    }

user = User.objects.get(pk=123)
UserProfileMultiForm(instance={
    'user': user,
    'profile': user.profile,
})
```

## Работа с CreateView

Довольно легко использовать **MultiModelForm** с [CreateView](https://docs.djangoproject.com/en/1.5/ref/class-based-views/generic-editing/#django.views.generic.edit.CreateView) Django, обычно вам придется переопределить метод [form\_valid()](https://docs.djangoproject.com/en/1.5/ref/class-based-views/mixins-editing/#django.views.generic.edit.FormMixin.form\_valid), чтобы выполнить некоторые специфические функции сохранения. Например, у вас может быть форма регистрации, которая создает пользователя и объект профиля пользователя в одном:

```python
# forms.py
from django import forms
from authtools.forms import UserCreationForm
from betterforms.multiform import MultiModelForm
from .models import UserProfile

class UserProfileForm(forms.ModelForm):
    class Meta:
        fields = ('favorite_color',)

class UserCreationMultiForm(MultiModelForm):
    form_classes = {
        'user': UserCreationForm,
        'profile': UserProfileForm,
    }

# views.py
from django.views.generic import CreateView
from django.core.urlresolvers import reverse_lazy
from django.shortcuts import redirect
from .forms import UserCreationMultiForm

class UserSignupView(CreateView):
    form_class = UserCreationMultiForm
    success_url = reverse_lazy('home')

    def form_valid(self, form):
        # Сначала сохраните пользователя, потому что профиль
        # нуждается в пользователе, прежде чем его можно будет сохранить.
        user = form['user'].save()
        profile = form['profile'].save(commit=False)
        profile.user = user
        profile.save()
        return redirect(self.get_success_url())
```

{% hint style="info" %}
В этом примере мы использовали форму **UserCreationForm** из пакета [django-authtools](https://pypi.org/project/django-authtools/) просто для краткости. Конечно, вы можете использовать любую модель **ModelForm**, которую захотите.
{% endhint %}

Конечно, мы могли бы поместить логику сохранения в саму форму **UserCreationMultiForm**, переопределив метод MultiModelForm.save().

```python
class UserCreationMultiForm(MultiModelForm):
    form_classes = {
        'user': UserCreationForm,
        'profile': UserProfileForm,
    }

    def save(self, commit=True):
        objects = super(UserCreationMultiForm, self).save(commit=False)

        if commit:
            user = objects['user']
            user.save()
            profile = objects['profile']
            profile.user = user
            profile.save()

        return objects
```

Если мы это сделаем, мы можем упростить наше представление до этого:

```python
class UserSignupView(CreateView):
    form_class = UserCreationMultiForm
    success_url = reverse_lazy('home')
```

## Работа с UpdateView

Работать с [UpdateView](https://docs.djangoproject.com/en/1.5/ref/class-based-views/generic-editing/#django.views.generic.edit.UpdateView) также довольно просто, но вам, скорее всего, придется переопределить метод **django.views.generic.edit.FormMixin.get\_form\_kwargs**, чтобы передавать экземпляры, с которыми вы хотите работать. Если мы продолжим пример с пользователем/профилем, это будет выглядеть примерно так:

```python
# forms.py
from django import forms
from django.contrib.auth import get_user_model
from betterforms.multiform import MultiModelForm
from .models import UserProfile

User = get_user_model()

class UserEditForm(forms.ModelForm):
    class Meta:
        fields = ('email',)

class UserProfileForm(forms.ModelForm):
    class Meta:
        fields = ('favorite_color',)

class UserEditMultiForm(MultiModelForm):
    form_classes = {
        'user': UserEditForm,
        'profile': UserProfileForm,
    }

# views.py
from django.views.generic import UpdateView
from django.core.urlresolvers import reverse_lazy
from django.shortcuts import redirect
from django.contrib.auth import get_user_model
from .forms import UserEditMultiForm

User = get_user_model()

class UserSignupView(UpdateView):
    model = User
    form_class = UserEditMultiForm
    success_url = reverse_lazy('home')

    def get_form_kwargs(self):
        kwargs = super(UserSignupView, self).get_form_kwargs()
        kwargs.update(instance={
            'user': self.object,
            'profile': self.object.profile,
        })
        return kwargs
```

## Работа с WizardView

MultiForm также поддерживает классы **WizardView**, предоставляемые [django-formtools](https://django-formtools.readthedocs.io/en/latest/wizard.html) (или Django до версии 1.8), однако вы должны установить атрибут **base\_fields** в своем классе формы.

```python
# forms.py
from django import forms
from betterforms.multiform import MultiForm

class Step1Form(MultiModelForm):
    # Мы должны установить base_fields в словарь,
    # потому что WizardView пытается его изучить.
    base_fields = {}

    form_classes = {
        'user': UserEditForm,
        'profile': UserProfileForm,
    }
```

Затем вы можете использовать его как обычно.

```python
# views.py
try:
    from django.contrib.formtools.wizard.views import SessionWizardView
except ImportError:  # Django >= 1.8
    from formtools.wizard.views import SessionWizardView

from .forms import Step1Form, Step2Form

class MyWizardView(SessionWizardView):
    def done(self, form_list, form_dict, **kwargs):
        step1form = form_dict['1']
        # Вы можете получить данные для пользовательской формы следующим образом:
        user = step1form['user'].save()
        # ...

wizard_view = MyWizardView.as_view([Step1Form, Step2Form])
```

Причина, по которой мы должны установить **base\_fields** в словарь, заключается в том, что **WizardView** выполняет некоторый самоанализ, чтобы определить, принимает ли какая-либо из форм файлы, а затем убеждается, что в **WizardView** есть файловое\_хранилище. Установив для **base\_fields** пустой словарь, мы можем обойти эту проверку.

{% hint style="warning" %}
Если у вас есть какие-либо формы, которые принимают файлы, вы должны настроить атрибут **file\_storage** для вашего **WizardView**.
{% endhint %}
