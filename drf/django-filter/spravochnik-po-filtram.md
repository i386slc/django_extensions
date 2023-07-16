# Справочник по фильтрам

Это справочный документ со списком фильтров и их аргументов.

## Основные аргументы

Ниже приведены основные аргументы, применимые **ко всем фильтрам**. Обратите внимание, что они объединяются для создания полного [выражения поиска](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups), которое является левой частью вызова ORM `.filter()`.

### field\_name

Имя поля модели, по которому выполняется фильтрация. Если этот аргумент не указан, по умолчанию используется имя атрибута фильтра в классе **FilterSet**. Имена полей могут пересекать отношения, соединяя связанные части с помощью разделителя поиска ORM (`__`). например, `manufacturer__name`.

### lookup\_expr

[Поиск поля](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups), который должен выполняться при вызове фильтра. По умолчанию **exact**. **Lookup\_expr** может содержать преобразования, если части выражения соединены разделителем поиска ORM (`__`). например, отфильтруйте дату и время по части года `year__gt`.

## Только ключевые аргументы (ключ-значение)

Ниже приведены **необязательные** аргументы, которые можно использовать для изменения поведения всех фильтров.

### label

Метка, как она будет отображаться в HTML, аналогична аргументу **label** поля формы. Если **label** не указана, будет создана подробная метка на основе поля **field\_name** и частей **lookup\_expr** (см.: <mark style="color:purple;">FILTERS\_VERBOSE\_LOOKUPS</mark>).

### method

Необязательный аргумент, указывающий фильтру, как обрабатывать набор запросов. Он может принимать либо вызываемый объект, либо имя метода в **FilterSet**. Вызываемый объект получает **QuerySet**, имя поля модели для фильтрации и значение для фильтрации. Он должен вернуть отфильтрованный набор запросов.

Обратите внимание, что значение проверяется полем `Filter.field`, поэтому преобразование необработанных значений и проверка пустых значений не нужны.

```python
class F(FilterSet):
    """Отфильтровать книги по тому, опубликованы ли книги или нет"""
    published = BooleanFilter(field_name='published_on', method='filter_published')

    def filter_published(self, queryset, name, value):
        # построить полное выражение поиска.
        lookup = '__'.join([name, 'isnull'])
        return queryset.filter(**{lookup: False})

        # в качестве альтернативы вы можете жестко закодировать поиск, например,
        # return queryset.filter(published_on__isnull=False)

    class Meta:
        model = Book
        fields = ['published']


# Вызываемые объекты также могут быть определены вне области видимости класса.
def filter_not_empty(queryset, name, value):
    lookup = '__'.join([name, 'isnull'])
    return queryset.filter(**{lookup: False})

class F(FilterSet):
    """Отфильтровать книги по тому, опубликованы ли книги или нет"""
    published = BooleanFilter(field_name='published_on', method=filter_not_empty)

    class Meta:
        model = Book
        fields = ['published']
```

### distinct

Логическое значение, указывающее, будет ли фильтр использовать **distinct** в наборе запросов. Этот параметр можно использовать для устранения повторяющихся результатов при использовании фильтров, охватывающих взаимосвязи. По умолчанию имеет значение `False`.

### exclude

Логическое значение, указывающее, должен ли фильтр использовать **filter** или **exclude** в наборе запросов. По умолчанию имеет значение `False`.

### required

Логическое значение, указывающее, требуется ли фильтр или нет. По умолчанию имеет значение `False`.

### \*\*kwargs

Любые дополнительные аргументы ключевого слова сохраняются как параметр фильтра **extra**. Они предоставляются сопровождающему полю формы **Field** и могут использоваться для предоставления аргументов, таких как **choices**. Некоторые аргументы, связанные с полем:

### widget

Класс **Widget** `django.form`, который будет представлять **Filter**. В дополнение к виджетам, включенным в Django, которые вы можете использовать, есть дополнительные, которые предоставляет **django-filter**, которые могут быть полезны:

* <mark style="color:purple;">LinkWidget</mark> — отображает параметры аналогично тому, как это делает админка Django, в виде серии ссылок. Ссылка для выбранного варианта будет иметь `class="selected"`.
* <mark style="color:purple;">BooleanWidget</mark> — этот виджет преобразует входные данные в значения `True/False` Python. Он преобразует все варианты регистра `True` и `False` во внутренние значения Python.
* <mark style="color:purple;">CSVWidget</mark> — этот виджет ожидает значение, разделенное запятыми, и преобразует его в список строковых значений. Ожидается, что класс поля обрабатывает список значений, а также преобразование типов.
* <mark style="color:purple;">RangeWidget</mark> — этот виджет используется с **RangeFilter** для создания двух элементов ввода формы с использованием одного поля.

## Аргументы ModelChoiceFilter и ModelMultipleChoiceFilter

Эти аргументы применяются только к **ModelChoiceFilter** и **ModelMultipleChoiceFilter**.

### queryset

Для работы с **ModelChoiceFilter** и **ModelMultipleChoiceFilter** требуется **queryset**, который должен передаваться как **kwarg**.

