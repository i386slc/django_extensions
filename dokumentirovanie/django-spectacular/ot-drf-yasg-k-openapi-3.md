# От drf-yasg к OpenAPI 3

[drf-yasg](https://pypi.org/project/drf-yasg) — отличная библиотека и самый популярный выбор для создания схем OpenAPI 2.0 (ранее известного как Swagger 2.0) с помощью [Django REST Framework](https://pypi.org/project/djangorestframework/). К сожалению, в настоящее время он не поддерживает OpenAPI 3.x. Переход с **drf-yasg** на **drf-spectacular** требует некоторых модификаций, сложность которых зависит от используемых функций.

{% hint style="info" %}
В отличие от **drf-yasg**, мы не упаковываем пользовательский интерфейс **Redoc** и **Swagger UI**, а вместо этого обслуживаем их через CDN с гиперссылками. Если вы хотите или должны обслуживать эти файлы самостоятельно, вы можете сделать это с помощью дополнительного пакета [drf-spectacular-sidecar](https://github.com/tfranzel/drf-spectacular-sidecar). Дополнительные сведения см. в [инструкциях по установке](./#ustanovka).
{% endhint %}

## Декораторы

* **@swagger\_auto\_schema** во многом эквивалентен **@extend\_schema**.
  * Аргумент **operation\_description** называется **description**
  * аргумент **operation\_summary** называется **summary**
  * Аргументы **manual\_parameters** и **query\_serializer** объединены в один аргумент **parameters**
  * аргумент **security** называется **auth**
  * Аргументы **request\_body** называются **request**
    * Используйте `None` вместо `drf_yasg.utils.no_body`
  * Аргумент **method** не существует, вместо этого используйте **methods** (также поддерживается **drf-yasg**)
  * **auto\_schema** не имеет эквивалента.
  * **extra\_overrides** не имеет эквивалента.
  * **field\_inspectors** не имеет эквивалента.
  * **filter\_inspectors** не имеет эквивалента.
  * **paginator\_inspectors** не имеет эквивалента.
  * Также доступны дополнительные аргументы: **excluse**, **operation**, **versions**, **examples**.
* **@swagger\_serializer\_method** эквивалентен **@extend\_schema\_field**.
  * Можно указать **component\_name**, чтобы выделить поле как отдельный компонент.
* **@extend\_schema\_serializer** доступен для переопределения поведения сериализаторов.
* Вместо использования **@method\_decorator** используйте **@extend\_schema\_view**.
* Вместо использования **swagger\_schema\_field** используйте **@extend\_schema\_field** или **@extend\_schema\_serializer**.

## Классы Helper

* **Parameter** примерно эквивалентен **OpenApiParameter**.
  * аргумент **in\_** называется **location**.
  * аргумент **schema** должен быть передан как **type**.
  * Аргумент **format**  объединяется с аргументом **type** с помощью **OpenApiTypes**.
  * установка для аргумента **many** значения `True` приводит к тому, что аргумент принимает массив значений и создает схему, аналогичную использованию класса drf\_yasg **Items** для свойства **items**. Тип элементов массива определяется аргументом **type**.
* **Response** во многом идентичен **OpenApiResponse**.
  * аргумент **schema** называется **response**
  * Порядок аргументов отличается, поэтому используйте аргументы ключевого слова.
* **OpenApiExample** доступен для предоставления примеров **@extend\_schema**.
* **Schema** не требуется и может быть удалена. Вместо этого используйте простой **dict**.

## Типы и форматы

Вместо отдельных констант `drf_yasg.openapi.TYPE_*` и `drf_yasg.openapi.FORMAT_*` drf-spectacular предоставляет перечисление **OpenApiTypes**:

* **TYPE\_BOOLEAN** называется **BOOL**, но вы можете использовать **bool**.
* **TYPE\_FILE** следует заменить на **BINARY**
* **TYPE\_INTEGER** называется **INT**, но вы можете использовать **int**.
* **TYPE\_INTEGER** с **FORMAT\_INT32** называется **INT32**
* **TYPE\_INTEGER** с **FORMAT\_INT64** называется **INT64**
* **TYPE\_NUMBER** называется **NUMBER**
* **TYPE\_NUMBER** с **FORMAT\_FLOAT** называется **FLOAT**, но вы можете использовать **float**.
* **TYPE\_NUMBER** с **FORMAT\_DOUBLE** называется **DOUBLE** (или **DECIMAL**, но вы можете использовать **Decimal**)
* **TYPE\_OBJECT** называется **OBJECT**, но вы можете использовать **dict**.
* **TYPE\_STRING** называется **STR**, но вы можете использовать **str**.
* **TYPE\_STRING** с **FORMAT\_BASE64** называется **BYTE** (который кодируется **base64**).
* **TYPE\_STRING** с **FORMAT\_BINARY** называется **BINARY**, но вы можете использовать **bytes**.
* **TYPE\_STRING** с **FORMAT\_DATETIME** называется **DATETIME**, но вы можете использовать **datetime.datetime**.
* **TYPE\_STRING** с **FORMAT\_DATE** называется **DATE**, но вы можете использовать **datetime.date**.
* **TYPE\_STRING** с **FORMAT\_EMAIL** называется **EMAIL**
* **TYPE\_STRING** с **FORMAT\_IPV4** называется **IP4**, но вы можете использовать **ipaddress.IPv4Address**.
* **TYPE\_STRING** с **FORMAT\_IPV6** называется **IP6**, но вы можете использовать **ipaddress.IPv6Address**.
* **TYPE\_STRING** с **FORMAT\_PASSWORD** называется **PASSWORD**
* **TYPE\_STRING** с **FORMAT\_URI** называется **URI**
* **TYPE\_STRING** с **FORMAT\_UUID** называется **UUID**, но вы можете использовать **uuid.UUID**.
* **TYPE\_STRING** с **FORMAT\_SLUG** не имеет прямого эквивалента. Вместо этого используйте **STR** или **str**.
* **TYPE\_ARRAY** обрабатывается путем предоставления **OpenApiParameter** со значением `many=True` в качестве параметра. Нет необходимости устанавливать свойство **items** для параметра - наличие `many=True` превращает параметр в параметр массива.
* Также доступны следующие дополнительные типы:
  * **ANY**, для которого вы можете использовать **type.Any**.
  * **DURATION**, для которого вы можете использовать **datetime.timedelta**.
  * **HOSTNAME**
  * **IDN\_EMAIL**
  * **IDN\_HOSTNAME**
  * **IRI\_REF**
  * **IRI**
  * **JSON\_PTR\_REL**
  * **JSON\_PTR**
  * **NONE**, для которого вы можете использовать **None**.
  * **REGEX**
  * **TIME**, для которого вы можете использовать **datetime.time**.
  * **URI\_REF**
  * **URI\_TPL**

## Параметр Location

Константы `drf_yasg.openapi.IN_*` примерно эквивалентны константам, определенным в классе **OpenApiParameter**:

* **IN\_PATH** называется **PATH**
* **IN\_QUERY** называется **QUERY**
* **IN\_HEADER** называется **HEADER**
* **IN\_BODY** и **IN\_FORM** не имеют прямого эквивалента. Вместо этого вы можете использовать `@extend_schema(request={"": ...})`.
* **COOKIE** также доступен.

## Парсинг Docstring

**drf-yasg** имеет специальную обработку строк документации, которая не поддерживается **drf-spectacular**.

Он пытается отделить первую строку от остальной части строки документации для использования в качестве сводки операции, а оставшаяся часть используется в качестве описания операции. **drf-spectacular** использует всю строку документации в качестве описания. Вместо этого используйте аргументы **summary** и **decription** в **@extend\_schema**. При желании строку документации можно использовать для заполнения описания операции.

```python
# Поддерживается drf-yasg:
class UserViewSet(ViewSet):
    def list(self, request):
        """
        List all the users.

        Return a list of all usernames in the system.
        """
        ...

# Обновлено для drf-spectacular с использованием декоратора для описания:
class UserViewSet(ViewSet):
    @extend_schema(
        summary="List all the users.",
        description="Return a list of all usernames in the system.",
    )
    def list(self, request):
        ...

# Обновлено для drf-spectacular с использованием строки документации для описания:
class UserViewSet(ViewSet):
    @extend_schema(summary="List all the users.")
    def list(self, request):
        """Возвращает список всех имен пользователей в системе."""
        ...
```

Кроме того, **drf-yasg** также поддерживает [именованные разделы](https://www.django-rest-framework.org/coreapi/schemas/#schemas-as-documentation), но они не поддерживаются **drf-spectacular**. Опять же, вместо этого используйте аргументы **summary** и **description** в **@extend\_schema**:

```python
# Поддерживается drf-yasg:
class UserViewSet(ViewSet):
    """
    list:
        List all the users.

        Return a list of all usernames in the system.

    retrieve:
        Retrieve user

        Get details of a specific user
    """
    ...

# Обновлено для drf-spectacular с использованием декоратора для описания:
@extend_schema_view(
    list=extend_schema(
        summary="List all the users.",
        description="Return a list of all usernames in the system.",
    ),
    retrieve=extend_schema(
        summary="Retrieve user",
        description="Get details of a specific user",
    ),
)
class UserViewSet(ViewSet):
    ...
```

## Аутентификация

В **drf-yasg** нужно было [вручную прописывать схемы аутентификации](https://drf-yasg.readthedocs.io/en/stable/security.html).

В **drf-spectacular** есть поддержка автоматического создания определений безопасности для ряда классов аутентификации, встроенных в DRF, а также в другие популярные сторонние пакеты. **OpenApiAuthenticationExtension** помогает связать пользовательские классы аутентификации — см. [руководство по настройке](nastroika-rabochego-processa-i-skhemy.md#ukazhite-autentifikaciyu-s-pomoshyu-openapiauthenticationextension.).

## Совместимость

Для совместимости реализованы следующие функции **drf-yasg**:

* Поддерживается **ref\_name** в **Serializer Meta** (за исключением встраивания с `ref_name=None`)
  * Дополнительные сведения см. в [документации drf-yasg](https://drf-yasg.readthedocs.io/en/stable/custom\_spec.html#swagger-schema-fields).
  * Эквивалентом в **drf-spectacular** является `@extend_schema_serializer(component_name="...")`
* **swagger\_fake\_view** доступен как атрибут в представлениях, чтобы сигнализировать о генерации схемы
