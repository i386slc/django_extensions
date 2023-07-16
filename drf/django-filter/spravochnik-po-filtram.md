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

### DateRangeFilter

Фильтр аналогичен дате списка изменений админки, он имеет ряд общих настроек для работы с полями даты.

### DateFromToRangeFilter

Аналогичен **RangeFilter**, за исключением того, что он использует даты вместо числовых значений. Его можно использовать с **DateField**. Он также работает с **DateTimeField**, но учитывает только дату.

Пример использования поля **DateField**:

```python
class Comment(models.Model):
    date = models.DateField()
    time = models.TimeField()

class F(FilterSet):
    date = DateFromToRangeFilter()

    class Meta:
        model = Comment
        fields = ['date']

# Range: Комментарии, добавленные между 2016-01-01 и 2016-02-01
f = F({'date_after': '2016-01-01', 'date_before': '2016-02-01'})

# Min-Only: Комментарии, добавленные после 2016-01-01
f = F({'date_after': '2016-01-01'})

# Max-Only: Комментарии, добавленные до 2016-02-01
f = F({'date_before': '2016-02-01'})
```

{% hint style="info" %}
При фильтрации диапазонов, происходящих в даты перехода на летнее время DST, **DateFromToRangeFilter** будет использовать первый допустимый час дня для даты и времени начала и последний допустимый час дня для даты и времени окончания. Это нормально для большинства приложений, но если вы хотите настроить это поведение, вы должны расширить **DateFromToRangeFilter** и создать для него настраиваемое поле.
{% endhint %}

{% hint style="warning" %}
Если вы используете Django до версии 1.9, вы можете столкнуться с **AmbiguousTimeError** или **NonExistentTimeError**, когда дата начала/конца соответствует началу/концу летнего времени соответственно. Это происходит из-за того, что версии до 1.9 не позволяют изменить поведение летнего времени для информирования о дате и времени.
{% endhint %}

Пример использования поля **DateTimeField**:

```python
class Article(models.Model):
    published = models.DateTimeField()

class F(FilterSet):
    published = DateFromToRangeFilter()

    class Meta:
        model = Article
        fields = ['published']

Article.objects.create(published='2016-01-01 8:00')
Article.objects.create(published='2016-01-20 10:00')
Article.objects.create(published='2016-02-10 12:00')

# Range: Статьи, опубликованные между 2016-01-01 и 2016-02-01
f = F({'published_after': '2016-01-01', 'published_before': '2016-02-01'})
assert len(f.qs) == 2

# Min-Only: Статьи, опубликованные после 2016-01-01
f = F({'published_after': '2016-01-01'})
assert len(f.qs) == 3

# Max-Only: Статьи, опубликованные до 2016-02-01
f = F({'published_before': '2016-02-01'})
assert len(f.qs) == 2
```

### DateTimeFromToRangeFilter

Аналогичен **RangeFilter**, за исключением того, что он использует значения формата даты и времени вместо числовых значений. Его можно использовать с **DateTimeField**.

Пример:

```python
class Article(models.Model):
    published = models.DateTimeField()

class F(FilterSet):
    published = DateTimeFromToRangeFilter()

    class Meta:
        model = Article
        fields = ['published']

Article.objects.create(published='2016-01-01 8:00')
Article.objects.create(published='2016-01-01 9:30')
Article.objects.create(published='2016-01-02 8:00')

# Range: Статьи, опубликованные 2016-01-01 между 8:00 и 10:00
f = F({'published_after': '2016-01-01 8:00', 'published_before': '2016-01-01 10:00'})
assert len(f.qs) == 2

# Min-Only: Статьи, опубликованные после 2016-01-01 8:00
f = F({'published_after': '2016-01-01 8:00'})
assert len(f.qs) == 3

# Max-Only: Статьи, опубликованные до 2016-01-01 10:00
f = F({'published_before': '2016-01-01 10:00'})
assert len(f.qs) == 2
```

### IsoDateTimeFromToRangeFilter

