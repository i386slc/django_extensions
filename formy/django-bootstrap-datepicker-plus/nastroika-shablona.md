# Настройка шаблона

Календарь пока недоступен для настройки, но шаблон поля ввода можно настроить. Сначала вам нужно создать HTML-шаблон для ввода виджета.

```django
<!-- файл: my_app/templates/my_app/custom-input.html -->

<h5>This is a customized input from template</h5>
<div class="input-group dbdp">
    {% raw %}
{% include 'django/forms/widgets/text.html' %}
{% endraw %}
    <div class="input-group-addon input-group-append input-group-text">
        <i class="{{ addon_icon_class }}"></i>
    </div>
</div>
```

Затем добавьте его в настройки **BOOTSTRAP\_DATEPICKER\_PLUS**.

```python
BOOTSTRAP_DATEPICKER_PLUS = {
    "template_name": "my_app/custom-input.html",
}
```
