# API templatetags

## _class_ crispy\_forms\_tags.BasicNode(_form_, _helper_, _template\_pack=None_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#BasicNode).

Базовый объект **Node**, на который мы можем положиться для объектов **Node** в обычных тегах шаблона. Я создал это, потому что большинству тегов, которые мы будем использовать, потребуются как объект формы, так и вспомогательная строка. Это обрабатывает как объект формы, так и разбирает вспомогательную строку на атрибуты, которые могут легко обрабатываться шаблонами.

### get\_render(_context_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#BasicNode.get\_render).

Возвращает объект **Context** со всем необходимым для рендеринга формы.

#### Параметры

* **context** - Переменная **django.template.Context**, содержащая контекст для узла

**self.form** и **self.helper** разрешаются в реальные объекты Python, разрешая их из контекста. **Actual\_form** может быть формой или набором форм. Если это набор форм, для **is\_formset** установлено значение `True`. Если у помощника есть макет, мы используем его для рендеринга формы или форм набора форм.

### get\_response\_dict(_helper_, _context_, _is\_formset_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#BasicNode.get\_response\_dict).

Возвращает словарь со всеми параметрами, необходимыми для отображения формы/набора форм в шаблоне.

#### Параметры

* **context** - _**django.template.Context**_ для узла
* **is\_formset** - Логическое значение. Если установлено значение `True`, это означает, что мы работаем с набором форм.

## _class_ crispy\_forms\_tags.CrispyFormNode(_form_, _helper_, _template\_pack=None_)

### render(_context_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#CrispyFormNode.render).

Возвращает узел, представленный в виде строки.

## _class_ crispy\_forms\_tags.ForLoopSimulator(_formset_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#ForLoopSimulator).

Точно имитирует тег **forloop**:

```django
{% raw %}
{% for form in formset.forms %}
{% endraw %}
```

Если `{% crispy %}` рендерит набор форм с помощью помощника, мы вводим объект **ForLoopSimulator** в контекст как **forloop**, чтобы формы набора форм могли делать такие вещи, как:

```python
Fieldset("Item {{ forloop.counter }}", [...])
HTML("{% raw %}
{% if forloop.first %}First form text{% endif %}
{% endraw %}"
```

### iterate()

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#ForLoopSimulator.iterate).

Обновляет значения, как если бы мы перебирали **for**.

## crispy\_forms\_tags.do\_uni\_form(_parser_, _token_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#do\_uni\_form).

Вам нужно передать как минимум объект `form/formset`, а также можно передать необязательный объект **crispy\_forms.helpers.FormHelper**.

**helper** (необязательно): объект **crispy\_forms.helper.FormHelper**.

Применение:

```django
{% raw %}
{% load crispy_tags %}
{% crispy form form.helper %}
{% endraw %}
```

Вы также можете указать пакет шаблонов в качестве третьего аргумента:

```django
{% raw %}
{% crispy form form.helper 'bootstrap' %}
{% endraw %}
```

Если атрибут **FormHelper** называется **helper**, вы можете просто сделать:

```django
{% raw %}
{% crispy form %}
{% crispy form 'bootstrap' %}
{% endraw %}
```

## crispy\_forms\_tags.whole\_uni\_form\_template(_template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#whole\_uni\_form\_template).

## crispy\_forms\_tags.whole\_uni\_formset\_template(_template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#whole\_uni\_formset\_template).

## crispy\_forms\_filters.as\_crispy\_errors(_form_, _template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#as\_crispy\_errors).

Отображает только ошибки формы так же, как **django-crispy-forms**:

```django
{% raw %}
{% load crispy_forms_tags %}
{% endraw %}
{{ form|as_crispy_errors }}
```

или:

```django
{{ form|as_crispy_errors:"bootstrap4" }}
```

## crispy\_forms\_filters.as\_crispy\_field(_field_, _template\_pack=\<SimpleLazyObject: 'bootstrap4'>_, _label\_class=''_, _field\_class=''_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#as\_crispy\_field).

Визуализирует поле формы как поле **django-crispy-forms**:

