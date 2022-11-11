# API Layout

## _class_ layout.BaseInput(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#BaseInput).

Базовый класс для уменьшения объема кода в классах ввода.

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

#### Методы

**render(**_**form**_**, **_**context**_**, **_**template\_pack=\<SimpleLazyObject: 'bootstrap4'>**_**, **_**\*\*kwargs**_**)**

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#BaseInput.render).

Отображает `<input />` если контейнер используется как объект **Layout**. Значение кнопки ввода может быть переменной в контексте.

## _class_ layout.Button(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Button).

Используется для создания дескриптора кнопки для тега шаблона `{% crispy %}` .

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

Примеры

{% hint style="info" %}
атрибут **form** для **render()** не требуется для унаследованных объектов **BaseInput**.
{% endhint %}

```python
>>> button = Button('Button 1', 'Press Me!')
>>> button.render("", "", Context())
'<input type="button" name="button-1" value="Press Me!" '
'class="btn" id="button-id-button-1"/>'
```

```python
>>> button = Button('Button 1', 'Press Me!', css_id="custom-id",
                     css_class="custom class", my_attr=True, data="my-data")
>>> button.render("", "", Context())
'<input type="button" name="button-1" value="Press Me!" '
'class="btn custom class" id="custom-id" data="my-data" my-attr/>'
```

Обычно вы не будете вызывать метод рендеринга для объекта напрямую. Вместо этого добавьте его в свой **Layout** вручную или используйте метод **add\_input**:

```python
class ExampleForm(forms.Form):
[...]
def __init__(self, *args, **kwargs):
    super().__init__(*args, **kwargs)
    self.helper = FormHelper()
    self.helper.add_input(Button('Button 1', 'Press Me!'))
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

## _class_ layout.ButtonHolder(_\*fields_, _css\_id=None_, _css\_class=None_, _template=None_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#ButtonHolder).

Объект макета. Он оборачивает поля в `<div class="buttonHolder">`.

Здесь вы должны разместить объекты макета, которые отображают кнопки формы.

#### Параметры

* **\*fields** (_HTML или BaseInput_) - Объекты макета для отображения в **ButtonHolder**. Он должен содержать только унаследованные объекты **HTML** и **BaseInput**.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.

Примеры

Пример использования **ButtonHolder** в вашем макете:

```python
ButtonHolder(
    HTML(<span style="display: hidden;">Information Saved</span>),
    Submit('Save', 'Save')
)
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

## _class_ layout.Column(_\*fields_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Column).

Объект макета. Он заключает поля в `<div>`, а шаблон добавляет соответствующий класс для отображения содержимого в столбце, например, **col-md** при использовании пакета шаблонов **Bootstrap4**.

#### Параметры

* **\*fields** (_str, Layout Object_) - Любое количество полей в качестве позиционных аргументов для отображения в `<div>`.
* **css\_id** (_str_) (опционально) - Идентификатор DOM для объекта макета, который будет добавлен в `<div>`, если он предоставлен. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS, которые должны применяться в дополнение к тем, которые объявлены самим классом. При использовании пакета шаблонов **Bootstrap4** по умолчанию удаляется **col-md**, если эта строка содержит другой **col-** класс. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<div>`.

Примеры

В своем **Layout** вы можете:

```python
Column('form_field_1', 'form_field_2', css_id='col-example')
```

Также возможно вкладывать объекты **Layout** в строку **Row**:

```python
Div(
    Column(
        Field('form_field', css_class='field-class'),
        css_class='col-sm,
    ),
    Column('form_field_2', css_class='col-sm'),
)
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
* **css\_class** (_str_) (опционально) - Классы CSS, применяемые к элементу `<div>`. По умолчанию `None`.

## _class_ layout.Div(_\*fields_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Div).

Объект макета. Он оборачивает поля в `<div>`.

#### Параметры

* **\*fields** (_str, Layout Object_) - Любое количество полей в качестве позиционных аргументов для отображения в `<div>`.
* **css\_id** (_str_) (опционально) - Идентификатор DOM для объекта макета, который будет добавлен в `<div>`, если он предоставлен. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS, которые должны применяться в дополнение к тем, которые объявлены самим классом. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<div>`.

Примеры

В своем **Layout** вы можете:

```python
Div(
    'form_field_1',
    'form_field_2',
    css_id='div-example',
    css_class='divs',
)
```

Также возможно вкладывать объекты макета в **Div**:

```python
Div(
    Div(
        Field('form_field', css_class='field-class'),
        css_class='div-class',
    ),
    Div('form_field_2', css_class='div-class'),
)
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
* **css\_class** (_str_) (опционально) - Классы CSS, применяемые к элементу `<div>`. По умолчанию `None`.

## _class_ layout.Field(_\*fields_, _css\_class=None_, _wrapper\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Field).

Объект **Layout**, обычно содержащий одно имя поля, к которому можно легко добавлять атрибуты.

#### Параметры

