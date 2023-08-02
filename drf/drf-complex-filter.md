# drf-complex-filter

DRF-фильтр для сложных запросов

## Установка

Для установки используйте **pip**

```bash
pip install drf-complex-filter
```

## Применение

Добавьте **ComplexQueryFilter** в **filter\_backends**:

```python
from drf_complex_filter.filters import ComplexQueryFilter

class UserViewSet(ModelViewSet):
      queryset = User.objects.all()
      serializer_class = UserSerializer
      filter_backends = [ComplexQueryFilter]
```

И получить некоторые записи

```http
GET /users?filters={
    "type":"operator",
    "data":{
        "attribute":"first_name",
        "operator":"=",
        "value":"John"
    }
}
```

## Оператор фильтра

Оператор может быть одного из трех типов

```python
# Будет возвращать Q(field_name=value_for_compare)
operator_filter = {
    "type": "operator",
    "data": {
        "attribute": "field_name",
        "operator": "=",
        "value": "value_for_compare",
    }
}

# Будет комбинировать с AND все операторы в "data"
and_filter = {
    "type": "and",
    "data": []
}

# Будет комбинировать с OR все операторы в "data"
or_filter = {
    "type": "or",
    "data": []
}
```

## Операторы поиска

В пакете есть несколько основных операторов, но вы можете заменить или расширить этот список.

| Метка оператора                 | Оператор запроса |
| ------------------------------- | ---------------- |
| Является                        | =                |
| Не является                     | !=               |
| Без учета регистра содержит     | \*               |
| Регистронезависимый не содержит | !                |
| Больше                          | >                |
| Больше или равно                | >=               |
| Меньше                          | <                |
| Меньше или равно                | <=               |
| Содержит значение из списк      | in               |
| Не содержит значение из списк   | not\_in          |
| Текущий пользователь            | me               |
| Не текущий пользователь         | not\_me          |

## Добавление операторов

Сначала создайте класс, содержащий ваши операторы. Он должен содержать как минимум метод `"get_operators"`, который возвращает словарь с вашими операторами.

```python
class YourClassWithOperators:
    def get_operators(self):
        return {
            "simple_operator": lambda f, v, r, m: Q(**{f"{f}": v}),
            "complex_operator": self.complex_operator,
        }

    @staticmethod
    def complex_operator(field: str, value=None, request=None, model: Model = None)
        return Q(**{f"{field}": value})
```

Далее укажите этот класс в конфигурации.

```python
COMPLEX_FILTER_SETTINGS = {
    "COMPARISON_CLASSES": [
        "drf_complex_filter.comparisons.CommonComparison",
        "drf_complex_filter.comparisons.DynamicComparison",
        "path.to.your.module.YourClassWithOperators",
    ],
}
```

Теперь вы можете использовать эти операторы для фильтрации моделей.

## Вычисленное значение

Иногда вам нужно получить значение динамически на стороне сервера, а не записывать его непосредственно в фильтр. Для этого вы можете создать класс, содержащий метод `"get_functions"`.

```python
class YourClassWithFunctions:
    def get_functions(self):
        return {
            "calculate_value": self.calculate_value,
        }

    @staticmethod
    def calculate_value(request, model, my_arg):
        return str(my_arg)
```

Затем зарегистрируйте этот класс в настройках.

```python
COMPLEX_FILTER_SETTINGS = {
    "VALUE_FUNCTIONS": [
        "drf_complex_filter.functions.DateFunctions",
        "path.to.your.module.YourClassWithFunctions",
    ],
}
```

И создайте оператор со значением, подобным этому:

```python
value = {
    "func": "name_of_func",
    "kwargs": { "my_arg": "value_of_my_arg" },
}

operator_filter = {
    "type": "operator",
    "data": {
      "attribute": "field_name",
      "operator": "=",
      "value": value,
    }
}
```

Где:

* **func** - имя вызываемого метода
* **kwargs** - словарь с аргументами для передачи в метод

Значение будет рассчитано перед передачей оператору. Что позволяет использовать полученное таким образом значение с любым оператором, способным его корректно обработать.

## Вычисление подзапроса

Если у вас есть один большой запрос, который необходимо выполнять по частям (не одно большое выполнение, а всего несколько небольших выполнений в связанных моделях), вы можете добавить конструкцию **RelatedModelName\_\_\_** к имени вашего атрибута в операторе. После этого эта конструкция выполняется в отдельном запрос.

```python
operator_filter = {
    "type": "operator",
    "data": {
      "attribute": "RelatedModelName___field_name",
      "operator": "=",
      "value": "value_for_compare",
    }
}
  
# если этот RelatedModelName.objects.filter(field_name="value_for_compare")
# возвращает объекты с идентификаторами `2, 5, 9`, то этот `operator_filter`
# эквивалентен
  
new_filter = {
    "type": "operator",
    "data": {
      "attribute": "RelatedModelName_id",
      "operator": "in",
      "value": [2, 5, 9],
    }
}
  
# и имеет два выбора в БД:
# `select id from RelatedModelName where field_name = 'value_for_compare'`
# и
# `select * from MainTable where RelatedModelName_id in (2, 5, 9)`

```
