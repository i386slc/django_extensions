# Приступая к работе (django-filter)

**Django-filter** предоставляет простой способ отфильтровать набор запросов на основе параметров, предоставленных пользователем. Скажем, у нас есть модель **Product**, и мы хотим, чтобы наши пользователи могли фильтровать, какие продукты они видят на странице списка.

{% hint style="info" %}
Если вы используете **django-filter** с **Django Rest Framework**, рекомендуется прочитать документы «[Интеграция с DRF](https://django-filter.readthedocs.io/en/main/guide/rest\_framework.html#drf-integration)» после этого руководства.
{% endhint %}

## Модель

Начнем с нашей модели:

```python
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=255)
    price = models.DecimalField(max_digits=5, decimal_places=2)
    description = models.TextField()
    release_date = models.DateField()
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
```

## Фильтр

У нас есть несколько полей, и мы хотим, чтобы наши пользователи могли фильтровать их по названию, цене или дате выпуска. Для этого мы создаем **FilterSet**:

```python
import django_filters

class ProductFilter(django_filters.FilterSet):
    name = django_filters.CharFilter(lookup_expr='iexact')

    class Meta:
        model = Product
        fields = ['price', 'release_date']
```

Как видите, здесь используется API, очень похожий на **ModelForm** Django. Как и в случае с **ModelForm**, мы также можем переопределять фильтры или добавлять новые, используя декларативный синтаксис.

### Объявление фильтров

Декларативный синтаксис обеспечивает наибольшую гибкость при создании фильтров, однако он довольно многословен. Мы будем использовать приведенный ниже пример, чтобы обрисовать [основные аргументы фильтра](https://django-filter.readthedocs.io/en/main/ref/filters.html#core-arguments) в **FilterSet**:

```python
class ProductFilter(django_filters.FilterSet):
    price = django_filters.NumberFilter()
    price__gt = django_filters.NumberFilter(field_name='price', lookup_expr='gt')
    price__lt = django_filters.NumberFilter(field_name='price', lookup_expr='lt')

    release_year = django_filters.NumberFilter(
        field_name='release_date', lookup_expr='year'
    )
    release_year__gt = django_filters.NumberFilter(
        field_name='release_date', lookup_expr='year__gt'
    )
    release_year__lt = django_filters.NumberFilter(
        field_name='release_date', lookup_expr='year__lt'
    )

    manufacturer__name = django_filters.CharFilter(lookup_expr='icontains')

    class Meta:
        model = Product
        fields = ['price', 'release_date', 'manufacturer']
```

Есть два основных аргумента в пользу фильтров:

* **field\_name**: имя поля модели для фильтрации. Вы можете перемещаться по «путям отношений», используя синтаксис Django `__` для фильтрации полей в связанной модели, например, `manufacturer__name`.
* **lookup\_expr**: [поле поиска](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups) для использования при фильтрации. Синтаксис Django `__` снова можно использовать для поддержки преобразований поиска, например, `year__gte`.

Вместе поля **field\_name** и **lookup\_expr** представляют полное выражение поиска Django. Подробное объяснение выражений поиска приведено в [справочнике по поиску Django](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups). **django-filter** поддерживает выражения, содержащие как преобразования, так и окончательный поиск.

### Генерация фильтров с помощью Meta.fields

Мета-класс **FilterSet** предоставляет атрибут **fields**, который можно использовать для простого указания нескольких фильтров без значительного дублирования кода. Базовый синтаксис поддерживает список из нескольких имен полей:

```python
import django_filters

class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ['price', 'release_date']
```

Приведенное выше генерирует `'exact'` запросы для полей `'price'` и `'release_date'`.

Кроме того, можно использовать словарь для указания нескольких выражений поиска для каждого поля:

```python
import django_filters

class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            'price': ['lt', 'gt'],
            'release_date': ['exact', 'year__gt'],
        }
```

Вышеприведенное сгенерирует фильтры `'price__lt'`, `'price__gt'`, `'release_date'` и `'release_date__year__gt'`.

{% hint style="info" %}
Тип фильтра поиска `'exact'` является неявным значением по умолчанию и поэтому никогда не добавляется к имени фильтра. В приведенном выше примере точным фильтром даты выпуска является `'release_date'`, а не `'release_date__exact'`. Это можно переопределить настройкой **FILTERS\_DEFAULT\_LOOKUP\_EXPR**.
{% endhint %}

Элементы в последовательности полей в классе **Meta** могут включать «relationship path», использующие синтаксис Django `__` для фильтрации полей в связанной модели:

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ['manufacturer__country']
```

### Переопределение фильтров по умолчанию

Подобно `django.contrib.admin.ModelAdmin`, можно переопределить фильтры по умолчанию для всех полей моделей одного типа, используя **filter\_overrides** в классе **Meta**:

```python
class ProductFilter(django_filters.FilterSet):

    class Meta:
        model = Product
        fields = {
            'name': ['exact'],
            'release_date': ['isnull'],
        }
        filter_overrides = {
            models.CharField: {
                'filter_class': django_filters.CharFilter,
                'extra': lambda f: {
                    'lookup_expr': 'icontains',
                },
            },
            models.BooleanField: {
                'filter_class': django_filters.BooleanFilter,
                'extra': lambda f: {
                    'widget': forms.CheckboxInput,
                },
            },
        }
```

### Фильтрация на основе запросов

**FilterSet** может быть инициализирован необязательным аргументом **request**. Если объект запроса передан, вы можете получить доступ к запросу во время фильтрации. Это позволяет фильтровать запросы по свойствам, таким как текущий вошедший в систему пользователь или заголовок **Accepts-Languages**.

{% hint style="info" %}
Не гарантируется, что запрос **request** будет передан экземпляру **FilterSet**. Любой код, зависящий от запроса, должен обрабатывать случай `None`.
{% endhint %}

### Фильтрация основного .qs

Чтобы отфильтровать основной набор запросов **queryset**, просто переопределите свойство `FilterSet.qs`. Например, вы можете отфильтровать статьи блога, выбрав только те, которые опубликованы, и те, которые принадлежат вошедшему в систему пользователю (предположительно, черновики статей автора).

```python
class ArticleFilter(django_filters.FilterSet):

    class Meta:
        model = Article
        fields = [...]

    @property
    def qs(self):
        parent = super().qs
        author = getattr(self.request, 'user', None)

        return parent.filter(is_published=True) \
            | parent.filter(author=author)
```

### Фильтрация связанного queryset для ModelChoiceFilter

Аргумент набора запросов **queryset** для **ModelChoiceFilter** и **ModelMultipleChoiceFilter** поддерживает вызываемое поведение. Если **callable** передается, он будет вызываться с запросом **request** в качестве единственного аргумента. Это позволяет выполнять те же виды фильтрации на основе запросов, не прибегая к переопределению `FilterSet.__init__`.

```python
def departments(request):
    if request is None:
        return Department.objects.none()

    company = request.user.company
    return company.department_set.all()

class EmployeeFilter(filters.FilterSet):
    department = filters.ModelChoiceFilter(queryset=departments)
    ...
```

### Настройка фильтрации с помощью Filter.method

Вы можете управлять поведением фильтра, указав **method** для выполнения фильтрации. Дополнительные сведения см. в [справочнике по методу](https://django-filter.readthedocs.io/en/main/ref/filters.html#filter-method). Обратите внимание, что вы можете получить доступ к свойствам набора фильтров, например к **request**.

```python
class F(django_filters.FilterSet):
    username = CharFilter(method='my_custom_filter')

    class Meta:
        model = User
        fields = ['username']

    def my_custom_filter(self, queryset, name, value):
        return queryset.filter(**{
            name: value,
        })
```

## Вид (представление)

Теперь нам нужно написать представление:

```python
def product_list(request):
    f = ProductFilter(request.GET, queryset=Product.objects.all())
    return render(request, 'my_app/template.html', {'filter': f})
```

Если аргумент **queryset** не указан, будут использоваться все элементы в диспетчере модели по умолчанию.

Если вы хотите получить доступ к отфильтрованным объектам в ваших представлениях, например, если вы хотите разбить их на страницы, вы можете сделать это. Они в `f.qs`

## Конфигурация URL

Нам нужен шаблон URL для вызова представления:

```python
path('list/', views.product_list, name="product-list")
```

## Шаблон

И, наконец, нам нужен шаблон:

```django
{% raw %}
{% extends "base.html" %}

{% block content %}
    <form method="get">
        {{ filter.form.as_p }}
        <input type="submit" />
    </form>
    {% for obj in filter.qs %}
        {{ obj.name }} - ${{ obj.price }}<br />
    {% endfor %}
{% endblock %}
{% endraw %}
```

И это все! Атрибут **form** содержит обычную форму Django, и когда мы перебираем `FilterSet.qs`, мы получаем объекты в результирующем наборе запросов.

## Общий вид (представление) и конфигурация

В дополнение к описанному выше использованию существует также общее представление на основе классов, включенное в **django-filter**, которое находится по адресу `django_filters.views.FilterView`. Вы должны предоставить либо **model**, либо аргумент **filterset\_class**, аналогичный **ListView** в самом Django:

```python
# urls.py
from django.urls import path
from django_filters.views import FilterView
from myapp.models import Product

urlpatterns = [
    path("list/", FilterView.as_view(model=Product), name="product-list"),
]
```

Если вы дополнительно предоставляете **model**, вы можете установить **filterset\_fields**, чтобы указать список или кортеж полей, которые вы хотите включить для автоматического построения класса набора фильтров.

Вы должны предоставить шаблон в `<app>/<model>_filter.html`, который получает параметра контекста **filter**. Кроме того, контекст будет содержать **object\_list**, содержащий отфильтрованный набор запросов.

Устаревшее функциональное универсальное представление по-прежнему включено в **django-filter**, хотя его использование устарело. Его можно найти по адресу `django_filters.views.object_filter`. Вы должны предоставить ему те же аргументы, что и представление на основе класса:

```python
# urls.py
from django.urls import path
from django_filters.views import object_filter
from myapp.models import Product

urlpatterns = [
    path("list/", object_filter, {'model': Product}, name="product-list"),
]
```

Необходимый шаблон и его переменные контекста также будут такими же, как и в представлении на основе классов выше.
