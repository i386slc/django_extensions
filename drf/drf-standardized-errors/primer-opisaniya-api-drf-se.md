# Пример описания API drf-se

Ознакомьтесь с [советами и рекомендациями](integraciya-s-drf-spectacular.md#sekrety-i-ulovki), чтобы понять, как использовать этот образец описания API.

## Обзор

Здесь вы, вероятно, сделаете обзор API и добавите для него описание.

## Аутентификация

Поскольку API требует проверки подлинности, вы можете также добавить раздел, описывающий процесс проверки подлинности.

## Ошибки

Теперь это важный раздел в этом примере. В этом разделе вы можете перечислить ответы об ошибках, которые появляются в каждой операции, с некоторыми пояснениями. Это может выглядеть так:

### 401 Unauthorized

Эти ошибки возвращаются с кодом состояния **401** всякий раз, когда аутентификация завершается сбоем или делается запрос к конечной точке без предоставления информации аутентификации как части запроса. Вот 2 возможные ошибки, которые могут быть возвращены.

```json
{
    "type": "client_error",
    "errors": [
        {
            "code": "authentication_failed",
            "detail": "Incorrect authentication credentials.",
            "attr": null
        }
    ]
}
```

```json
{
    "type": "client_error",
    "errors": [
        {
            "code": "not_authenticated",
            "detail": "Authentication credentials were not provided.",
            "attr": null
        }
    ]
}
```

### 405 Method Not Allowed

Это возвращается, когда конечная точка вызывается с неожиданным методом **http**. Например, если для обновления пользователя требуется запрос **POST**, а вместо этого выдается **PATCH**, возвращается эта ошибка. Вот как это выглядит:

```json
{
    "type": "client_error",
    "errors": [
        {
            "code": "method_not_allowed",
            "detail": "Method “patch” not allowed.",
            "attr": null
        }
    ]
}
```

### 406 Not Acceptable

Это возвращается, если заголовок **Accept** отправлен и содержит значение, отличное от `application/json`. Вот как будет выглядеть ответ:

```json
{
    "type": "client_error",
    "errors": [
        {
            "code": "not_acceptable",
            "detail": "Could not satisfy the request Accept header.",
            "attr": null
        }
    ]
}
```

### 415 Unsupported Media Type

Это возвращается, если тип содержимого запроса не **json**. Вот как будет выглядеть ответ:

```json
{
    "type": "client_error",
    "errors": [
        {
            "code": "not_acceptable",
            "detail": "Unsupported media type “application/xml” in request.",
            "attr": null
        }
    ]
}
```

### 500 Internal Server Error

Это возвращается, когда сервер API обнаруживает непредвиденную ошибку. Вот как будет выглядеть ответ:

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
