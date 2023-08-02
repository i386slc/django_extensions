# drf-aggregation

DRF Mixin для получения агрегатов

Ключевая особенность:

* может вычислять процентиль (работает только на PostgreSQL) и процент
* группировка по нескольким полям
* временной ряд (не работает на SQLite)
* ограничение количества отображаемых записей

Внимание! API еще не стабилизирован

## Установка

Для установки используйте&#x20;

```bash
$ pip install drf-aggregation
```

## Применение

Создайте **ViewSet**, используя **ViewSet** из пакета или добавив миксин в существующий.

```python
from drf_aggregation.mixins import AggregationMixin
from drf_aggregation.viewsets import AggregationViewSet


class MyCustomUserViewSet(AggregationMixin, GenericViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer

class UserViewSet(AggregationViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```

Зарегистрируйте URL-адрес

```python
from drf_aggregation.routers import AggregationRouter


aggregation_router = AggregationRouter()
aggregation_router.register("user", UserViewSet)

urlpatterns = [
    path("my/custom/endpoint", UserViewSet.as_view({"get": "aggregation"})),
]

urlpatterns += aggregation_router.urls
```

Получение агрегатов

```
# Получить количество пользователей
/user/aggregation?aggregation=count

# Дата последней поездки
/trip/aggregation?aggregation=maximum&aggregationField=duration

# Средняя зарплата
/position/aggregation?aggregation=percentile&aggregationField=salary&percentile=0.5

# Топ-5 распорядителей билетов
/ticket/aggregation?aggregation=count&groupBy=assigned_to&limit=5&order=desc

# Percentage of open tickets by service
/ticket/aggregation?aggregation=percent&groupBy=service&additionalFilter={"type":"operator","data":{"attribute":"state","operator":"=","value":"open"}}

# Продолжительность жизни в зависимости от года рождения
/person/aggregation?aggregation=average&aggregationField=age&annotations={"birth_year":{"field":"birth_date","kind":"year"}}&groupBy=birth_year
```

## Поддерживаемые типы полей

* IntegerField
* FloatField
* DateField (только минимум или максимум)
* DateTimeField (только минимум или максимум)
* DurationField

## Агрегация

При отправке единственного параметра "aggregation" возвращается словарь с полем "value"

```python
?aggregation=average

{"value":42.5}
```

Доступные типы агрегации:

* count
* sum
* average
* minimum
* maximum
* percentile
* percent (возвращает два дополнительных значения: "numerator" и "denominator")

### Дополнительные параметры для разных типов агрегаций

* aggregationField - обязательно для агрегирования: сумма, среднее, минимум, максимум, процентиль
* percentile - от 0 до 1, обязательно для процентиля
* additionalFilter - парсер фильтра используется из пакета [drf-complex-filter](https://github.com/kit-oz/drf-complex-filter), обязательный для процентов

## Группировка результатов

Для группировки результата передается список обязательных полей через запятую

```python
?aggregation=count&groupBy=field1,field2

[
    {"field1":"value1","field2":"value3","value":2},
    {"field1":"value2","field2":"value3","value":1},
    {"field1":"value2","field2":"value4","value":3}
]
```

## Сортировка результата

При группировке по одному полю достаточно передать список полей, по которым нужно отсортировать результат

```python
?aggregation=count&groupBy=field1&orderBy=field1

[
    {"field1":"value1","value":2},
    {"field1":"value2","value":1},
    {"field1":"value3","value":3}
]
```

Для сортировки по результату агрегирования используйте «value».

```python
?aggregation=count&groupBy=field1&orderBy=-value

[
    {"field1":"value3","value":3},
    {"field1":"value1","value":2},
    {"field1":"value2","value":1}
]
```

Для сортировки при группировке по двум или более полям необходимо сначала добавить серверную часть фильтра **ColumnIndexFilter** в свой **ViewSet**.

```python
from drf_aggregation.filters import ColumnIndexFilter

class ModelViewSet(AggregationViewSet):
    filter_backends = [ColumnIndexFilter]
```

Этот фильтр группирует исходный набор запросов по указанному полю и сохраняет сортировку элементов. После этого вы можете использовать этот индекс для сортировки данных, сгруппированных желаемым образом.

```python
?aggregation=count&groupBy=field1,field2&columnIndex=field1&orderBy=-field1__index,-value

[
    {"field1":"value2","field2":"value4","value":3},
    {"field1":"value2","field2":"value3","value":1},
    {"field1":"value1","field2":"value3","value":2}
]
```

## Ограничение количества отображаемых групп

Если у вас большое количество категорий или вам нужно отображать только топ-H, есть возможность ограничить количество возвращаемых записей

```python
?aggregation=count&groupBy=field1&orderBy=-value&limit=2

[
    {"field1":"value1","value":10},
    {"field1":"value2","value":9}
]
```

Также возможно отобразить все остальные группы как одну дополнительную категорию.

```python
?aggregation=count&groupBy=field1orderBy=-value&&limit=2&showOther=1

[
    {"field1":"value1","value":10},
    {"field1":"value2","value":9},
    {"field1":"Other","value":45}
]
```

Дополнительные опции при наличии ограничения на количество отображаемых групп:

* **limitBy** - поле для выбора значений, которые останутся, если не пройти, используется первое поле для группировки
* **showOther** - если передано "1", то все группы, не вошедшие в топ, будут отображаться как одна дополнительная категория
* **otherGroupName** - метка для дополнительной категории, по умолчанию "Other"

## Временная последовательность

Предупреждение! Не работает на **SQLite**, потому что у него нет полей даты/времени.

Чтобы отобразить временные ряды, вы должны сначала добавить серверную часть фильтра **TruncateDateFilter** в свой **ViewSet**.

```python
from drf_aggregation.filters import TruncateDateFilter

class ModelViewSet(AggregationViewSet):
    filter_backends = [TruncateDateFilter]
```

Этот фильтр позволит вам добавить поля даты, округленные до нужного уровня, по которым вы сможете группировать и сортировать результат

```python
?truncateDate=created_at=day&groupBy=created_at__trunc__day

[
    {"created_at__trunc__day": date(2020, 10, 4), "value": 1},
    {"created_at__trunc__day": date(2020, 11, 4), "value": 2},
]
```

Доступные сокращения:

* year
* quarter
* month
* week
* day
* hour
* minute
* second

Подробнее об усечениях читайте в [Django Docs](https://docs.djangoproject.com/en/3.1/ref/models/database-functions/#trunc).
