# Использование

## Использование в общем представлении GenericView

```python
# файл: views.py
from bootstrap_datepicker_plus.widgets import DateTimePickerInput
from django.views import generic
from .models import Question

class CreateView(generic.edit.CreateView):
    model = Question
    fields = ["question_text", "pub_date"]
    def get_form(self, form_class):
        form = super().get_form(form_class)
        form.fields["pub_date"].widget = DateTimePickerInput()
        return form
```

## Использование пользовательской формы

```python
# файл: forms.py
from bootstrap_datepicker_plus.widgets import DatePickerInput
from .models import Event
from django import forms

class ToDoForm(forms.Form):
    todo = forms.CharField()
    date = forms.DateField(widget=DatePickerInput())
```

## Использование формы модели

```python
# файл: forms.py
from bootstrap_datepicker_plus.widgets import DatePickerInput
from .models import Event
from django import forms

class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ["name", "start_date", "end_date"]
        widgets = {
            "start_date": DatePickerInput(),
            "end_date": DatePickerInput(options={"format": "MM/DD/YYYY"}),
        }
```

## Типы DatePickers

Виджет содержит все типы выбора даты **date-picker**, которые могут вам когда-либо понадобиться.

```python
# файл: forms.py
from bootstrap_datepicker_plus.widgets import (
    DatePickerInput, TimePickerInput, DateTimePickerInput,
    MonthPickerInput, YearPickerInput
)
from .models import Event
from django import forms

class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = [
            "start_date", "start_time", "start_datetime",
            "start_month", "start_year"
        ]
        widgets = {
            "start_date": DatePickerInput(),
            "start_time": TimePickerInput(),
            "start_datetime": DateTimePickerInput(),
            "start_month": MonthPickerInput(),
            "start_year": YearPickerInput(),
        }
```

## Реализация выбора диапазона дат date-range-picker

**DatePickers** можно связать для выбора диапазона дат или времени.

```python
# файл: forms.py
from bootstrap_datepicker_plus.widgets import DatePickerInput, TimePickerInput
from .models import Event
from django import forms

class EventForm(forms.ModelForm):
    class Meta:
        model = Event
        fields = ["name", "start_date", "end_date", "start_time", "end_time"]
        widgets = {
            "start_date": DatePickerInput(),
            "end_date": DatePickerInput(range_from="start_date"),
            "start_time": TimePickerInput(),
            "end_time": TimePickerInput(range_from="start_time"),
        }
```