Аналогичен **RangeFilter**, за исключением того, что он использует форматированные значения ISO 8601 вместо числовых значений. Его можно использовать с **IsoDateTimeField**.

Пример:

```python
class Article(models.Model):
    published = django_filters.IsoDateTimeField()

class F(FilterSet):
    published = IsoDateTimeFromToRangeFilter()

    class Meta:
        model = Article
        fields = ['published']

Article.objects.create(published='2016-01-01T8:00:00+01:00')
Article.objects.create(published='2016-01-01T9:30:00+01:00')
Article.objects.create(published='2016-01-02T8:00:00+01:00')

# Range: Статьи, опубликованные 2016-01-01 между 8:00 и 10:00
f = F({'published_after': '2016-01-01T8:00:00+01:00', 'published_before': '2016-01-01T10:00:00+01:00'})
assert len(f.qs) == 2

# Min-Only: Статьи, опубликованные после 2016-01-01 8:00
f = F({'published_after': '2016-01-01T8:00:00+01:00'})
assert len(f.qs) == 3

# Max-Only: Статьи, опубликованные до 2016-01-01 10:00
f = F({'published_before': '2016-01-01T10:00:00+0100'})
assert len(f.qs) == 2
```

### TimeRangeFilter

Аналогичен **RangeFilter**, за исключением того, что он использует значения формата времени вместо числовых значений. Его можно использовать с **TimeField**.

Пример:

```python
class Comment(models.Model):
    date = models.DateField()
    time = models.TimeField()

class F(FilterSet):
    time = TimeRangeFilter()

    class Meta:
        model = Comment
        fields = ['time']

# Range: Комментарии, добавленые между 8:00 и 10:00
f = F({'time_after': '8:00', 'time_before': '10:00'})

# Min-Only: Комментарии, добавленые после 8:00
f = F({'time_after': '8:00'})

# Max-Only: Комментарии, добавленые до 10:00
f = F({'time_before': '10:00'})
```

### AllValuesFilter

Это **ChoiceFilter**, варианты выбора которого являются текущими значениями в базе данных. Итак, если в БД для данного поля у вас есть значения 5, 7 и 9, каждое из них присутствует в качестве опции. Это похоже на поведение админки по умолчанию.

### AllValuesMultipleFilter

Это **MultipleChoiceFilter**, варианты выбора которого являются текущими значениями в базе данных. Итак, если в БД для данного поля у вас есть значения 5, 7 и 9, каждое из них присутствует в качестве опции. Это похоже на поведение админки по умолчанию.

### LookupChoiceFilter

Комбинированный фильтр, который позволяет пользователям выбирать выражение поиска из раскрывающегося списка.

* **lookup\_choices** — это необязательный аргумент, который принимает несколько входных форматов и в конечном итоге нормализуется как варианты, используемые в раскрывающемся списке поиска. См. `.get_lookup_choices()` для получения дополнительной информации.
* **field\_class** — это необязательный аргумент, который позволяет вам установить класс поля внутренней формы, используемый для проверки значения. По умолчанию: `forms.CharField`

Например:

```python
price = django_filters.LookupChoiceFilter(
    field_class=forms.DecimalField,
    lookup_choices=[
        ('exact', 'Equals'),
        ('gt', 'Greater than'),
        ('lt', 'Less than'),
    ]
)
```

### BaseInFilter

Это базовый класс, используемый для создания фильтров поиска **IN**. Ожидается, что этот класс фильтра используется в сочетании с другим классом фильтра, поскольку этот класс проверяет только то, что входящее значение разделено запятыми. Затем вторичный фильтр используется для проверки отдельных значений.

Пример:

```python
class NumberInFilter(BaseInFilter, NumberFilter):
    pass

class F(FilterSet):
    id__in = NumberInFilter(field_name='id', lookup_expr='in')

    class Meta:
        model = User

User.objects.create(username='alex')
User.objects.create(username='jacob')
User.objects.create(username='aaron')
User.objects.create(username='carl')

# In: Пользователь с ID 1 и 3.
f = F({'id__in': '1,3'})
assert len(f.qs) == 2
```

