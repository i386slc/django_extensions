# Формат ответа об ошибке drf-se

Формат ответа об ошибке по умолчанию выглядит так

```json
{
  "type": "validation_error",
  "errors": [
    {
      "code": "required",
      "detail": "This field is required.",
      "attr": "name"
    }
  ]
}
```

* **type** - может быть **validation\_error**, **client\_error** или **server\_error**
* **code** - короткая строка с описанием ошибки. Может использоваться потребителями API для настройки своего поведения
* **detail** - удобный текст, описывающий ошибку
* **attr** - устанавливается только в том случае, если тип ошибки — **validation\_error** и сопоставляется с именем поля сериализатора или параметрами **settings.NON\_FIELD\_ERRORS\_KEY**.

## Типы ошибок

### Ошибки валидации

Это вызвано возникновением ошибки `rest_framework.exceptions.ValidationError`. Это единственный тип ошибки, который может иметь более 1 ошибки. Список соответствующих кодов ошибок зависит от сериализатора и его полей.

### Ошибки клиента

Покрывает все ошибки 4xx, кроме ошибок проверки. Вот ссылка на все возможные коды ошибок и соответствующие исключения:

| Код ошибки               | Код статуса | Исключение DRF                                                                                           |
| ------------------------ | ----------- | -------------------------------------------------------------------------------------------------------- |
| parse\_error             | 400         | [ParseError](https://www.django-rest-framework.org/api-guide/exceptions/#parseerror)                     |
| authentication\_failed   | 401         | [AuthenticationFailed](https://www.django-rest-framework.org/api-guide/exceptions/#authenticationfailed) |
| not\_authenticated       | 401         | [NotAuthenticated](https://www.django-rest-framework.org/api-guide/exceptions/#notauthenticated)         |
| permission\_denied       | 403         | [PermissionDenied](https://www.django-rest-framework.org/api-guide/exceptions/#permissiondenied)         |
| not\_found               | 404         | [NotFound](https://www.django-rest-framework.org/api-guide/exceptions/#notfound)                         |
| method\_not\_allowed     | 405         | [MethodNotAllowed](https://www.django-rest-framework.org/api-guide/exceptions/#methodnotallowed)         |
| not\_acceptable          | 406         | [NotAcceptable](https://www.django-rest-framework.org/api-guide/exceptions/#notacceptable)               |
| unsupported\_media\_type | 415         | [UnsupportedMediaType](https://www.django-rest-framework.org/api-guide/exceptions/#unsupportedmediatype) |
| throttled                | 429         | [Throttled](https://www.django-rest-framework.org/api-guide/exceptions/#throttled)                       |

### Ошибки сервера

Это вызвано возникновением исключения `rest_framework.exceptions.APIException` или необработанными исключениями. Соответствующий код ошибки — **error**, а код состояния — **500**.

## Поддержка нескольких ошибок

По умолчанию только ошибки проверки приводят к ответу об ошибке с несколькими ошибками. Ошибки могут быть для одного и того же поля или для разных полей. Итак, пример ошибки DRF выглядит следующим образом:

```json
{
    "phone": [
        ErrorDetail("The phone number entered is not valid.", code="invalid_phone_number")
    ],
    "password": [
        ErrorDetail("This password is too short.", code="password_too_short"),
        ErrorDetail("The password is too similar to the username.", code="password_too_similar"),
    ],
}
```

будет преобразовано в:

```json
{
    "type": "validation_error",
    "errors": [
        {
            "code": "invalid_phone_number",
            "detail": "The phone number entered is not valid.",
            "attr": "phone"
        },
        {
            "code": "password_too_short",
            "detail": "This password is too short.",
            "attr": "password"
        },
        {
            "code": "password_too_similar",
            "detail": "The password is too similar to the username.",
            "attr": "password"
        }
    ]
}
```

## Поддержка вложенных сериализаторов

Взяв этот пример

```json
{
    "shipping_address": {
        "non_field_errors": [
            ErrorDetail(
                "We do not support shipping to the provided address.",
                code="unsupported"
            )
        ]
    }
}
```

Он будет преобразован в:

```json
{
    "code": "unsupported",
    "detail": "We do not support shipping to the provided address.",
    "attr": "shipping_address.non_field_errors"
}
```

Обратите внимание, что **attr** представляет собой комбинированное значение имени поля родительского сериализатора и вложенного поля. Они разделены `'.'` по умолчанию, но это можно изменить с помощью параметра **NESTED\_FIELD\_SEPARATOR**.

## Список поддерживаемых сериализаторов

Этот пример

```json
{
    "recipients": [
        {"name": [ErrorDetail("This field is required.", code="required")]},
        {"email": [ErrorDetail("Enter a valid email address.", code="invalid")]},
    ]
}
```

был бы преобразован в

```json
{
    "type": "validation_error",
    "errors": [
        {
            "code": "required",
            "detail": "This field is required.",
            "attr": "recipients.0.name"
        },
        {
            "code": "invalid",
            "detail": "Enter a valid email address.",
            "attr": "recipients.1.email"
        }
    ]
}
```

Обратите внимание, что различение ошибок в разных объектах в сериализаторе вложенных списков выполняется с помощью индексации на основе 0.