* **\*fields** (_str_) - Обычно одно поле, но может быть любое количество полей, которые должны отображаться с применением одних и тех же атрибутов.
* **css\_class** (_str_) (опционально) - Классы CSS, которые будут применяться к полю. Они добавляются к любым классам, включенным в словарь **attrs**. По умолчанию `None`.
* **wrapper\_class** (_str_) (опционально) - Классы CSS, которые будут использоваться при рендеринге поля. Этот класс обычно применяется к `<div>`, который обертывает теги `<label>` и `<input>` поля. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<div>`.

Примеры

```python
Field('field_name', style="color: #333;", css_class="whatever", id="field_name")
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
* **attrs** (_dict_) (опционально) - Атрибуты, применяемые к полю. Они преобразуются в атрибуты html, например, `data_id: 'test'` в словаре **attrs** станет `data-id='test'` в поле `<input>`.

## _class_ layout.Fieldset(_legend_, _\*fields_, _css\_class=None_, _css\_id=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Fieldset).

Объект макета, который оборачивает поля в `<fieldset>`.

#### Параметры

* **legend** (_str_) - Содержимое набора полей `<legend>`. Этот текст зависит от контекста, чтобы воплотить его в жизнь, см. Раздел примеров.
* **\*fields** (_str_) - Любое количество полей в качестве позиционных аргументов для отображения в `<fieldset>`.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в `<fieldset>`.

Примеры

Объект Fieldset Layout добавляется к вашему макету **Layout**, например:

```python
Fieldset("Text for the legend",
    "form_field_1",
    "form_field_2",
    css_id="my-fieldset-id",
    css_class="my-fieldset-class",
    data="my-data"
)
```

Приведенный выше макет будет отображаться как:

```html
<fieldset id="fieldset-id" class="my-fieldset-class" data="my-data">
   <legend>Text for the legend</legend>
   <!-- здесь отображаются поля формы -->
</fieldset>
```

Первый параметр — это текст легенды набора полей. Этот текст зависит от контекста, поэтому вы можете делать такие вещи, как:

```python
Fieldset("Data for {{ user.username }}",
    'form_field_1',
    'form_field_2'
)
```

## _class_ layout.HTML(_html_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#HTML).

Объект макета **Layout**. Он может содержать чистый HTML и имеет доступ ко всему контексту страницы, на которой отображается форма.

Примеры

```python
HTML("{% raw %}
{% if saved %}Data saved{% endif %}
{% endraw %}")
HTML('<input type="hidden" name="{{ step_field }}" value="{{ step0 }}" />')
```

## _class_ layout.Hidden(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Hidden).

Используется для создания скрытого дескриптора ввода для тега шаблона `{% crispy %}` .

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

Примеры

{% hint style="info" %}
атрибут **form** для **render()** не требуется для унаследованных объектов **BaseInput**.
{% endhint %}

```python
>>> hidden = Hidden("hidden", "hide-me")
>>> hidden.render("", "", Context())
'<input type="hidden" name="hidden" value="hide-me"/>'
```

Обычно вы не будете вызывать метод рендеринга для объекта напрямую. Вместо этого добавьте его в свой макет вручную или используйте метод **add\_input**:

```python
class ExampleForm(forms.Form):
[...]
def __init__(self, *args, **kwargs):
    super().__init__(*args, **kwargs)
    self.helper = FormHelper()
    self.helper.add_input(Hidden("hidden", "hide-me"))
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

## _class_ layout.Layout(_\*fields_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Layout).

Макет формы. Это соответствует объектам макета: **Fieldset**, **Row**, **Column**, **MultiField**, **HTML**, **ButtonHolder**, **Button**, **Hidden**, **Reset**, **Submit** и полям. Поля формы должны быть строками. Объекты макета **Fieldset**, **Row**, **Column**, **MultiField** и **ButtonHolder** могут содержать внутри себя другие объекты макета. Хотя **ButtonHolder** должен содержать только унаследованные классы **HTML** и **BaseInput**: **Button**, **Hidden**, **Reset** и **Submit**.

Пример:

```python
helper.layout = Layout(
    Fieldset('Company data',
        'is_company'
    ),
    Fieldset(_('Contact details'),
        'email',
        Row('password1', 'password2'),
        'first_name',
        'last_name',
        HTML('<img src="/media/somepicture.jpg"/>'),
        'company'
    ),
    ButtonHolder(
        Submit('Save', 'Save', css_class='button white'),
    ),
)
```

## _class_ layout.MultiField(_label_, _\*fields_, _label\_class=None_, _help\_text=None_, _css\_class=None_, _css\_id=None_, _template=None_, _field\_template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#MultiField).

Контейнер **MultiField** для **Bootstrap3**. Визуализирует в **MultiField** тег `<div>`.

#### Параметры

* **label** (_str_) - Метка для **MultiField**.
* **\*fields** (_str_) - Поля, которые должны отображаться в **MultiField**.
* **label\_class** (_str_) (опционально) - Классы CSS для добавления в метку с несколькими полями. По умолчанию `None`.
* **help\_text** (_str_) (опционально) - Текст справки будет доступен в контексте шаблона с несколькими полями. Это не используется в предоставленном шаблоне **bootstrap3**. По умолчанию `None`.
* **css\_id** (_str_) (опционально) - Идентификатор DOM для объекта макета, который будет добавлен в обертку `<div>`, если он предоставлен. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **field\_template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в `<fieldset>`.

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
* **field\_template** (_str_) - Шаблон, с которым будут отображаться поля.

## _class_ layout.MultiWidgetField(_\*fields_, _attrs=None_, _template=None_, _wrapper\_class=None_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#MultiWidgetField).

Объект макета. Для полей с **MultiWidget** в качестве виджета вы можете передать дополнительные атрибуты каждому виджету.

#### Параметры

* **\*fields** (_str_) - Обычно одно поле, но может быть любое количество полей, которые должны отображаться с применением одних и тех же атрибутов.
* **attrs** (_str_) (опционально) - Дополнительные атрибуты, которые будут добавлены к каждому виджету. Они добавляются к любым классам, включенным в словарь **attrs**. По умолчанию `None`.
* **wrapper\_class** (_str_) (опционально) - Классы CSS, которые будут использоваться при рендеринге поля. Этот класс обычно применяется к `<div>`, который обертывает теги `<label>` и `<input>` поля. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.

Пример:

```python
MultiWidgetField(
    'multiwidget_field_name',
    attrs=(
        {'style': 'width: 30px;'},
        {'class': 'second_widget_class'}
    ),
)
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

