# Руководство django-mptt

## Проблема

Вы создали проект Django, и вам нужно управлять некоторыми иерархическими данными. Например, у вас есть куча иерархических страниц в CMS, и иногда страницы являются дочерними элементами других страниц.

Теперь предположим, что вы хотите отобразить навигационную цепочку на своем сайте, например:

```bash
Home > Products > Food > Meat > Spam > Spammy McDelicious
```

Чтобы получить все эти заголовки страниц, вы можете сделать что-то вроде этого:

```python
titles = []
while page:
    titles.append(page.title)
    page = page.parent
```

Это один запрос к базе данных для каждой страницы в хлебной крошке, а запросы к базе данных выполняются медленно. Давайте сделаем это лучше.

## Решение

Модифицированный обход дерева предзаказов поначалу может показаться немного сложным, но это один из лучших способов решить эту проблему.

Если вы хотите углубиться в детали, здесь есть хорошее объяснение: [Хранение иерархических данных в базе данных](https://www.sitepoint.com/hierarchical-data-database/) или [Управление иерархическими данными в Mysql](http://mikehillyer.com/articles/managing-hierarchical-data-in-mysql/).

Вкратце: MPTT делает большинство операций с деревьями намного дешевле с точки зрения запросов. На самом деле все эти операции занимают максимум один запрос, а иногда и ноль:

* получить потомков узла
* получить предков узла
* получить все узлы на заданном уровне
* получить листовые узлы

И этот принимает ноль запросов:

* подсчитать потомков данного узла

Достаточно вступления. Давайте начнем.

## Приступаем к работе

### Добавляем mptt в INSTALLED\_APPS

Как и в большинстве приложений Django, вы должны добавить **mptt** в **INSTALLED\_APPS** в вашем файле `settings.py`:

```python
INSTALLED_APPS = (
    'django.contrib.auth',
    # ...
    'mptt',
)
```

### Настройка своей модели

Начните с базового подкласса **MPTTModel**, примерно так:

```python
from django.db import models
from mptt.models import MPTTModel, TreeForeignKey

class Genre(MPTTModel):
    name = models.CharField(max_length=50, unique=True)
    parent = TreeForeignKey(
        'self',
        on_delete=models.CASCADE,
        null=True, blank=True,
        related_name='children'
    )

    class MPTTMeta:
        order_insertion_by = ['name']
```

Вы должны определить родительское поле **parent**, которое является **TreeForeignKey** для **«self»**. **TreeForeignKey** — это обычный **ForeignKey**, который по-разному отображает поля формы в админке и некоторых других местах.

Поскольку вы наследуете **MPTTModel**, ваша модель также будет иметь ряд других полей: **level**, **lft**, **rght** и **tree\_id**. Эти поля управляются алгоритмом **MPTT**. В большинстве случаев вам не нужно будет использовать эти поля напрямую.

Этот класс **MPTTMeta** добавляет некоторые настройки в **django-mptt** — в данном случае просто **order\_insertion\_by**. Это указывает на естественный порядок данных в дереве.

Теперь создайте и примените миграции для создания таблицы в базе данных:

```bash
python manage.py makemigrations <your_app>
python manage.py migrate
```

### Создание некоторых данных

Запустите оболочку **django**:

```bash
python manage.py shell
```

Теперь создайте некоторые данные для тестирования:

```python
from myapp.models import Genre

rock = Genre.objects.create(name="Rock")
blues = Genre.objects.create(name="Blues")
Genre.objects.create(name="Hard Rock", parent=rock)
Genre.objects.create(name="Pop Rock", parent=rock)
```

### Сделать вид view

Это довольно просто на данный момент. Добавьте это облегченное представление в свой `views.py`:

```python
def show_genres(request):
    return render(request, "genres.html", {'genres': Genre.objects.all()})
```

И добавьте URL для него в `urls.py`:

```python
(r'^genres/$', show_genres),
```

### Шаблон

**django-mptt** также включает в себя несколько тегов шаблонов, упрощающих эту задачу. Создайте шаблон с именем `genres.html` в каталоге шаблонов и поместите в него это:

```django
{% raw %}
{% load mptt_tags %}
<ul>
    {% recursetree genres %}
        <li>
            {{ node.name }}
            {% if not node.is_leaf_node %}
                <ul class="children">
                    {{ children }}
                </ul>
            {% endif %}
        </li>
    {% endrecursetree %}
{% endraw %}
</ul>
```

Этот тег **recursetree** будет рекурсивно отображать этот фрагмент шаблона для всех узлов. Попробуйте, перейдя в `/genres/`.

Есть больше; [ознакомьтесь с документацией](http://django-mptt.github.com/django-mptt/) для пользовательских материалов сайта администратора, дополнительных тегов шаблонов, функций перестроения дерева и т. д.

Теперь вы можете перестать думать о том, как делать деревья, и начать делать отличное приложение django!

### "order\_insertion\_by" gotcha

В приведенном выше примере мы использовали параметр **order\_insertion\_by**, который позволяет **django-mptt** упорядочивать элементы в дереве автоматически, используя ключ имени **name**. Что это означает технически? Что ж, если вы добавляете элементы неупорядоченным образом, **django-mptt** обновит базу данных, чтобы они были упорядочены в дереве.

Так почему это именно ловушка?

Ну, это не так. Пока вы не храните экземпляры со ссылками на старые данные. Но, скорее всего, вы храните их и даже не знаете об этом.

Если вы это сделаете, вам нужно будет перезагрузить свои предметы из базы данных, иначе у вас останутся странные ошибки, похожие на несогласованность данных.

Единственная причина этого в том, что мы не можем сказать Django перезагрузить каждый отдельный экземпляр **django.db.models.Model** на основе строки таблицы. Вам нужно будет перезагрузить вручную, вызвав [Model.refresh\_from\_db()](https://docs.djangoproject.com/en/dev/ref/models/instances/#refreshing-objects-from-database).

Например, используя эту модель из предыдущего фрагмента кода:

```python
>>> root = Genre.objects.create(name="")
#
# Имейте в виду, что мы собираемся добавлять детей неупорядоченным образом:
#
>>> b = OrderedInsertion.objects.create(name="b", parent=root)
>>> a = OrderedInsertion.objects.create(name="a", parent=root)
#
# На этом этапе дерево будет реорганизовано в базе данных,
# и если вы не обновите экземпляр «b», он останется со старыми данными,
# что, в свою очередь, приведет к таким ошибкам, как:
#
>>> a in a.get_ancestors(include_self=True)
True
>>> b in b.get_ancestors(include_self=True)
False
#
# Что? Это неверно! Давайте перезагрузим
#
>>> b.refresh_from_db()
>>> b in b.get_ancestors(include_self=True)
True
# Все в норме.
```
