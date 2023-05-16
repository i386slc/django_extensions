# Поиск ошибок drf-se

## Написание тестов

### TL;DR

Если:

* вы настроили обработчик исключений и,
* представление вызывает исключение, которое вызывает код состояния 5xx и,
* вы пишете тест, который гарантирует, что представление вернет правильный ответ при возникновении исключения

Затем обязательно передайте `raise_request_exception=False`, иначе тест будет продолжать давать сбой. `raise_request_exception=False` [позволяет вернуть ответ 500 вместо возбуждения исключения](https://docs.djangoproject.com/en/stable/topics/testing/tools/#exceptions).

### Длинная версия

Я столкнулся с этим при написании теста для этого пакета, поэтому я хотел поделиться им, если кто-то еще наткнется на него. Я тестировал пользовательский форматировщик исключений, чтобы убедиться, что он используется, если он установлен в настройках, и что формат ответа об ошибке соответствует моим ожиданиям. Итак, вот тест

```python
# views.py
from rest_framework.views import APIView

class ErrorView(APIView):
    def get(self, request, *args, **kwargs):
        raise Exception("Internal server error.")
```

```python
# urls.py
from django.urls import path

from .views import ErrorView

urlpatterns = [
    path("error/", ErrorView.as_view()),
]
```

```python
# tests.py
import pytest
from rest_framework.test import APIClient

from drf_standardized_errors.formatter import ExceptionFormatter
from drf_standardized_errors.types import ErrorResponse


@pytest.fixture
def api_client():
    return APIClient()


def test_custom_exception_formatter_class(settings, api_client):
    settings.DRF_STANDARDIZED_ERRORS = {
        "EXCEPTION_FORMATTER_CLASS": "tests.CustomExceptionFormatter"
    }
    response = api_client.get("/error/")
    assert response.status_code == 500
    assert response.data["type"] == "server_error"
    assert response.data["code"] == "error"
    assert response.data["message"] == "Internal server error."
    assert response.data["field_name"] is None


class CustomExceptionFormatter(ExceptionFormatter):
    def format_error_response(self, error_response: ErrorResponse):
        """возвращает по одной ошибке за раз и изменяет имена ключей ответа на ошибку"""
        error = error_response.errors[0]
        return {
            "type": error_response.type,
            "code": error.code,
            "message": error.detail,
            "field_name": error.attr,
        }
```

Этот тест продолжал давать сбои и отображал трассировку, включая `raise Exception(«Internal server error»)`. Мне казалось, что обработчик исключений не выполняет свою работу.

Запустив тест в режиме отладки, я смог увидеть, что ответ, возвращаемый представлением, действительно соответствует моим ожиданиям, однако тест по-прежнему терпит неудачу.

Снова взглянув на тестовую трассировку и прочитав [соответствующий код в тестовом клиенте django](https://github.com/django/django/blob/0b31e024873681e187b574fe1c4afe5e48aeeecf/django/test/client.py#L803-L810), я понял, что происходит: тестовый клиент определяет получателя для сигнала **got\_request\_exception**, и если этот сигнал отправлен, он делает вывод, что возникла проблема, и вызывает исключение. В моем тесте я вызывал `Exception("Internal server error")`, которое считается ошибкой сервера, поэтому сигнал отправляется обработчиком исключений, и django не проходит тест, поскольку получает сигнал.

Что касается того, почему сигнал отправляется обработчиком исключений в первую очередь, это потому, что инструменты мониторинга ошибок (такие как **Sentry**) полагаются на него для сбора информации об исключении и предоставления ее через свой пользовательский интерфейс. Кроме того, как было обнаружено во время отладки этой проблемы, тестовый клиент django нуждается в нем, чтобы определить, вызвало ли рассматриваемое представление исключение или нет, и уведомить разработчика.

Приятно то, что [тестовый клиент django позволяет получить ответ, не вызывая исключения](https://docs.djangoproject.com/en/stable/topics/testing/tools/#exceptions). Это возможно, если передать `raise_request_exception=False` при создании экземпляра тестового клиента.
