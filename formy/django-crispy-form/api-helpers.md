# API helpers

## _class_ helper.FormHelper(_form=None_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/helper.html#FormHelper).

Этот класс управляет поведением рендеринга формы, передаваемой тегу `{% crispy %}` . Для этого вам нужно будет установить его атрибуты и передать соответствующий вспомогательный объект тегу:

```django
{% raw %}
{% crispy form form.helper %}
{% endraw %}
```

Давайте посмотрим, какие атрибуты вы можете установить и к какому поведению формы они относятся:

### form\_method

Задает атрибут метода формы. Вы можете установить его на **POST** или **GET**. По умолчанию **POST**.

### form\_action

Применяется к атрибуту **action** формы. Может быть именованным URL-адресом в вашей конфигурации URL-адресов, который может быть выполнен с помощью тега шаблона `{% url %}`. Пример: `«show_my_profile»`. В вашем **URLconf** у вас может быть что-то вроде:

```python
path('show/profile/', 'show_my_profile_view', name = 'show_my_profile')
```

Он может просто указывать на URL `‘/whatever/blabla/’` .

### form\_id

Создает **id** формы для идентификации домена. Если идентификатор не указан, то в форме не создается атрибут **id**.

### form\_class

Строка, содержащая отдельные классы CSS, применяемые для формирования атрибута **class**.

### form\_group\_wrapper\_class

Строка, содержащая отдельные классы CSS, применяемые к каждой строке входных данных.

### form\_tag

Он указывает, должны ли теги `<form></form>` отображаться при использовании макета. Если установлено значение `False`, форма отображается без тегов `<form></form>`. По умолчанию `True`.

### form\_error\_title

Если форма содержит **non\_field\_errors** для отображения, они отображаются в **div**. Вы можете установить **div** заголовка с помощью этого атрибута. Пример: `«Ooooops!»` или `«Form Errors»`.

### formset\_error\_title

Если набор форм содержит **non\_form\_errors** для отображения, они отображаются в **div**. Вы можете установить **div** заголовка с помощью этого атрибута.

### include\_media

Следует ли автоматически включать media формы. Установите значение `False`, если вы хотите вручную включить media формы вне формы. По умолчанию `True`.

Публичные методы:

### add\_input(input)

Вы можете добавить кнопки ввода, используя этот метод. Входные данные, добавленные с помощью этого метода, будут отображаться в конце формы/набора форм.

### add\_layout(layout)

Вы можете добавить объект **Layout** в **FormHelper**. Макет указывает простым, чистым и DRY способом, как должны отображаться поля формы. Вы можете оборачивать поля, упорядочивать их, настраивать почти все в форме.

Лучший способ добавить помощника в форму — это добавить в форму свойство с именем **helper**, которое возвращает настроенный объект **FormHelper**:

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit

class MyForm(forms.Form):
    title = forms.CharField(_("Title"))

    @property
    def helper(self):
        helper = FormHelper()
        helper.form_id = 'this-form-rocks'
        helper.form_class = 'search'
        helper.add_input(Submit('save', 'save'))
        [...]
        return helper
```

Вы можете использовать его в шаблоне, выполняя:

```django
{% raw %}
{% load crispy_forms_tags %}
{% crispy form %}
{% endraw %}
```

## get\_attributes(_template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/helper.html#FormHelper.get\_attributes).

Используется **crispy\_forms\_tags** для получения вспомогательных атрибутов

## render\_layout(_form_, _context_, _template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/helper.html#FormHelper.render\_layout).

Возвращает безопасный html рендеринга макета.
