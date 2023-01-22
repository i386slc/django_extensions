# Модуль parler.utils.context

Контекстные менеджеры для временного переключения языка.

## _class_ parler.utils.context.smart\_override(_language\_code_)

Это более умная версия **translation.override**, которая позволяет избежать переключения языка, если нет никаких изменений. Этот метод можно использовать вместо **translation.override**:

```python
with smart_override(self.get_current_language()):
    return reverse('myobject-details', args=(self.id,))
```

Это гарантирует, что любые URL-адреса, заключенные в `i18n_patterns()`, получат правильный префикс кода языка. Если URL-адрес также содержит переведенные поля (например, **slug**), вместо этого используйте **switch\_language**.

### \_\_init\_\_(language\_code)

Инициализирует себя. См. `help(type(self))` для точной сигнатуры.

## _class_ parler.utils.context.switch\_language(_object_, _language\_code=None_)

Менеджер контекста для переключения перевода объекта.

Он временно меняет как язык перевода, так и объектный язык.

Этот контекстный менеджер можно использовать для переключения переводов Django на текущий объектный язык. Его также можно использовать для рендеринга объектов на другом языке:

```python
with switch_language(object, 'nl'):
    print object.title
```

Это особенно полезно для функции `get_absolute_url()`. При использовании этого контекстного менеджера объектный язык будет идентичен текущему языку Django.

```python
def get_absolute_url(self):
    with switch_language(self):
        return reverse('myobject-details', args=(self.slug,))
```

{% hint style="info" %}
Когда объект совместно используется потоками, это не является потокобезопасным. Вместо этого используйте `safe_translation_getter()` для чтения определенного поля.
{% endhint %}

### \_\_init\_\_(_object_, _language\_code=None_)

Инициализирует себя. См. `help(type(self))` для точной сигнатуры.
