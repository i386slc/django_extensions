# Справочник по полям

## IsoDateTimeField

Расширяет `django.forms.DateTimeField`, позволяя анализировать даты в формате ISO 8601 в дополнение к существующим форматам.

Определяет атрибут уровня класса **ISO\_8601** как константу для формата.

Устанавливает `input_formats = [ISO_8601]` — это означает, что по умолчанию **IsoDateTimeField** будет анализировать только даты в формате ISO 8601.

Вы можете установить **input\_formats** в свой список необходимых форматов в соответствии с документацией **DateTimeField**, используя атрибут уровня класса **ISO\_8601**, чтобы указать формат ISO 8601.

```python
f = IsoDateTimeField()
f.input_formats = [IsoDateTimeField.ISO_8601] + DateTimeField.input_formats
```
