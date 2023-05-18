# Работа с деревом в шаблонах

## Приступаем к работе

Прежде чем вы сможете использовать эти теги/фильтры, вы должны:

* добавьте `"mptt"` к вашим **INSTALLED\_APPS** в `settings.py`
* добавьте `{% load mptt_tags %}` в свой шаблон

## Рекурсивные теги

_Новое в версии 0.4_.

Для большинства настроек рекурсивные теги — самый простой и эффективный способ рендеринга деревьев.

### recursetree

_Новое в версии 0.4_.

Этот тег рекурсивно отображает раздел вашего шаблона для каждого узла в вашем дереве.

Например:

```django
<ul class="root">
    {% raw %}
{% recursetree nodes %}
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

Обратите внимание на специальные переменные **node** и **children**. Они волшебным образом вставляются в ваш контекст, пока вы находитесь внутри тега **recursetree**.

* **node** - является экземпляром вашей модели **MPTT**.
* **children** - Эта переменная содержит отображаемый HTML для дочерних элементов **node**.

{% hint style="info" %}
Если у вас уже есть переменные с именами **node** или **children** в вашем шаблоне, и вам нужно получить к ним доступ внутри блока **recursetree**, вам нужно сначала присвоить им другое имя:

```django
{% raw %}
{% with node as friendly_node %}
    {% recursetree nodes %}
        {{ node.name }} is friends with {{ friendly_node.name }}
        {{ children }}
    {% endrecursetree %}
{% endwith %}
{% endraw %}
```
{% endhint %}

## Теги итерации

Почему? Эти теги лучше подходят для необычно глубоких деревьев. Если вы ожидаете иметь деревья с глубиной больше **20**, вы должны использовать их вместо приведенных выше.

### full\_tree\_for\_model

Заполняет переменную шаблона **QuerySet**, содержащим полное дерево для данной модели.

Применение:

```django
{% raw %}
{% full_tree_for_model [model] as [varname] %}
{% endraw %}
```

Модель указывается в формате `[appname].[modelname]`.

Пример:

```django
{% raw %}
{% full_tree_for_model tests.Genre as genres %}
{% endraw %}
```

### drilldown\_tree\_for\_node

Заполняет переменную шаблона деревом детализации для данного узла, дополнительно подсчитывая количество элементов, связанных с его дочерними элементами.

Дерево детализации состоит из предков узла, самого узла и его непосредственных дочерних элементов или всех потомков. Например, дерево детализации для категории книг `"Personal Finance"` может выглядеть примерно так:

```bash
Books
   Business, Finance & Law
      Personal Finance
         Budgeting (220)
         Financial Planning (670)
```

Использование:

```django
{% raw %}
{% drilldown_tree_for_node [node] as [varname] %}
{% endraw %}
```

Расширенное использование:

```django
{% raw %}
{% drilldown_tree_for_node [node] as [varname] all_descendants %}
{% drilldown_tree_for_node [node] as [varname] count [foreign_key] in [count_attr] %}
{% drilldown_tree_for_node [node] as [varname] cumulative count [foreign_key] in [count_attr] %}
{% endraw %}
```

Внешний ключ указывается в формате `[appname].[modelname].[fieldname]`, где имя поля **fieldname** — это имя поля в указанной модели, которое связывает его с моделью данного узла.

При использовании этой формы атрибут **count\_attr** для каждого дочернего элемента данного узла в дереве детализации будет содержать количество элементов, связанных с ним посредством данного внешнего ключа.

Если также указано **cumulative**, это количество будет для элементов, связанных с дочерним узлом и всеми его потомками.

Примеры:

```django
{% raw %}
{% drilldown_tree_for_node genre as drilldown %}
{% drilldown_tree_for_node genre as drilldown count tests.Game.genre in game_count %}
{% drilldown_tree_for_node genre as drilldown cumulative count tests.Game.genre in game_count %}
{% endraw %}
```

См. [Примеры](rabota-s-derevom-v-shablonakh.md#primery) для примера того, как визуализировать дерево детализации как вложенный список.

## Фильтры

### Фильтр tree\_info

Получив список элементов дерева, выполняет итерацию по списку, генерируя два кортежа текущего элемента дерева и словарь **dict**, содержащий информацию о структуре дерева вокруг элемента, со следующими ключами:

* **new\_level** - `True`, если текущий элемент является началом нового уровня в дереве, в противном случае — `False`.
* **closed\_levels** - Список уровней, которые заканчиваются после текущего элемента. Это будет пустой список, если уровень следующего элемента такой же или выше, чем уровень текущего элемента.

Можно указать необязательный аргумент, чтобы указать дополнительные сведения о структуре, которая должна отображаться в словаре **dict**. Это должен быть список имен функций, разделенных запятыми. Допустимые имена функций:

* **ancestors** - Добавляет список представлений Unicode предков текущего узла в порядке убывания (сначала корневой узел, последним непосредственный родитель) под ключом `"ancestors"`. Например: для приведенного ниже образца дерева содержимое списка, которое будет доступно под ключом `"ancestors"`, приведено справа:

```bash
Books                    ->  []
   Sci-fi                ->  ['Books']
      Dystopian Futures  ->  ['Books', 'Sci-fi']
```

Используя этот фильтр с распаковкой в теге `{% for %}`, вы должны иметь достаточно информации о структуре дерева для создания иерархического представления дерева.

Пример:

```django
{% raw %}
{% for genre,structure in genres|tree_info %}
    {% if structure.new_level %}<ul><li>{% else %}</li><li>{% endif %}
        {{ genre.name }}
    {% for level in structure.closed_levels %}</li></ul>{% endfor %}
{% endfor %}
{% endraw %}
```

### Фильтр tree\_path

Создает древовидный путь, представленный списком элементов, объединяя элементы с помощью разделителя, который может быть предоставлен в качестве необязательного аргумента, по умолчанию `' :: '`.

Каждый элемент пути будет переведен в юникод, поэтому при необходимости может быть предоставлен список экземпляров модели.

Пример:

```django
{{ some_list|tree_path }}
{{ some_node.get_ancestors|tree_path:" > " }}
```

## Примеры

Использование **drilldown\_tree\_for\_node** и **tree\_info** вместе для отображения раскрывающегося меню для узла с кумулятивным количеством связанных элементов для дочерних элементов узла:

```django
{% raw %}
{% drilldown_tree_for_node genre as drilldown cumulative count tests.Game.genre in game_count %}
{% for node,structure in drilldown|tree_info %}
    {% if structure.new_level %}<ul><li>{% else %}</li><li>{% endif %}
    {% if node == genre %}
        <strong>{{ node.name }}</strong>
    {% else %}
        <a href="{{ node.get_absolute_url }}">{{ node.name }}</a>
        {% if node.parent_id == genre.pk %}({{ node.game_count }}){% endif %}
    {% endif %}
    {% for level in structure.closed_levels %}</li></ul>{% endfor %}
{% endfor %}
{% endraw %}
```

Использование **tree\_info** (с необязательным аргументом) и **tree\_path** вместе для создания множественного выбора, который:

* не содержит корневых узлов
* отображает полный путь к каждому узлу

```django
<select name="classifiers" multiple="multiple" size="10">
    {% raw %}
{% for node,structure in classifiers|tree_info:"ancestors" %}
        {% if node.is_child_node %}
            <option value="{{ node.pk }}">
                {{ structure.ancestors|tree_path }} :: {{ node }}
            </option>
        {% endif %}
    {% endfor %}
{% endraw %}
</select>
```
