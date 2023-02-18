# Настройка

## Настроить все входы

Чтобы настроить внешний вид и функции, скопируйте [блок настроек](https://github.com/monim67/django-bootstrap-datepicker-plus/blob/5.0.0/dev/mysite/settings.py#L140-L250) в файл **settings.py** и настройте его, следуя комментариям к инструкции. Настройки применяются глобально ко всем виджетам, используемым на вашем сайте.

```python
# По ссылке выше есть все настройки
BOOTSTRAP_DATEPICKER_PLUS = {
    "options": {
        "locale": "bn",
    },
    "variant_options": {
        "date": {
            "format": "MM/DD/YYYY",
        },
    }
}
```

Вы можете установить параметры перехвата даты и события с помощью **JavaScript**.

```javascript
window.dbdpOptions = {
    widgetParent: jQuery("#myWidgetParent"),
}
window.dbdpEvents = {
    "dp.show": e => console.log("Calendar opened"),
}
```

## Настроить один вход

Вы должны использовать параметры **options** в файле **settings.py** для применения ко всем экземплярам виджета. Если вам нужно настроить отдельный ввод виджета, передайте атрибуты и параметры непосредственно экземпляру виджета.

```python
# файл: forms.py
from bootstrap_datepicker_plus.widgets import DatePickerInput
from .models import Event
from django import forms

class ToDoForm(forms.Form):
    todo = forms.CharField()
    deadline_date = forms.DateField(widget=DatePickerInput(
        attrs={"class": "my-exclusive-input"},
        options={
            "format": "MM/DD/YYYY",
            "showTodayButton": False,
        },
    ))
```

Аналогичным образом установите параметры даты и события, используя **JavaScript**.

```javascript
window.dbdpOptions_deadline_date = {
    widgetParent: jQuery("#myWidgetParent"),
}
window.dbdpEvents_deadline_date = {
    "dp.show": e => console.log("Calendar opened"),
}
```
