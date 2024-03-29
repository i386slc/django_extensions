# Справочник по настройкам

Вот список всех доступных настроек django-filter и их значения по умолчанию. Все настройки имеют префикс **FILTERS\_**, хотя это немного многословно, но помогает легко идентифицировать эти настройки.

## FILTERS\_DEFAULT\_LOOKUP\_EXPR

По умолчанию: `'exact'`

Задает генерируемое выражение поиска по умолчанию, если оно не определено.

## FILTERS\_EMPTY\_CHOICE\_LABEL

По умолчанию: `'---------'`

Устанавливает значение по умолчанию для `ChoiceFilter.empty_label`. Вы можете отключить пустой выбор, установив для него значение `None`.

## FILTERS\_NULL\_CHOICE\_LABEL

По умолчанию: `None`

Устанавливает значение по умолчанию для `ChoiceFilter.null_label`. Вы можете включить нулевой выбор, установив значение, отличное от `None`.

## FILTERS\_NULL\_CHOICE\_VALUE

По умолчанию: `'null'`

Устанавливает значение по умолчанию для `ChoiceFilter.null_value`. Вы можете изменить это значение, если строка `'null'` по умолчанию конфликтует с фактическим выбором.

## FILTERS\_DISABLE\_HELP\_TEXT

По умолчанию: `False`

Некоторые фильтры предоставляют информационный **help\_text**. Например, фильтры на основе csv (`filters.BaseCSVFilter`) информируют пользователей о том, что «несколько значений могут быть разделены запятыми».

Вы можете установить значение `True`, чтобы отключить **help\_text** для всех фильтров, удалив текст из вывода отображаемой формы.

## FILTERS\_VERBOSE\_LOOKUPS

{% hint style="info" %}
Это считается расширенной настройкой и может быть изменено.
{% endhint %}

По умолчанию:

```python
# Ссылается на 'django_filters.conf.DEFAULTS'
'VERBOSE_LOOKUPS': {
    'exact': _(''),
    'iexact': _(''),
    'contains': _('contains'),
    'icontains': _('contains'),
    ...
}
```

Этот параметр управляет подробным выводом для сгенерированных меток фильтра. Вместо получения частей выражения, таких как `«lt»` и `«contained_by»`, подробная метка будет содержать `«is less then»` и `«is contained by»`. Подробный вывод можно отключить, установив для этого параметра ложное значение.

Этот параметр также принимает вызываемые объекты. Вызываемый объект не должен требовать аргументов и должен возвращать словарь. Это полезно для расширения или переопределения терминов по умолчанию без необходимости копировать весь набор терминов в ваши настройки. Например, вы можете добавить подробный вывод для `"exact"` поисков.

```python
# settings.py
def FILTERS_VERBOSE_LOOKUPS():
    from django_filters.conf import DEFAULTS

    verbose_lookups = DEFAULTS['VERBOSE_LOOKUPS'].copy()
    verbose_lookups.update({
        'exact': 'is equal to',
    })
    return verbose_lookups
```
