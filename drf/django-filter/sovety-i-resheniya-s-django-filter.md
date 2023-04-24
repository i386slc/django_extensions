# Советы и решения с django-filter

## Общие проблемы для объявленных фильтров

Ниже приведены некоторые распространенные проблемы, возникающие при объявлении фильтров. Рекомендуется прочитать это, так как оно дает более полное представление о том, как работают фильтры.

### Фильтр field\_name и lookup\_expr не настроены

Хотя **field\_name** и **lookup\_expr** являются необязательными, рекомендуется указать их. По умолчанию, если **field\_name** не указано, будет использоваться имя фильтра в классе **FilterSet**. Кроме того, **lookup\_expr** по умолчанию имеет значение **exact**. Ниже приведен пример неправильно настроенного ценового фильтра:

```python
class ProductFilter(django_filters.FilterSet):
    price__gt = django_filters.NumberFilter()
```

Экземпляр фильтра будет иметь имя поля **price\_\_gt** и тип поиска **exact**. Под капотом это будет неправильно решено как:

```python
Product.objects.filter(price__gt__exact=value)
```

Вышеприведенное, скорее всего, вызовет ошибку **FieldError**. Правильная конфигурация будет:

```python
class ProductFilter(django_filters.FilterSet):
    price__gt = django_filters.NumberFilter(field_name='price', lookup_expr='gt')
```

### Отсутствует lookup\_expr для фильтров текстового поиска

Довольно часто забывают установить выражение поиска для **CharField** и **TextField** и задаются вопросом, почему поиск `«foo»` не возвращает результатов для `«foobar»`. Это связано с тем, что тип поиска по умолчанию является точным **exact**, но вы, вероятно, захотите выполнить поиск по **icontains**.

### Несоответствие выражения фильтра и поиска (**in**, **range**, **isnull**)

Не всегда уместно напрямую сопоставлять фильтр с типом поля модели, так как некоторые операции поиска предполагают разные типы значений. Это часто встречающаяся проблема с поиском **in**, **range** и **isnull**. Рассмотрим следующую модель продукта:

```python
class Product(models.Model):
    category = models.ForeignKey(Category, null=True)
```

Учитывая, что категория не является обязательной, разумно включить поиск товаров без категорий. Ниже показан неправильно настроенный фильтр **isnull**:

```python
class ProductFilter(django_filters.FilterSet):
    uncategorized = django_filters.NumberFilter(
        field_name='category', lookup_expr='isnull'
    )
```

Так в чем проблема? Хотя базовый тип столбца для **category** является целым числом, при поиске **isnull** ожидается логическое значение. Однако **NumberFilter** проверяет только числа. Фильтры не «_осведомлены о выражениях_» и не будут изменять поведение на основе их **lookup\_expr**. Вы должны использовать фильтры, соответствующие _**типу данных выражения поиска**_, а не типу данных, лежащему в основе поля модели. Следующее правильно позволит вам искать как продукты без категорий, так и продукты для набора категорий:

```python
class NumberInFilter(django_filters.BaseInFilter, django_filters.NumberFilter):
    pass

class ProductFilter(django_filters.FilterSet):
    categories = NumberInFilter(field_name='category', lookup_expr='in')
    uncategorized = django_filters.BooleanFilter(
        field_name='category', lookup_expr='isnull'
    )
```

