# Руководство по миграциям

## Включение предупреждений

Чтобы просмотреть устаревшие версии, вам может потребоваться включить предупреждения в Python. Этого можно добиться либо с помощью [флага](https://docs.python.org/3.6/using/cmdline.html#cmdoption-W) `-W`, либо с помощью [переменной окружения](https://docs.python.org/3.6/using/cmdline.html#envvar-PYTHONWARNINGS) **PYTHONWARNINGS**. Например, вы можете запустить свой набор тестов следующим образом:

```bash
$ python -W once manage.py test
```

Приведенное выше выводит все предупреждения один раз, когда они появляются впервые. Это полезно, чтобы знать, какие нарушения существуют в вашем коде (или иногда в стороннем коде). Однако он печатает только последнюю строку трассировки стека. Вместо этого вы можете использовать следующее, чтобы вызвать полное исключение:

```bash
$ python -W error manage.py test
```

## Переход на 2.0

Этот выпуск содержит несколько изменений, которые нарушают совместимость вперед. Сюда входят удаленные функции, переименованные атрибуты и аргументы, а также некоторые переработанные функции. Из-за характера этих изменений невозможно выпустить полностью совместимый с предыдущими версиями миграционный выпуск. Просмотрите приведенный ниже список изменений и соответствующим образом обновите свой код.

### Форма списка Filter.lookup\_expr удалена ([#851](https://github.com/carltongibson/django-filter/pull/851)).

Аргумент `Filter.lookup_expr` больше не принимает `None` или список выражений. Вместо этого используйте <mark style="color:purple;">LookupChoiceFilter</mark>.

### FilterSet  filter\_for\_reverse\_field удален ([#915](https://github.com/carltongibson/django-filter/pull/915))

Метод **filter\_for\_field** теперь создает фильтры для обратных отношений, устраняя необходимость в **filter\_for\_reverse\_field**. В результате обратные отношения теперь также подчиняются `Meta.filter_overrides`.

### Атрибуты представления View переименованы ([#867](https://github.com/carltongibson/django-filter/pull/867))

Несколько атрибутов, связанных с представлением, были переименованы для улучшения согласованности с другими частями библиотеки. Затронуты следующие классы:

* DRF `ViewSet.filter_class` => `filterset_class`
* DRF `ViewSet.filter_fields` => `filterset_fields`
* `DjangoFilterBackend.default_filter_set` => `filterset_base`
* `DjangoFilterBackend.get_filter_class()` => `get_filterset_class()`
* `FilterMixin.filter_fields` => `filterset_fields`

### Параметр FilterSet Meta.together удален ([#791](https://github.com/carltongibson/django-filter/pull/791)).

`Meta.together` устарел в пользу пользовательских реализаций, которые переопределяют чистый метод класса `Meta.form`. Пример будет представлен в разделе «рецепты» в будущих документах.

### Обработка «stricness» FilterSet перемещена в представление ([#788](https://github.com/carltongibson/django-filter/pull/788))

Обработка строгости была удалена из **FilterSet** и добавлена к слою представления. В результате параметр **FILTERS\_STRICTNESS**, параметр `Meta.strict` и аргумент **strict** для инициализатора **FilterSet** были удалены.

Чтобы изменить поведение строгости, следует переопределить соответствующий код представления. Более подробная информация будет представлена в будущих документах.

### Filter.name переименован в Filter.field\_name ([#792](https://github.com/carltongibson/django-filter/pull/792))

Имя фильтра было переименовано в **field\_name**, чтобы устранить неоднозначность имени атрибута фильтра в его классе **FilterSet** и **field\_name**, используемого для целей фильтрации.

### Filter.widget и Filter.required удалены ([#734](https://github.com/carltongibson/django-filter/pull/734))

Класс фильтра больше не хранит напрямую аргументы, переданные в его поле формы. Все аргументы находятся в словаре `.extra` фильтра.

### MultiWidget заменен на SuffixedMultiWidget ([#770](https://github.com/carltongibson/django-filter/pull/770))

**RangeWidget**, **DateRangeWidget** и **LookupTypeWidget** теперь наследуются от **SuffixedMultiWidget**, изменяя суффиксы имен своих параметров запроса. Например, **RangeWidget** теперь имеет суффиксы **\_min** и **\_max** вместо **\_0** и **\_1**.

### Такие фильтры, как RangeFilter, DateRangeFilter, DateTimeFromToRangeFilter... ([#770](https://github.com/carltongibson/django-filter/pull/770))

#### Поскольку они зависят от MultiWidget, их необходимо настроить. В версии 1.0 параметры

были предоставлены с использованием `_0` и `_1` в качестве суффикса` `` `. Например, параметр **create\_date**, использующий **DateRangeFilter**, будет ожидать **creation\_date\_after** и **creation\_date\_before**, а не **creation\_date\_0** и **creation\_date\_1**.

## Переход на 1.0

В выпуске `1.0` **django-filter** представлено несколько изменений и усовершенствований API, которые нарушают совместимость вперед. Ниже приведен список устаревших версий и инструкции по переходу на версию 1.0. Также был создан совместимый с предыдущими версиями выпуск `0.15`, чтобы помочь с миграцией. Он совместим как с существующими, так и с новыми API и будет выдавать предупреждения об устаревшем поведении.

### MethodFilter и Filter.action заменены на Filter.method ([#382](https://github.com/carltongibson/django-filter/pull/382))

Функции **MethodFilter** и `Filter.action` были объединены и заменены параметром `Filter.method`. Параметр **method** принимает либо вызываемый объект, либо имя метода **FilterSet**. Теперь сигнатура принимает дополнительный аргумент **name**, который является именем поля модели, по которому будет выполняться фильтрация.

Поскольку **method** теперь является параметром всех фильтров, входные данные проверяются и очищаются его **field\_class**. Функция получит очищенное значение вместо необработанного значения.

```python
# 0.x
class UserFilter(FilterSet):
    last_login = filters.MethodFilter()

    def filter_last_login(self, qs, value):
        # пробует преобразовать значение в дату и время, что может привести к сбою.
        if value and looks_like_a_date(value):
            value = datetime(value)

        return qs.filter(last_login=value})


# 1.0
class UserFilter(FilterSet):
    last_login = filters.CharFilter(method='filter_last_login')

    def filter_last_login(self, qs, name, value):
        return qs.filter(**{name: value})
```

### Методы QuerySet больше не проксируются ([#440](https://github.com/carltongibson/django-filter/pull/440))

Методы `__iter__()`, `__len__()`, `__getitem__()`, `count()` больше не проксируются из набора запросов. Чтобы исправить это, вызовите методы самого свойства `.qs`.

```python
f = UserFilter(request.GET, queryset=User.objects.all())

# 0.x
for obj in f:
    ...

# 1.0
for obj in f.qs:
    ...
```

### Фильтры больше не генерируются автоматически, если поле Meta.fields не указано ([#450](https://github.com/carltongibson/django-filter/pull/450))

У FilterSets было недокументированное поведение автоматического создания фильтров для всех полей модели, когда либо `Meta.fields` не был указан, либо когда установлено значение `None`. Это может привести к потенциально небезопасному раскрытию данных или схемы, и было объявлено устаревшим в пользу явной установки для `Meta.fields` специального значения `"__all__"`. Вы также можете внести поля в черный список, установив атрибут `Meta.exclude`.

```python
class UserFilter(FilterSet):
    class Meta:
        model = User
        fields = '__all__'

# или
class UserFilter(FilterSet):
    class Meta:
        model = User
        exclude = ['password']
```

### Переместите параметры FilterSet в метакласс ([#430](https://github.com/carltongibson/django-filter/issues/430))

Несколько параметров **FilterSet** были перемещены в класс **Meta**, чтобы предотвратить потенциальные конфликты с объявленными именами фильтров. Это включает в себя:

* `filter_overrides`
* `strict`
* `order_by_field`

```python
# 0.x
class UserFilter(FilterSet):
    filter_overrides = {}
    strict = STRICTNESS.RAISE_VALIDATION_ERROR
    order_by_field = 'order'
    ...

# 1.0
class UserFilter(FilterSet):
    ...

    class Meta:
        filter_overrides = {}
        strict = STRICTNESS.RAISE_VALIDATION_ERROR
        order_by_field = 'order'
```

### FilterSet ordering заменен на OrderingFilter ([#472](https://github.com/carltongibson/django-filter/pull/472))

Параметры и методы упорядочения **FilterSet** устарели и заменены <mark style="color:purple;">OrderingFilter</mark>. Устаревшие параметры включают:

* `Meta.order_by`
* `Meta.order_by_field`

Эти параметры сохраняют обратную совместимость со следующими оговорками:

* **order\_by** утверждает, что `Meta.fields` не использует синтаксис словаря. Ранее это было неопределенным поведением, однако код миграции не может его поддерживать.
* Ранее, если в запросе не был указан порядок, **FilterSet** неявно фильтровал по первому параметру в опции **order\_by**. Это поведение не может быть легко сымитировано, но его можно исправить, убедившись, что переданный набор запросов явно вызывает `.order_by()`.

```python
filterset = MyFilterSet(queryset=MyModel.objects.order_by('field'))
```

Следующие методы устарели и вызовут предупреждение, если они присутствуют в **FilterSet**:

* `.get_order_by()`
* `.get_ordering_field()`

Чтобы исправить это, просто удалите методы из вашего класса. Вы можете создать подкласс **OrderingFilter** для переноса любой пользовательской логики.

### Устарели FILTERS\_HELP\_TEXT\_FILTER и FILTERS\_HELP\_TEXT\_EXCLUDE ([#437](https://github.com/carltongibson/django-filter/pull/437))

Сгенерированные метки фильтров в версии 1.0 будут более описательными, включая гуманизированный текст о выполняемом поиске и о том, является ли фильтр фильтром исключения.

Эти настройки больше не будут действовать и будут удалены в версии 1.0.

### Backend фильтра DRF вызывает исключение TemplateDoesNotExist ([#562](https://github.com/carltongibson/django-filter/issues/562))

Шаблоны теперь предоставляются **django-filter**. Если вы получаете эту ошибку, вам может потребоваться добавить `'django_filters'` в настройку **INSTALLED\_APPS**. Кроме того, вы можете предоставить свои собственные шаблоны.
