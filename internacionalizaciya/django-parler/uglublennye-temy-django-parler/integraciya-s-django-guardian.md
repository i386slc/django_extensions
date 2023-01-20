# Интеграция с django-guardian

## Объединение TranslatableAdmin с GuardedModelAdmin

Чтобы объединить **TranslatableAdmin** с **GuardedModelAdmin** из [django-guardian](https://github.com/lukaszb/django-guardian), нужно отметить несколько вещей.

В зависимости от порядка наследования вкладки языка _parler_ или кнопка _guardian_ «Разрешения объекта» могут больше не отображаться.

Чтобы исправить это, вам нужно убедиться, что обе части шаблона включены на страницу.

Оба класса переопределяют значение **change\_form\_template**:

* **GuardedModelAdmin** явно устанавливает для него значение `admin/guardian/model/change_form.html`.
* **TranslatableAdmin** задает для него значение `admin/parler/change_form.html`, но наследует исходный шаблон, который в противном случае администратор выбрал бы автоматически.

## Использование TranslatableAdmin в качестве первого класса

Когда **TranslatableAdmin** является первым унаследованным классом:

```python
class ProjectAdmin(TranslatableAdmin, GuardedModelAdmin):
    pass
```

Вы можете создать шаблон, такой как `myapp/project/change_form.html`, который наследует шаблон _guardian_:

```django
{% raw %}
{% extends "admin/guardian/model/change_form.html" %}
{% endraw %}
```

Теперь **django-parler** загрузит этот шаблон в `admin/parler/change_form.html`, чтобы было видно содержимое как _guardian_, так и _parler_.

## Использование GuardedModelAdmin в качестве первого класса

Когда **GuardedModelAdmin** является первым унаследованным классом:

```python
class ProjectAdmin(TranslatableAdmin, GuardedModelAdmin):
    change_form_template = 'myapp/project/change_form.html'
```

**change\_form\_template** необходимо установить вручную. Его можно установить как `admin/parler/change_form.html` или использовать собственный шаблон, который включает обе части:

```django
{% raw %}
{% extends "admin/guardian/model/change_form.html" %}

{# восстановить вкладки django-parler #}
{% block field_sets %}
{% include "admin/parler/language_tabs.html" %}
{{ block.super }}
{% endblock %}
{% endraw %}
```
