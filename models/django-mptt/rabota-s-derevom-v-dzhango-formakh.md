# Работа с деревом в Джанго формах

## Поля

В пакете **mptt.forms** предусмотрены следующие настраиваемые поля формы.

### TreeNodeChoiceField

Это поле формы по умолчанию, используемое **TreeForeignKey**.

Подкласс [ModelChoiceField](https://docs.djangoproject.com/en/dev/ref/forms/fields/#django.forms.ModelChoiceField), который представляет уровень дерева каждого узла при создании меток параметров.

Пример, где форма, которая использовала **ModelChoiceField**:

```python
category = ModelChoiceField(queryset=Category.objects.all())
```

… приведет к выбору со следующими параметрами:

```bash
---------
Root 1
Child 1.1
Child 1.1.1
Root 2
Child 2.1
Child 2.1.1
```

Вместо этого используйте **TreeNodeChoiceField**:

```python
category = TreeNodeChoiceField(queryset=Category.objects.all())
```

… приведет к выбору со следующими параметрами:

```bash
Root 1
--- Child 1.1
------ Child 1.1.1
Root 2
--- Child 2.1
------ Child 2.1.1
```

Текст, используемый для обозначения уровня дерева, можно настроить, указав аргумент **level\_indicator**:

```python
category = TreeNodeChoiceField(queryset=Category.objects.all(),
                               level_indicator='+--')
```

… что для этого примера приведет к выбору со следующими параметрами:

```bash
Root 1
+-- Child 1.1
+--+-- Child 1.1.1
Root 2
+-- Child 2.1
+--+-- Child 2.1.1
```

Можно установить начальный уровень, чтобы наборы запросов, не включающие корневой объект, по-прежнему отображались удобным способом. Используйте аргумент **start\_level**, чтобы установить начальную точку для уровней:

```python
obj = Category.objects.get(pk=1)
category = TreeNodeChoiceField(queryset=obj.get_descendants(),
                               start_level=obj.level)
```

… что для этого примера приведет к выбору со следующими параметрами:

```bash
--- Child 1.1.1
```

### TreeNodeMultipleChoiceField

Так же, как **TreeNodeChoiceField**, но принимает более одного значения.

### TreeNodePositionField

Подкласс [ChoiceField](https://docs.djangoproject.com/en/dev/ref/forms/fields/#choicefield), выбор которого по умолчанию соответствует допустимым аргументам [метода move\_to](modeli-i-menedzhery-django-mptt.md#move\_to-target-position-first-child).

## Формы

Следующая пользовательская форма предоставляется в пакете **mptt.forms**.

### MoveNodeForm

Форма, которая позволяет пользователю перемещать заданный узел из одного места в его дереве в другое с необязательным ограничением узлов, которые являются допустимыми целевыми узлами для _метода move\_to_.

### Поля

Форма содержит следующие поля:

* **target** - **TreeNodeChoiceField** для выбора целевого узла target для перемещения узла node. Целевые узлы будут отображаться как `<select>` с установленным атрибутом **size**, поэтому пользователь может прокручивать целевые узлы без необходимости сначала открывать раскрывающийся список.
* **position** - **TreeNodePositionField** для выбора позиции перемещения узла node относительно целевого узла target.

### Конструкция

Требуемые аргументы:

* **node** - При построении формы экземпляр модели, представляющий перемещаемый узел, должен быть передан в качестве первого аргумента.

Необязательные аргументы:

* **valid\_targets** - Если предоставлен, этот аргумент ключевого слова будет определять список узлов, которые допустимы для выбора в форме. В противном случае для выбора будет доступен любой экземпляр того же класса модели, что и перемещаемый узел, кроме самого узла и любых его потомков. Например, если вы хотите ограничить перемещение узла в пределах собственного дерева, передайте **QuerySet**, содержащий все в дереве узла, кроме самого себя и его потомков (для предотвращения недопустимых перемещений) и корневого узла (поскольку пользователь может сделать так, чтобы узел родственный корневому узлу).
* **target\_select\_size** - Если предоставлен, этот аргумент ключевого слова будет использоваться для установки размера выбора, используемого для целевого узла. По умолчанию **10**.
* **position\_choices** - Кортеж разрешенных вариантов позиций и их описания.
* **level\_indicator** - Строка, которая будет использоваться для представления одного уровня дерева в целевых параметрах.

### Метод save()

Когда вызывается метод формы `save()`, он попытается выполнить перемещение узла, как указано в форме.

Если будет предпринята попытка недопустимого перемещения, сообщение об ошибке будет добавлено к ошибкам формы, не связанным с полем (доступно с помощью `{{ form.non_field_errors }}` в шаблонах), и связанный **mptt.exceptions.InvalidMove** будет повторно вызван.

Рекомендуется попытаться отловить эту ошибку и, если она обнаружена, позволить вашему представлению снова перейти к рендерингу формы, чтобы сообщение об ошибке отображалось для пользователя.

### Пример использования

Образец представления, показывающий основное использование формы, приведен ниже:

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render_to_response

from faqs.models import Category
from mptt.exceptions import InvalidMove
from mptt.forms import MoveNodeForm

def move_category(request, category_pk):
    category = get_object_or_404(Category, pk=category_pk)
    if request.method == 'POST':
        form = MoveNodeForm(category, request.POST)
        if form.is_valid():
            try:
                category = form.save()
                return HttpResponseRedirect(category.get_absolute_url())
            except InvalidMove:
                pass
    else:
        form = MoveNodeForm(category)

    return render_to_response('faqs/move_category.html', {
        'form': form,
        'category': category,
        'category_tree': Category.objects.all(),
    })
```
