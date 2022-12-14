# Обновление макетов на ходу

Макеты можно изменять, адаптировать и генерировать программно.

В следующих разделах объясняется, как выбирать части макета и обновлять их. Мы будем использовать этот API из экземпляра **FormHelper**, а не самого макета. Основное поведение этого API состоит в выборе части макета для управления и цепочке методов, которые после этого изменяют его.

## Выделение объектов макета срезами

Вы можете получить фрагмент макета, используя знакомый оператор `[]` Python:

```python
form.helper[1:3]
form.helper[2]
form.helper[:-1]
```

В основном вы можете делать все виды срезов, те же самые, которые поддерживаются списками Python. Вы также можете объединить их. Если бы у вас был такой макет:

```python
Layout(
    Div('email')
)
```

Вы можете получить доступ к строке `'email'`, выполнив:

```python
form.helper[0][0]
```

## wrap

Одно полезное действие, которое вы можете применить к фрагменту, — это **wrap**, которое оборачивает каждое выбранное поле с использованием типа объекта макета и переданных параметров. Давайте посмотрим пример. Если бы у нас была такая раскладка:

```python
Layout(
    'field_1',
    'field_2',
    'field_3'
)
```

Мы могли бы сделать:

```python
form.helper[1:3].wrap(Field, css_class="hello")
```

Обратите внимание, как **wrap** влияет _**на каждый выбранный**_ объект макета. Если вы хотите _**объединить**_ поля field\_2 и field\_3 в объекте макета поля, вам придется использовать **wrap\_together**.

Помните, что срез `[1:3]` работает только на первом уровне глубины макета. Итак, если предыдущий макет был таким:

```python
Layout(
    'field_1',
    Div('field_2'),
    'field_3'
)
```

`helper[1:3]` вернет этот макет:

```python
Layout(
    'field_1',
    Field(Div('field_2'), css_class="hello"),
    Field('field_3', css_class="hello")
)
```

Параметры, переданные в **wrap** или **wrap\_together**, будут использоваться для создания объекта макета, который оборачивает выбранные поля. Вы можете передавать **args** и **kwargs**. Если вы используете объект макета, такой как **Fieldset**, которому требуется строка в качестве обязательного первого аргумента, wrap не будет работать должным образом, если вы не предоставите текст легенды в качестве аргумента для **wrap**. Давайте посмотрим на действительный пример:

```python
form.helper[1:3].wrap(Fieldset, "legend of the fieldset")
```

Также вы можете передавать **args** и **kwargs**:

```python
form.helper[1:3].wrap(Fieldset, "legend of the fieldset", css_class="fieldsets")
```

## wrap\_together

**wrap\_together** оборачивает весь фрагмент в тип объекта макета с переданными параметрами. Давайте посмотрим пример. Если бы у нас была такая раскладка:

```python
Layout(
    'field_1',
    'field_2',
    'field_3'
)
```

Мы могли бы сделать:

```python
form.helper[0:3].wrap_together(Field, css_class="hello")
```

В итоге у нас получился бы такой макет:

```python
Layout(
    Field(
        'field_1',
        'field_2',
        'field_3',
        css_class='hello'
    )
)
```

## update\_attributes

Обновляет атрибуты каждого объекта макета, содержащегося в срезе:

```python
Layout(
    'field_1',
    Field('field_2'),
    Field('field_3')
)
```

Мы могли бы сделать:

```python
form.helper[0:3].update_attributes(css_class="hello")
```

Макет превратится в:

```python
Layout(
    'field_1',
    Field('field_2', css_class='hello'),
    Field('field_3', css_class='hello')
)
```

Мы также можем применить его к имени поля, обернутому в объект макета:

```python
form.helper['field_2'].update_attributes(css_class="hello")
```

Однако следующее будет неверным:

```python
form.helper['field_1'].update_attributes(css_class="hello")
```

Потому что это изменит атрибуты **Layout**. Ваша задача правильно его завернуть.

## all

Этот метод выбирает _**все объекты**_ макета глубины первого уровня:

```python
form.helper.all().wrap(Field, css_class="hello")
```

## Выбор имени поля

Если вы передаете строку с именем поля, это имя поля будет жадно искаться по всем уровням глубины макета. Представьте, что у нас есть такой макет:

```python
Layout(
    'field_1',
    Div(
        Div('password')
    ),
    'field_3'
)
```

Если мы делаем:

```python
form.helper['password'].wrap(Field, css_class="hero")
```

Предыдущий макет станет:

```python
Layout(
    'field_1',
    Div(
        Div(
            Field('password', css_class="hero")
        )
    ),
    'field_3'
)
```

## filter

Этот метод позволит фильтровать объекты макета по типу его класса, применяя к ним действия:

```python
form.helper.filter(basestring).wrap(Field, css_class="hello")
form.helper.filter(Div).wrap(Field, css_class="hello")
```

Вы можете одновременно фильтровать несколько типов объектов макета:

```python
form.helper.filter(basestring, Div).wrap(Div, css_class="hello")
```

По умолчанию **filter** не является жадным, поэтому он ищет только первый уровень глубины. Но вы можете настроить его для поиска на разных уровнях глубины с помощью _kwarg_ **max\_level** (по умолчанию установлено значение **0**). Давайте посмотрим несколько примеров, чтобы прояснить это. Представьте, что у нас есть такой макет:

```python
Layout(
    'field_1',
    Div(
        Div('password')
    ),
    'field_3'
)
```

