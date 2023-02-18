# Приступаем к работе

## Требования

* Python >= 3.7
* Django >= 2.0
* Bootstrap >= 3
* jquery >= 1.7.1

## Установка

Установите пакет PyPI через pip

```bash
pip install django-bootstrap-datepicker-plus
```

Добавьте **bootstrap\_datepicker\_plus** в список **INSTALLED\_APPS** в файле `settings.py`.

```python
INSTALLED_APPS = [
    # Добавьте следующее
    "bootstrap_datepicker_plus",
]
```

## Настройка шаблона

Для следующего шага в вашем шаблоне должны присутствовать файлы **jQuery** и **Bootstrap JS/CSS**. Вы также можете использовать **django-bootstrap3**, **django-bootstrap4**, **django-bootstrap5** или **django-crispy-forms** для отображения формы.

```django
<!-- Файл: example-template.html -->
{{ form.media }}  {# Добавляет файлы виджетов JS/CSS из CDN #}
<form method="post">
  {% raw %}
{% csrf_token %}
  {% bootstrap_form form %}
{% endraw %}  {# Рендерит поля формы, используя django-bootstrapX #}
</form>
```

Если вы используете **django-crispy-forms**, вместо этого используйте фильтр **crispy** для отображения полей формы.

```django
<!-- Файл: example-template.html -->
{{ form.media }}  {# Добавляет файлы виджетов JS/CSS из CDN #}
<form method="post">
  {% raw %}
{% csrf_token %}
{% endraw %}
  {{ form | crispy }}  {# Рендерит поля формы #}
</form>
```

В качестве альтернативы вы можете использовать тег `{% crispy %}` для отображения всей формы.

```django
<!-- Файл: example-template.html -->
{% raw %}
{% crispy form %}
{% endraw %} {# Добавляет файлы виджетов JS/CSS из CDN и рендерит поля формы #}
```

Затем перейдите на страницу «[Использование](ispolzovanie.md)», чтобы узнать, как использовать его в формах и представлениях.