## _class_ layout.Reset(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Reset).

Используется для создания дескриптора кнопки сброса для тега шаблона `{% crispy %}` .

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

Примеры

{% hint style="info" %}
атрибут **form** для **render()** не требуется для унаследованных объектов **BaseInput**.
{% endhint %}

```python
>>> reset = Reset('Reset This Form', 'Revert Me!')
>>> reset.render("", "", Context())
'<input type="reset" name="reset-this-form" value="Revert Me!" '
'class="btn btn-inverse" id="reset-id-reset-this-form"/>'
```

```python
>>> reset = Reset('Reset This Form', 'Revert Me!', css_id="custom-id",
                     css_class="custom class", my_attr=True, data="my-data")
>>> reset.render("", "", Context())
'<input type="reset" name="reset-this-form" value="Revert Me!" '
'class="btn btn-inverse custom class" id="custom-id" data="my-data" my-attr/>'
```

Обычно вы не будете вызывать метод рендеринга для объекта напрямую. Вместо этого добавьте его в свой макет вручную вручную или используйте метод **add\_input**:

```python
class ExampleForm(forms.Form):
[...]
def __init__(self, *args, **kwargs):
    super().__init__(*args, **kwargs)
    self.helper = FormHelper()
    self.helper.add_input(Reset('Reset This Form', 'Revert Me!'))
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

## _class_ layout.Row(_\*fields_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Row).

Объект макета. Он заключает поля в `<divЮ`, а шаблон добавляет соответствующий класс для отображения содержимого в строке, например, **form-row** при использовании пакета шаблонов **Bootstrap4**.

#### Параметры

* **\*fields** (_str, Layout Object_) - Любое количество полей в качестве позиционных аргументов для отображения в `<div>`.
* **css\_id** (_str_) (опционально) - Идентификатор DOM для объекта макета, который будет добавлен в `<div>`, если он предоставлен. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS, которые должны применяться в дополнение к тем, которые объявлены самим классом. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<div>`.

Примеры

В своем макете **Layout** вы можете:

```python
Row('form_field_1', 'form_field_2', css_id='row-example')
```

Также возможно вкладывать объекты макета **Layout** в строку **Row**:

```python
Row(
    Div(
        Field('form_field', css_class='field-class'),
        css_class='div-class',
    ),
    Div('form_field_2', css_class='div-class'),
)
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
* **css\_class** (_str_) (опционально) - Классы CSS, применяемые к элементу `<div>`. По умолчанию `None`.

## _class_ layout.Submit(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Submit).

Используется для создания дескриптора кнопки **Submit** для тега шаблона `{% crispy %}` .

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

Примеры

{% hint style="info" %}
атрибут **form** для **render()** не требуется для унаследованных объектов **BaseInput**.
{% endhint %}

```python
>>> submit = Submit('Search the Site', 'search this site')
>>> submit.render("", "", Context())
'<input type="submit" name="search-the-site" value="search this site" '
'class="btn btn-primary" id="submit-id-search-the-site"/>'
```

```python
>>> submit = Submit('Search the Site', 'search this site', css_id="custom-id",
                     css_class="custom class", my_attr=True, data="my-data")
>>> submit.render("", "", Context())
'<input type="submit" name="search-the-site" value="search this site" '
'class="btn btn-primary custom class" id="custom-id" data="my-data" my-attr/>'
```

Обычно вы не будете вызывать метод рендеринга для объекта напрямую. Вместо этого добавьте его в свой макет вручную вручную или используйте метод **add\_input**:

```python
class ExampleForm(forms.Form):
[...]
def __init__(self, *args, **kwargs):
    super().__init__(*args, **kwargs)
    self.helper = FormHelper()
    self.helper.add_input(Submit('submit', 'Submit'))
```

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.
