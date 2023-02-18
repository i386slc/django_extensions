# Краткое руководство

Это руководство начнется с того места, где заканчивается официальное руководство по django [Написание вашего первого приложения Django, часть 4](https://docs.djangoproject.com/en/2.1/intro/tutorial04/). Если у вас нет проекта, вы можете клонировать следующий репозиторий и отладить до завершения учебника 04.

```bash
git clone https://github.com/monim67/django-polls
cd django-polls
git checkout d2.1t4
```

Модель **Question** имеет поле даты и времени. Мы собираемся создать страницу для добавления новых вопросов для опроса и страницу для их редактирования с помощью календаря выбора даты и времени в поле даты и времени. Здесь мы будем использовать Bootstrap 4. Если вы используете Bootstrap 3, просто замените 4 на 3 в приведенных ниже кодах и инструкциях. Установите следующие пакеты:

```bash
pip install django-bootstrap4
pip install django-bootstrap-datepicker-plus
```

Добавьте эти пакеты в список **INSTALLED\_APPS**, как вы делали здесь, в [Уроке 02](https://docs.djangoproject.com/en/2.1/intro/tutorial02/#activating-models).

```python
# файл: mysite/settings.py
INSTALLED_APPS = [
    'polls.apps.PollsConfig',
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    # Добавленные либы
    "bootstrap4",
    "bootstrap_datepicker_plus",
]
```

## CreateView для модели Question

Добавьте модель **CreateView** для **Question**. Метод **get\_form** используется для указания виджетов в полях формы.

```python
# файл: polls/views.py
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from bootstrap_datepicker_plus.widgets import DateTimePickerInput

from .models import Choice, Question


class CreateView(generic.edit.CreateView):
    model = Question
    fields = ['question_text', 'pub_date']
    def get_form(self):
        form = super().get_form()
        form.fields['pub_date'].widget = DateTimePickerInput()
        return form

# Остальные классы оставить без изменений
```

Создайте шаблон с именем `question_form.html` в своем приложении для отображения формы. Если вы используете другое имя, вы должны установить свойство **template\_name** класса **CreateView** в файле `views.py` выше.

```django
<!-- файл: polls/templates/polls/question_form.html -->
{% raw %}
{% load bootstrap4 %}
{% bootstrap_css %}
{% bootstrap_javascript jquery='full' %}
{{ form.media }}

<form method="post">{% csrf_token %}
    {% bootstrap_form form %}
{% endraw %}
    <input type="submit" value="Save">
</form>
```

Добавьте метод **get\_absolute\_url** в свою модель **Question**.

```python
# файл: polls/models.py
import datetime

from django.db import models
from django.urls import reverse
from django.utils import timezone


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    def get_absolute_url(self):
        return reverse('polls:detail', kwargs={'pk': self.pk})
```

Добавьте **urlpattern** для создания нового вопроса для опроса.

```python
# файл: polls/urls.py
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    # новая строка
    path('create', views.CreateView.as_view(), name='create'),
    
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Теперь запустите сервер разработки и посетите `http://localhost:8000/polls/create`, если все работает нормально, вы можете обернуть свой шаблон в правильный HTML.

```django
<!-- файл: polls/templates/polls/question_form.html -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    {% raw %}
{% load bootstrap4 %}
    {% bootstrap_css %}
    {% bootstrap_javascript jquery='full' %}
    {{ form.media }}
</head>
<body>
    <div class="container">
        <div class="col-md-3">
        <form method="post">{% csrf_token %}
            {% bootstrap_form form %}
            {% buttons %}
            <button type="submit" class="btn btn-primary">Save</button>
            {% endbuttons %}
{% endraw %}
        </form>
        </div>
    </div>
</body>
</html>
```

## UpdateView для модели Question

Теперь мы можем добавить страницу для обновления вопроса опроса. Сначала мы добавляем **UpdateView** к нашим представлениям.

```python
# файл: добавить это к polls/views.py
class UpdateView(generic.edit.UpdateView):
    model = Question
    fields = ['question_text', 'pub_date']
    def get_form(self):
        form = super().get_form()
        form.fields['pub_date'].widget = DateTimePickerInput()
        return form
```

Затем добавьте **urlpattern** для доступа к странице обновления вопроса.

```python
# файл: polls/urls.py
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('create', views.CreateView.as_view(), name='create'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    # новая строка
    path('<int:pk>/update', views.UpdateView.as_view(), name='update'),
    
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Вот и все, запустите сервер разработки и посетите `http://localhost:8000/polls/1/update`. Если все работает нормально, вы можете проверить использование в пользовательской форме и форме модели на странице «[Использование](ispolzovanie.md)» документации.
