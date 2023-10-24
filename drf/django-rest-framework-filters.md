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

Пример вызова фильтра:

```http
/api/companies?department__name=Accounting
/api/companies?department__manager__name__startswith=Bob
```

### queryset как вызываемый объект

Поскольку фильтр **RelatedFilter** является подклассом **ModelChoiceFilter**, аргумент **queryset** поддерживает вызываемое поведение. В следующем примере набор отделов ограничен подразделениями компании пользователя.

```python
def departments(request):
    company = request.user.company
    return company.department_set.all()

class EmployeeFilter(filters.FilterSet):
    department = filters.RelatedFilter(
        filterset=DepartmentFilter, queryset=departments
    )
    ...
```

### Рекурсивные и циклические отношения

Также поддерживаются рекурсивные отношения. Укажите путь к модулю в виде строки вместо класса набора фильтров FilterSet.

```python
class PersonFilter(filters.FilterSet):
    name = filters.AllLookupsFilter(field_name='name')
    best_friend = filters.RelatedFilter(
        'people.views.PersonFilter', field_name='best_friend',
        queryset=Person.objects.all()
    )

    class Meta:
        model = Person
```

Эта функция также полезна для циклических связей, когда связанный набор фильтров еще не создан. Обратите внимание, что вы можете передать связанный набор фильтров по имени, если он расположен в том же модуле, что и родительский набор фильтров.

```python
class BlogFilter(filters.FilterSet):
    post = filters.RelatedFilter('PostFilter', queryset=Post.objects.all())

class PostFilter(filters.FilterSet):
    blog = filters.RelatedFilter('BlogFilter', queryset=Blog.objects.all())
```

### Поддержка Filter.method

`django_filters.MethodFilter` устарел и переопределен как аргумент **method** для всех классов фильтров. Он включает в себя некоторые детали реализации старого `rest_framework_filters.MethodFilter`, но требует меньше шаблонов и его проще написания.

* Больше нет необходимости выполнять проверку пустых/нулевых значений.
* Вы можете использовать любой класс фильтра (**CharFilter**, **BooleanFilter** и т. д.), который будет проверять входные значения за вас.
* Сигнатура аргумента изменилась с `(name, qs, value)` на `(qs, name, value)`.

```python
class PostFilter(filters.FilterSet):
    # Обратите внимание на использование BooleanFilter, имени исходного поля модели
    # и аргумента метода.
    is_published = filters.BooleanFilter(
        field_name='date_published', method='filter_is_published'
    )

    class Meta:
        model = Post
        fields = ['title', 'content']

    def filter_is_published(self, qs, name, value):
        """
        `is_published` основан на поле модели `date_published`.
        Если дата публикации равна нулю, сообщение не публикуется.
        """
        # входящее значение нормализуется как логическое значение
        # с помощью BooleanFilter
        isnull = not value
        lookup_expr = LOOKUP_SEP.join([name, 'isnull'])

        return qs.filter(**{lookup_expr: isnull})

class AuthorFilter(filters.FilterSet):
    posts = filters.RelatedFilter('PostFilter', queryset=Post.objects.all())

    class Meta:
        model = Author
        fields = ['name']
```

Вышеупомянутое позволит включить следующие вызовы фильтров:

```http
/api/posts?is_published=true
/api/authors?posts__is_published=true
```

При первом вызове API метод фильтра получает набор queryset сообщений. Во втором случае он получает набор запросов (queryset) пользователей. Метод фильтра в примере изменяет имя поиска для работы во всей связи, позволяя находить опубликованные сообщения или авторов, которые опубликовали сообщения.

### Автоматический фильтр Neagtion/Exclusion

Наборы фильтров поддерживают автоматическое исключение, используя простой синтаксис `param!=value`. Этот синтаксис внутренне устанавливает свойство **exclude** для фильтра.

```http
/api/page?title!=The%20Park
```

Этот синтаксис поддерживает обычную фильтрацию в сочетании с фильтрацией исключений. Например, следующая команда будет искать все статьи, содержащие `"Hello"` в заголовке, исключая те, которые содержат `"World"`.

```http
/api/articles?title__contains=Hello&title__contains!=World
```

Обратите внимание, что большинство фильтров принимают только один параметр запроса. В приведенном выше примере **title\_\_contains** и **title\_\_contains!** интерпретируются как два отдельных параметра запроса. Следующее, вероятно, будет неверным, хотя это зависит от особенностей конкретного класса фильтра:

