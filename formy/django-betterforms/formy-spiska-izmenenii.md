# Формы списка изменений

Формы списка изменений **Changelist Forms** предназначены для облегчения поиска и сортировки моделей django, а также для предоставления основы для создания других функций, связанных с операциями над списками экземпляров модели.

## _class_ changelist.ChangeListForm

Базовый класс формы для всех форм списка изменений.

* **установка queryset**: Все формы списка изменений должны знать, с какого набора запросов начинать. Вы можете сделать это, либо передав именованный параметр ключевого слова в конструктор формы, либо определив атрибут модели в классе.

## _class_ changelist.SearchForm

Класс формы, упрощающий поиск по заданному набору полей для модели. Эта форма добавляет поле в модель **q** для поискового запроса ([исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/changelist.html#SearchForm)).

### SEARCH\_FIELDS

Список полей, по которым будет производиться поиск.

### CASE\_SENSITIVE

Должен ли поиск учитывать регистр.

Вот простой пример [SearchForm](formy-spiska-izmenenii.md#class-changelist.searchform) для поиска среди пользователей.

```python
# my_app/forms.py
from django.contrib.auth.models import get_user_model
from betterforms.forms import SearchForm

class UserSearchForm(SearchForm):
    SEARCH_FIELDS = ('username', 'email', 'name')
    model = get_user_model()

 # my_app.views.py
 from my_app.forms import UserSearchForm

 def user_list_view(request):
     form = UserSearchForm(request.GET)
     context = {'form': form}
     if form.is_valid:
         context['queryset'] = form.get_queryset()
     return render_to_response(context, ...)
```

[SearchForm](formy-spiska-izmenenii.md#class-changelist.searchform) проверяет, присутствует ли значение запроса в каком-либо из полей, объявленных в **SEARCH\_FIELDS**, путем объединения объектов **Q** с помощью запросов **\_\_contains** или **\_\_icontains** к этим полям.

## _class_ changelist.SortForm

Форма, облегчающая сортировку экземпляров модели ([исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/changelist.html#SortForm)). Эта форма добавляет скрытое поле сортировки в модель, которое используется для определения столбцов, которые должны быть отсортированы и в каком порядке.

### HEADERS

Список объектов [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true) для сортировки.

Заголовки могут быть объявлены несколькими способами.

* как созданные экземпляры [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true):

```python
# my_app/forms.py
from betterforms.forms import SortForm, Header

class UserSortForm(SortForm):
    HEADERS = (
        Header('username', ..),
        Header('email', ..),
        Header('name', ..),
    )
    model = User
```

* как строка

```python
# my_app/forms.py
from betterforms.forms import SortForm

class UserSortForm(SortForm):
    HEADERS = (
        'username',
        'email',
        'name',
    )
    model = User
```

* Как итерация **\*args**, которая будет использоваться для создания экземпляра объекта [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true):

```python
# my_app/forms.py
from betterforms.forms import SortForm

class UserSortForm(SortForm):
    HEADERS = (
        ('username', ..),
        ('email', ..),
        ('name', ..),
    )
    model = User
```

* Как 2-кортеж из имени заголовка **header name** и **\*\*kwargs**. Имя и предоставленные **\*\*kwargs** будут использоваться для создания экземпляров объектов [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true).

```python
# my_app/forms.py
from betterforms.forms import SortForm

class UserSortForm(SortForm):
    HEADERS = (
        ('username', {..}),
        ('email', {..}),
        ('name', {..}),
    )
    model = User
```

Все эти примеры примерно эквивалентны, в результате чего форма имеет три сортируемых заголовка `('username', 'email', 'name')`, которые будут отображаться в этих именованных полях в модели пользователя **User**.

См. документацию по классу [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true) для получения дополнительной информации о том, как можно настроить заголовки сортировки.

### get\_order\_by()

Возвращает список имен столбцов, которые используются в вызове **order\_by** для возвращенного набора запросов queryset.

Во время создания экземпляра все объявленные заголовки в **form.HEADERS** преобразуются в объекты [Header](formy-spiska-izmenenii.md#class-changelist.header-name-label-none-column\_name-none-is\_sortable-true) и доступны из **form.headers**.

```python
>>> [header for header in form.headers]  # Перебрать все заголовки.
>>> form.headers[2]  #  Получить заголовок по индексу - 2
>>> form.headers['username']  #  Получить заголовок с именем 'username'
```

## _class_ changelist.Header(_name_, _label=None_, _column\_name=None_, _is\_sortable=True)_

Заголовки — это механизм, через который сияет [SortForm](formy-spiska-izmenenii.md#class-changelist.sortform). Они предоставляют строки запроса для операций, связанных с сортировкой по любому параметру запроса, к которому привязан заголовок, а также другие значения, полезные во время рендеринга. [Исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/changelist.html#Header).

#### name

Имя заголовка.

#### label

Удобочитаемое имя заголовка.

#### is\_active

Логическое значение, указывающее, используется ли этот заголовок в настоящее время для сортировки.

#### is\_ascending

Логическое значение, указывающее, используется ли этот заголовок для сортировки данных в порядке возрастания.

#### is\_descending

Логическое значение, определяющее, используется ли этот заголовок для сортировки данных в порядке убывания.

#### css\_classes

Разделенный пробелами список классов css, подходящих для вывода в шаблоне в качестве атрибута класса css элемента HTML.

#### priority

1-индексированное число относительно приоритета этого заголовка в списке сортировок. Возвращает `None`, если заголовок не активен.

#### querystring

Строка запроса, которая будет сортироваться по этому заголовку в качестве основной сортировки, перемещая все остальные активные сортировки в их текущем порядке после этого. Сохраняет все остальные параметры запроса.

#### remove\_querystring

Строка запроса, которая удалит этот заголовок из сортировок. Сохраняет все остальные параметры запроса.

#### singular\_querystring

Строка запроса, которая будет сортироваться по этому заголовку и деактивировать все остальные активные сортировки. Сохраняет все остальные параметры запроса.

## Работа со списками изменений

Вывод заголовков формы сортировки можно выполнить с помощью предоставленного частичного шаблона, расположенного по адресу `betterforms/sort_form_header.html`.

```python
<th class="{{ header.css_classes }}">
  {% raw %}
{% if header.is_sortable %}
    <a href="?{{ header.querystring }}">{{ header.label }}</a>
    {% if header.is_active %}
      {% if header.is_ascending %}
        ▾
      {% elif header.is_descending %}
        ▴
      {% endif %}
      <a href="" data-sort_by="title" data-direction="up"></a>
      <span class="filterActive"><span>{{ header.priority }}</span> <a href="?{{ header.remove_querystring }}">x</a></span>
    {% endif %}
  {% else %}
    {{ header.label }}
  {% endif %}
{% endraw %}
</th>
```

В этом примере предполагается, что вы используете таблицу для вывода данных. Это должно быть тривиально, чтобы изменить это в соответствии с вашими потребностями.

## _class_ views.BrowseView

Представление на основе классов для работы со списками изменений ([исходник](https://django-betterforms.readthedocs.io/en/latest/\_modules/betterforms/views.html#BrowseView)). Это комбинация **FormView** и **ListView** в том смысле, что он обрабатывает отправку форм и предоставляет необязательный набор запросов с разбивкой на страницы для рендеринга в шаблоне.

Работает аналогично стандартному классу **FormView**, предоставляемому django, за исключением того, что форма создается с помощью **request.GET**, а **object\_list**, передаваемый в контекст шаблона, поступает из `form.get_queryset()`.