### to\_field\_name

Если вы передадите **to\_field\_name** (которое перенаправляется в поле Django), оно будет использоваться также в реализации **get\_filter\_predicate** по умолчанию в качестве атрибута модели.

## Фильтры

### CharFilter

Этот фильтр выполняет простые сопоставления символов, используемые по умолчанию с **CharField** и **TextField**.

### UUIDFilter

Этот фильтр соответствует значениям UUID, используемым с `models.UUIDField` по умолчанию.

### BooleanFilter

Этот фильтр соответствует логическому значению `True` или `False`, используемому с **BooleanField** и **NullBooleanField** по умолчанию.

### ChoiceFilter

Этот фильтр сопоставляет значения в своем аргументе **choices**. Выбор **choices** должен быть явно передан при объявлении фильтра в **FilterSet**. Например,

```python
class User(models.Model):
    username = models.CharField(max_length=255)
    first_name = SubCharField(max_length=100)
    last_name = SubSubCharField(max_length=100)

    status = models.IntegerField(choices=STATUS_CHOICES, default=0)

STATUS_CHOICES = (
    (0, 'Regular'),
    (1, 'Manager'),
    (2, 'Admin'),
)

class F(FilterSet):
    status = ChoiceFilter(choices=STATUS_CHOICES)
    class Meta:
        model = User
        fields = ['status']
```

**ChoiceFilter** также имеет аргументы, которые позволяют отказаться от фильтрации, а также выбрать фильтрацию по значениям `None`. Каждый из аргументов имеет соответствующую глобальную настройку (<mark style="color:purple;">Справочник по настройкам</mark>).

* **empty\_label**: Отображаемая метка, используемая для выбора без фильтрации. Выбор можно отключить, установив для этого аргумента значение `None`. По умолчанию **FILTERS\_EMPTY\_CHOICE\_LABEL**.
* **null\_label**: отображаемая метка, используемая для выбора фильтрации по значениям `None`. Выбор можно отключить, установив для этого аргумента значение `None`. По умолчанию **FILTERS\_NULL\_CHOICE\_LABEL**.
* **null\_value**: специальное значение для включения фильтрации по значениям `None`. Это значение по умолчанию **FILTERS\_NULL\_CHOICE\_VALUE** и должно быть непустым значением (`''`, `None`, `[]`, `()`, `{}`).

### TypedChoiceFilter

То же, что и **ChoiceFilter**, но с добавленной возможностью конвертировать значение для соответствия. Это можно сделать с помощью параметра **coerce**. Пример использования ограничивает логические варианты для сопоставления, поэтому только некоторые предопределенные строки могут использоваться в качестве входных данных логического фильтра:

```python
import django_filters
from distutils.util import strtobool

BOOLEAN_CHOICES = (('false', 'False'), ('true', 'True'),)

class YourFilterSet(django_filters.FilterSet):
    ...
    flag = django_filters.TypedChoiceFilter(choices=BOOLEAN_CHOICES,
                                            coerce=strtobool)
```

### MultipleChoiceFilter

То же, что и **ChoiceFilter**, за исключением того, что пользователь может выбрать несколько вариантов, и фильтр по умолчанию формирует **OR** этих вариантов для сопоставления элементов. Фильтр сформирует **AND** выбранных вариантов, когда аргумент `conjoined=True` будет передан этому классу.

Несколько вариантов представлены в строке запроса путем повторного использования одного и того же ключа с разными значениями (например, `"?status=Regular&status=Admin"`).

Значение **distinct** по умолчанию равно `True`, так как это обычно требуется для отношений ко-многим.

Расширенное использование: в зависимости от логики вашего приложения, когда выбраны все варианты или нет, фильтрация может быть нулевой. В этом случае вы можете захотеть избежать накладных расходов на фильтрацию, особенно отдельного вызова.

Установите для **always\_filter** значение `False` после создания экземпляра, чтобы включить тест **is\_noop** по умолчанию.

Переопределите **is\_noop**, если вам требуется другой тест для вашего приложения.

### TypedMultipleChoiceFilter

Аналогичен **MultipleChoiceFilter**, но дополнительно принимает параметр **coerce**, как и в **TypedChoiceFilter**.

### DateFilter

Совпадения по дате. Используется с **DateField** по умолчанию.

### TimeFilter

Совпадения по времени. Используется с **TimeField** по умолчанию.

### DateTimeFilter

Совпадение по дате и времени. Используется с **DateTimeField** по умолчанию.

### IsoDateTimeFilter

Использует **IsoDateTimeField** для поддержки фильтрации дат в формате **ISO 8601**, которые часто используются в API и используются по умолчанию в Django REST Framework.

Пример:

```python
class F(FilterSet):
    """Фильтрация книг по дате публикации с использованием дат в формате ISO 8601."""
    published = IsoDateTimeFilter()

    class Meta:
        model = Book
        fields = ['published']
```

### DurationFilter

Совпадение по продолжительности. Используется с **DurationField** по умолчанию.

