# Настройка рабочего процесса и схемы

Вы не удовлетворены сгенерированной схемой? Выполните следующие шаги, чтобы приблизить вашу схему к вашему API.

{% hint style="info" %}
Предупреждения, выдаваемые `./manage.py visible --file schema.yaml --validate`, предназначены для указания того, где **drf-spectacular** обнаружил проблемы. Разумные запасные варианты используются везде, где это возможно, и некоторые предупреждения могут даже не относиться к вам. Остальные проблемы можно решить, выполнив следующие действия.
{% endhint %}

## Шаг 1: queryset и serializer\_class

Самоанализ в значительной степени зависит от этих двух атрибутов. `get_serializer_class()` и `get_serializer()` также используются, если они доступны. Вы также можете установить их в **APIView**. Несмотря на то, что это не поддерживается DRF, **drf-spectacular** найдет и использует их.

## Шаг 2: @extend\_schema

Оберните свои функции представления с помощью декоратора **@extend\_schema**. Существует множество вариантов переопределения, но вам нужно переопределить только то, что не было должным образом обнаружено при самоанализе.

```python
class PersonView(viewsets.GenericViewSet):
    @extend_schema(
        parameters=[
          # поля сериализатора преобразуются в параметры
          QuerySerializer,
          # объект сериализатора преобразуется в параметр
          OpenApiParameter("nested", QuerySerializer),
          OpenApiParameter("queryparam1", OpenApiTypes.UUID, OpenApiParameter.QUERY),
          # переменная пути была переопределена
          OpenApiParameter("pk", OpenApiTypes.UUID, OpenApiParameter.PATH),
        ],
        request=YourRequestSerializer,
        responses=YourResponseSerializer,
        # больше настроек
    )
    def retrieve(self, request, pk, *args, **kwargs)
        # ваш код
```

{% hint style="info" %}
**responses** можно детализировать, предоставив вместо этого словарь. Это может быть, например, `{201: YourResponseSerializer, ...}` или `{(200, 'application/pdf'): OpenApiTypes.BINARY, ...}`.
{% endhint %}

{% hint style="info" %}
Для простых ответов вам может не понадобиться писать явный класс сериализатора. В таких случаях вы можете просто указать запрос/ответ с помощью вызова **inline\_serializer**. Это позволяет вам удобно определять встроенную схему конечной точки без фактического написания класса сериализатора.
{% endhint %}

{% hint style="info" %}
Если вы хотите аннотировать методы, предоставляемые базовыми классами представления, вам не к чему прикреплять **@extend\_schema**. В этих случаях вы можете использовать **@extend\_schema\_view**, чтобы удобно аннотировать реализации по умолчанию.

```python
class XViewset(mixins.ListModelMixin, viewsets.GenericViewSet):
    @extend_schema(description='text')
    def list(self, request, *args, **kwargs):
        return super().list(request, *args, **kwargs)
```

эквивалентно

```python
@extend_schema_view(
    list=extend_schema(description='text')
)
class XViewset(mixins.ListModelMixin, viewsets.GenericViewSet):
    ...
```
{% endhint %}

{% hint style="info" %}
Вы также можете использовать **@extend\_schema** в представлениях, чтобы прикреплять аннотации ко всем методам в этом представлении (например, теги). Аннотации методов будут иметь приоритет над аннотациями представления.
{% endhint %}

## Шаг 3: @extend\_schema\_field и подсказки ввода

Пользовательский **SerializerField** может быть неправильно подобран. Вы можете сообщить **drf-spectacular** о том, чего следует ожидать, с помощью декоратора **@extend\_schema\_field**. В качестве аргумента он принимает либо базовые типы, либо сериализатор. В случае базовых типов (например, **str**, **int** и т. д.) подсказки типа уже достаточно.

```python
@extend_schema_field(OpenApiTypes.BYTE)  # также принимает основные типы Python
class CustomField(serializers.Field):
    def to_representation(self, value):
        return urlsafe_base64_encode(b'\xf0\xf1\xf2')
```

Вы также можете применить его к методу **SerializerMethodField**.