Дополнительные сведения о [создании фильтров](https://django-filter.readthedocs.io/en/main/ref/filters.html#base-in-filter) **in** и **range** csv.

## Фильтрация по пустым значениям

В ряде случаев может потребоваться фильтрация по пустым или нулевым значениям. Ниже приведены некоторые распространенные решения этих проблем:

### Фильтрация по нулевым значениям null

Как объяснялось выше в разделе «Несоответствие выражений фильтрации и поиска», распространенная проблема заключается в том, как правильно фильтровать значения **NULL** в поле.

### Решение 1. Использование BooleanFilter с isnull

Использование **BooleanFilter** с поиском **isnull** — это встроенное решение, используемое при автоматической генерации фильтров в **FilterSet**. Чтобы сделать это вручную, просто добавьте:

```python
class ProductFilter(django_filters.FilterSet):
    uncategorized = django_filters.BooleanFilter(
        field_name='category', lookup_expr='isnull'
    )
```

{% hint style="info" %}
Помните, что класс фильтра проверяет входное значение. Базовый тип поля режима здесь не имеет значения.
{% endhint %}

Вы также можете изменить логику с помощью параметра **exclude**.

```python
class ProductFilter(django_filters.FilterSet):
    has_category = django_filters.BooleanFilter(
        field_name='category', lookup_expr='isnull', exclude=True
    )
```

### Решение 2. Использование нулевого выбора ChoiceFilter

Если вы используете ChoiceFilter, вы также можете фильтровать по нулевым значениям, включив параметр null\_label. Дополнительные сведения см. в [справочной документации](https://django-filter.readthedocs.io/en/main/ref/filters.html#choice-filter) **ChoiceFilter**.

```python
class ProductFilter(django_filters.FilterSet):
    category = django_filters.ModelChoiceFilter(
        field_name='category', lookup_expr='isnull',
        null_label='Uncategorized',
        queryset=Category.objects.all(),
    )
```

### Решение 3. Объединение полей с MultiValueField

Альтернативный подход — использовать **MultiValueField** Django для ручного добавления **BooleanField** для обработки нулевых значений. Доказательство концепции: [https://github.com/carltongibson/django-filter/issues/446](https://github.com/carltongibson/django-filter/issues/446).

### Фильтрация по пустой строке

В настоящее время фильтрация по пустой строке невозможна, так как пустые значения интерпретируются как пропущенный фильтр.

```http
GET http://localhost/api/my-model?myfield=
```

### Решение 1. Магические значения

Вы можете переопределить метод `filter()` класса фильтра, чтобы специально проверять магические значения. Это похоже на обработку нулевого значения **ChoiceFilter**.

```http
GET http://localhost/api/my-model?myfield=EMPTY
```

```python
class MyCharFilter(filters.CharFilter):
    empty_value = 'EMPTY'

    def filter(self, qs, value):
        if value != self.empty_value:
            return super().filter(qs, value)

        qs = self.get_method(qs)(**{'%s__%s' % (self.field_name, self.lookup_expr): ""})
        return qs.distinct() if self.distinct else qs
```

### Решение 2. Фильтр пустой строки

Также можно было бы создать фильтр пустого значения, который демонстрирует то же поведение, что и фильтр **isnull**.

```http
GET http://localhost/api/my-model?myfield__isempty=false
```

```python
from django.core.validators import EMPTY_VALUES

class EmptyStringFilter(filters.BooleanFilter):
    def filter(self, qs, value):
        if value in EMPTY_VALUES:
            return qs

        exclude = self.exclude ^ (value is False)
        method = qs.exclude if exclude else qs.filter

        return method(**{self.field_name: ""})

class MyFilterSet(filters.FilterSet):
    myfield__isempty = EmptyStringFilter(field_name='myfield')

    class Meta:
        model = MyModel
        fields = []
```

## Фильтрация по относительному времени

Учитывая модель с полем метки времени, может быть полезно фильтровать на основе относительного времени. Например, возможно, мы хотим получить данные за последние **n** часов. Этого можно добиться с помощью **NumberFilter**, который вызывает пользовательский метод.

```python
from django.utils import timezone
from datetime import timedelta
...

class DataModel(models.Model):
    time_stamp = models.DateTimeField()

class DataFilter(django_filters.FilterSet):
    hours = django_filters.NumberFilter(
        field_name='time_stamp', method='get_past_n_hours', label="Past n hours")

    def get_past_n_hours(self, queryset, field_name, value):
        time_threshold = timezone.now() - timedelta(hours=int(value))
        return queryset.filter(time_stamp__gte=time_threshold)

    class Meta:
        model = DataModel
        fields = ('hours',)
```

## Использование начальных значений initial по умолчанию

В версиях **django-filter** до `1.0` начальное значение поля фильтра использовалось по умолчанию, если значение не было отправлено. Это поведение официально не поддерживалось и с тех пор было удалено.

{% hint style="warning" %}
Рекомендуется НЕ реализовывать приведенное ниже, так как это отрицательно влияет на удобство использования. Формы Django не обеспечивают такого поведения по какой-то причине.

* Использование начальных значений по умолчанию несовместимо с поведением форм Django.
* Значения по умолчанию не позволяют пользователям фильтровать по пустым значениям.
* Значения по умолчанию не позволяют пользователям пропускать этот фильтр.
{% endhint %}

Однако, если необходимы значения по умолчанию, следующее должно имитировать поведение до версии 1.0:

```python
class BaseFilterSet(FilterSet):

    def __init__(self, data=None, *args, **kwargs):
        # если filterset привязан, используйте начальные значения по умолчанию
        if data is not None:
            # получить изменяемую копию QueryDict
            data = data.copy()

            for name, f in self.base_filters.items():
                initial = f.extra.get('initial')

                # параметр filter либо отсутствует, либо пуст,
                # используйте initial по умолчанию
                if not data.get(name) and initial:
                    data[name] = initial

        super().__init__(data, *args, **kwargs)
```

## Добавление поля модели help\_text в фильтры

Поле модели **help\_text** не используется фильтрами по умолчанию. Его можно добавить с помощью простого базового класса **FilterSet**:

```python
class HelpfulFilterSet(django_filters.FilterSet):
    @classmethod
    def filter_for_field(cls, f, name, lookup_expr):
        filter = super(HelpfulFilterSet, cls).filter_for_field(f, name, lookup_expr)
        filter.extra['help_text'] = f.help_text
        return filter
```
