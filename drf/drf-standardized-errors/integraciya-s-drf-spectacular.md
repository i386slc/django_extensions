# Интеграция с drf-spectacular

## Конфигурирование

Установите класс схемы по умолчанию на тот, который предоставляется пакетом

```python
REST_FRAMEWORK = {
    # другие настройки
    "DEFAULT_SCHEMA_CLASS": "drf_standardized_errors.openapi.AutoSchema"
}
```

или на основе представления (особенно если вы делает это в версионном API)

```python
from drf_standardized_errors.openapi import AutoSchema
from rest_framework.views import APIView

class MyAPIView(APIView):
    schema = AutoSchema()
```

Затем добавьте следующее в настройку drf\_spectacular **ENUM\_NAME\_OVERRIDES**. Это позволит избежать нескольких предупреждений, выдаваемых drf-spectacular из-за того, что один и тот же набор кодов ошибок появляется в нескольких операциях.

```python
SPECTACULAR_SETTINGS = {
    # другие настройки
    "ENUM_NAME_OVERRIDES": {
        "ValidationErrorEnum": "drf_standardized_errors.openapi_serializers.ValidationErrorEnum.values",
        "ClientErrorEnum": "drf_standardized_errors.openapi_serializers.ClientErrorEnum.values",
        "ServerErrorEnum": "drf_standardized_errors.openapi_serializers.ServerErrorEnum.values",
        "ErrorCode401Enum": "drf_standardized_errors.openapi_serializers.ErrorCode401Enum.values",
        "ErrorCode403Enum": "drf_standardized_errors.openapi_serializers.ErrorCode403Enum.values",
        "ErrorCode404Enum": "drf_standardized_errors.openapi_serializers.ErrorCode404Enum.values",
        "ErrorCode405Enum": "drf_standardized_errors.openapi_serializers.ErrorCode405Enum.values",
        "ErrorCode406Enum": "drf_standardized_errors.openapi_serializers.ErrorCode406Enum.values",
        "ErrorCode415Enum": "drf_standardized_errors.openapi_serializers.ErrorCode415Enum.values",
        "ErrorCode429Enum": "drf_standardized_errors.openapi_serializers.ErrorCode429Enum.values",
        "ErrorCode500Enum": "drf_standardized_errors.openapi_serializers.ErrorCode500Enum.values",
    },
}
```

Наконец, если вы не переопределяете настройку хука постобработки из **drf-spectacular**, установите для нее значение

```python
SPECTACULAR_SETTINGS = {
    # другие настройки
    "POSTPROCESSING_HOOKS": ["drf_standardized_errors.openapi_hooks.postprocess_schema_enums"]
}
```

Но если вы уже переопределяете его, обязательно замените хук постобработки enums из drf-spectacular на хук из этого пакета. Хук не будет выдавать предупреждения о динамически создаваемых перечислениях кодов ошибок для каждого поля.