```http
/api/articles?title__contains=Hello&title__contains!=World&title_contains!=Friend
```

### Разрешение любого типа поиска в поле

Если вам нужно включить несколько поисков для поля, **django-filter** предоставляет синтаксис dict для `Meta.fields`.

```python
class ProductFilter(filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            'price': ['exact', 'lt', 'gt', ...],
        }
```

**django-rest-framework-filters** также позволяет вам включить все возможные поиски для любого поля. Этого можно достичь с помощью **AllLookupsFilter** или значения `"__all__"` в синтаксисе стиля dict `Meta.fields`. Созданные фильтры (`Meta.fields`, `AllLookupsFilter`) никогда не переопределят объявленные вами фильтры.

Обратите внимание, что использование всех поисков сопровождается теми же предупреждениями, что и включение полей `"__all__"` в формах django ([документация](https://docs.djangoproject.com/en/1.10/topics/forms/modelforms/#selecting-the-fields-to-use)). Раскрытие всех поисков может позволить пользователям создавать запросы, которые непреднамеренно приводят к утечке данных. Используйте эту функцию ответственно.

```python
class ProductFilter(filters.FilterSet):
    # Не переопределено `__all__`
    price__gt = filters.NumberFilter(
        field_name='price', lookup_expr='gt', label='Minimum price'
    )

    class Meta:
        model = Product
        fields = {
            'price': '__all__',
        }

# или

class ProductFilter(filters.FilterSet):
    price = filters.AllLookupsFilter()

    # Не переопределено `AllLookupsFilter`
    price__gt = filters.NumberFilter(
        field_name='price', lookup_expr='gt', label='Minimum price'
    )

    class Meta:
        model = Product
```

Вы не можете комбинировать **AllLookupsFilter** с **RelatedFilter**, так как имена фильтров будут конфликтовать.

```python
class ProductFilter(filters.FilterSet):
    manufacturer = filters.RelatedFilter(
        'ManufacturerFilter', queryset=Manufacturer.objects.all()
    )
    manufacturer = filters.AllLookupsFilter()
```

Чтобы обойти эту проблему, у вас есть следующие варианты:

```python
class ProductFilter(filters.FilterSet):
    manufacturer = filters.RelatedFilter(
        'ManufacturerFilter', queryset=Manufacturer.objects.all()
    )

    class Meta:
        model = Product
        fields = {
            'manufacturer': '__all__',
        }

# или

class ProductFilter(filters.FilterSet):
    manufacturer = filters.RelatedFilter(
        'ManufacturerFilter', queryset=Manufacturer.objects.all(), lookups='__all__'
    )  # `lookups` также принимает список

    class Meta:
        model = Product
```

### Могу ли я смешивать и сочетать фильтры django-filter и django-rest-framework-filters?

Да, ты можешь. **django-rest-framework-filters** — это просто расширение **django-filter**. Обратите внимание, что фильтр **RelatedFilter** и другие функции **django-rest-framework-filters** предназначены для работы с `rest_framework_filters.FilterSet` и не будут работать с `django_filters.FilterSet`. Однако целевой объект `RelatedFilter.filterset` может указывать на **FilterSet** из любого пакета, и обе реализации **FilterSet** совместимы с другим DRF backend.

```python
# правильно
class VanillaFilter(django_filters.FilterSet):
    ...

class DRFFilter(rest_framework_filters.FilterSet):
    vanilla = rest_framework_filters.RelatedFilter(
        filterset=VanillaFilter, queryset=...
    )


# неправильно
class DRFFilter(rest_framework_filters.FilterSet):
    ...

class VanillaFilter(django_filters.FilterSet):
    drf = rest_framework_filters.RelatedFilter(filterset=DRFFilter, queryset=...)
```

### Предостережения и ограничения

#### MultiWidget несовместим

**djangorestframework-filters** несовместим с виджетами форм, которые анализируют имена запросов, отличающиеся от имени атрибута фильтра. Хотя это практически применимо только к **MultiWidget**, это общее ограничение, которое затрагивает пользовательские виджеты, которые также имеют такое поведение. Затронутые фильтры включают **RangeFilter**, **DateTimeFromToRangeFilter**, **DateFromToRangeFilter**, **TimeRangeFilter** и **NumericRangeFilter**.

Чтобы продемонстрировать несовместимость, возьмем следующий набор фильтров:

```python
class PostFilter(FilterSet):
    publish_date = filters.DateFromToRangeFilter()
```

Приведенный выше фильтр позволяет пользователям выполнять запрос диапазона **range** по дате публикации. Класс фильтра внутренне использует **MultiWidget** для отдельного анализа значений верхней и нижней границы. Несовместимость заключается в том, что **MultiWidget** добавляет индекс к своим внутренним именам виджетов. Вместо анализа **publish\_date** он ожидает **publish\_date\_0** и **publish\_date\_1**. Это можно исправить, включив имя атрибута в строку запроса, хотя это не рекомендуется.

```http
?publish_date_0=2016-01-01&publish_date_1=2016-02-01&publish_date=
```

**MultiWidget** также не рекомендуется, поскольку:

* самоанализ поля `core-api` терпит неудачу по тем же причинам
* `_0` и `_1` менее удобны для API, чем `_min` и `_max`.

Рекомендуемые решения:

* Создайте отдельные фильтры для каждого из вложенных виджетов (например, `publish_date_min` и `publish_date_max`).
* Используйте фильтр на основе CSV, например фильтр, полученный из `BaseCSVFilter`/`BaseInFilter`/`BaseRangeFilter`. Например,

```http
?publish_date__range=2016-01-01,2016-02-01
```

## Комплексные операции

**ComplexFilterBackend** определяет пользовательский синтаксис строки запроса и процесс кодирования, который позволяет выражать [сложные запросы](https://docs.djangoproject.com/en/2.0/topics/db/queries/#complex-lookups-with-q-objects). Этот синтаксис расширяет стандартные строки запросов, позволяя определять несколько наборов параметров и операторов для объединения запросов.

{% hint style="warning" %}
Обратите внимание, что эта функция является экспериментальной. Могут возникнуть ошибки, а серверная часть может быть изменена.
{% endhint %}

Чтобы лучше понять серверную часть, рассмотрим запрос на поиск всех статей, заголовки которых начинаются с `"Who"` или `"What"`. Базовый запрос может быть представлен следующим образом:

```python
q1 = Article.objects.filter(title__startswith='Who')
q2 = Article.objects.filter(title__startswith='What')
return q1 | q2
```

Теперь рассмотрим запрос, но модифицированный верхней и нижней границей даты:

```python
q1 = Article.objects.filter(
    title__startswith='Who'
).filter(publish_date__lte='2005-01-01')
q2 = Article.objects.filter(
    title__startswith='What'
).filter(publish_date__gte='2010-01-01')
return q1 | q2
```

Используя только **FilterSet**, безусловно, можно представить первый запрос, написав собственный класс фильтра. Однако для последнего запроса это менее осуществимо, поскольку необходимо проверить несколько наборов данных различных типов и поисков. Напротив, **ComplexFilterBackend** может создать этот сложный запрос с помощью произвольной комбинации простого фильтра. Для поддержки вышеизложенного необходимо создать строку запроса с минимальными изменениями. Некодированный пример:

```
(title__startswith=Who&publish_date__lte=2005-01-01) | (title__startswith=What&publish_date__gte=2010-01-01)
```

По умолчанию серверная часть объединяет запросы с операторами `&` (AND) и `|` (OR) и поддерживает унарное отрицание `~`. Например,

```
(param1=value1) & (param2=value2) | ~(param3=value3)
```

Бэкэнд поддерживает как стандартные, так и сложные запросы. Для выполнения сложных запросов запрос должен быть закодирован и установлен как значение **complex\_filter\_param** (по умолчанию — **filters**). Для выполнения стандартных запросов используйте серверную часть так же, как **RestFrameworkFilterBackend**.

### Настройка ComplexFilterBackend

Как и в случае с другими серверными модулями, **ComplexFilterBackend** необходимо добавить к атрибуту **filter\_backends** представления. Либо добавьте его в параметр **DEFAULT\_FILTER\_BACKENDS**, либо установите в качестве бэкэнда в классе представления.

```python
REST_FRAMEWORK = {
    'DEFAULT_FILTER_BACKENDS': (
        'rest_framework_filters.backends.ComplexFilterBackend',
    ),
}

# или

class MyViewSet(generics.ListAPIView):
    filter_backends = (rest_framework_filters.backends.ComplexFilterBackend, )
    ...
```

Вы можете настроить способ объединения запросов, создав подкласс **ComplexFilterBackend** и переопределив атрибут **operators**. **operators** — это отображение символов операторов в функциях, которые объединяют два набора запросов. Например, карту можно переопределить, чтобы использовать `QuerySet.intersection()` и `QuerySet.union()` вместо `&` и `|`.

```python
class CustomizedBackend(ComplexFilterBackend):
    operators = {
        '&': QuerySet.intersection,
        '|': QuerySet.union,
        '-': QuerySet.difference,
    }
```

Унарное отрицание **negation** зависит от внутреннего устройства ORM и при определенных обстоятельствах может давать сбои. Если с этой функцией возникают проблемы, ее можно отключить, установив для атрибута **negation** значение `False` в серверном классе. Если у вас возникли ошибки, пожалуйста, откройте проблему в [системе отслеживания ошибок](https://github.com/philipn/django-rest-framework-filters/issues/).

### Сложная кодировка строки запроса

Ниже приведена процедура кодирования сложного запроса:

* Преобразуйте параметры запроса в отдельные строки запроса.
* URL-кодирование отдельных строк запроса.
* Заключите закодированные строки в круглые скобки и объедините их с помощью операторов.
* URL-кодирование всей строки запроса.
* Установите в качестве значения параметра сложного фильтра (например, `?filters=<complex querystring>`).

Обратите внимание, что **filters** — это имя параметра по умолчанию, и их можно переопределить в классе серверной части.

Используя первый пример, эти шаги можно визуализировать следующим образом:

* `title__startswith=Who`, `title__startswith=What`
* `title__startswith%3DWho`, `title__startswith%3DWhat`
* `(title__startswith%3DWho) | (title__startswith%3DWhat)`
* `%28title__startswith%253DWho%29%20%7C%20%28title__startswith%253DWhat%29`
* `filters=%28title__startswith%253DWho%29%20%7C%20%28title__startswith%253DWhat%29`

### Обработка ошибок

**ComplexFilterBackend** выдаст любые ошибки декодирования под сложным именем параметра фильтрации. Например,

```json
{
    "filters": [
        "Invalid querystring operator. Matched: 'foo'."
    ]
}
```

При фильтрации наборов запросов ошибки проверки набора фильтров будут собираться и возникать под сложным именем параметра фильтрации, а затем под декодированной строкой запроса набора фильтров. Для сложного запроса типа `(a=1&b=2) | (c=3&d=4)`, ошибки будут возникать следующим образом:

```json
{
    "filters": {
        "a=1&b=2": {
            "a": ["..."]
        },
        "c=3&d=4": {
            "c": ["..."]
        }
    }
{
```

## Миграция на 1.0

### Серверная часть переименована, предоставляет новые шаблоны

Серверная часть была переименована с **DjangoFilterBackend** в **RestFrameworkFilterBackend** и теперь использует собственные пути к шаблонам, расположенные в **rest\_framework\_filters** вместо `django_filters/rest_framework`.

Для загрузки включенных шаблонов необходимо добавить **rest\_framework\_filters** в настройку **INSTALLED\_APPS**.

### ReleaseFilter.queryset теперь обязателен.

Модель связанного набора фильтров больше не используется для предоставления значения по умолчанию для `RelatedFilter.queryset`. Это изменение снижает вероятность непреднамеренного раскрытия данных в отображаемых формах фильтра. Теперь вы должны явно указать аргумент **queryset** или переопределить метод `get_queryset()` (см. [вызываемые объекты набора запросов](django-rest-framework-filters.md#queryset-kak-vyzyvaemyi-obekt)).

### get\_filters() переименован в get\_request\_filters()

**django-filter** добавил в свой API метод класса `get_filters()`, поэтому этот метод был переименован.

## Публикация

```bash
$ pip install -U twine setuptools wheel
$ rm -rf dist/ build/
$ python setup.py sdist bdist_wheel
$ twine upload dist/*
```

## Авторские права и лицензия

Авторские права (c) Филип Нейстром, 2013–2015 гг., и Райан П. Килби, 2016–2019 гг. Подробности см. в разделе [ЛИЦЕНЗИЯ](https://github.com/philipn/django-rest-framework-filters/blob/master/LICENSE).
