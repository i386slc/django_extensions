# API templatetags

## _class_ templatetags.crispy\_forms\_tags.BasicNode(_form_, _helper_, _template\_pack=None_)

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

## _class_ templatetags.crispy\_forms\_tags.CrispyFormNode(_form_, _helper_, _template\_pack=None_)

### render(_context_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/templatetags/crispy\_forms\_tags.html#CrispyFormNode.render).

Возвращает узел, представленный в виде строки.

## _class_ templatetags.crispy\_forms\_tags.ForLoopSimulator(_formset_)

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

## templatetags.crispy\_forms\_tags.do\_uni\_form(_parser_, _token_)

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
