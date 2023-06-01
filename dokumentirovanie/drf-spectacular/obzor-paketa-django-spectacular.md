# Обзор пакета django-spectacular

## drf\_spectacular.utils

### OpenApiCallback

#### _class_ drf\_spectacular.utils.OpenApiCallback(_name: str_, _path: str_, _decorator: Union\[Callable\[\[drf\_spectacular.utils.F], drf\_spectacular.utils.F], Dict\[str, Callable\[\[drf\_spectacular.utils.F], drf\_spectacular.utils.F]], Dict\[str, Any]]_)

```python
class drf_spectacular.utils.OpenApiCallback(
    name: str,
    path: str,
    decorator: Union[
        Callable[[drf_spectacular.utils.F], drf_spectacular.utils.F],
        Dict[str, Callable[[drf_spectacular.utils.F], drf_spectacular.utils.F]],
        Dict[str, Any]
    ]
)
```

Bases: `drf_spectacular.utils.OpenApiSchemaBase`

Вспомогательный класс для связывания определения обратного вызова. Это определяет представление на стороне вызываемого абонента, эффективно заявляя об ожиданиях принимающей стороны. Обратите внимание, что этот конкретный экземпляр **@extend\_schema** работает с точки зрения источника обратного вызова, что означает, что **request** определяет исходящий запрос.

Для удобства мы предполагаем, что обратный вызов отправляет `application/json` и возвращает 200. Если этого недостаточно, вы можете использовать перегруженные **request** и **responses**, как обычно.

#### Параметры:

* **name** - Имя, под которым этот обратный вызов указан в схеме.
* **path** - Путь, по которому выполняется операция обратного вызова. Чтобы сослаться на содержимое тела запроса, обратитесь к [ключевым выражениям](https://swagger.io/specification/#key-expression) спецификации OpenAPI для допустимых вариантов.
* **decorator** - Декоратор **@extend\_schema**, указывающий принимающую конечную точку. В этом специальном контексте разрешенными параметрами являются **requests**, **responses**, **summary**, **description**, **deprecated**.

### OpenApiExample

#### _class_ drf\_spectacular.utils.OpenApiExample(_name: str_, _value: Optional\[Any] = None_, _external\_value: str = ''_, _summary: str = ''_, _description: str = ''_, _request\_only: bool = False_, _response\_only: bool = False_, _parameter\_only: Optional\[Tuple\[str, typing\_extensions.Literal\[query, path, header, cookie]]] = None_, _media\_type: Optional\[str] = None_, _status\_codes: Optional\[Sequence\[Union\[str, int]]] = None_)

```python
class drf_spectacular.utils.OpenApiExample(
    name: str,
    value: Optional[Any] = None,
    external_value: str = '',
    summary: str = '',
    description: str = '',
    request_only: bool = False,
    response_only: bool = False,
    parameter_only: Optional[
        Tuple[str, typing_extensions.Literal[query, path, header, cookie]]
    ] = None,
    media_type: Optional[str] = None,
    status_codes: Optional[Sequence[Union[str, int]]] = None
)
```

Bases: `drf_spectacular.utils.OpenApiSchemaBase`

Вспомогательный класс для документирования параметра API/тела запроса/тела ответа с конкретным примером значения.

Рекомендуется указать значение примера в единственном числе, так как ответы на разбивку на страницы и список обрабатываются **drf-spectacular**.

Пример будет присоединен к объекту операции там, где это уместно, т. е. там, где заданные **media\_type**, **status\_code** и модификаторы совпадают. Примеры, не соответствующие ни одному сценарию, игнорируются.

* **media\_type** по умолчанию будет иметь значение `'application/json'`, если только это не указано неявно через **OpenApiResponse**.
* **status\_codes** по умолчанию будет \[200, 201], если это не указано неявно через **OpenApiResponse**.

### OpenApiParameter

#### _class_ drf\_spectacular.utils.OpenApiParameter(_name: str, type: Union\[rest\_framework.serializers.Serializer, Type\[rest\_framework.serializers.Serializer], Type\[Union\[str, float, bool, bytes, int, dict, uuid.UUID, decimal.Decimal, datetime.datetime, datetime.date, datetime.time, datetime.timedelta, ipaddress.IPv4Address, ipaddress.IPv6Address]], drf\_spectacular.types.OpenApiTypes, dict] = \<class 'str'>, location: typing\_extensions.Literal\[query, path, header, cookie] = 'query', required: bool = False, description: str = '', enum: Optional\[Sequence\[Any]] = None, pattern: Optional\[str] = None, deprecated: bool = False, style: Optional\[str] = None, explode: Optional\[bool] = None, default: Optional\[Any] = None, allow\_blank: bool = True, many: Optional\[bool] = None, examples: Optional\[Sequence\[drf\_spectacular.utils.OpenApiExample]] = None, extensions: Optional\[Dict\[str, Any]] = None, exclude: bool = False, response: Union\[bool, Sequence\[Union\[str, int]]] = False_)

```python
class drf_spectacular.utils.OpenApiParameter(
    name: str,
    type: Union[
        rest_framework.serializers.Serializer,
        Type[rest_framework.serializers.Serializer],
        Type[Union[
            str, float, bool, bytes, int, dict, uuid.UUID, decimal.Decimal,
            datetime.datetime, datetime.date, datetime.time, datetime.timedelta,
            ipaddress.IPv4Address, ipaddress.IPv6Address
        ]],
        drf_spectacular.types.OpenApiTypes, dict
    ] = <class 'str'>,
    location: typing_extensions.Literal[query, path, header, cookie] = 'query',
    required: bool = False,
    description: str = '',
    enum: Optional[Sequence[Any]] = None,
    pattern: Optional[str] = None,
    deprecated: bool = False,
    style: Optional[str] = None,
    explode: Optional[bool] = None,
    default: Optional[Any] = None,
    allow_blank: bool = True,
    many: Optional[bool] = None,
    examples: Optional[Sequence[drf_spectacular.utils.OpenApiExample]] = None,
    extensions: Optional[Dict[str, Any]] = None,
    exclude: bool = False,
    response: Union[bool, Sequence[Union[str, int]]] = False
)
```

Bases: `drf_spectacular.utils.OpenApiSchemaBase`

Вспомогательный класс для документирования параметров запроса/пути/заголовка/cookie запроса. Также может использоваться для документирования заголовков ответов.

Обратите внимание, что не все аргументы применимы ко всем вариантам **location**/**type**/направления, например, параметры пути `required = True` по определению.

Для правильного выбора стиля обратитесь к [спецификации OpenAPI](https://swagger.io/specification/#style-values).

COOKIE_: typing\_extensions.Final = 'cookie'_

HEADER_: typing\_extensions.Final = 'header'_

PATH_: typing\_extensions.Final = 'path'_

QUERY_: typing\_extensions.Final = 'query'_
