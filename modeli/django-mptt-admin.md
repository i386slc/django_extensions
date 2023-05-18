# django-mptt-admin

**Django-mptt-admin** предоставляет приятный интерфейс администрирования Django для [моделей django-mptt](https://mbraak.github.io/django-mptt-admin/).

* Исходный код доступен на [https://github.com/mbraak/django-mptt-admin](https://github.com/mbraak/django-mptt-admin).
* Документация доступна на [https://mbraak.github.io/django-mptt-admin/](https://mbraak.github.io/django-mptt-admin/).

<figure><img src="../.gitbook/assets/django-mptt-admin.png" alt=""><figcaption></figcaption></figure>

## Требования

Пакет протестирован с Django (3.2, 4.0 и 4.1) и django-mptt (0.13). Также с Python 3.6–3.10.

Старые версии:

* Версия 1.0.x поддерживает Django 2.0 и 2.1.
* Версия 0.7.2 поддерживает Django 1.11 и Python 2.7.

## Установка

Установите пакет:

```bash
$ pip install django-mptt-admin
```

Добавьте **django\_mptt\_admin** к вашим установленным приложениям в **settings.py**.

```python
INSTALLED_APPS = (
    ..
    'django_mptt_admin',
)
```

Используйте класс **DjangoMpttAdmin** в **admin.py**:

```python
from django.contrib import admin
from django_mptt_admin.admin import DjangoMpttAdmin
from models import Country

class CountryAdmin(DjangoMpttAdmin):
    pass

admin.site.register(Country, CountryAdmin)
```

## Параметры

### tree\_animation\_speed

Скорость анимации открытия/закрытия в миллисекундах. По умолчанию 200 миллисекунд.

### **tree\_auto\_open**

Автоматически открывающийся узел. Значение по умолчанию — 1.

Значения:

* **True**: автоматически открывать все узлы
* **False**: не открывать автоматически
* **Integer**: автооткрытие до этого уровня

### tree\_load\_on\_demand

Загрузка по запросу (True/False/level). Значение по умолчанию — `True`.

* **True**: загружать узлы по запросу
* **False**: не загружать узлы по запросу
* **int**: загружать узлы по запросу до этого уровня

### autoescape

Автоэкранирование (True/False). Значение по умолчанию — `True`.

Автоэкранирование заголовков в дереве.

### filter\_tree\_queryset

Переопределите метод **filter\_tree\_queryset**, чтобы отфильтровать набор запросов для дерева.

```python
class CountryAdmin(DjangoMpttAdmin):
  def filter_tree_queryset(self, queryset):
    return queryset.filter(name='abc')
```

### is\_drag\_and\_drop\_enabled

Переопределите метод **is\_drag\_and\_drop\_enabled**, чтобы отключить перетаскивание. По умолчанию перетаскивание включено.

```python
class CountryAdmin(DjangoMpttAdmin):
  def is_drag_and_drop_enabled(self):
    return False
```

### use\_context\_menu

Захват события контекстного меню. NB: событие контекстного меню запускается при щелчке правой кнопкой мыши.

* **True**: захват события контекстного меню.
* Это полезно, если вы хотите написать собственный javascript для перехвата события **tree.contextmenu**.
* Также см. [https://mbraak.github.io/jqTree/#usecontextmenu](https://mbraak.github.io/jqTree/#usecontextmenu) и [https://mbraak.github.io/jqTree/#event-tree-contextmenu](https://mbraak.github.io/jqTree/#event-tree-contextmenu)
* **False** (по умолчанию): не захватывать событие **contextmenu**.

### item\_label\_field\_name

Определите, какое поле модели должно быть меткой для элементов дерева.

Возможные значения:

* **string**: имя поля модели или метод свойства модели для использования в качестве метки элемента дерева.
* **None** (по умолчанию): метка элемента дерева объявлений модели используется в юникоде.

Пример:

<pre class="language-python"><code class="lang-python"><strong>class MyMpttModel(MPTTModel):
</strong>    title = models.CharField(......

    @property
    def title_for_admin(self):
          return '%s %s' % (self.pk, self.title)

class MyMpttModelAdminClass(MPTTModelAdmin):
    item_label_field_name = 'title_for_admin'
</code></pre>

## Фильтры

Если вы хотите использовать фильтры, вы можете установить опцию **list\_filter**. См. [документацию Джанго](https://docs.djangoproject.com/en/1.10/ref/contrib/admin/#django.contrib.admin.ModelAdmin.list\_filter).

```python
from django_mptt_admin.admin import DjangoMpttAdmin

class CountryAdmin(DjangoMpttAdmin):
    list_filter = ('continent',)
```

Также см. пример проекта для полного фильтра континентов.

## История изменений

#### 2.3.0 (4 августа 2022)

* Обновление **jqtree** до версии 1.6.2. Это устраняет проблему с фокусом клавиатуры при использовании загрузки по запросу.
* Поддержка Django 4.1

#### 2.2.0 (8 декабря 2021)

* Поддержка Django 4.0

#### 2.1.0 (6 апреля 2021)

* Проблема #353: поддержка Django 3.2

И т.д.....
