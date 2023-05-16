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

### Пользовательский формат ошибки

### Настройка кодов ошибок в зависимости от операции

### extend\_validation\_errors()