### BaseRangeFilter

Это базовый класс, используемый для создания фильтров поиска **RANGE**. Он ведет себя так же, как **BaseInFilter**, за исключением того, что он ожидает только два значения, разделенных запятыми.

Пример:

```python
class NumberRangeFilter(BaseRangeFilter, NumberFilter):
    pass

class F(FilterSet):
    id__range = NumberRangeFilter(field_name='id', lookup_expr='range')

    class Meta:
        model = User

User.objects.create(username='alex')
User.objects.create(username='jacob')
User.objects.create(username='aaron')
User.objects.create(username='carl')

# Range: Пользователь с ID между 1 и 3.
f = F({'id__range': '1,3'})
assert len(f.qs) == 3
```

### OrderingFilter

Включить упорядочивание наборов запросов. Являясь расширением **ChoiceFilter**, он принимает два дополнительных аргумента, которые используются для построения вариантов упорядочивания.

* **fields** — это сопоставление {имя поля модели: имя параметра}. Имена параметров отображаются в вариантах выбора и маскируют/псевдонимы имен полей, используемых в вызове `order_by()`. Подобно выбору полей **choices**, **fields** принимают синтаксис «список из двух кортежей», который сохраняет порядок. **fields** также могут быть просто итерируемыми строками. В этом случае имена полей просто удваиваются как имена открытых параметров.
* **field\_labels** — это необязательный аргумент, который позволяет настроить отображаемую метку для соответствующего параметра. Он принимает сопоставление {имя поля: удобочитаемая метка}. Имейте в виду, что ключ — это имя поля, а не имя открытого параметра.

```python
class UserFilter(FilterSet):
    account = CharFilter(field_name='username')
    status = NumberFilter(field_name='status')

    o = OrderingFilter(
        # кортеж-сопоставление сохраняет порядок
        fields=(
            ('username', 'account'),
            ('first_name', 'first_name'),
            ('last_name', 'last_name'),
        ),

        # метки не должны сохранять порядок
        field_labels={
            'username': 'User account',
        }
    )

    class Meta:
        model = User
        fields = ['first_name', 'last_name']

>>> UserFilter().filters['o'].field.choices
[
    ('account', 'User account'),
    ('-account', 'User account (descending)'),
    ('first_name', 'First name'),
    ('-first_name', 'First name (descending)'),
    ('last_name', 'Last name'),
    ('-last_name', 'Last name (descending)'),
]
```

Кроме того, вы можете просто предоставить свой собственный выбор **choices**, если вам требуется явный контроль над открытыми параметрами. Например, когда вы можете отключить параметры сортировки по убыванию.

```python
class UserFilter(FilterSet):
    account = CharFilter(field_name='username')
    status = NumberFilter(field_name='status')

    o = OrderingFilter(
        choices=(
            ('account', 'Account'),
        ),
        fields={
            'username': 'account',
        },
    )
```

Этот фильтр также основан на CSV и принимает несколько параметров упорядочения. Виджет выбора по умолчанию не позволяет использовать это, но полезно для API. Виджеты **SelectMultiple** несовместимы, поскольку они не могут сохранять порядок выбора.

### Добавление пользовательских вариантов фильтра

Если вы хотите сортировать по полям, не относящимся к модели, вам нужно добавить пользовательскую обработку в подкласс **OrderingFilter**. Например, если вы хотите отсортировать по вычисляемому фактору «релевантности», вам нужно будет сделать что-то вроде следующего:

```python
class CustomOrderingFilter(django_filters.OrderingFilter):

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.extra['choices'] += [
            ('relevance', 'Relevance'),
            ('-relevance', 'Relevance (descending)'),
        ]

    def filter(self, qs, value):
        # OrderingFilter основан на CSV, поэтому `value` - это список
        if any(v in ['relevance', '-relevance'] for v in value):
            # сортирует queryset по релевантности
            return ...

        return super().filter(qs, value)
```