Поддерживает длительности как в формате Django (`'%d %H:%M:%S.%f'`), так и в формате ISO 8601 (но только разделы, которые принимаются временной дельтой Python, поэтому нет обозначений года, месяца и недели, например, `'P3DT10H22M'`).

### ModelChoiceFilter

Аналогичен **ChoiceFilter**, за исключением того, что он работает со связанными моделями, используемыми для **ForeignKey** по умолчанию.

При автоматическом создании экземпляра **ModelChoiceFilter** будет использоваться **QuerySet** по умолчанию для связанного поля. Если экземпляр создается вручную, вы должны указать **queryset** kwarg.

Пример:

```python
class F(FilterSet):
    """Фильтр книг по автору"""
    author = ModelChoiceFilter(queryset=Author.objects.all())

    class Meta:
        model = Book
        fields = ['author']
```

Аргумент **queryset** также поддерживает вызываемое поведение. Если **callable** передается, он будет вызываться с `Filterset.request` в качестве единственного аргумента. Это позволяет легко фильтровать по свойствам объекта запроса без переопределения `FilterSet.__init__`.

{% hint style="info" %}
Вы должны ожидать, что объект запроса может быть **None**.
{% endhint %}

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

### ModelMultipleChoiceFilter

Аналогичен **MultipleChoiceFilter**, за исключением того, что он работает со связанными моделями, используемыми для **ManyToManyField** по умолчанию.

Как и в случае с **ModelChoiceFilter**, при автоматическом создании экземпляра **ModelMultipleChoiceFilter** будет использоваться набор запросов **Queryset** по умолчанию для связанного поля. Если экземпляр создается вручную, вы должны указать **queryset** kwarg. Как и **ModelChoiceFilter**, аргумент **queryset** имеет вызываемое поведение.

Чтобы использовать имя пользовательского поля для поиска, вы можете использовать **to\_field\_name**:

```python
class FooFilter(BaseFilterSet):
    foo = django_filters.filters.ModelMultipleChoiceFilter(
        field_name='attr__uuid',
        to_field_name='uuid',
        queryset=Foo.objects.all(),
    )
```

Если вы хотите использовать собственный набор запросов, например, чтобы добавить аннотированные поля, это можно сделать следующим образом:

```python
class MyMultipleChoiceFilter(django_filters.ModelMultipleChoiceFilter):
    def get_filter_predicate(self, v):
        return {'annotated_field': v.annotated_field}

    def filter(self, qs, value):
        if value:
            qs = qs.annotate_with_custom_field()
            qs = super().filter(qs, value)
        return qs

foo = MyMultipleChoiceFilter(
    to_field_name='annotated_field',
    queryset=Model.objects.annotate_with_custom_field(),
)
```

Метод **annotate\_with\_custom\_field** будет определен через пользовательский набор запросов **Queryset**, который затем будет использоваться в качестве менеджера модели:

```python
class CustomQuerySet(models.QuerySet):
    def annotate_with_custom_field(self):
        return self.annotate(
            custom_field=Case(
                When(foo__isnull=False,
                     then=F('foo__uuid')),
                When(bar__isnull=False,
                     then=F('bar__uuid')),
                default=None,
            ),
        )

class MyModel(models.Model):
    objects = CustomQuerySet.as_manager()
```

### NumberFilter

Фильтры на основе числового значения, используемые по умолчанию с **IntegerField**, **FloatField** и **DecimalField**.

#### NumberFilter.get\_max\_validator()

Возвращает экземпляр **MaxValueValidator**, который будет добавлен в `field.validators`. По умолчанию используется предельное значение `1e50`. Возвращает `None`, чтобы отключить проверку максимального значения.

### NumericRangeFilter

Фильтрует, когда значение находится между двумя числовыми значениями или больше минимума или меньше максимума, если указано только одно предельное значение. Этот фильтр предназначен для работы с полями числового диапазона **Postgres**, включая **IntegerRangeField**, **BigIntegerRangeField** и **FloatRangeField** (доступно начиная с Django 1.8). По умолчанию используется виджет **RangeField**.

Обычные поиски по полям доступны в дополнение к нескольким поискам вложений, включая **overlap**, **contains** и **contained\_by**. Подробнее в [документации Django](https://docs.djangoproject.com/en/stable/ref/contrib/postgres/fields/#querying-range-fields).

Если задано нижнее предельное значение, фильтр автоматически ищет по умолчанию **startswith** и **endswith**, если указано только верхнее предельное значение.

### RangeFilter

Фильтрует, когда значение находится между двумя числовыми значениями или больше минимума или меньше максимума, если указано только одно предельное значение.

```python
class F(FilterSet):
    """Filter for Books by Price"""
    price = RangeFilter()

    class Meta:
        model = Book
        fields = ['price']

qs = Book.objects.all().order_by('title')

# Range: Книги от 5€ до 15€
f = F({'price_min': '5', 'price_max': '15'}, queryset=qs)

# Min-Only: Книги дороже 11€
f = F({'price_min': '11'}, queryset=qs)

# Max-Only: Книги стоимостью менее 19 €
f = F({'price_max': '19'}, queryset=qs)
```
