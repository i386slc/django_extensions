# django-spectacular

Разумная и гибкая генерация схемы OpenAPI 3.0 для среды Django REST.

У этого проекта 3 цели:

1. Извлеките как можно больше информации о схеме из DRF.
2. Обеспечьте гибкость, чтобы сделать схему пригодной для использования в реальном мире (не только в игрушечных примерах).
3. Создайте схему, которая хорошо работает с наиболее популярными генераторами клиентов.

Код представляет собой сильно модифицированный форк генератора DRF OpenAPI, в котором отсутствуют все перечисленные ниже функции.

Функции:

* Сериализаторы моделируются как компоненты (поддерживается произвольная вложенность и рекурсия)
* Декоратор @extend\_schema для настройки APIView, наборов представлений Viewsets, представлений на основе функций и **@action**&#x20;
  * дополнительные параметры
  * переопределение сериализатора запроса/ответа (с кодами состояния)
  * полиморфные ответы либо вручную с помощью помощника **PolymorphicProxySerializer**, либо с помощью **rest\_polymorphic** из **PolymorphicSerializer**)
  * … и другие возможности настройки
* Поддержка аутентификации (включая встроенные функции DRF, легко расширяемые)
* Поддержка пользовательского класса сериализатора (легко расширяемая)
* Тип `SerializerMethodField()` через подсказку типа (hint) или **@extend\_schema\_field**&#x20;
* поддержка i18n
* Извлечение тегов
* Примеры запросов/ответов/параметров
* Извлечение описания из строк документации
* Расширения спецификации поставщика (`x-*`) в информации, операциях, параметрах, компонентах и схемах безопасности
* Разумные запасные варианты
* Разумное именование **operation\_id** (на основе пути)
* Обслуживание схемы с помощью **SpectacularAPIView** (также доступны представления Redoc и Swagger-UI)
* Дополнительный компонент сериализатора ввода/вывода
* Обратные вызовы
* Включена поддержка:
  * [django-polymorphic](https://github.com/django-polymorphic/django-polymorphic) / [django-rest-polymorphic](https://github.com/apirobot/django-rest-polymorphic)
  * [SimpleJWT](https://github.com/jazzband/djangorestframework-simplejwt)
  * [DjangoOAuthToolkit](https://github.com/jazzband/django-oauth-toolkit)
  * [djangorestframework-jwt](https://github.com/jpadilla/django-rest-framework-jwt) (протестированный форк [drf-jwt](https://github.com/Styria-Digital/django-rest-framework-jwt))
  * [dj-rest-auth](https://github.com/iMerica/dj-rest-auth) (поддерживается форк [django-rest-auth](https://github.com/Tivix/django-rest-auth))
  * [djangorestframework-camel-case](https://github.com/vbabiy/djangorestframework-camel-case) (через хук постобработки **camelize\_serializer\_fields**)
  * [django-filter](https://github.com/carltongibson/django-filter)
  * [drf-nested-routers](https://github.com/alanjds/drf-nested-routers)
  * [djangorestframework-recursive](https://github.com/heywbj/django-rest-framework-recursive)
  * [djangorestframework-dataclasses](https://github.com/oxan/djangorestframework-dataclasses)
  * [django-rest-framework-gis](https://github.com/openwisp/django-rest-framework-gis)

Для получения дополнительной информации посетите [документацию](https://drf-spectacular.readthedocs.io/).

## Лицензия

Предоставлено [T. Franzel](https://github.com/tfranzel). [Лицензия под 3-Clause BSD](https://github.com/tfranzel/drf-spectacular/blob/master/LICENSE).

## Требования

* Python >= 3.6
* Django (2.2, 3.2, 4.0, 4.1, 4.2)
* Django REST Framework (3.10.3, 3.11, 3.12, 3.13, 3.14)

## Установка

Установить с помощью **pip**…

```bash
$ pip install drf-spectacular
```

затем добавьте **drf-spectacular** к установленным приложениям в **settings.py**.

```python
INSTALLED_APPS = [
    # ALL YOUR APPS
    'drf_spectacular',
]
```

и, наконец, зарегистрируйте нашу впечатляющую **AutoSchema** в DRF.

```python
REST_FRAMEWORK = {
    # YOUR SETTINGS
    'DEFAULT_SCHEMA_CLASS': 'drf_spectacular.openapi.AutoSchema',
}
```

**drf-spectacular** поставляется с разумными [настройками по умолчанию](https://drf-spectacular.readthedocs.io/en/latest/settings.html), которые должны работать достаточно хорошо из коробки. Никаких настроек указывать не обязательно, но рекомендуем указать хотя бы некоторые метаданные.

```python
SPECTACULAR_SETTINGS = {
    'TITLE': 'Your Project API',
    'DESCRIPTION': 'Your project description',
    'VERSION': '1.0.0',
    'SERVE_INCLUDE_SCHEMA': False,
    # OTHER SETTINGS
}
```

### Автономная установка пользовательского интерфейса

Некоторые среды не имеют прямого доступа к Интернету и поэтому не могут получить пользовательский интерфейс Swagger UI или Redoc из CDN. [drf-spectacular-sidecar](https://github.com/tfranzel/drf-spectacular-sidecar) предоставляет эти статические файлы в виде отдельного дополнительного пакета. Использование выглядит следующим образом:

```bash
$ pip install drf-spectacular[sidecar]
```

```python
INSTALLED_APPS = [
    # ALL YOUR APPS
    'drf_spectacular',
    'drf_spectacular_sidecar',  # требуется для обнаружения collectstatic Django
]
SPECTACULAR_SETTINGS = {
    'SWAGGER_UI_DIST': 'SIDECAR',  # сокращение для использования sidecar вместо этого
    'SWAGGER_UI_FAVICON_HREF': 'SIDECAR',
    'REDOC_DIST': 'SIDECAR',
    # OTHER SETTINGS
}
```

### Управление выпуском

**drf-spectacular** преднамеренно остается ниже версии 1.x.x, чтобы сигнализировать о том, что каждая новая версия потенциально может сломать вас. Для производства мы настоятельно рекомендуем закрепить версию и проверить различия схемы при обновлении.

С учетом сказанного, мы стремимся быть чрезвычайно оборонительными по отношению к нарушениям изменений API. Тем не менее, мы также признаем тот факт, что даже небольшие изменения схемы могут нарушить работу вашей цепочки инструментов, поскольку любая существующая ошибка может каким-то образом также использоваться в качестве функции.

Мы определяем приращения версий со следующей семантикой. Инкременты y-потока могут содержать потенциально критические изменения как в API, так и в схеме. Инкременты z-потока никогда не нарушат работу API и могут содержать только изменения схемы, вероятность которых сломать вас невелика.

## Попробуйте это

Создайте свою схему с помощью CLI:

```bash
$ ./manage.py spectacular --color --file schema.yml
$ docker run -p 80:8080 -e SWAGGER_JSON=/schema.yml -v ${PWD}/schema.yml:/schema.yml swaggerapi/swagger-ui
```

Если вы также хотите проверить свою схему, добавьте флаг **--validate**. Или обслуживайте свою схему непосредственно из вашего API. Мы также предоставляем удобные оболочки для **swagger-ui** или **redoc**.

```python
from drf_spectacular.views import (
    SpectacularAPIView, SpectacularRedocView, SpectacularSwaggerView
)

urlpatterns = [
    # YOUR PATTERNS
    path('api/schema/', SpectacularAPIView.as_view(), name='schema'),
    # опциональные UI:
    path(
        'api/schema/swagger-ui/',
        SpectacularSwaggerView.as_view(url_name='schema'), name='swagger-ui'
    ),
    path(
        'api/schema/redoc/',
        SpectacularRedocView.as_view(url_name='schema'), name='redoc'
    ),
]
```

## Применение

**drf-spectacular** работает довольно хорошо из коробки. Вы также можете установить некоторые метаданные для своего API. Просто создайте словарь **SPECTACULAR\_SETTINGS** в файле **settings.py** и переопределите значения по умолчанию. Посмотрите [доступные настройки](https://drf-spectacular.readthedocs.io/en/latest/settings.html).

Игрушечные примеры не охватывают ваши случаи? Нет проблем, вы можете сильно настроить способ отображения вашей схемы.

### Настройка с помощью @extend\_schema

Большинство случаев настройки должны быть покрыты декоратором **extend\_schema**. Обычно мы довольно далеко заходим с указанием **OpenApiParameter** и разделением сериализаторов запроса/ответа, но предела нет.

```python
from drf_spectacular.utils import (
    extend_schema, OpenApiParameter, OpenApiExample
)
from drf_spectacular.types import OpenApiTypes

class AlbumViewset(viewset.ModelViewset):
    serializer_class = AlbumSerializer

    @extend_schema(
        request=AlbumCreationSerializer,
        responses={201: AlbumSerializer},
    )
    def create(self, request):
        # ваше нестандартное поведение
        return super().create(request)

    @extend_schema(
        # в схему добавлены дополнительные параметры
        parameters=[
            OpenApiParameter(
                name='artist', description='Filter by artist',
                required=False, type=str
            ),
            OpenApiParameter(
                name='release',
                type=OpenApiTypes.DATE,
                location=OpenApiParameter.QUERY,
                description='Filter by release date',
                examples=[
                    OpenApiExample(
                        'Example 1',
                        summary='short optional summary',
                        description='longer description',
                        value='1993-08-23'
                    ),
                    ...
                ],
            ),
        ],
        # переопределить извлечение строки документации по умолчанию
        description='More descriptive text',
        # предоставить класс Authentication, который отличается от представлений
        # по умолчанию
        auth=None,
        # изменить автоматически сгенерированное имя операции
        operation_id=None,
        # или даже полностью переопределить то, что будет генерировать AutoSchema.
        # Предоставьте необработанную спецификацию Open API как Dict.
        operation=None,
        # прикрепите примеры запроса/ответа к операции.
        examples=[
            OpenApiExample(
                'Example 1',
                description='longer description',
                value=...
            ),
            ...
        ],
    )
    def list(self, request):
        # ваше нестандартное поведение
        return super().list(request)

    @extend_schema(
        request=AlbumLikeSerializer,
        responses={204: None},
        methods=["POST"]
    )
    @extend_schema(description='Override a specific method', methods=["GET"])
    @action(detail=True, methods=['post', 'get'])
    def set_password(self, request, pk=None):
        # ваше поведение действия
        ...
```

### Больше настроек

Все еще не удовлетворены? Ты хочешь больше! Мы все еще вас прикрыли. Посетите страницу [настройки](https://drf-spectacular.readthedocs.io/en/latest/customization.html) для получения дополнительной информации.

## Тестирование

Установите требования к тестированию.

```bash
$ pip install -r requirements.txt
```

Запустите с помощью runtests.

```bash
$ ./runtests.py
```

Вы также можете использовать отличный инструмент для тестирования tox для запуска тестов со всеми поддерживаемыми версиями Python и Django. Установите tox глобально, а затем просто запустите:

```bash
$ tox
```
