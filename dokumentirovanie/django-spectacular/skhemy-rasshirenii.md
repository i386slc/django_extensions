# Схемы расширений

Blueprints — это набор исправлений схемы для приложений Django и REST Framework. Некоторые библиотеки/приложения плохо взаимодействуют с автоматическим самоанализом **drf-spectacular**. С расширениями вы можете вручную предоставить необходимую информацию для создания лучшей схемы.

Нет плана для приложения, которое вы ищете? Нет проблем, вы можете легко написать расширения самостоятельно. Возьмите чертежи здесь в качестве примеров и взгляните на [настройку рабочего процесса и схемы](nastroika-rabochego-processa-i-skhemy.md). Не стесняйтесь вносить новые или исправления с [PR](https://github.com/tfranzel/drf-spectacular/pulls). Файлы чертежей можно найти [здесь](https://github.com/tfranzel/drf-spectacular/tree/master/docs/blueprints).

{% hint style="info" %}
Просто скопируйте и вставьте фрагменты в свою кодовую базу. Расширения регистрируются автоматически. Просто убедитесь, что интерпретатор Python видит их хотя бы один раз. С этой целью мы предлагаем создать файл `PROJECT/schema.py` и импортировать его в свой `PROJECT/__init__.py` (тот же каталог, что и **settings.py** и **urls.py**) с помощью `import PROJECT.schema`. Теперь все готово.
{% endhint %}

## dj-stripe

Полосатые модели для Django: [dj-stripe](https://github.com/dj-stripe/dj-stripe)

```python
from djstripe.contrib.rest_framework.serializers import (
    CreateSubscriptionSerializer, SubscriptionSerializer
)

from drf_spectacular.extensions import OpenApiViewExtension
from drf_spectacular.utils import extend_schema


class FixDjstripeSubscriptionRestView(OpenApiViewExtension):
    target_class = 'djstripe.contrib.rest_framework.views.SubscriptionRestView'

    def view_replacement(self):
        class Fixed(self.target_class):
            serializer_class = SubscriptionSerializer

            @extend_schema(
                request=CreateSubscriptionSerializer,
                responses=CreateSubscriptionSerializer
            )
            def post(self, request, *args, **kwargs):
                pass

        return Fixed
```

## django-oscar-api

RESTful API для django-oscar: [django-oscar-api](https://github.com/django-oscar/django-oscar-api)

```python
from rest_framework import serializers

from drf_spectacular.extensions import (
    OpenApiSerializerExtension, OpenApiSerializerFieldExtension, OpenApiViewExtension
)
from drf_spectacular.plumbing import build_basic_type
from drf_spectacular.types import OpenApiTypes
from drf_spectacular.utils import OpenApiParameter, extend_schema, extend_schema_field


class Fix1(OpenApiViewExtension):
    target_class = 'oscarapi.views.root.api_root'

    def view_replacement(self):
        return extend_schema(responses=OpenApiTypes.OBJECT)(self.target_class)


class Fix2(OpenApiViewExtension):
    target_class = 'oscarapi.views.product.ProductAvailability'

    def view_replacement(self):
        from oscarapi.serializers.product import AvailabilitySerializer

        class Fixed(self.target_class):
            serializer_class = AvailabilitySerializer
        return Fixed


class Fix3(OpenApiViewExtension):
    target_class = 'oscarapi.views.product.ProductPrice'

    def view_replacement(self):
        from oscarapi.serializers.checkout import PriceSerializer

        class Fixed(self.target_class):
            serializer_class = PriceSerializer
        return Fixed


class Fix4(OpenApiViewExtension):
    target_class = 'oscarapi.views.checkout.UserAddressDetail'

    def view_replacement(self):
        from oscar.apps.address.models import UserAddress

        class Fixed(self.target_class):
            queryset = UserAddress.objects.none()
        return Fixed


class Fix5(OpenApiViewExtension):
    target_class = 'oscarapi.views.product.CategoryList'

    def view_replacement(self):
        class Fixed(self.target_class):
            @extend_schema(parameters=[
                OpenApiParameter(name='breadcrumbs', type=OpenApiTypes.STR, location=OpenApiParameter.PATH)
            ])
            def get(self, request, *args, **kwargs):
                pass

        return Fixed


class Fix6(OpenApiSerializerExtension):
    target_class = 'oscarapi.serializers.checkout.OrderSerializer'

    def map_serializer(self, auto_schema, direction):
        from oscarapi.serializers.checkout import OrderOfferDiscountSerializer, OrderVoucherOfferSerializer

        class Fixed(self.target_class):
            @extend_schema_field(OrderOfferDiscountSerializer(many=True))
            def get_offer_discounts(self):
                pass

            @extend_schema_field(OpenApiTypes.URI)
            def get_payment_url(self):
                pass

            @extend_schema_field(OrderVoucherOfferSerializer(many=True))
            def get_voucher_discounts(self):
                pass

        return auto_schema._map_serializer(Fixed, direction)


class Fix7(OpenApiSerializerFieldExtension):
    target_class = 'oscarapi.serializers.fields.CategoryField'

    def map_serializer_field(self, auto_schema, direction):
        return build_basic_type(OpenApiTypes.STR)


class Fix8(OpenApiSerializerFieldExtension):
    target_class = 'oscarapi.serializers.fields.AttributeValueField'

    def map_serializer_field(self, auto_schema, direction):
        return {
            'oneOf': [
                build_basic_type(OpenApiTypes.STR),
            ]
        }


class Fix9(OpenApiSerializerExtension):
    target_class = 'oscarapi.serializers.basket.BasketSerializer'

    def map_serializer(self, auto_schema, direction):
        class Fixed(self.target_class):
            is_tax_known = serializers.SerializerMethodField()

            def get_is_tax_known(self) -> bool:
                pass

        return auto_schema._map_serializer(Fixed, direction)


class Fix10(Fix9):
    target_class = 'oscarapi.serializers.basket.BasketLineSerializer'
```

## djangorestframework-api-key

Поскольку [djangorestframework-api-key](https://github.com/florimondmanca/djangorestframework-api-key) не имеет записи в **authentication\_classes**, drf-spectacular не может подобрать эту библиотеку. Чтобы устранить этот недостаток, вы можете вручную добавить соответствующую схему безопасности.

{% hint style="info" %}
Использование параметра **SECURITY** не рекомендуется, за исключением особых обстоятельств, как, например, в примере. Почти во всех случаях настоятельно рекомендуется использовать **OpenApiAuthenticationExtension**, поскольку **SECURITY** будет добавляться к каждой конечной точке в схеме независимо от эффективности.
{% endhint %}

```python
SPECTACULAR_SETTINGS = {
    "APPEND_COMPONENTS": {
        "securitySchemes": {
            "ApiKeyAuth": {
                "type": "apiKey",
                "in": "header",
                "name": "Authorization"
            }
        }
    },
    "SECURITY": [{"ApiKeyAuth": [], }],
     ...
}
```

## Полиморфные модели

Использование полиморфных моделей/сериализаторов, к сожалению, дает плоские сериализаторы из-за того, как они построены. Это означает, что полиморфные сериализаторы не имеют иерархии наследования, представляющей общую функциональность. Эти расширения задним числом выстраивают иерархию, объединяя поля «общего знаменателя» в базовые компоненты и импортируя их в подкомпоненты через **allOf**. Это приводит к компонентам, которые лучше представляют структуру базовых сериализаторов/моделей, из которых они произошли.

Компоненты отлично работают без этого расширения, но в некоторых случаях сгенерированный клиентский код испытывает трудности с дизъюнктивной природой немодифицированных компонентов. Этот план предназначен для решения этой проблемы.

```python
from drf_spectacular.contrib.rest_polymorphic import PolymorphicSerializerExtension
from drf_spectacular.plumbing import ResolvedComponent
from drf_spectacular.serializers import PolymorphicProxySerializerExtension
from drf_spectacular.settings import spectacular_settings


class RollupMixin:
    """
    Это помощник схемы, который извлекает поля «общего знаменателя» из дочерних
    компонентов в их родительский компонент. Это применимо только к PolymorphicSerializer,
    а также к PolymorphicProxySerializer, где существует (неявная) иерархия наследования.

    Фактическая функциональность реализуется через расширения, определенные ниже.
    """
    def map_serializer(self, auto_schema, direction):
        schema = super().map_serializer(auto_schema, direction)

        if isinstance(self, PolymorphicProxySerializerExtension):
            sub_serializers = self.target.serializers
        else:
            sub_serializers = [
                self.target._get_serializer_from_model_or_instance(sub_model)
                for sub_model in self.target.model_serializer_mapping
            ]

        resolved_sub_serializers = [
            auto_schema.resolve_serializer(sub, direction) for sub in sub_serializers
        ]
        # это будет сгенерировано только при возврате map_serializer,
        # так что пока смоделируйте его
        mocked_component = ResolvedComponent(
            name=auto_schema._get_serializer_name(self.target, direction),
            type=ResolvedComponent.SCHEMA,
            object=self.target,
            schema=schema
        )

        # хак для рекурсивных моделей. во время выполнения расширения не все схемы
        # субсериализатора были сгенерированы, поэтому сведение невозможно.
        # регистрируя хук postproc с локальной переменной, мы откладываем это выполнение
        # до конца, где присутствуют все схемы.
        def postprocessing_rollup_hook(generator, result, **kwargs):
            rollup_properties(mocked_component, resolved_sub_serializers)
            result['components'] = generator.registry.build({})
            return result

        # зарегистрировать постпроцессорный хук. должен запускаться перед
        # enum postproc из-за перестройки реестра
        spectacular_settings.POSTPROCESSING_HOOKS.insert(0, postprocessing_rollup_hook)
        # и ничего не делать пока
        return schema


def rollup_properties(component, resolved_sub_serializers):
    # сворачивание уже произошло (впечатляющая ошибка spectacular и обычно не нужная)
    if any('allOf' in r.schema for r in resolved_sub_serializers):
        return

    all_field_sets = [
        set(list(r.schema['properties'])) for r in resolved_sub_serializers
    ]
    common_fields = all_field_sets[0].intersection(*all_field_sets[1:])
    common_schema = {
        'properties': {},
        'required': set(),
    }

    # заменить общие поля подсериализаторов базовым классом
    for r in resolved_sub_serializers:
        for cf in sorted(common_fields):
            if cf in r.schema['properties']:
                common_schema['properties'][cf] = r.schema['properties'][cf]
                del r.schema['properties'][cf]
                if cf in r.schema.get('required', []):
                    common_schema['required'].add(cf)
        r.schema = {'allOf': [component.ref, r.schema]}

    # изменить обычную схему для свертки полей
    del component.schema['oneOf']
    component.schema['properties'] = common_schema['properties']
    if common_schema['required']:
        component.schema['required'] = sorted(common_schema['required'])


class PolymorphicRollupSerializerExtension(RollupMixin, PolymorphicSerializerExtension):
    priority = 1


class PolymorphicProxyRollupSerializerExtension(RollupMixin, PolymorphicProxySerializerExtension):
    priority = 1
```

## RapiDoc

[RapiDoc](https://mrin9.github.io/RapiDoc/) — это инструмент документирования, который можно использовать в качестве альтернативы пользовательскому интерфейсу **Redoc** или **Swagger UI**.

```python
from rest_framework.renderers import TemplateHTMLRenderer
from rest_framework.response import Response
from rest_framework.reverse import reverse
from rest_framework.views import APIView

from drf_spectacular.plumbing import get_relative_url, set_query_parameters
from drf_spectacular.settings import spectacular_settings
from drf_spectacular.utils import extend_schema
from drf_spectacular.views import AUTHENTICATION_CLASSES


class SpectacularRapiDocView(APIView):
    renderer_classes = [TemplateHTMLRenderer]
    permission_classes = spectacular_settings.SERVE_PERMISSIONS
    authentication_classes = AUTHENTICATION_CLASSES
    url_name = 'schema'
    url = None
    template_name = 'rapidoc.html'
    title = spectacular_settings.TITLE

    @extend_schema(exclude=True)
    def get(self, request, *args, **kwargs):
        schema_url = self.url or get_relative_url(reverse(self.url_name, request=request))
        schema_url = set_query_parameters(schema_url, lang=request.GET.get('lang'))
        return Response(
            data={
                'title': self.title,
                'dist': 'https://cdn.jsdelivr.net/npm/rapidoc@latest',
                'schema_url': schema_url,
            },
            template_name=self.template_name,
        )
```

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{{ title|default:"RapiDoc" }}</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <script type="module" src="{{ dist }}/dist/rapidoc-min.js"></script>
  </head>
  <body>
    <rapi-doc spec-url="{{ schema_url }}"></rapi-doc>
  </body>
</html>
```

## Elements

[Elements](https://stoplight.io/open-source/elements) — это еще один инструмент документирования, который можно использовать в качестве альтернативы интерфейсу **Redoc** или **Swagger UI**.

```python
from rest_framework.renderers import TemplateHTMLRenderer
from rest_framework.response import Response
from rest_framework.reverse import reverse
from rest_framework.views import APIView

from drf_spectacular.plumbing import get_relative_url, set_query_parameters
from drf_spectacular.settings import spectacular_settings
from drf_spectacular.utils import extend_schema
from drf_spectacular.views import AUTHENTICATION_CLASSES


class SpectacularElementsView(APIView):
     renderer_classes = [TemplateHTMLRenderer]
     permission_classes = spectacular_settings.SERVE_PERMISSIONS
     authentication_classes = AUTHENTICATION_CLASSES
     url_name = 'schema'
     url = None
     template_name = 'elements.html'
     title = spectacular_settings.TITLE

     @extend_schema(exclude=True)
     def get(self, request, *args, **kwargs):
        schema_url = self.url or get_relative_url(reverse(self.url_name, request=request))
        schema_url = set_query_parameters(schema_url, lang=request.GET.get('lang'), version=request.GET.get('version'))
        return Response(
            data={
                'title': self.title,
                'js_dist': 'https://unpkg.com/@stoplight/elements/web-components.min.js',
                'css_dist': 'https://unpkg.com/@stoplight/elements/styles.min.css',
                'schema_url': self._get_schema_url(request),
            },
            template_name=self.template_name
        )
```

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{{ title|default:"Elements" }}</title>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <script src="{{ js_dist }}"></script>
    <link rel="stylesheet" href="{{ css_dist }}">
  </head>
  <body>
    <elements-api apiDescriptionUrl="{{ schema_url }}" router="hash" />
  </body>
</html>
```

## drf-rw-serializers

[drf-rw-serializers](https://github.com/vintasoftware/drf-rw-serializers) предоставляет общие представления, наборы представлений и миксины, которые расширяют Django REST Framework, добавляя отдельные сериализаторы для операций чтения и записи.

**drf-spectacular** требует лишь небольшого расширения **AutoSchema**, чтобы он знал о **drf-rw-serializers**. Не забудьте заменить **AutoSchema** в **DEFAULT\_SCHEMA\_CLASS**.

```python
from drf_rw_serializers.generics import GenericAPIView as RWGenericAPIView

from drf_spectacular.openapi import AutoSchema


class CustomAutoSchema(AutoSchema):
    """ Используйте пользовательские методы drf_rw_serializers для направленных сериализаторов. """

    def get_request_serializer(self):
        if isinstance(self.view, RWGenericAPIView):
            return self.view.get_write_serializer()
        return self._get_serializer()

    def get_response_serializers(self):
        if isinstance(self.view, RWGenericAPIView):
            return self.view.get_read_serializer()
        return self._get_serializer()
```

## drf-extra-fields Base64FileField

[drf-extra-fields](https://github.com/Hipo/drf-extra-fields) предоставляет **Base64FileField** и **Base64ImageField**, которые автоматически представляют двоичные файлы как строки в кодировке base64. Это полезный способ встраивать файлы в более крупный JSON API и хранить все данные в одном дереве и обслуживать их с помощью одного запроса или ответа.

Поскольку для запросов к этим полям требуется строка в кодировке base64, а ответы могут быть либо URI, либо содержимым base64 (если `submit_as_base64=True`), требуется логика создания пользовательской схемы, поскольку она отличается от **FileField** DRF по умолчанию.

```python
from drf_spectacular.extensions import OpenApiSerializerFieldExtension
from drf_spectacular.openapi import AutoSchema
from drf_spectacular.plumbing import append_meta
from drf_spectacular.plumbing import build_basic_type
from drf_spectacular.types import OpenApiTypes
from drf_spectacular.utils import Direction


class Base64FileFieldSchema(OpenApiSerializerFieldExtension):
    target_class = "drf_extra_fields.fields.Base64FileField"

    def map_serializer_field(self, auto_schema, direction):
        if direction == "request":
            return build_basic_type(OpenApiTypes.BYTE)
        elif direction == "response":
            if self.target.represent_in_base64:
                return build_basic_type(OpenApiTypes.BYTE)
            else:
                return build_basic_type(OpenApiTypes.URI)


class Base64ImageFieldSchema(Base64FileFieldSchema):
    target_class = "drf_extra_fields.fields.Base64ImageField"


class PresentablePrimaryKeyRelatedFieldSchema(OpenApiSerializerFieldExtension):
    target_class = 'drf_extra_fields.relations.PresentablePrimaryKeyRelatedField'

    def map_serializer_field(self, auto_schema: AutoSchema, direction: Direction):
        if direction == 'request':
            return build_basic_type(OpenApiTypes.INT)

        meta = auto_schema._get_serializer_field_meta(self.target, direction)
        schema = auto_schema.resolve_serializer(
            self.target.presentation_serializer(
                context=self.target.context, **self.target.presentation_serializer_kwargs,
            ),
            direction,
        ).ref
        return append_meta(schema, meta)
```

## django-auth-adfs

[django-auth-adfs](https://github.com/snok/django-auth-adfs) предоставляет «серверную часть аутентификации Django для Microsoft ADFS и Azure AD». Схема работает для руководства по настройке Azure AD (см. [https://django-auth-adfs.readthedocs.io/en/latest/azure\_ad\_config\_guide.html](https://django-auth-adfs.readthedocs.io/en/latest/azure\_ad\_config\_guide.html)).

```python
from drf_spectacular.extensions import OpenApiAuthenticationExtension
from drf_spectacular.plumbing import build_bearer_security_scheme_object

class AdfsAccessTokenAuthenticationScheme(OpenApiAuthenticationExtension):
    target_class = 'django_auth_adfs.rest_framework.AdfsAccessTokenAuthentication'
    name = 'jwtAuth'

    def get_security_definition(self, auto_schema):
        return build_bearer_security_scheme_object(header_name='AUTHORIZATION',
                                                   token_prefix='Bearer',
                                                   bearer_format='JWT')
```

## django-parler-rest

Интеграция [django-parler-rest](https://github.com/django-parler/django-parler-rest) для пакета перевода [django-parler](https://github.com/django-parler/django-parler).

```python
from django.conf import settings

from drf_spectacular.extensions import OpenApiSerializerFieldExtension
from drf_spectacular.plumbing import build_object_type


class TranslationsFieldFix(OpenApiSerializerFieldExtension):
    target_class = 'parler_rest.fields.TranslatedFieldsField'

    def map_serializer_field(self, auto_schema, direction):
        # Получить автоматически сгенерированный суб-сериализатор из parler_rest
        # Содержит поля, завернутые в parler.models.TranslatedFields()
        translation_serializer = self.target.serializer_class
        # преобразовать вспомогательный сериализатор перевода в повторно используемый компонент.
        translation_component = auto_schema.resolve_serializer(
            translation_serializer, direction
        )
        # использовать каждый язык, представленный в PARLER_LANGUAGES
        return build_object_type(
            properties={
                i['code']: translation_component.ref for i in settings.PARLER_LANGUAGES[None]
            }
        )
```
