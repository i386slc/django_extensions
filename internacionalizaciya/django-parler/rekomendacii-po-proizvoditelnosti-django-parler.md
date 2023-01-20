# Рекомендации по производительности django-parler

Переводы каждой модели хранятся в отдельной таблице. В некоторых случаях это может вызвать проблему с N-запросом. **django-parler** предлагает два способа управления производительностью базы данных.

## Кеширование

Все переведенное содержимое кэшируется по умолчанию. Следовательно, при повторном чтении объекта запрос не выполняется. Это работает из коробки, когда проект использует правильное кэширование:

```python
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',
        'KEY_PREFIX': 'mysite.production',  # Измени это
        'LOCATION': '127.0.0.1:11211',
        'TIMEOUT': 24*3600
    },
}
```

Вы должны убедиться, что ваш проект имеет надлежащую внутреннюю поддержку:

```bash
pip install python-memcached
```

Теперь таблицу перевода нужно читать только один раз в день.

## Предварительная выборка запроса

С помощью `prefetch_related()` все переводы могут быть получены одним запросом:

```python
object_list = MyModel.objects.prefetch_related('translations')
for obj in object_list:
    # читает переведенный заголовок из предварительно загруженного набора запросов
    print obj.title
```

Обратите внимание, что предварительная выборка считывает информацию обо всех языках, а не только об активном в данный момент языке.

Когда вы отображаете переведенные объекты в форме, например, список выбора, вы также можете выполнить предварительную выборку набора запросов:

```python
class MyModelAdminForm(TranslatableModelForm):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['some_field'].queryset = self.fields['some_field'].queryset.prefetch_related('translations')
```