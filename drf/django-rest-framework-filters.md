# django-rest-framework-filters

**django-rest-framework-filters** — это расширение [фреймворка Django REST](https://github.com/tomchristie/django-rest-framework) и [Django filter](https://github.com/carltongibson/django-filter), которое позволяет легко фильтровать отношения. Исторически это расширение также предоставляло ряд дополнительных функций и исправлений, однако количество функций сократилось, поскольку они были объединены обратно в **django-filter**.

Используя **django-rest-framework-filters**, мы можем легко делать такие вещи, как:

```http
/api/article?author__first_name__icontains=john
/api/article?is_published!=true
```

{% hint style="warning" %}
Эти документы относятся к предстоящему выпуску 1.0. Текущие документы можно найти [здесь](https://github.com/philipn/django-rest-framework-filters/blob/v0.10.2/README.rst).
{% endhint %}

{% hint style="warning" %}
Предварительная версия 1.0 совместима с django-filter 2.x и может быть установлена с помощью `pip install --pre`.
{% endhint %}

## Особенности

* Легкая фильтрация по отношениям.
* Поддержка фильтрации методов по отношениям.
* Автоматическое отрицание фильтра с помощью простого синтаксиса `param!=value`.
* Серверная часть для сложных операций с несколькими фильтрованными наборами запросов, например, `q1 | q2`.

## Требования

* **Python**: 3.5, 3.6, 3.7, 3.8
* **Django**: 1.11, 2.0, 2.1, 2.2, 3.0, 3.1
* **DRF**: 3.11
* **django-filter**: 2.1, 2.2 (Django 2.0+)

## Установка

Установите с помощью **pip** или вашего предпочтительного менеджера пакетов:

```bash
$ pip install djangorestframework-filters
```

Добавьте в настройку **INSTALLED\_APPS**:

```python
INSTALLED_APPS = [
    'rest_framework_filters',
    ...
]
```

## Использование FilterSet

Обновление с **django-filter** до **django-rest-framework-filters** очень просто:

* Импортируйте из **rest\_framework\_filters** вместо **django\_filters**.
* Используйте серверную часть **rest\_framework\_filters** вместо той, которая предоставляется **django\_filter**.

```python
# django-filter
from django_filters.rest_framework import FilterSet, filters

class ProductFilter(FilterSet):
    manufacturer = filters.ModelChoiceFilter(queryset=Manufacturer.objects.all())
    ...

# django-rest-framework-filters
import rest_framework_filters as filters

class ProductFilter(filters.FilterSet):
    manufacturer = filters.ModelChoiceFilter(queryset=Manufacturer.objects.all())
    ...
```

Чтобы использовать бэкэнд **django-rest-framework-filters**, добавьте в свои настройки следующее:

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'rest_framework_filters.backends.RestFrameworkFilterBackend', ...
    ),
    ...
```

После настройки вы можете продолжать использовать все фильтры, найденные в **django-filter**.

### Фильтрация по отношениям

Вы можете легко перемещаться по нескольким отношениям при фильтрации с помощью связанного фильтра **RelatedFilter**:

```python
from rest_framework import viewsets
import rest_framework_filters as filters


class ManagerFilter(filters.FilterSet):
    class Meta:
        model = Manager
        fields = {'name': ['exact', 'in', 'startswith']}


class DepartmentFilter(filters.FilterSet):
    manager = filters.RelatedFilter(
        ManagerFilter, field_name='manager', queryset=Manager.objects.all()
    )

    class Meta:
        model = Department
        fields = {'name': ['exact', 'in', 'startswith']}


class CompanyFilter(filters.FilterSet):
    department = filters.RelatedFilter(
        DepartmentFilter, field_name='department', queryset=Department.objects.all()
    )

    class Meta:
        model = Company
        fields = {'name': ['exact', 'in', 'startswith']}


# company viewset
class CompanyView(viewsets.ModelViewSet):
    filter_class = CompanyFilter
    ...
```
