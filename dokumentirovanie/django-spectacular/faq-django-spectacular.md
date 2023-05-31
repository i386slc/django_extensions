# FAQ django-spectacular

## Я использую библиотеку/приложение XXX, и сгенерированная схема неверна или повреждена

Иногда библиотеки DRF плохо взаимодействуют с механикой самоанализа. Проверьте [шаблоны расширения](skhemy-rasshirenii.md) на наличие уже доступных исправлений. Если их нет, узнайте, как легко [настроить рабочий процесс и схему](nastroika-rabochego-processa-i-skhemy.md). Не стесняйтесь вносить недостающие исправления.

Если вы считаете, что это ошибка в **drf-spectacular**, откройте [вопрос](https://github.com/tfranzel/drf-spectacular/issues).

## Мой пользовательский интерфейс Swagger и/или страница Redoc пусты

Скорее всего, вы используете [django-csp](https://django-csp.readthedocs.io/en/latest/index.html). Загляните в консоль браузера и убедитесь, что у вас есть ошибки **Content Security Policy**. По умолчанию **django-csp** обычно ломает наши пользовательские интерфейсы по двум причинам: внешние ассеты и встроенные скрипты.

Использование [sidecar](https://github.com/tfranzel/drf-spectacular#self-contained-ui-installation) смягчит нарушение загрузки удаленных ресурсов, предоставляя ресурсы от **self**. В качестве альтернативы вы также можете адаптировать **CSP\_DEFAULT\_SRC**, чтобы разрешить использование этих ресурсов CDN.

### Решение для пользовательского интерфейса Swagger UI:

```python
# Option: SIDECAR
SPECTACULAR_SETTINGS = {
     ...
    'SWAGGER_UI_DIST': 'SIDECAR',
    'SWAGGER_UI_FAVICON_HREF': 'SIDECAR',
}
CSP_DEFAULT_SRC = ("'self'", "'unsafe-inline'")
CSP_IMG_SRC = ("'self'", "data:")

# Option: CDN
CSP_DEFAULT_SRC = ("'self'", "'unsafe-inline'", "cdn.jsdelivr.net")
CSP_IMG_SRC = ("'self'", "data:", "cdn.jsdelivr.net")
```

{% hint style="info" %}
В зависимости от того, насколько вы параноики, вы можете избежать использования **unsafe-inline**, используя вместо этого **SpectacularSwaggerSplitView**, который выполняет отдельный запрос для сценария. Однако обратите внимание, что некоторые развертывания с перезаписью URL-адресов сломают его. Используйте эту опцию только в том случае, если вам это действительно необходимо.
{% endhint %}

### Решение для Redoc:

```python
# Option: SIDECAR
SPECTACULAR_SETTINGS = {
     ...
    'REDOC_DIST': 'SIDECAR',
}
# Option: CDN
CSP_DEFAULT_SRC = ("'self'", "cdn.jsdelivr.net")

# требуется для обоих CDN и SIDECAR
CSP_WORKER_SRC = ("'self'", "blob:")
CSP_IMG_SRC = ("'self'", "data:", "cdn.redoc.ly")
CSP_STYLE_SRC = ("'self'", "'unsafe-inline'", "fonts.googleapis.com")
CSP_FONT_SRC = ("'self'", "fonts.gstatic.com")
```

## Я не могу использовать @extend\_schema в коде библиотеки

Вы можете легко адаптировать самоанализ для библиотек/приложений с помощью механизма расширения. Расширения обеспечивают простой способ присоединения информации о схеме к коду, который вы не можете изменить иначе. Взгляните на [настройку рабочего процесса и схемы](nastroika-rabochego-processa-i-skhemy.md), чтобы узнать, как использовать расширения.

## Я получаю пустую схему или отсутствуют конечные точки

Обычно это происходит из-за версий (реже из-за разрешений).

Если вы используете управление версиями на всех конечных точках, это может быть ожидаемым результатом. По умолчанию схема будет содержать только неверсированные конечные точки. Явно укажите, какую версию вы хотите сгенерировать.

```bash
./manage.py spectacular --api-version 'YOUR_VERSION'
```

Она будет содержать конечные точки без версии вместе с конечными точками для указанной версии.

Для представлений схемы вы можете либо установить класс управления версиями (неявное управление версиями через запрос), либо явно переопределить версию с помощью `SpectacularAPIView.as_view(api_version='YOUR_VERSION')`.

## Я ожидал другую схему

Иногда представления объявляют одно (через **serializer\_class** и **queryset**), а делают совершенно другое. Обычно это объясняется тем, что код библиотеки становится гибким в различных ситуациях. В таких случаях лучше отменить то, что решил самоанализ, и прямо указать, чего следует ожидать. Выполните шаги в [настройке рабочего процесса и схемы](nastroika-rabochego-processa-i-skhemy.md), чтобы адаптировать свою схему.

## Я получаю повторяющиеся операции с суффиксом {format}

Ваше приложение, вероятно, использует **format\_suffix\_patterns** DRF. Если эти операции нежелательны в вашей схеме, вы можете просто исключить их с помощью уже предоставленного [хука предварительной обработки](nastroika-rabochego-processa-i-skhemy.md#shag-7-preprocessing-khukov).

## Я получаю много предупреждений

Предупреждения выдаются, чтобы информировать вас об обнаруженных проблемах со схемой. Некоторые шаблоны использования, такие как **@api\_view** или **APIView**, предоставляют очень мало информации о вашем API. В таких случаях вы можете легко дополнить эти конечные точки и сериализаторы дополнительной информацией. Посмотрите на параметры [настройки рабочего процесса и схемы](nastroika-rabochego-processa-i-skhemy.md), чтобы заполнить эти пробелы и убрать предупреждения.

## Я получаю предупреждения относительно моего Enum или у моих имен Enum есть странный суффикс

Это связано с тем, что по умолчанию активируется хук постобработки **Enum**, который пытается найти имя для набора вариантов перечисления.

Механизм именования использует имя поля и, возможно, имя компонента, за которым следует суффикс, если это необходимо, если есть конфликты (если есть два поля перечисления с одинаковым именем, но с разным набором вариантов). Это автоматически обработает все возникающие проблемы, а также уведомит вас о потенциальных проблемах двух видов:

* несколько имен создаются для одного и того же набора значений из-за разных имен полей (например, если у вас есть одно перечисление валюты, используемое разными полями с именами, такими как **payment\_currency** и **preferred\_currency**, механизм именования по умолчанию будет рассматривать это как два разных перечисления, но выдает предупреждение).
* конфликты, которые приводят к необходимости суффикса, как указано выше.

Вы можете решить (или отключить) проблемы с перечислением, добавив запись в параметр **ENUM\_NAME\_OVERRIDES**. Значения могут принимать форму вариантов (список кортежей), списков значений (список строк) или импортированных строк. Также поддерживаются классы Django **models.Choices** и Python **Enum**. Ключ — это строка, которую вы выбираете в качестве имени для этого набора значений.

Например:

```python
SPECTACULAR_SETTINGS = {
    ...
    'ENUM_NAME_OVERRIDES': {
        # переменная, содержащая список кортежей, например, [('US', 'US'), ('RU', 'RU'),]
        'LanguageEnum': language_choices,
        # настоящий Enum или класс models.Choices
        'CountryEnum': 'import_path.enums.CountryEnum',
        # choices — это атрибут класса CurrencyContainer, содержащий список кортежей.
        'CurrencyEnum': 'import_path.CurrencyContainer.choices',
    }
}
```

Если у вас есть несколько семантически различных перечислений с одинаковым набором значений, и вам нужны разные имена для них, этот механизм не будет работать.

## Мои конечные точки используют разные сериализаторы в зависимости от ситуации

Добро пожаловать в реальный мир! Используйте **@extend\_schema** в сочетании с **PolymorphicProxySerializer** следующим образом:

```python
class PersonView(viewsets.GenericViewSet):
    @extend_schema(responses={
        200: PolymorphicProxySerializer(
                component_name='Person',
                # на 200 возвращается либо юридическое, либо физическое лицо
                serializers=[LegalPersonSerializer, NaturalPersonSerializer],
                resource_type_field_name='type',
        ),
        500: YourOptionalErrorSerializer,
    })
    def retrieve(self, request, *args, **kwargs)
        pass
```

## Мой метод аутентификации не поддерживается

Вы можете легко указать пользовательскую аутентификацию с помощью **OpenApiAuthenticationExtension**. Взгляните на настройку рабочего процесса и схемы, чтобы узнать, как использовать расширения.

## Как я могу интернационализировать i18n свою схему и пользовательский интерфейс?

Вы можете использовать интернационализацию Django, как обычно. Рабочий процесс такой, как и следовало ожидать: `USE_I18N=True`, настраиваются языки, **makemessages** и **compilemessages**.

Инструмент CLI принимает параметр языка (`./manage.py эффектный --lang="de-de"`) для автономной генерации. Представление схемы, а также представления пользовательского интерфейса принимают параметр запроса **lang** для явного запроса языка (`example.com/api/schema?lang=de`). Если **i18n** включен и параметр запроса не указан, используется заголовок **ACCEPT\_LANGUAGE**. В противном случае перевод возвращается к языку по умолчанию.

```python
from django.utils.translation import gettext_lazy as _

class PersonView(viewsets.GenericViewSet):
    __doc__ = _("""
    More lengthy explanation of the view
    """)

    @extend_schema(summary=_('Main endpoint for creating person'))
    def retrieve(self, request, *args, **kwargs):
        pass
```

## FileField (ImageField) неправильно обрабатывается в схеме

В отличие от большинства других полей, **FileField** ведет себя по-разному для запросов и ответов. Эту двойственность невозможно представить в однокомпонентной схеме.

Для этих случаев есть возможность разделить компоненты на части запроса и ответа, установив `COMPONENT_SPLIT_REQUEST = True`. Обратите внимание, что это влияет на всю схему, а не только на компоненты с **FileFields**.

Также рассмотрите возможность явной установки `parser_classes = [parsers.MultiPartParser]` (или любого другого парсера, совместимого с файлами) в вашем представлении **View** или напишите собственный **get\_parser\_classes**. Эти поля не работают с **JsonParser** по умолчанию, и этот факт должен быть представлен в схеме.

## Я использую @action(detail=False), но схема ответа не является списком

`detail=True/False` указывает только, должно ли действие быть перенаправлено на `x/{id}/action` или `x/action`. Сам по себе параметр **detail** ничего не говорит об ответе действия. Также обратите внимание, что по умолчанию для недоопределенных конечных точек используется ответ без списка. Чтобы сообщить о перечисленном ответе, вы можете использовать `@extend_schema(responses=XSerializer(many=True))`.

## Использование @extend\_schema в APIView не имеет никакого эффекта

**@extend\_schema** необходимо применять к методу точки входа представления. Для представлений, производных от **Viewset**, это такие методы, как **retrieve**, **list**, **create**. Для представлений на основе **APIView** это **get**, **post**, **create**. Эта путаница обычно возникает при использовании удобных классов, таких как **ListAPIView**. **ListAPIView** на самом деле имеет метод **list** (через миксин), но фактическая точка входа по-прежнему является методом **get**, а вызов списка передается через точку входа.

## Где я должен разместить свои расширения? / мои расширения не обнаружены

Расширения регистрируются автоматически. Просто убедитесь, что интерпретатор Python видит их хотя бы один раз. С этой целью мы предлагаем создать файл `PROJECT/schema.py` и импортировать его в свой `PROJECT/__init__.py` (тот же каталог, что и **settings.py** и **urls.py**) с помощью `import PROJECT.schema`.

## Мой @action ошибочно разбит на страницы или имеет параметры фильтра, которые мне не нужны

Обычно это происходит при использовании `@extend_schema(responses=XSerializer(many=True))` . Действия наследуют фильтры и классы разбиения на страницы от их **ViewSet**. Если затем ответ помечается как список, срабатывает **pagination\_class**. Поскольку действия обрабатываются пользователем вручную, такое поведение обычно не сразу бросается в глаза. Чтобы сделать ваши намерения понятными для **drf-spectacular**, вам необходимо очистить оскорбительные классы в декораторе действий, например, установив `pagination_class = None`.

Пользователи **django-filter** также могут увидеть нежелательные параметры запроса. Поскольку здесь применима та же механика, вы можете удалить эти параметры, сбросив серверные части фильтра с помощью `@action(...,filter_backends=[])`.

```python
class XViewset(viewsets.ModelViewSet):
    queryset = SimpleModel.objects.all()
    pagination_class = pagination.LimitOffsetPagination

    @extend_schema(responses=SimpleSerializer(many=True))
    @action(methods=['GET'], detail=False, pagination_class=None)
    def custom_action(self):
        pass
```

## Как я могу обернуть свои ответы? / Мои конечные точки заключены в общий конверт

Это неродное поведение можно удобно смоделировать с помощью простой вспомогательной функции. Вам просто нужно обернуть реальный сериализатор вашим сериализатором оболочки и предоставить его **@extend\_schema**.

Вот пример того, как построить функцию оболочки **enveloper**. В этом примере фактический сериализатор помещается в поле **data**, а **status** — это произвольное поле конверта. Адаптируйте к вашим конкретным требованиям.

```python
def enveloper(serializer_class, many):
    component_name = 'Enveloped{}{}'.format(
        serializer_class.__name__.replace("Serializer", ""),
        "List" if many else "",
    )

    @extend_schema_serializer(many=False, component_name=component_name)
    class EnvelopeSerializer(serializers.Serializer):
        status = serializers.BooleanField()  # some arbitrary envelope field
        data = serializer_class(many=many)  # объемлющая часть

    return EnvelopeSerializer


class XViewset(GenericViewSet):
    @extend_schema(responses=enveloper(XSerializer, True))
    def list(self, request, *args, **kwargs):
        ...
```

## Как я могу иметь несколько SpectacularAPIView с разными настройками

Сначала определите свои базовые настройки в **settings.py** с помощью **SPECTACULAR\_SETTINGS**. Затем, если вам нужна другая схема с другими настройками, вы можете предоставить переопределение области действия, указав аргумент **custom\_settings**. **custom\_settings** ожидает словарь и разрешает только ключи, представляющие допустимые имена настроек.

Имейте в виду, что использование этой механики в настоящее время не является потокобезопасным.

Также обратите внимание, что переопределение **SERVE\_\*** или **DEFAULT\_GENERATOR\_CLASS** в **custom\_settings** не допускается. **SpectacularAPIView** имеет специальные аргументы для переопределения этих настроек.

```python
urlpatterns = [
    path('api/schema/', SpectacularAPIView.as_view(),
    path('api/schema-custom/', SpectacularAPIView.as_view(
        custom_settings={
            'TITLE': 'your custom title',
            'SCHEMA_PATH_PREFIX': 'your custom regex',
            ...
        }
    ), name='schema-custom'),
]
```

## Как правильно аннотировать представления на основе функций, которые используют @api\_view()

DRF предоставляет удобный способ написания представлений на основе функций. `@api_view()` по сути оборачивает обычную функцию и неявно преобразует ее в класс **APIView**. Для случаев с одним методом просто используйте **@extend\_schema** так же, как и с обычным методом представления.

```python
@extend_schema(request=XSerializer, responses=XSerializer)
@api_view(['POST'])
def view_func(request, format=None):
    return ...
```

Для функций, предоставляющих несколько методов, рекомендуется использовать **@extend\_schema\_view** и разбирать каждый случай отдельно.

```python
@extend_schema_view(
    get=extend_schema(description='get desc', responses=XSerializer),
    post=extend_schema(
        description='post desc', request=None, responses=OpenApiTypes.UUID
    ),
)
@api_view(['GET', 'POST'])
def view_func(request, format=None):
    return ...
```

## Мой get\_queryset() зависит от некоторых атрибутов, недоступных во время генерации схемы

В определенных ситуациях нам нужно вызвать **get\_serializer**, который, в свою очередь, вызывает **get\_queryset**. Если ваш **get\_queryset** (или **get\_serializer\_class**) зависит от атрибутов, недоступных во время генерации схемы (например, `request.user.is_authenticated`), вам необходимо предоставить запасной вариант, который позволит нам вызывать этот метод. Пока схема генерируется, вы можете проверить атрибут представления **swagger\_fake\_view** и просто вернуть пустой набор запросов правильной модели.

```python
class XViewset(viewsets.ModelViewset):
    ...

    def get_queryset(self):
        if getattr(self, 'swagger_fake_view', False):  # drf-yasg comp
            return YourModel.objects.none()
        # твоя обычная логика
```

## Как обслуживать сгенерированные в памяти файлы или файлы вообще за пределами FileField

DRF предоставляет удобный **FileField** для постоянного хранения файлов в модели **Model**. **drf-spectacular** корректно обрабатывает их по умолчанию. Но чтобы обслуживать двоичные файлы, созданные в памяти, следуйте следующему рецепту. В этом примере используется метод, [рекомендованный Django](https://docs.djangoproject.com/en/4.0/ref/request-response/#telling-the-browser-to-treat-the-response-as-a-file-attachment) для обработки ответа **Response** как файла, и настраивается соответствующий **Renderer**, который будет обрабатывать клиентский заголовок **Accept** для этого типа содержимого ответа. `responses=bytes` указывает, что ответ является двоичным BLOB-объектом без дополнительных сведений о его структуре.

```python
from django.http import HttpResponse
from rest_framework.renderers import BaseRenderer


class BinaryRenderer(BaseRenderer):
    media_type = "application/octet-stream"
    format = "bin"


class FileViewSet(RetrieveModelMixin, GenericViewSet):
    ...
    renderer_classes = [BinaryRenderer]

    @extend_schema(responses=bytes)
    def retrieve(self, request, *args, **kwargs):
        export_data = b"..."
        return HttpResponse(
            export_data,
            content_type=BinaryRenderer.media_type,
            headers={
                "Content-Disposition": "attachment; filename=out.bin",
            },
        )
```