```python
class ErrorDetailSerializer(serializers.Serializer):
    field_custom = serializers.SerializerMethodField()

    @extend_schema_field(OpenApiTypes.DATETIME)
    def get_field_custom(self, object):
        return '2020-03-06 20:54:00.104248'
```

## Шаг 4: @extend\_schema\_serializer

Вы также можете обернуть свой сериализатор **@extend\_schema\_serializer**. В основном используется для исключения определенных полей из схемы или прикрепления примеров запроса/ответа. В редких случаях (например, сериализаторы конвертов) может пригодиться переопределение определения списка с помощью `many=False`.

```python
@extend_schema_serializer(
    exclude_fields=('single',), # схема игнорирует эти поля
    examples = [
         OpenApiExample(
            'Valid example 1',
            summary='short summary',
            description='longer description',
            value={
                'songs': {'top10': True},
                'single': {'top10': True}
            },
            request_only=True, # сигнализировать, что пример применим только к запросам
            response_only=False, # сигнализировать, что пример применим только к ответам
        ),
    ]
)
class AlbumSerializer(serializers.ModelSerializer):
    songs = SongSerializer(many=True)
    single = SongSerializer(read_only=True)

    class Meta:
        fields = '__all__'
        model = Album
```

## Шаг 5: Расширения

Основная цель расширений — сделать вышеуказанные механизмы настройки доступными и для кода библиотеки. Обычно вы не можете легко декорировать или модифицировать **View**, **Serializer** или **Field** из библиотек. Расширения предоставляют способ подключиться к самоанализу, фактически не касаясь библиотеки.

Все расширения работают по одному принципу. Вы предоставляете **target\_class** (строка пути импорта или фактический класс), а затем указываете, что **drf-spectular** должен использовать вместо то, что он обычно обнаруживает.

{% hint style="success" %}
Расширения регистрируются автоматически. Просто убедитесь, что интерпретатор Python видит их хотя бы один раз.

Рекомендуется собирать расширения в `YOUR_MAIN_APP_NAME/schema.py` и импортировать этот файл в `YOUR_MAIN_APP_NAME/apps.py`. Каждое правильное приложение Django уже будет иметь автоматически сгенерированный файл **apps.py**. Хотя это и не является строго необходимым, выполнение импорта в `ready()` является наиболее надежным подходом. Это позволит убедиться, что ваша среда (например, настройки) правильно настроена перед загрузкой.

```python
# your_main_app_name/apps.py
class YourMainAppNameConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "your_main_app_name"

    def ready(self):
        import your_main_app_name.schema  # noqa: E402
```
{% endhint %}

{% hint style="info" %}
Используется только первое расширение, соответствующее критериям. Установив атрибут приоритета **priority** для вашего расширения, вы можете влиять на порядок сопоставления (по умолчанию `0`). Встроенные расширения имеют приоритет `-1`. Если вы создаете подклассы для встроенных расширений, не забудьте увеличить приоритет.
{% endhint %}

### Замените представления view на OpenApiViewExtension.

Многие библиотеки используют **@api\_view** или **APIView** вместо **ViewSet** или **GenericAPIView**. В таких случаях интроспекция имеет очень мало возможностей для работы. Целью этого расширения является расширение или отключение встречающегося представления (только для генерации схемы). Простое расширение обнаруженного класса `class Fixed(self.target_class)` с помощью атрибута **queryset** или **serializer\_class** часто решает большинство проблем.

```python
class Fix4(OpenApiViewExtension):
    target_class = 'oscarapi.views.checkout.UserAddressDetail'

    def view_replacement(self):
        from oscar.apps.address.models import UserAddress

        class Fixed(self.target_class):
            queryset = UserAddress.objects.none()
        return Fixed
```

### Укажите аутентификацию с помощью OpenApiAuthenticationExtension.

