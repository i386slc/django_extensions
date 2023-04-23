# Установка django-filter

## Установка

**Django-filter** можно установить из PyPI с помощью таких инструментов, как **pip**:

```bash
$ pip install django-filter
```

Затем добавьте `'django_filters'` в **INSTALLED\_APPS**.

```python
INSTALLED_APPS = [
    ...
    'django_filters',
]
```

## Требования

Для **Django-filter** требуется текущая версия [Django](https://www.djangoproject.com/download/#supported-versions), и он протестирован на всех поддерживаемых версиях Python, а также на последней версии Django REST Framework ([DRF](http://www.django-rest-framework.org/)).