If we did:

```python
form.helper.filter(basestring).wrap(Field, css_class="hello")
```

Только **field\_1** и **field\_3** будут обернуты, что приведет к:

```python
Layout(
    Field('field_1', css_class="hello"),
    Div(
        Div('password')
    ),
    Field('field_3', css_class="hello"),
)
```

Если бы мы хотели искать глубже, оборачивая **password**, нам нужно было бы установить **max\_level** равным **2** или больше:

```python
form.helper.filter(basestring, max_level=2).wrap(Field, css_class="hello")
```

Другими словами, **max\_level** указывает количество шагов, которые crispy-forms могут сделать в объекте макета для сопоставления. В этом случае попадание в первый **Div** будет одним шагом, а попадание в следующий **Div** будет вторым шагом, таким образом, `max_level=2`.

Мы можем сделать фильтр жадным, заставив его искать как можно глубже, установив для **greedy** значение `True`:

```python
form.helper.filter(basestring, greedy=True).wrap(Div, css_class="hello")
```

#### Параметры:

* **max\_level** - Целое число, представляющее количество шагов, которое должны сделать crispy-forms при фильтрации. По умолчанию `0`.
* **greedy** - Логическое значение, указывающее, следует ли фильтровать жадно или нет. По умолчанию имеет значение `False`.

## filter\_by\_widget

Соответствует всем полям типа виджета. Этот метод предполагает, что вы используете помощника с прикрепленной формой, см. раздел [FormHelper с прикрепленной формой (макет по умолчанию)](formhelper.md#formhelper-s-prikreplennoi-formoi-maket-po-umolchaniyu), вы можете фильтровать по типу виджета, выполнив:

```python
form.helper.filter_by_widget(forms.PasswordInput).wrap(Field, css_class="hero")
```

**filter\_by\_widget** по умолчанию является жадным, поэтому выполняет глубокий поиск. Давайте посмотрим на пример использования, представьте, что у нас есть этот макет:

```python
Layout(
    'username',
    Div('password1'),
    Div('password2')
)
```

Предположим, что поля **password1** и **password2** используют виджет **PasswordInput**, который превратится в:

```python
Layout(
    'username',
    Div(Field('password1', css_class="hero")),
    Div(Field('password2', css_class="hero"))
)
```

Интересным реальным примером использования здесь может быть обертка всех **SelectInputs** с помощью пользовательского **ChosenField**, который отображает поле, используя выбранное поле, совместимое с js.

## exclude\_by\_widget

Исключает все поля типа виджета. Этот метод предполагает, что вы используете помощника с прикрепленной формой, см. раздел [FormHelper с прикрепленной формой (макет по умолчанию)](formhelper.md#formhelper-s-prikreplennoi-formoi-maket-po-umolchaniyu):

```python
form.helper.exclude_by_widget(forms.PasswordInput).wrap(Field, css_class="hero")
```

**exclude\_by\_widget** по умолчанию является жадным, поэтому выполняет поиск в глубину. Давайте посмотрим на пример использования, представьте, что у нас есть этот макет:

```
Layout(
    'username',
    Div('password1'),
    Div('password2')
)
```

Предположим, что поля **password1** и **password2** используют виджет **PasswordInput**, который превратится в:

```python
Layout(
    Field('username', css_class="hero"),
    Div('password1'),
    Div('password2')
)
```

## Управление макетом

Помимо выбора объектов макета и применения к ним действий, вы также можете легко манипулировать самими макетами и объектами макета, как если бы они были списками. Мы будем делать это не из хелпера, а из макета и самих объектов макета. Считайте это API более низкого уровня.

Все объекты макета, которые могут обертывать другие, содержат внутренний атрибут **fields**, который представляют собой список, а не словарь, как в формах Django. Вы можете легко применить к ним любые методы **Layout**. Помните, что макет ведет себя как другие объекты макета, такие как **Div**, с той лишь разницей, что он является корнем дерева.

Вот как вы замените объект макета на другой:

```python
layout[0][3][1] = Div('field_1')
```

Вот как вы могли бы добавить один объект макета в конец **Layout**:

```python
layout.append(HTML("<p>whatever</p>"))
```

Вот как вы можете добавить один объект макета в конец другого объекта внутри макета:

```python
layout[0].append(HTML("<p>whatever</p>"))
```

Вот как вы можете добавить несколько объектов макета в сам **Layout**:

```python
layout.extend([
    HTML("<p>whatever</p>"),
    Div('add_field_on_the_go')
])
```

Вот как вы можете добавить несколько объектов макета внутрь другого объекта макета:

```python
layout[0][2].extend([
    HTML("<p>whatever</p>"),
    Div('add_field_on_the_go')
])
```

Вот как вы могли бы удалить второй объект макета в **Layout**:

```python
layout.pop(1)
```

Вот как вы можете удалить второй объект макета внутри второго объекта макета:

```python
layout[1].pop(1)
```

Вот как вы можете вставить объект макета во вторую позицию базового **Layout**:

```python
layout.insert(1, HTML("<p>что угодно</p>"))
```

Вот как вы можете вставить объект макета во вторую позицию второго объекта макета:

```python
layout[1].insert(1, HTML("<p>whatever</p>"))
```

{% hint style="warning" %}
Всегда помните, что если вы собираетесь манипулировать помощником или макетом в представлении или любой части вашего кода, вам лучше использовать переменную уровня экземпляра.
{% endhint %}
