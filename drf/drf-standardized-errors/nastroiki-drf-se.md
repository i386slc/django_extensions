# Настройки drf-se

Вот все доступные настройки со значениями по умолчанию, вы можете переопределить их в настройках вашего проекта.

```python
DRF_STANDARDIZED_ERRORS = {
    # класс, отвечающий за обработку исключений. Может быть подклассом,
    # чтобы изменить, какие исключения обрабатываются по умолчанию,
    # чтобы обновить, какие исключения сообщаются инструментам мониторинга ошибок
    # (например, Sentry), ...
    "EXCEPTION_HANDLER_CLASS": "drf_standardized_errors.handler.ExceptionHandler",
    # класс, отвечающий за генерацию вывода ответа об ошибке.
    # Может быть подклассом для изменения формата ответа об ошибке.
    "EXCEPTION_FORMATTER_CLASS": "drf_standardized_errors.formatter.ExceptionFormatter",
    # включает стандартизированные ошибки, когда DEBUG=True для необработанных исключений.
    # По умолчанию установлено значение False, поэтому вы можете просмотреть
    # трассировку в терминале и получить дополнительную информацию об исключении.
    "ENABLE_IN_DEBUG_FOR_UNHANDLED_EXCEPTIONS": False,
    # Когда во вложенном сериализаторе возникает ошибка проверки, ключ "attr"
    # ответа об ошибке будет выглядеть так:
    # {field}{NESTED_FIELD_SEPARATOR}{nested_field}
    # например: 'shipping_address.zipcode'
    "NESTED_FIELD_SEPARATOR": ".",

    # Приведенные ниже настройки предназначены для генерации схемы OpenAPI 3.

    # ТОЛЬКО ответы, соответствующие этим кодам состояния,
    # будут отображаться в схеме API.
    "ALLOWED_ERROR_STATUS_CODES": [
        "400",
        "401",
        "403",
        "404",
        "405",
        "406",
        "415",
        "429",
        "500",
    ],

    # Сопоставление, используемое для переопределения сериализаторов по умолчанию,
    # используемых для описания ответа на ошибку. Ключ — это код состояния,
    # а значение — строка, представляющая путь к классу сериализатора,
    # описывающему реакцию на ошибку.
    "ERROR_SCHEMAS": None,

    # Когда в сериализаторах списков возникает ошибка проверки, возвращаемый
    # атрибут «attr» будет выглядеть примерно так: "0.email", "1.email", "2.email", ...
    # Таким образом, для описания кодов ошибок, связанных с одним и тем же полем
    # в сериализаторе списка, поле появится в схеме с именем "INDEX.email"
    "LIST_INDEX_IN_API_SCHEMA": "INDEX",

    # Когда в DictField возникает ошибка проверки с именем «extra_data»,
    # возвращаемый «attr» будет похож на sth "extra_data.<key1>", "extra_data.<key2>",
    # "extra_data.<key3>", ... Поскольку ключи DictField заранее не определены,
    # этот параметр используется в качестве общего имени для использования в схеме API.
    # Таким образом, соответствующее значение «attr» для предыдущего примера будет
    # "extra_data.KEY"
    "DICT_KEY_IN_API_SCHEMA": "KEY",

    # должен быть уникальным для компонентов ошибок, так как он используется
    # для идентификации компонентов ошибок, сгенерированных динамически,
    # чтобы исключить их обработку хуком постобработки. Это позволяет избежать
    # появления предупреждений для «code» и «attr», которые могут иметь
    # одинаковый выбор в нескольких сериализаторах.
    "ERROR_COMPONENT_NAME_SUFFIX": "ErrorComponent",
}
```