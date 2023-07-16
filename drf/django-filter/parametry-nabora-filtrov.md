# Параметры набора фильтров

Этот документ содержит руководство по использованию дополнительных функций FilterSet.

## Опции Meta

* [model](parametry-nabora-filtrov.md#avtomaticheskoe-sozdanie-filtra-s-model)
* [fields](parametry-nabora-filtrov.md#obyavlenie-filtruemykh-polei-fields)
* [exclude](parametry-nabora-filtrov.md#otklyuchit-polya-filtra-s-exclude)
* [form](parametry-nabora-filtrov.md#polzovatelskie-formy-s-ispolzovaniem-form)
* [filter\_overrides](parametry-nabora-filtrov.md#nastroite-generaciyu-filtrov-s-pomoshyu-filter\_overrides)

### Автоматическое создание фильтра с model

**FilterSet** способен автоматически генерировать фильтры для заданных полей модели **model**. Подобно **ModelForm** в Django, фильтры создаются на основе типа поля базовой модели. Этот параметр должен сочетаться либо с параметрами **fields**, либо с параметром **exclude**, что является тем же требованием для класса Django **ModelForm**, подробно описанного [здесь](https://docs.djangoproject.com/en/stable/topics/forms/modelforms/#selecting-the-fields-to-use).

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        fields = ['username', 'last_login']
```

### Объявление фильтруемых полей fields

Параметр **fields** сочетается с моделью **model** для автоматического создания фильтров. Обратите внимание, что сгенерированные фильтры _не перезаписывают_ фильтры, объявленные в **FilterSet**. Параметр **fields** принимает два синтаксиса:

* список имен полей
* словарь имен полей, сопоставленный со списком поисковых запросов

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        fields = ['username', 'last_login']

# или

class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        fields = {
            'username': ['exact', 'contains'],
            'last_login': ['exact', 'year__gt'],
        }
```

Синтаксис списка создаст фильтр поиска **exact** для каждого поля, включенного в **fields**. Синтаксис словаря создаст фильтр для каждого выражения поиска, объявленного для соответствующего поля модели. Эти выражения могут включать как преобразования, так и поиски, как подробно описано в [справочнике поиска](https://docs.djangoproject.com/en/stable/ref/models/lookups/#module-django.db.models.lookups).

Обратите внимание, что нет необходимости включать объявленные фильтры в список полей **fields** — это повлияет только на порядок, в котором поля появляются в форме **FilterSet**. Включение декларативных псевдонимов в словарь полей **fields** вызовет ошибку.

```python
class UserFilter(django_filters.FilterSet):
    username = filters.CharFilter()
    login_timestamp = filters.IsoDateTimeFilter(field_name='last_login')

    class Meta:
        model = User
        fields = {
            'username': ['exact', 'contains'],
            'login_timestamp': ['exact'],
        }

TypeError(
    "'Meta.fields' contains fields that are not defined on this FilterSet: login_timestamp"
)
```

### Отключить поля фильтра с exclude

Опция **exclude** принимает черный список имен полей, которые нужно исключить из автоматической генерации фильтра. Обратите внимание, что этот параметр не отключает фильтры, объявленные непосредственно в **FilterSet.**

```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        exclude = ['password']
```

### Пользовательские формы с использованием form

Внутренний класс **Meta** также принимает необязательный аргумент **form**. Это класс формы, от которого будет подклассом `FilterSet.form`. Это работает аналогично параметру **form** в **ModelAdmin**.

### Настройте генерацию фильтров с помощью filter\_overrides

Внутренний класс **Meta** также принимает необязательный аргумент **filter\_overrides**. Это карта полей модели для фильтрации классов с параметрами:

```python
class ProductFilter(django_filters.FilterSet):

     class Meta:
         model = Product
         fields = ['name', 'release_date']
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

## Переопределение методов FilterSet

При переопределении методов класса вызов `super(MyFilterSet, cls)` может привести к исключению **NameError**. Это связано с тем, что класс **FilterSetMetaclass** вызывает эти методы класса до того, как класс **FilterSet** будет полностью создан. Есть два рекомендуемых обходных пути:

1. Если вы используете Python 3.6 или новее, используйте синтаксис `super()` без аргументов.
2. Для более старых версий Python используйте промежуточный класс. Например:

```python
class Intermediate(django_filters.FilterSet):

    @classmethod
    def method(cls, arg):
        super(Intermediate, cls).method(arg)
        ...

class ProductFilter(Intermediate):
    class Meta:
        model = Product
        fields = ['...']
```

### filter\_for\_lookup()

До версии `0.13.0` при генерации фильтра не учитывалось используемое **lookup\_expr**. Это обычно приводило к созданию искаженных фильтров для поиска `"isnull"`, `"in"` и `"range"` (а также преобразованных поисков). Текущая реализация обеспечивает следующее поведение:

* поиск `"isnull"` возвращает **BooleanFilter**
* Поиск `"in"` возвращает фильтр, полученный из **BaseInFilter** на основе CSV.
* Поиск по `"range"` возвращает фильтр, полученный из **BaseRangeFilter** на основе CSV.

Если вы хотите переопределить **filter\_class** и **params**, используемые для создания экземпляров фильтров для поля модели, вы можете переопределить `filter_for_lookup()`. Например:

```python
class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            'release_date': ['exact', 'range'],
        }

    @classmethod
    def filter_for_lookup(cls, f, lookup_type):
        # переопределить поиск диапазона дат
        if isinstance(f, models.DateField) and lookup_type == 'range':
            return django_filters.DateRangeFilter, {}

        # в противном случае используйте поведение по умолчанию
        return super().filter_for_lookup(f, lookup_type)
```