```django
{% raw %}
{% load crispy_forms_tags %}
{% endraw %}
{{ form.field|as_crispy_field }}
```

или:

```django
{{ form.field|as_crispy_field:"bootstrap4" }}
```

## crispy\_forms\_filters.as\_crispy\_form(_form_, _template\_pack=\<SimpleLazyObject: 'bootstrap4'>_, _label\_class=''_, _field\_class=''_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#as\_crispy\_form).

Оригинальный и все еще очень полезный способ создания элегантной формы/набора форм **div**:

```django
{% raw %}
{% load crispy_forms_tags %}

<form class="my-class" method="post">
    {% csrf_token %}
{% endraw %}
    {{ myform|crispy }}
</form>
```

или, если вы хотите явно установить пакет шаблонов:

```django
{{ myform|crispy:"bootstrap4" }}
```

В **bootstrap3** или **bootstrap4** для горизонтальных форм вы можете сделать:

```django
{{ myform|label_class:"col-lg-2",field_class:"col-lg-8" }}
```

## crispy\_forms\_filters.flatatt\_filter(_attrs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#flatatt\_filter).

## crispy\_forms\_filters.optgroups(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#optgroups).

Фильтр шаблонов, помогающий отображать поля с помощью групп опций.

#### Возвращает

Кортеж из label, option, index.

* **label** - Групповая метка для сгруппированных групп опций (`None`, если входы не сгруппированы).
* **option** - Словарь, содержащий информацию для отображения опции:

```python
{
    "name": "checkbox_select_multiple",
    "value": 1,
    "label": 1,
    "selected": False,
    "index": "0",
    "attrs": {"id": "id_checkbox_select_multiple_0"},
    "type": "checkbox",
    "template_name": "django/forms/widgets/checkbox_option.html",
    "wrap_label": True,
}
```

* **index** - Индекс группы

## crispy\_forms\_filters.uni\_form\_template(_template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#uni\_form\_template).

## crispy\_forms\_filters.uni\_formset\_template(_template\_pack=\<SimpleLazyObject: 'bootstrap4'>_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_filters.html#uni\_formset\_template).

## _class_ crispy\_forms\_field.CrispyFieldNode(_field_, _attrs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#CrispyFieldNode).

### render(context)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#CrispyFieldNode.render).

Возвращает узел, представленный в виде строки.

## crispy\_forms\_field.classes(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#classes).

Возвращает классы CSS поля.

## crispy\_forms\_field.crispy\_addon(_field_, _append=''_, _prepend=''_, _form\_show\_labels=True_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#crispy\_addon).

Визуализирует поле формы, используя добавленный bootstrap перед или после текст:

```django
{% raw %}
{% crispy_addon form.my_field prepend="$" append=".00" %}
{% endraw %}
```

Вы также можете просто добавить перед или после так:

```django
{% raw %}
{% crispy_addon form.my_field prepend=”$” %}
{% crispy_addon form.my_field append=”.00” %}
{% endraw %}
```

## crispy\_forms\_field.crispy\_field(_parser_, _token_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#crispy\_field).

```django
{% raw %}
{% crispy_field field attrs %}
{% endraw %}
```

## crispy\_forms\_field.css\_class(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#css\_class).

Возвращает имя класса виджетов в нижнем регистре

## crispy\_forms\_field.is\_checkbox(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_checkbox).

## crispy\_forms\_field.is\_checkboxselectmultiple(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_checkboxselectmultiple).

## crispy\_forms\_field.is\_clearable\_file(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_clearable\_file).

## crispy\_forms\_field.is\_file(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_file).

## crispy\_forms\_field.is\_multivalue(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_multivalue).

## crispy\_forms\_field.is\_password(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_password).

## crispy\_forms\_field.is\_radioselect(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_radioselect).

## crispy\_forms\_field.is\_select(_field_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#is\_select).

## crispy\_forms\_field.pairwise(_iterable_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_field.html#pairwise).

s -> (s0,s1), (s2,s3), (s4, s5), …