Классы аутентификации, не имеющие поддержки сторонних производителей, будут выдавать предупреждения и игнорироваться. К счастью, расширения аутентификации очень просты в реализации. Взгляните на [расширения метода аутентификации по умолчанию](https://github.com/tfranzel/drf-spectacular/blob/master/drf\_spectacular/authentication.py). Простая пользовательская аутентификация на основе заголовка HTTP может быть достигнута следующим образом:

```python
class MyAuthenticationScheme(OpenApiAuthenticationExtension):
    target_class = 'my_app.MyAuthentication'  # полный путь импорта ИЛИ ссылка на класс
    name = 'MyAuthentication'  # имя, используемое в схеме

    def get_security_definition(self, auto_schema):
        return {
            'type': 'apiKey',
            'in': 'header',
            'name': 'api_key',
        }
```

### Объявить вывод поля с помощью OpenApiSerializerFieldExtension

Это в основном предназначено для пользовательских **SerializerField**, которые находятся в коде библиотеки. Это расширение функционально эквивалентно **@extend\_schema\_field**.

```python
class CategoryFieldFix(OpenApiSerializerFieldExtension):
    target_class = 'oscarapi.serializers.fields.CategoryField'

    def map_serializer_field(self, auto_schema, direction):
        # эквивалентно возврату {'type': 'string'}
        return build_basic_type(OpenApiTypes.STR)
```

### Объявите магию сериализатора с помощью OpenApiSerializerExtension

Это один из наиболее сложных механизмов расширения. **drf-spectacular** использует их для реализации [полиморфных сериализаторов](https://github.com/tfranzel/drf-spectacular/blob/master/drf\_spectacular/serializers.py). Использование этого расширения требуется редко, потому что поведение большинства пользовательских классов **Serializer** очень близко к поведению по умолчанию.

Если ваш **Serializer** использует пользовательский **ListSerializer** (т. е. пользовательский `to_representation()`), вы можете написать для него специальные расширения. Обычно это тот случай, когда `many=True` приводит не к простому списку, а к расширенному объекту с дополнительными полями (например, конверты).

### Объявите пользовательские/библиотечные фильтры с помощью OpenApiFilterExtension

Это расширение применяется только к классам фильтрации и разбиения на страницы и используется редко. С этим расширением реализована встроенная поддержка **django-filter**. **OpenApiFilterExtension** заменяет собственные параметры фильтра **get\_schema\_operation\_parameters** вашей настроенной версией, где у вас есть полный доступ к более продвинутым функциям самоанализа **drf-spectacular**.

## Шаг 6: Хуки постобработки

Сгенерированная схема вам все еще не нравится? Вы не простой клиент, но есть еще одна вещь, которую вы можете сделать. Хуки постобработки запускаются в самом конце генерации схемы. Вот как выбранные **Enum** объединяются в составные объекты. Вы можете зарегистрировать хуки с настройкой **POSTPROCESSING\_HOOKS**.

```python
def custom_postprocessing_hook(result, generator, request, public):
    # ваши изменения схемы в результате параметра
    return result
```

{% hint style="info" %}
Обратите внимание, что параметр **POSTPROCESSING\_HOOKS** переопределяет значение по умолчанию. Если вы намерены сохранить хук **Enum**, обязательно добавьте `"drf_spectacular.hooks.postprocess_schema_enums"` обратно в список.
{% endhint %}

## Шаг 7: Препроцессинг хуков

Хуки предварительной обработки применяются вскоре после сбора всех операций API и до начала фактического создания схемы. Они обеспечивают простой механизм для изменения того, какие операции должны быть представлены в вашей схеме. Вы можете исключить определенные операции, префиксные пути, ввести или жестко закодировать параметры пути или изменить инициацию представления. дополнительные хуки с настройкой **PREPROCESSING\_HOOKS**.

```python
def custom_preprocessing_hook(endpoints):
    # ваши изменения в списке операций, представленные в схеме
    for (path, path_regex, method, callback) in endpoints:
        pass
    return endpoints
```

{% hint style="info" %}
Распространенным вариантом использования будет удаление повторяющихся операций с суффиксом `{format}`, для которых мы уже предоставили хук `drf_spectacular.hooks.preprocess_exclude_path_format`. Вы можете просто включить этот хук, добавив строку пути импорта в файл **PREPROCESSING\_HOOKS**.
{% endhint %}

## Поздравления

Теперь у вас больше не должно быть предупреждений и эффектная схема, удовлетворяющая всем вашим требованиям. Если это не так, не стесняйтесь открывать [проблему](https://github.com/tfranzel/drf-spectacular/issues) и вносить предложения по улучшению.
