# Быстрый старт drf-se

Установка с **pip**:

```bash
pip install drf-standardized-errors
```

Добавьте **drf-standardized-errors** в установленные приложения.

```python
INSTALLED_APPS = [
    # другие приложения
    "drf_standardized_errors",
]
```

Установите обработчик исключений для всех представлений API.

```python
REST_FRAMEWORK = {
    # другие настройки
    "EXCEPTION_HANDLER": "drf_standardized_errors.handler.exception_handler"
}
```

или на основе просмотра (особенно если вы представляете это в версионном API)

```python
from drf_standardized_errors.handler import exception_handler
from rest_framework.views import APIView

class MyAPIView(APIView):
    def get_exception_handler(self):
        return exception_handler
```

Теперь ваши ответы об ошибках API для ошибок 4xx и 5xx будут выглядеть следующим образом.

```json
{
  "type": "validation_error",
  "errors": [
    {
      "code": "required",
      "detail": "This field is required.",
      "attr": "name"
    },
    {
      "code": "max_length",
      "detail": "Ensure this value has at most 100 characters.",
      "attr": "title"
    }
  ]
}
```

или

```json
{
  "type": "server_error",
  "errors": [
    {
      "code": "error",
      "detail": "A server error occurred.",
      "attr": null
    }
  ]
}
```

## Важные заметки

* Стандартные ответы на ошибки, когда `DEBUG=True` для **необработанных исключений**, по умолчанию отключены. Это позволит вам получить больше информации из трассировки. Вместо этого вы можете включить стандартизированные ошибки с помощью:

```python
DRF_STANDARDIZED_ERRORS = {"ENABLE_IN_DEBUG_FOR_UNHANDLED_EXCEPTIONS": True}
```

* Случаи, когда вы явно возвращаете ответ с кодом состояния 4xx или 5xx в вашем **APIView**, не проходят через обработчик исключений и, следовательно, не будут иметь стандартизированного формата ошибки. Поэтому мы рекомендуем вызывать исключение с помощью `raise APIException("Service temporarily unavailable", code="service_unavailable")` вместо `return Response(data, status=500)`. Таким образом, форматирование ответа об ошибке обрабатывается автоматически. Но имейте в виду, что об исключениях, которые приводят к ответу 5xx, сообщается инструментам мониторинга ошибок (например, **Sentry**), если вы их используете.

## Интеграция с DRF spectular

Если вы планируете использовать [drf-spectacular](https://github.com/tfranzel/drf-spectacular) для создания схемы **OpenAPI 3**, установите команду `pip install drf-standardized-errors[openapi]`. После этого проверьте страницу документа для настройки интеграции.
