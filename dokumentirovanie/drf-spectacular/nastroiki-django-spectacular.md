# Настройки django-spectacular

Настройки устанавливаются в **settings.py** в области **SPECTACULAR\_SETTINGS**. Вы можете переопределить любой параметр, в противном случае используются значения по умолчанию, указанные ниже.

```python
SPECTACULAR_DEFAULTS: Dict[str, Any] = {
    # Регулярное выражение, указывающее общий знаменатель для всех путей операций.
    # Если для SCHEMA_PATH_PREFIX установлено значение None, drf-spectacular
    # попытается оценить общий префикс. Используйте '' для отключения.
    # В основном используется для извлечения тегов, где такие пути, как
    # '/api/v1/albums' с регуляркой SCHEMA_PATH_PREFIX '/api/v[0-9]'
    # будут возвращать тег 'albums'.
    'SCHEMA_PATH_PREFIX': None,
    # Удалите соответствующий SCHEMA_PATH_PREFIX из рабочего пути.
    # Обычно используется в сочетании с добавленными префиксами в SERVERS.
    'SCHEMA_PATH_PREFIX_TRIM': False,
    # Вставьте префикс пути вручную к рабочему пути, например, '/service/backend'.
    # Используйте это, например, для выравнивания путей, когда API монтируется
    # как подресурс за прокси-сервером, а Django не знает об этом. Кроме того,
    # префиксы также могут быть указаны через SERVERS, но это делает путь операции
    # более явным.
    'SCHEMA_PATH_PREFIX_INSERT': '',
    # Принуждение определения {pk} как {id} контролируется SCHEMA_COERCE_PATH_PK.
    # Кроме того, некоторые библиотеки (например, drf-nested-routers) используют
    # переменные пути с суффиксом "_pk".
    # Этот параметр глобально принуждает использовать переменные пути,
    # такие как "{user_pk}" как "{user_id}".
    'SCHEMA_COERCE_PATH_PK_SUFFIX': False,

    # Параметры генерации схемы, влияющие на то, как создаются компоненты.
    # Некоторые функции схемы могут не соответствовать вашей цели.
    # Демультиплексирование/изменение компонентов может помочь решить эти проблемы.
    'DEFAULT_GENERATOR_CLASS': 'drf_spectacular.generators.SchemaGenerator',
    # Создайте отдельные компоненты для конечных точек PATCH (без обязательного списка)
    'COMPONENT_SPLIT_PATCH': True,
    # Разделите компоненты на части запроса и ответа, где это необходимо
    # Этот параметр настоятельно рекомендуется для достижения наиболее точного
    # описания API, однако это достигается за счет большего количества компонентов.
    'COMPONENT_SPLIT_REQUEST': False,
    # Помогите целевым объектам генератора клиентов, у которых есть проблемы
    # со свойствами, доступными только для чтения.
    'COMPONENT_NO_READ_ONLY_REQUIRED': False,

    # Добавляет «minLength: 1» к полям, которые не допускают пустых строк.
    # Деактивировано по умолчанию, потому что сериализаторы строго не применяют
    # это к ответам, поэтому «minLength: 1» не всегда точно описывает поведение API.
    # Неявно включается COMPONENT_SPLIT_REQUEST, поскольку это можно точно
    # смоделировать, когда компоненты запроса и ответа разделены.
    'ENFORCE_NON_BLANK_FIELDS': False,

    # Конфигурация для обслуживания подмножества схемы с помощью SpectacularAPIView
    'SERVE_URLCONF': None,
    # полная общедоступная схема или подмножество на основе запрашивающего пользователя
    'SERVE_PUBLIC': True,
    # включить конечную точку схемы в схеме
    'SERVE_INCLUDE_SCHEMA': True,
    # список классов аутентификации/разрешения для представлений view.
    'SERVE_PERMISSIONS': ['rest_framework.permissions.AllowAny'],
    # None по умолчанию будет для DRF's AUTHENTICATION_CLASSES
    'SERVE_AUTHENTICATION': None,

    # Словарь общей конфигурации, чтобы перейти к SwaggerUI({ ... })
    # https://swagger.io/docs/open-source-tools/swagger-ui/usage/configuration/
    # Настройки сериализуются с помощью json.dumps(). Если вам нужен настраиваемый JS,
    # используйте вместо этого строку. Затем строка должна содержать действительный JS
    # и передается без изменений.
    'SWAGGER_UI_SETTINGS': {
        'deepLinking': True,
    },
    # Инициализируйте SwaggerUI с дополнительной конфигурацией OAuth2.
    # https://swagger.io/docs/open-source-tools/swagger-ui/usage/oauth2/
    'SWAGGER_UI_OAUTH2_CONFIG': {},

    # Словарь общей конфигурации, чтобы перейти к Redoc.init({ ... })
    # https://github.com/Redocly/redoc#redoc-options-object
    # Настройки сериализуются с помощью json.dumps(). Если вам нужен настраиваемый JS,
    # используйте вместо этого строку. Затем строка должна содержать действительный JS
    # и передается без изменений.
    'REDOC_UI_SETTINGS': {},

    # CDN для swagger и redoc. Вы можете изменить версию или даже разместить
    # свою собственную в зависимости от ваших требований. Для самостоятельного
    # размещения посмотрите опцию sidecar в README.
    'SWAGGER_UI_DIST': 'https://cdn.jsdelivr.net/npm/swagger-ui-dist@latest',
    'SWAGGER_UI_FAVICON_HREF': 'https://cdn.jsdelivr.net/npm/swagger-ui-dist@latest/favicon-32x32.png',
    'REDOC_DIST': 'https://cdn.jsdelivr.net/npm/redoc@latest',

    # Добавляйте объекты OpenAPI к пути и компонентам в дополнение
    # к сгенерированным объектам.
    'APPEND_PATHS': {},
    'APPEND_COMPONENTS': {},

    # DISCOURAGED - пожалуйста, не используйте это больше, так как это имеет
    # каверзные последствия, которые трудно понять правильно. Для аутентификации
    # настоятельно рекомендуется использовать OpenApiAuthenticationExtension,
    # поскольку они более надежны и просты в написании.
    # Однако, если они используются, список методов добавляется к каждой конечной точке
    # в схеме!
    'SECURITY': [],

    # Функции постобработки, которые запускаются в конце генерации схемы.
    # должен удовлетворять интерфейсу result = hook(generator, request, public, result)
    'POSTPROCESSING_HOOKS': [
        'drf_spectacular.hooks.postprocess_schema_enums'
    ],

    # Функции предварительной обработки, которые запускаются перед созданием схемы.
    # должен удовлетворять интерфейсу result = hook(endpoints=result) где result -
    # это список Tuples (path, path_regex, method, callback).
    # Пример: 'drf_spectacular.hooks.preprocess_exclude_path_format'
    'PREPROCESSING_HOOKS': [],

    # Определяет, как должны быть отсортированы операции. Если вы собираетесь выполнять
    # сортировку с помощью PREPROCESSING_HOOKS, обязательно отключите этот параметр.
    # Если настроено, сортировка применяется после PREPROCESSING_HOOKS.
    # Принимает либо True (drf-spectacular's alpha-sorter), False, или вызываемый объект
    # для аргумента ключа сортировки.
    'SORT_OPERATIONS': True,

    # переопределяет имя перечисления. Словарь с ключами "YourEnum" и их значениями
    # выбора "field.choices"
    # например, {'SomeEnum': ['A', 'B'], 'OtherEnum': 'import.path.to.choices'}
    'ENUM_NAME_OVERRIDES': {},
    # Добавляет "blank" и "null" выбора перечисления, где это уместно.
    # Отключить при проблемах с генерацией клиентов
    'ENUM_ADD_EXPLICIT_BLANK_NULL_CHOICE': True,
    # Добавляет/вставляет список (``choice value`` - имя выбора) в строку
    # описания перечисления.
    'ENUM_GENERATE_CHOICE_DESCRIPTION': True,

    # функция, которая возвращает список всех классов, которые следует исключить
    # из извлечения строки документа
    'GET_LIB_DOC_EXCLUDES': 'drf_spectacular.plumbing.get_lib_doc_excludes',

    # Функция, которая возвращает имитированный запрос на обработку представления.
    # Для использования с CLI original_request будет None.
    # интерфейс: request = build_mock_request(method, path, view, original_request, **kwargs)
    'GET_MOCK_REQUEST': 'drf_spectacular.plumbing.build_mock_request',

    # Camelize имена, такие как «operationId» и имена параметров пути Camelize
    # самой схемы операции требует добавления
    # 'drf_spectacular.contrib.djangorestframework_camel_case.camelize_serializer_fields'
    # к POSTPROCESSING_HOOKS. Обратите внимание, что хук зависит от
    # ``djangorestframework_camel_case``, в то время как сама не CAMELIZE_NAMES.
    'CAMELIZE_NAMES': False,

    # Определяет, следует ли создавать в схеме 'additionalProperties' произвольной
    # формы и каким образом. Некоторые цели генератора кода чувствительны к этому.
    # None отключает общие 'additionalProperties'.
    # допустимые значения 'dict', 'bool', None
    'GENERIC_ADDITIONAL_PROPERTIES': 'dict',

    # Переопределение схемы преобразователя пути (например, <int:foo>).
    # Может использоваться либо для изменения поведения по умолчанию,
    # либо для предоставления схемы для пользовательских преобразователей,
    # зарегистрированных с помощью register_converter(...).
    # Принимает метки преобразователя в качестве ключей и либо базовые типы Python,
    # OpenApiType, либо необработанные схемы в качестве значений.
    # Пример: {'aint': OpenApiTypes.INT, 'bint': str, 'cint': {'type': ...}}
    'PATH_CONVERTER_OVERRIDES': {},

    # Определяет, должны ли параметры операции сортироваться в алфавитно-цифровом
    # порядке или только в том порядке, в котором они были получены.
    # Принимает либо True, False, либо вызываемый аргумент для ключа сортировки.
    'SORT_OPERATION_PARAMETERS': True,

    # @extend_schema позволяет указывать коды состояния помимо 200.
    # Эта функциональность обычно используется для описания ответов на ошибки,
    # которые редко используют механику списка. Поэтому мы по умолчанию отключаем
    # перечисление (разбивку на страницы и фильтрацию) для кодов состояния,
    # отличных от 2XX. Переключите это, чтобы включить ответы списка
    # с ListSerializers/many=True независимо от кода состояния.
    'ENABLE_LIST_MECHANICS_ON_NON_2XX': False,

    # Этот параметр позволяет вам отклониться от диспетчера по умолчанию, обратившись
    # к другому свойству модели. Мы используем 'objects' по умолчанию из соображений
    # совместимости. Использование "_default_manager", скорее всего, решит большинство
    # проблем, хотя вы можете выбрать любое имя.
    "DEFAULT_QUERY_MANAGER": 'objects',

    # Определяет, какие методы проверки подлинности предоставляются в схеме.
    # Если не None, будут скрыты классы проверки подлинности, не содержащиеся
    # в белом списке. Используйте полные пути импорта, например
    # ['rest_framework.authentication.TokenAuthentication', ...].
    # Пустой список ([]) будет скрывать все методы аутентификации.
    # По умолчанию None будет показывать все.
    'AUTHENTICATION_WHITELIST': None,
    # Управляет тем, какие синтаксические анализаторы предоставляются в схеме.
    # Работает аналогично AUTHENTICATION_WHITELIST.
    # Список разрешенных парсеров или None, чтобы разрешить все.
    'PARSER_WHITELIST': None,
    # Определяет, какие средства визуализации отображаются в схеме.
    # Работает аналогично AUTHENTICATION_WHITELIST.
    # rest_framework.renderers.BrowsableAPIRenderer игнорируется по умолчанию,
    # если белый список равен None
    'RENDERER_WHITELIST': None,

    # Возможность отключения сообщений об ошибках и предупреждений
    'DISABLE_ERRORS_AND_WARNINGS': False,

    # Запускает образцовую генерацию схемы и выдает предупреждения
    # как часть "./manage.py check --deploy"
    'ENABLE_DJANGO_DEPLOY_CHECK': True,

    # Общие метаданные схемы. Обратитесь к спецификации для допустимых входных данных
    # https://spec.openapis.org/oas/v3.0.3#openapi-object
    'TITLE': '',
    'DESCRIPTION': '',
    'TOS': None,
    # Необязательно: МОЖЕТ содержать "name", "url", "email"
    'CONTACT': {},
    # Необязательно: ДОЛЖЕН содержать "name", МОЖЕТ содержать URL
    'LICENSE': {},
    # Статически заданная версия схемы. Также может быть пустой строкой.
    # При использовании вместе с управлением версиями представления станет "0.0.0 (v2)"
    # для запросов с версией "v2".
    # Установите для VERSION значение None, если должна отображаться только версия запроса.
    'VERSION': '0.0.0',
    # Необязательный список серверов.
    # Каждая запись ДОЛЖНА содержать "url", МОЖЕТ содержать "description", "variables"
    # например, [{'url': 'https://example.com/v1', 'description': 'Text'}, ...]
    'SERVERS': [],
    # Теги, определенные в глобальной области
    'TAGS': [],
    # Необязательно: ДОЛЖЕН содержать 'url', может содержать "description"
    'EXTERNAL_DOCS': {},

    # Произвольные расширения спецификации, прикрепленные к информационному объекту схемы.
    # https://swagger.io/specification/#specification-extensions
    'EXTENSIONS_INFO': {},

    # Произвольные расширения спецификации, прикрепленные к корневому объекту схемы.
    # https://swagger.io/specification/#specification-extensions
    'EXTENSIONS_ROOT': {},

    # Настройки, связанные с Oauth2. Используется, например, django-oauth2-toolkit.
    # https://spec.openapis.org/oas/v3.0.3#oauthFlowsObject
    'OAUTH2_FLOWS': [],
    'OAUTH2_AUTHORIZATION_URL': None,
    'OAUTH2_TOKEN_URL': None,
    'OAUTH2_REFRESH_URL': None,
    'OAUTH2_SCOPES': None,
}
```
