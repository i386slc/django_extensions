# Кастомизация drf-se

Идея этого пакета состоит в том, чтобы стандартизировать ответы на ошибки и упростить настройку. Для этого обработчик исключений был переписан как класс, поэтому его легко создавать подклассы и вносить небольшие изменения. Сначала мы рассмотрим краткое описание процесса генерации ответа об ошибке, а затем проверим некоторые настройки, которые вы, возможно, захотите сделать.

## Поток обработки исключений

Вам рекомендуется прочитать исходный код, так как его не так много, но вот краткий обзор.

* Процесс начинается с преобразования известных исключений, таких как `django.core.exceptions.PermissionDenied` и `django.http.Http404`, в [исключения DRF](https://www.django-rest-framework.org/api-guide/exceptions/#api-reference).
* Затем любое необработанное исключение преобразуется в экземпляр `rest_framework.exceptions.APIException`.
* После этого данные об исключении извлекаются и форматируются, а ответ об ошибке генерируется с правильными заголовками.
* Наконец, если исключение является ошибкой сервера (код состояния 5xx), оно регистрируется и отправляется сигнал **got\_request\_exception**. Это помогает [тестовому клиенту django](https://github.com/django/django/blob/1b3c0d3b54d4ff5f75af57d3130180b1d22468e9/django/test/client.py#L712) или [инструменту мониторинга ошибок, такому как Sentry](https://github.com/getsentry/sentry-python/blob/d880f47add3876d5cedefb4178a1dcd4d85b5d1b/sentry\_sdk/integrations/django/\_\_init\_\_.py#L138), собирать сведения об исключении.

## Примеры настроек

### Обработка исключения, отличного от DRF

Это можно сделать так же, как [рекомендует DRF](https://www.django-rest-framework.org/api-guide/exceptions/#apiexception):

* Создайте новый класс исключений, унаследовав его от **APIException** и задав атрибуты **default\_detail** и **default\_code**.
* Также установите атрибут **status\_code**, но имейте в виду, что код состояния используется для определения типа ошибки. Код состояния 4xx приводит к ошибке **client\_error**, а код состояния 5xx — к ошибке **server\_error**.
* На ваш взгляд, теперь вы можете вызвать новое исключение, и оно будет обработано соответствующим образом.

Кроме того, вы можете настроить обработчик исключений вместо создания нового исключения в своем коде:

* Предполагая пример из документов DRF для исключения **ServiceUnavailable**

```python
from rest_framework.exceptions import APIException

class ServiceUnavailable(APIException):
    status_code = 503
    default_detail = 'Service temporarily unavailable, try again later.'
    default_code = 'service_unavailable'
```

* Вам нужно создать подкласс `drf_standardized_errors.handler.ExceptionHandler` и переопределить **convert\_known\_exceptions**

```python
import requests
from drf_standardized_errors.handler import ExceptionHandler

class MyExceptionHandler(ExceptionHandler):
    def convert_known_exceptions(self, exc: Exception) -> Exception:
        if isinstance(exc, requests.Timeout):
            return ServiceUnavailable()
        else:
            return super().convert_known_exceptions(exc)
```

Затем обновите параметр, чтобы он указывал на ваш класс обработчика исключений.

```python
DRF_STANDARDIZED_ERRORS = {"EXCEPTION_HANDLER_CLASS": "path.to.MyExceptionHandler"}
```

### Изменить формат ответа об ошибке

Допустим, вам не нужно возвращать несколько ошибок, и вам не нравятся имена некоторых ключей в ответе об ошибке: в частности, вы хотите изменить **detail** на **message** и **attr** на **field\_name**.

Вам нужно создать подкласс **ExceptionFormatter** и переопределить **format\_error\_response**.

```python
from drf_standardized_errors.formatter import ExceptionFormatter
from drf_standardized_errors.types import ErrorResponse

class MyExceptionFormatter(ExceptionFormatter):
    def format_error_response(self, error_response: ErrorResponse):
        error = error_response.errors[0]
        return {
            "type": error_response.type,
            "code": error.code,
            "message": error.detail,
            "field_name": error.attr
        }
```

Затем обновите соответствующий параметр

```python
DRF_STANDARDIZED_ERRORS = {"EXCEPTION_FORMATTER_CLASS": "path.to.MyExceptionFormatter"}
```