Вот и все, теперь ответы об ошибках будут автоматически генерироваться для каждой операции в вашей схеме. Вот [пример](https://user-images.githubusercontent.com/17159441/172224172-b117aad2-a5cb-4172-a34a-1302f623a5a6.png) того, как это будет выглядеть в пользовательском интерфейсе **swagger**.

## Примечания

* Реализация охватывает все коды состояния, возвращаемые DRF: 400, 401, 403, 404, 405, 406, 415, 429 и 500. Дополнительную информацию о каждом коде состояния и соответствующем исключении можно найти [здесь](https://www.django-rest-framework.org/api-guide/exceptions/#api-reference).
* Основная цель текущей реализации — создать точное определение схемы для ошибок валидации. Это означает **документирование всех возможных кодов ошибок на основе полей**. Это поможет потребителям API заранее знать все возможные возвращенные ошибки, чтобы они могли изменить сообщения об ошибках на основе кода ошибки или выполнить определенную логику для определенного кода ошибки.
* Реализация включает поддержку **django-filter** при его использовании. Это означает, что ответы об ошибках проверки генерируются для представлений списка с использованием **DjangoFilterBackend** и указанием **filterset\_class** или **filterset\_fields**.
* Для ошибок проверки коды ошибок для каждого поля сериализатора собираются из соответствующего атрибута **error\_messages** этого поля. Итак, чтобы этот пакет собирал пользовательские коды ошибок, рекомендуется следовать DRF-способу определения и вызова ошибок проверки. Ниже приведен пример определения сериализатора, который приведет к добавлению **unknown\_email\_domain** к возможным кодам ошибок, выдаваемым полем электронной почты, и **invalid\_date\_range** к списку кодов, связанных с **non\_field\_errors** сериализатора. Важно то, что пользовательские коды ошибок добавляются в **default\_error\_messages** соответствующего сериализатора или поля сериализатора. Обратите внимание, что вы также можете переопределить `__init__` и добавить коды ошибок непосредственно в `self.error_messages`.

```python
from rest_framework.fields import empty
from rest_framework import serializers


class CustomDomainEmailField(serializers.EmailField):
    default_error_messages = {"unknown_email_domain": "The email domain is invalid."}

    def run_validation(self, data=empty):
        data = super().run_validation(data)
        if data and not data.endswith("custom-domain.com"):
            self.fail("unknown_email_domain")
        return data


class CustomSerializer(serializers.Serializer):
    default_error_messages = {
        "invalid_date_range": "The end date should be after the start date."
    }
    name = serializers.CharField()
    email = CustomDomainEmailField()
    start_date = serializers.DateField()
    end_date = serializers.DateField()

    def validate(self, attrs):
        start_date = attrs.get("start_date")
        end_date = attrs.get("end_date")
        if start_date and end_date and end_date < start_date:
            self.fail("invalid_date_range")

        return attrs
```

## Секреты и уловки

### Скрыть ответы об ошибках, которые отображаются в каждой операции

По умолчанию в схему будет добавлен ответ об ошибке для всех поддерживаемых кодов состояния. Некоторые из этих кодов состояния фактически появляются в каждой операции: 500 (ошибка сервера) и 405 (метод не разрешен). Другие также могут появляться в каждой операции при определенных условиях:

* Если все операции требуют аутентификации, то в каждой из них появится 401.
* Если все конечные точки регулируются, произойдет то же самое.
* Кроме того, 406 (not acceptable) покажет, используете ли вы средство согласования контента по умолчанию и так далее.

В этом случае рекомендуется скрыть эти ответы об ошибках из схемы и использовать атрибут описания схемы, чтобы скрыть их.

Давайте возьмем пример API, где все конечные точки требуют аутентификации и принимают/возвращают только json. При этом имеем:

* 500 (server error) и 405 (method not allowed) в каждой операции (поведение пакета по умолчанию)
* 401 (unauthorized) почти в каждой операции (кроме входа/регистрации)
* 406 (not acceptable), появляющийся в каждой операции, поскольку API возвращает только json, а потребители API могут заполнять заголовок «Accept» значением, отличным от «application/json».
* 415 (unsupported media type), поскольку каждый потребитель API может отправлять содержимое запроса, отличное от json.

Теперь, когда мы определили ответы об ошибках, которые будут в каждой операции, мы можем добавить примечания о них в описание API. Поскольку описание может стать немного длинным, давайте добавим его в файл markdown (вместо того, чтобы добавлять его в файл Python). Кроме того, это означает, что за ним будет легче ухаживать. Вот [пример файла markdown](https://drf-standardized-errors.readthedocs.io/en/latest/openapi\_sample\_description.html) (вы можете скопировать содержимое с GitHub). Затем содержимое файла необходимо установить в качестве описания API.

```python
# settings.py
with open("/absolute/path/to/openapi_sample_description.md") as f:
    description = f.read()

SPECTACULAR_SETTINGS = {
    "TITLE": "Awesome API",
    "DESCRIPTION": description,
    # другие настройки
}
```

Теперь, когда сведения об ошибках, которые отображаются во всех операциях, являются частью документации, мы можем удалить их из списка ошибок, отображаемых в схеме API. Это должно составить список ответов об ошибках

```python
DRF_STANDARDIZED_ERRORS = {
    "ALLOWED_ERROR_STATUS_CODES": ["400", "403", "404", "429"]
}
```

Обратите внимание, что вы можете еще больше ограничить список кодов состояния при других обстоятельствах. Если в API используется управление версиями URL, то при каждой операции будет отображаться ошибка **404**. Кроме того, если вы предоставляете общедоступный API и ограничиваете все конечные точки, чтобы избежать злоупотреблений или в рамках бизнес-модели, то **429** лучше удалить, а примечания о нем добавить в описание API.

### Скрыть ответы об ошибках синтаксического анализа

Код состояния **400** охватывает как ошибки валидации, так и ошибки синтаксического анализа. Но, поскольку ошибки синтаксического анализа обычно появляются для каждой операции, вы можете скрыть их, но при этом показывать ошибки валидации. Для этого необходимо переопределить поведение генерации ответов об ошибках по умолчанию в классе **AutoSchema**:

```python
from drf_standardized_errors.handler import exception_handler as standardized_errors_handler
from drf_standardized_errors.openapi import AutoSchema

class CustomAutoSchema(AutoSchema):
    def _should_add_error_response(self, responses: dict, status_code: str) -> bool:
        if (
            status_code == "400"
            and status_code not in responses
            and self.view.get_exception_handler() is standardized_errors_handler
        ):
            # нет необходимости учитывать ошибки синтаксического анализа
            # при принятии решения о том, следует ли добавлять ответ об ошибке 400
            return self._should_add_validation_error_response()
        else:
            return super()._should_add_error_response(responses, status_code)

    def _get_http400_serializer(self):
        # удалена вся логика, связанная с наличием ошибок синтаксического анализа
        return self._get_serializer_for_validation_error_response()
```

После этого обновите параметр **DEFAULT\_SCHEMA\_CLASS**.

```python
REST_FRAMEWORK = {
    # другие настройки
    "DEFAULT_SCHEMA_CLASS": "path.to.CustomAutoSchema"
}
```

### Уже используется пользовательский класс AutoSchema

Если вы уже переопределяете класс **AutoSchema**, предоставленный **drf-spectacular**, обязательно наследуйте класс **AutoSchema**, предоставленный этим пакетом. Кроме того, если вы переопределяете **get\_examples** и/или **\_get\_response\_bodies**, обязательно вызовите **super**.

### Пользовательский код состояния

Это идет рука об руку с [обработкой исключений, отличных от DRF](https://drf-standardized-errors.readthedocs.io/en/latest/customization.html#handle-a-non-drf-exception). Итак, давайте предположим, что вы определили пользовательское исключение, которое может быть вызвано в любой операции:

```python
from rest_framework.exceptions import APIException

class ServiceUnavailable(APIException):
    status_code = 503
    default_detail = 'Service temporarily unavailable, try again later.'
    default_code = 'service_unavailable'
```

Затем вам нужно будет добавить соответствующий код состояния в настройки и определить класс сериализатора, который представляет возвращаемый ответ.

```python
# serializers.py
from django.db import models
from rest_framework import serializers
from drf_standardized_errors.openapi_serializers import ServerErrorEnum

class ErrorCode503Enum(models.TextChoices):
    SERVICE_UNAVAILABLE = "service_unavailable"

class Error503Serializer(serializers.Serializer):
    code = serializers.ChoiceField(choices=ErrorCode503Enum.choices)
    detail = serializers.CharField()
    attr = serializers.CharField(allow_null=True)

class ErrorResponse503Serializer(serializers.Serializer):
    type = serializers.ChoiceField(choices=ServerErrorEnum.choices)
    errors = Error503Serializer(many=True)
```

```python
# settings.py
DRF_STANDARDIZED_ERRORS = {
    "ALLOWED_ERROR_STATUS_CODES": ["400", "403", "404", "429", "503"],
    "ERROR_SCHEMAS": {"503": "path.to.ErrorResponse503Serializer"}
}
SPECTACULAR_SETTINGS = {
    # другие настройки
    "ENUM_NAME_OVERRIDES": {
        # чтобы избежать предупреждений, выдаваемых drf-spectacular,
        # добавьте следующую строку
        "ErrorCode503Enum": "path.to.ErrorCode503Enum.values",
    },
}
```

Если код состояния появляется только в определенных операциях, вы можете создать свою собственную **AutoSchema**, которая наследуется от схемы, предоставленной этим пакетом, а затем переопределить `AutoSchema._should_add_error_response`, чтобы определить критерии, управляющие добавлением ответа об ошибке в операцию. Например, добавление ответа **503** только в том случае, если метод операции — **GET**, выглядит так:

```python
from drf_standardized_errors.openapi import AutoSchema

class CustomAutoSchema(AutoSchema):
    def _should_add_error_response(self, responses: dict, status_code: str) -> bool:
        if status_code == "503":
            return self.method == "GET"
        else:
            return super()._should_add_error_response(responses, status_code)
```

Не забудьте обновить **DEFAULT\_SCHEMA\_CLASS**, чтобы в этом случае он указывал на **CustomAutoSchema**.

```python
REST_FRAMEWORK = {
    # другие настройки
    "DEFAULT_SCHEMA_CLASS": "path.to.CustomAutoSchema"
}
```

### Пользовательский формат ошибки

Эта запись охватывает изменения, необходимые, если вы измените формат ответа об ошибке по умолчанию. Основная идея заключается в том, что вам необходимо предоставить сериализаторы, описывающие каждый код состояния ошибки в **ALLOWED\_ERROR\_STATUS\_CODES**. Кроме того, вы должны предоставить примеры для каждого кода состояния или убедиться, что примеры по умолчанию не отображаются.

Давайте продолжим с примера в разделе «[Кастомизация](kastomizaciya-drf-se.md)» об [изменении формата ответа об ошибке](kastomizaciya-drf-se.md#izmenit-format-otveta-ob-oshibke). Стандартный ответ на ошибку выглядит следующим образом:

```json
{
    "type": "string",
    "code": "string",
    "message": "string",
    "field_name": "string"
}
```

Теперь предположим, что вам нужен точный ответ об ошибке на основе кода состояния. Это означает, что вы хотите, чтобы схема показывала, какие конкретные типы, коды и имена полей следует ожидать на основе кода состояния. Кроме того, чтобы пример не стал слишком длинным, для **ALLOWED\_ERROR\_STATUS\_CODES** будет установлено только значение `["400", "403", "404"]`. Это потому, что работа для других кодов состояния будет аналогична **403** и **404**. Однако генерация ответа об ошибке для **400** сложна по сравнению с другими, и поэтому он в списке.

Начнем с простых (**403** и **404**):

```python
from drf_standardized_errors.openapi_serializers import (
    ClientErrorEnum, ErrorCode403Enum, ErrorCode404Enum
)
from rest_framework import serializers


class ErrorResponse403Serializer(serializers.Serializer):
    type = serializers.ChoiceField(choices=ClientErrorEnum.choices)
    code = serializers.ChoiceField(choices=ErrorCode403Enum.choices)
    message = serializers.CharField()
    field_name = serializers.CharField(allow_null=True)


class ErrorResponse404Serializer(serializers.Serializer):
    type = serializers.ChoiceField(choices=ClientErrorEnum.choices)
    code = serializers.ChoiceField(choices=ErrorCode404Enum.choices)
    message = serializers.CharField()
    field_name = serializers.CharField(allow_null=True)
```

Далее обновим настройки

```python
DRF_STANDARDIZED_ERRORS = {
    "ALLOWED_ERROR_STATUS_CODES": ["400", "403", "404"],
    "ERROR_SCHEMAS": {
        "403": "path.to.ErrorResponse403Serializer",
        "404": "path.to.ErrorResponse404Serializer",
    }
}
```

Теперь давайте перейдем к **400**. Этот код состояния представляет ошибки синтаксического анализа, а также ошибки проверки, а ошибки проверки являются динамическими на основе сериализатора в соответствующей операции. Итак, нам нужно создать собственный класс **AutoSchema**, который возвращает правильный сериализатор ответа на ошибку на основе операции.

```python
from drf_spectacular.utils import PolymorphicProxySerializer
from drf_standardized_errors.openapi_serializers import (
    ClientErrorEnum, ParseErrorCodeEnum, ValidationErrorEnum
)
from drf_standardized_errors.openapi import AutoSchema
from drf_standardized_errors.settings import package_settings
from inflection import camelize
from rest_framework import serializers


class ParseErrorResponseSerializer(serializers.Serializer):
    type = serializers.ChoiceField(choices=ClientErrorEnum.choices)
    code = serializers.ChoiceField(choices=ParseErrorCodeEnum.choices)
    message = serializers.CharField()
    field_name = serializers.CharField(allow_null=True)


class CustomAutoSchema(AutoSchema):
    def _get_http400_serializer(self):
        operation_id = self.get_operation_id()
        component_name = f"{camelize(operation_id)}ErrorResponse400"

        http400_serializers = []
        if self._should_add_validation_error_response():
            fields_with_error_codes = self._determine_fields_with_error_codes()
            error_serializers = [
                get_serializer_for_validation_error_response(
                    operation_id, field.name, field.error_codes
                )
                for field in fields_with_error_codes
            ]
            http400_serializers.extend(error_serializers)
        if self._should_add_parse_error_response():
            http400_serializers.append(ParseErrorResponseSerializer)

        return PolymorphicProxySerializer(
            component_name=component_name,
            serializers=http400_serializers,
            resource_type_field_name="field_name",
        )


def get_serializer_for_validation_error_response(operation_id, field, error_codes):
    field_choices = [(field, field)]
    error_code_choices = sorted(zip(error_codes, error_codes))

    camelcase_operation_id = camelize(operation_id)
    attr_with_underscores = field.replace(package_settings.NESTED_FIELD_SEPARATOR, "_")
    camelcase_attr = camelize(attr_with_underscores)
    suffix = package_settings.ERROR_COMPONENT_NAME_SUFFIX
    component_name = f"{camelcase_operation_id}{camelcase_attr}{suffix}"

    class ValidationErrorSerializer(serializers.Serializer):
        type = serializers.ChoiceField(choices=ValidationErrorEnum.choices)
        code = serializers.ChoiceField(choices=error_code_choices)
        message = serializers.CharField()
        field_name = serializers.ChoiceField(choices=field_choices)

        class Meta:
            ref_name = component_name

    return ValidationErrorSerializer
```

Остается удалить примеры по умолчанию из класса **AutoSchema** или создать новые, которые соответствуют новому выводу ответа об ошибке. Удалить примеры по умолчанию легко, и это можно сделать, переопределив **get\_examples** и вернув пустой список, который оставляет генерацию примеров на уровне используемого пользовательского интерфейса **OpenAPI** (**swagger UI**, **redoc** и т. д.). Но если вы разборчивы в примерах и хотите показать, что атрибут **field\_name** всегда имеет значение **null** для ошибок, отличных от ошибок проверки, вы можете предоставить примеры. Поэтому приступим к созданию новых примеров для **403** и **404**.

```python
from drf_standardized_errors.openapi import AutoSchema
from rest_framework import exceptions
from drf_spectacular.utils import OpenApiExample


class CustomAutoSchema(AutoSchema):
    def get_examples(self):
        errors = [exceptions.PermissionDenied(), exceptions.NotFound()]
        return [get_example_from_exception(error) for error in errors]

def get_example_from_exception(exc: exceptions.APIException):
    return OpenApiExample(
        exc.__class__.__name__,
        value={
            "type": "client_error",
            "code": exc.get_codes(),
            "message": exc.detail,
            "field_name": None,
        },
        response_only=True,
        status_codes=[str(exc.status_code)],
    )
```

### Настройка кодов ошибок в зависимости от операции

При определении кодов ошибок в полевых условиях предполагается, что разработчик будет следовать примеру, приведенному в последнем пункте [Notes](integraciya-s-drf-spectacular.md#primechaniya). Однако есть ситуации, когда этого не происходит:

* При использовании сериализаторов, предоставляемых сторонними пакетами, пакет не добавляет коды ошибок в атрибут **error\_messages**.
* При использовании пользовательской формы для класса набора фильтров, и эта форма имеет метод **clean**, который включает проверку между несколькими полями (например, при наличии полей даты начала/окончания или минимальной/максимальной цены)
* При вызове **ValidationError** непосредственно внутри представления.
* …

В этих случаях мы можем использовать декоратор **@extend\_validation\_errors**, чтобы добавить дополнительные коды ошибок для поля к определенным действиям, методам и/или версиям. Вот один пример: у нас есть класс набора фильтров с полями **start\_date** и **end\_date**, и класс набора фильтров использует пользовательскую форму, которая проверяет, что **end\_date** больше или равен **start\_date** в методе `Form.clean`. **@extend\_validation\_errors** можно использовать в наборе представлений, чтобы добавить конкретный код ошибки в правильное поле в вашей схеме API (в этом случае `__all__` задается как имя поля, потому что django устанавливает его как имя поля для ошибок, возникающих в `Form.clean` ).

```python
from rest_framework.viewsets import ModelViewSet
from django import forms
from django.contrib.auth import get_user_model
from django_filters.rest_framework import DjangoFilterBackend, FilterSet, DateFilter
from drf_standardized_errors.openapi_validation_errors import extend_validation_errors

User = get_user_model()


class UserForm(forms.Form):
    start_date = forms.DateField()
    end_date = forms.DateField()
    
    def clean(self):
        cleaned_data = super().clean()
        start_date = cleaned_data.get("start_date")
        end_date = cleaned_data.get("end_date")
        if start_date and end_date and end_date < start_date:
            msg = "The end should be greater than or equal to the start date."
            raise forms.ValidationError(msg, code="invalid_date_range")
        
        return cleaned_data


class UserFilterSet(FilterSet):
    start_date = DateFilter(field_name="date_joined", lookup_expr="gte")
    end_date = DateFilter(field_name="date_joined", lookup_expr="lte")
    
    class Meta:
        model = User
        fields = ["start_date", "end_date"]
        form = UserForm


@extend_validation_errors(
    ["invalid_date_range"], field_name="__all__", actions=["list"], methods=["get"]
)
class UserViewSet(ModelViewSet):
    queryset = User.objects.all()
    serializer_class = ...
    filter_backends = (DjangoFilterBackend,)
    filterset_class = UserFilterSet
```

Несколько заметок о декораторе:

* Его можно применить к классу представления, классу набора представлений или функции представления, украшенной **@api\_view**.
* Его можно применять несколько раз к одному и тому же представлению.
* Он добавляет дополнительные коды ошибок к уже собранным **drf-standardized-errors** для определенного поля.
* Если он применяется к родительскому представлению, добавленные коды ошибок будут автоматически добавлены к дочернему представлению.
* Коды ошибок, добавленные в дочернее представление, переопределяют коды, добавленные в родительское представление для определенного поля, метода, действия и версии.

### extend\_validation\_errors()

#### drf\_standardized\_errors.openapi\_validation\_errors.extend\_validation\_errors(_error\_codes: List\[str]_, _field\_name: str | None = None_, _actions: List\[str] | None = None_, _methods: List\[str] | None = None_, _versions: List\[str] | None = None_) → Callable\[\[V], V]

Декоратор представления/набора представлений для добавления дополнительных кодов ошибок к ошибкам проверки. Этот декоратор не переопределяет коды ошибок, уже собранные **drf-standardized-errors**.

#### Параметры:

* **error\_codes** — список кодов ошибок для добавления.
* **field\_name** — имя сериализатора или поля формы, в которое будут добавляться коды ошибок. Можно установить значение **«non\_field\_errors»**, если коды ошибок соответствуют проверке внутри **Serializer.validate**, или **«\_\_all\_\_»**, если они соответствуют проверке внутри `Form.clean`. Его также можно оставить равным `None`, если проверка не связана с каким-либо сериализатором или формой (например, при вызове **serializers.ValidationError** непосредственно внутри представления или набора представлений).
* **actions** — можно задать при оформлении вьюсета. Ограничивает добавленные коды ошибок указанными действиями. По умолчанию коды ошибок добавляются ко всем действиям.
* **methods** — ограничивает добавленные коды ошибок указанными методами (**get**, **post**, …). По умолчанию добавляются коды ошибок независимо от метода.
* **versions** — ограничивает добавленные коды ошибок указанными версиями. По умолчанию добавляются коды ошибок независимо от версии.
