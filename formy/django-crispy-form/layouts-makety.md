# Layouts (макеты)

## Основы

**Django-crispy-forms** определяет еще один мощный класс под названием **Layout**, который позволяет вам изменять способ отображения полей формы. Это позволяет вам устанавливать порядок полей, заключать их в **div** или другие структуры, добавлять **html**, устанавливать идентификаторы **id**, классы **class** или атрибуты на все, что вы хотите, и т. д. И все это без написания собственного шаблона формы, используя программные макеты. Просто прикрепите макет к помощнику **helper.** Макеты необязательны, но, вероятно, это самая мощная вещь, которую может предложить **django-crispy-forms**.

Макет **layout** создается объектами макета, которые можно рассматривать как компоненты формы.

Все эти компоненты объясняются позже в [универсальных объектах макета](layouts-makety.md#obekty-universalnogo-maketa), сейчас вам нужно знать о них то, что каждый компонент отображает другой шаблон и имеет другое назначение. Давайте напишем пару разных макетов для нашей формы, продолжая наш пример с классом формы (обратите внимание, что полная форма больше не показана).

Давайте добавим **layout** в наш помощник:

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Layout, Fieldset, Submit

class ExampleForm(forms.Form):
    [...]
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.layout = Layout(
            Fieldset(
                'first arg is the legend of the fieldset',
                'like_website',
                'favorite_number',
                'favorite_color',
                'favorite_food',
                'notes'
            ),
            Submit('submit', 'Submit', css_class='button white'),
        )
```

Когда мы визуализируем форму, используя:

```django
{% raw %}
{% load crispy_forms_tags %}
{% crispy example_form %}
{% endraw %}
```

Мы получим поля, обернутые в набор полей, легенда которого будет установлена ​​на `«first arg is the legend of the fieldset»`. Порядок полей (из ранее созданной формы) будет следующим: **like\_website**, **favorite\_number**, **favorite\_color**, **favorite\_food** и **notes**. Мы также получаем кнопку отправки **Submit**, которая будет стилизована под классы Bootstrap `btn btn-primary`.

Это только вершина айсберга: теперь представьте, что вы хотите добавить объяснение того, что такое заметки, вы можете использовать объект макета **HTML**:

```python
Layout(
    Fieldset(
        'Tell us your favorite stuff {{ username }}',
        'like_website',
        'favorite_number',
        'favorite_color',
        'favorite_food',
        HTML("""
            <p>We use notes to get better, <strong>please help us {{ username }}</strong></p>
        """),
        'notes'
    )
)
```

Как вы заметите, легенда набора полей является _**контекстно-зависимой**_, и вы можете написать ее, как если бы она была фрагментом шаблона, в котором будет отображаться форма. Объект **HTML** добавит сообщение перед вводом заметок, а также контекстно-зависимый. Обратите внимание, как вы можете обернуть объекты макета в другие объекты макета. Объекты макета **Fieldset**, **Div**, **MultiField** и **ButtonHolder** могут содержать внутри себя другие объекты макета. Сделаем альтернативный макет для той же формы:

```python
Layout(
    MultiField(
        'Tell us your favorite stuff {{ username }}',
        Div(
            'like_website',
            'favorite_number',
            css_id = 'special-fields'
        ),
        'favorite_color',
        'favorite_food',
        'notes'
    )
)
```

На этот раз мы используем **MultiField**, объект макета, который, как правило, может использоваться в тех же местах, что и **Fieldset**. Основное отличие состоит в том, что при этом все поля отображаются в **div**, а когда при отправке формы есть ошибки, они отображаются в списке, а не по каждому полю, окружающему поле. Иногда лучший способ увидеть, что делают объекты макета, — это просто попробовать их и немного поиграть с ними.

## Атрибуты объектов макета

Для всех объектов макета вы можете установить **kwargs**, которые будут использоваться в качестве атрибутов **HTML**. Например, если вы хотите отключить автозаполнение для поля, вы можете сделать следующее:

```python
Field('field_name', autocomplete='off')
```

Если вы хотите установить атрибуты html со словами, разделенными дефисами, такими как **data-name**, поскольку Python не поддерживает дефисы в аргументах ключевых слов, а дефисы являются обычной записью в HTML, символы подчеркивания будут переведены в дефисы, поэтому вы должны сделать:

```python
Field('field_name', data_name="whatever")
```

Поскольку **class** является зарезервированным ключевым словом в Python, для него вам придется использовать **css\_class**. Например:

```python
Field('field_name', css_class="black-fields")
```

И атрибут **id** устанавливается с помощью **css\_id**:

```python
Field('field_name', css_id="custom_field_id")
```

## Объекты универсального макета

Они находятся в модуле **crispy\_forms.layout**. Это объекты макета, не относящиеся к пакету шаблонов. Мы пройдем по одному за другим, показывая примеры использования:

### Div

Оборачивает поля в тег `<div>`:

```python
Div('form_field_1', 'form_field_2', 'form_field_3', ...)
```

{% hint style="info" %}
В основном во всех объектах макета вы можете установить **kwargs**, которые будут использоваться в качестве атрибутов HTML. Поскольку **class** является зарезервированным ключевым словом в Python, для него вам придется использовать **css\_class**. Например:

```python
Div(
    'form_field_1',
    style="background: white;",
    title="Explication title",
    css_class="bigdivs"
)
```
{% endhint %}

### HTML

Очень мощный объект макета. Используйте его для рендеринга чистого HTML-кода. Фактически он ведет себя как шаблон Django и имеет доступ ко всему контексту страницы, на которой отображается форма. Этот объект макета не принимает никаких дополнительных параметров, кроме **html** для рендеринга, вы не можете установить атрибуты html, как в **Div**:

```python
HTML("{% raw %}
{% if success %} <p>Operation was successful</p> {% endif %}
{% endraw %}")
```

{% hint style="warning" %}
Помните, что это отображается в отдельном шаблоне, поэтому, если вы используете пользовательские теги шаблона или фильтры, не забудьте добавить свои `{% load custom_tags %}`
{% endhint %}

### Field

Чрезвычайно полезный объект макета. Вы можете использовать его для установки атрибутов в поле или отображения определенного поля с помощью настраиваемого шаблона. Таким образом вам не придется явно переопределять виджет поля и передавать уродливый словарь **attrs**:

```python
Field('password', id="password-field", css_class="passwordfields", title="Explanation")
Field('slider', template="custom-slider.html")
```

Этот объект макета можно использовать для простого расширения виджетов Django. Если вы хотите отобразить поле формы Django как _**скрытое**_, вы можете просто сделать:

```python
Field('field_name', type="hidden")
```

Если вам нужны атрибуты HTML5, вы можете легко сделать те, которые используют символы подчеркивания, **data\_name** kwarg здесь превратится в **data-name** в сгенерированном html:

```python
Field('field_name', data_name="special")
```

Поля в начальной загрузке заключены в `<div class="control-group">`. Вы можете установить дополнительные классы в этом **div**, для этого выполните:

```python
Field('field_name', wrapper_class="extra-class")
```

### Submit

Используется для создания кнопки отправки. Первый параметр — это атрибут **name** кнопки, второй параметр — это атрибут **value**:

```python
Submit('search', 'SEARCH')
```

Отображает в:

```html
<input type="submit" name="search" value="SEARCH" class="submit submitButton" id="submit-id-search" />
```

### Hidden

Используется для создания скрытого ввода:

```python
Hidden('name', 'value')
```

### Button

Создает кнопку:

```python
Button('name', 'value')
```

### Reset

Используется для создания ввода сброса:

```python
reset = Reset('name', 'value')
```

### Fieldset

Он оборачивает поля в `<fieldset>`. Первый параметр — это текст для легенды набора полей, как мы уже говорили, он ведет себя как шаблон Django:

```python
Fieldset("Text for the legend {{ username }}",
    'form_field_1',
    'form_field_2'
)
```

### ButtonHolder

Он оборачивает поля в `<div class="buttonHolder">`, это устаревший объект макета из пакета шаблонов **uni-form**:

```python
ButtonHolder(
    HTML('<span class="hidden">✓ Saved data</span>'),
    Submit('save', 'Save')
)
```

### MultiField

Он заключает поля в `<div>` с меткой label сверху. Когда в отправке формы есть ошибки, они отображаются в списке, а не в каждом поле, окружающем поле:

```python
MultiField("Text for the label {{ username }}",
    'form_field_1',
    'form_field_2'
)
```

## Объекты Bootstrap Layout

Они находятся в модуле **crispy\_forms.bootstrap**.

### FormActions

Оборачивает поля в `<div class="form-actions">`. Обычно используется для обертывания кнопок формы:

```python
FormActions(
    Submit('save', 'Save changes'),
    Button('cancel', 'Cancel')
)
```

<figure><img src="../../.gitbook/assets/form_actions.webp" alt=""><figcaption></figcaption></figure>

### AppendedText

Отображает ввод текста с добавлением **bootstrap**. Первый параметр — это имя поля **name**, в которое добавляется дополнительный текст, затем добавленный текст, который может быть похож на HTML. Существует необязательный параметр **active**, по умолчанию установленный в `False`, который вы можете установить в логическое значение, чтобы сделать добавленный текст активным. См. [input\_size](https://django-crispy-forms.readthedocs.io/en/latest/layouts.html#input-size), чтобы изменить размер этого ввода:

```python
# Синтаксис
AppendedText('field_name', 'appended text to show')
# Пример
AppendedText('field_name', '$', active=True)
```

<figure><img src="../../.gitbook/assets/appended_text.webp" alt=""><figcaption></figcaption></figure>

### PrependedText

Отображает ввод текста с добавлением **bootstrap**. Первый параметр — это имя поля **name**, в которое нужно добавить текст, а затем текст, который может быть похож на HTML. Существует необязательный параметр **active**, по умолчанию установленный в `False`, который вы можете установить в логическое значение, чтобы активировать предшествующий текст. См. [input\_size](https://django-crispy-forms.readthedocs.io/en/latest/layouts.html#input-size), чтобы изменить размер этого ввода:

```python
# Синтаксис
PrependedText('field_name', '<b>Prepended text</b> to show')
# Пример
PrependedText('field_name', '@', placeholder="username")
```

<figure><img src="../../.gitbook/assets/prepended_text.webp" alt=""><figcaption></figcaption></figure>

### PrependedAppendedText

Отображает комбинированный текст с добавлением спереди и сзади. Первый параметр — это имя поля **name**, затем добавляемый текст в начало и, наконец, добавляемый текст в конец. См. [input\_size](https://django-crispy-forms.readthedocs.io/en/latest/layouts.html#input-size), чтобы изменить размер этого ввода:

```python
PrependedAppendedText('field_name', '$', '.00'),
```

<figure><img src="../../.gitbook/assets/appended_prepended_text (1).webp" alt=""><figcaption></figcaption></figure>

### InlineCheckboxes

Отображает поле Django **forms.MultipleChoiceField**, используя встроенные флажки:

```python
InlineCheckboxes('field_name')
```

<figure><img src="../../.gitbook/assets/inline_checkboxes.webp" alt=""><figcaption></figcaption></figure>

### InlineRadios

Отображает поле Django **forms.ChoiceField** с его виджетом, установленным на **forms.RadioSelect**, используя встроенные переключатели:

```python
InlineRadios('field_name')
```

<figure><img src="../../.gitbook/assets/inline_radios.jpg" alt=""><figcaption></figcaption></figure>

### StrictButton

Он отображает кнопку, используя html-тег `<button>`, а не ввод **input**. По умолчанию для установлен тип **type** со значением **button**, а для класса **class** установлено значение **btn**:

```python
StrictButton("Button's content", name="go", value="go", css_class="extra")
StrictButton('Success', css_class="btn-success")
```

<figure><img src="../../.gitbook/assets/strict_button.webp" alt=""><figcaption></figcaption></figure>

### FieldWithButton

Вы можете создать ввод, связанный с кнопками:

```
# Размер поля можно настроить в пакете шаблонов Bootstrap4,
# передав класс модификатора размера в input_size.

FieldWithButtons('field_name', StrictButton("Go!"), input_size="input-group-sm")
```

<figure><img src="../../.gitbook/assets/field_with_buttons.webp" alt=""><figcaption></figcaption></figure>

### Tab & TabHolder

**Tab** отображает вкладку, разные вкладки должны быть обернуты в **TabHolder** для автоматического функционирования JavaScript, также вам понадобится **bootstrap-tab.js**, включенный в ваши статические файлы:

```python
TabHolder(
    Tab('First Tab',
        'field_name_1',
        Div('field_name_2')
    ),
    Tab('Second Tab',
        Field('field_name_3', css_class="extra")
    )
)
```

<figure><img src="../../.gitbook/assets/tab_and_tabholder.jpg" alt=""><figcaption></figcaption></figure>

### Accordion & AccordionGroup

**AccordionGroup** отображает панель аккордеона, различные группы должны быть заключены в **Accordeon** для автоматического функционирования JavaScript, также вам понадобится **bootstrap-tab.js**, включенный в ваши статические файлы:

```python
Accordion(
    AccordionGroup('First Group',
        'radio_buttons'
    ),
    AccordionGroup('Second Group',
        Field('field_name_3', css_class="extra")
    )
)
```

<figure><img src="../../.gitbook/assets/accordiongroup_and_accordion.jpg" alt=""><figcaption></figcaption></figure>

### Alert

**Alert** генерирует разметку в виде диалогового окна предупреждения:

```python
Alert(
    content="<strong>Warning!</strong> Best check yo self, you're not looking too good."
)
```

<figure><img src="../../.gitbook/assets/alert.webp" alt=""><figcaption></figcaption></figure>

### UneditableField

**UneditableField** отображает отключенное поле, используя класс **uneditable-input** начальной загрузки:

```python
UneditableField('text_input', css_class='form-control-lg')
```

<figure><img src="../../.gitbook/assets/field_disabled.webp" alt=""><figcaption></figcaption></figure>

### Modal

**Modal** отображает свои поля внутри модального окна начальной загрузки, которое можно настроить с помощью **kwargs** при инициализации. См. документы bootstrap для получения дополнительных примеров модальных окон и того, как управлять вашим модальным окном [с помощью атрибутов](https://getbootstrap.com/docs/4.0/components/modal/#via-data-attributes) или [с помощью javascript](https://getbootstrap.com/docs/4.0/components/modal/#via-javascript). Поддерживает только **Bootstrap v3** или выше:

```python
Layout(
    Modal(
        # email.help_text был установлен во время инициализации поля формы django
        Field('email', placeholder="Email", wrapper_class="mb-0"),
        Button(
            "submit",
            "Send Reset Email",
            id="email_reset",
            css_class="btn-primary mt-3",
            onClick="someJavasciptFunction()", # используется для отправки формы
        ),
        css_id="my_modal_id",
        title="This is my modal",
        title_class="w-100 text-center",
    )
)
```

<figure><img src="../../.gitbook/assets/modal.webp" alt=""><figcaption></figcaption></figure>

### Размер группы input

По умолчанию используются стандартные размеры ввода **Bootstrap**. Чтобы настроить размер группы ввода (**AppendedText**, **PrependedText**, **PrependedAppendedText**), добавьте соответствующий класс CSS:

```python
# Bootstrap 3 - Входы (input) и интервалы (span) требуют класса размера.
# Используйте `css_class`.
PrependedText('field_name', StrictButton("Go!"), css_class="input-sm")
PrependedText('field_name', StrictButton("Go!"), css_class="input-lg")

# Bootstrap 4 - Обертка div требует класса размера. Используйте `input_size`.
PrependedText('field_name', StrictButton("Go!"), input_size="input-group-sm")
PrependedText('field_name', StrictButton("Go!"), input_size="input-group-lg")
```

## Переопределение шаблонов объектов макета

Упомянутый набор [объектов универсального макета](layouts-makety.md#obekty-universalnogo-maketa) был тщательно разработан, чтобы быть гибким, совместимым со стандартами и поддерживать функции формы Django. Каждый объект **Layout** связан с другим шаблоном, который находится в каталоге `templates/{{ TEMPLATE_PACK_NAME }}/layout/`.

Некоторые опытные пользователи могут захотеть использовать свои собственные шаблоны, чтобы адаптировать объекты макета к своему использованию или потребностям. Существует три способа переопределения шаблона, используемого объектом макета.

**Глобально**: вы переопределяете шаблон объекта макета для всех экземпляров этого объекта макета, который вы используете:

```python
from crispy_forms.layout import Div
Div.template = 'my_div_template.html'
```

**Индивидуально**: Вы можете переопределить шаблон для определенного объекта макета в макете:

```python
Layout(
    Div(
        'field1',
        'field2',
        template='my_div_template.html'
    )
)
```

**Переопределение каталога шаблонов**: это означает имитацию структуры каталогов **crispy-forms** в вашем проекте, а затем копирование шаблонов, которые вы хотите переопределить, и, наконец, редактирование этих копий. Если вы используете этот подход, лучше просто копировать и редактировать шаблоны, которые вы будете настраивать, а не все.

## Переопределение шаблонов проектов

Вам нужно различать шаблоны объектов макета и шаблоны **django-crispy-forms**. Есть несколько шаблонов, которые находятся в `templates/{{ TEMPLATE_PACK_NAME }}`, которые определяют структуру формы/набора форм, способ отображения поля или ошибок и т. д. Они добавляют очень мало логики и в основном являются базовыми оболочками для остальной части **django-crispy-forms**. Чтобы переопределить их, у вас есть два варианта:

Атрибуты **template** и **field\_template** в **FormHelper**: начиная с версии `1.3.0` вы можете переопределить шаблон формы/формы и шаблон поля, используя вспомогательные атрибуты, см. раздел [Атрибуты помощника helper](formhelper.md#vspomogatelnye-atributy-helper-kotorye-vy-mozhete-ustanovit), которые вы можете установить. При этом вы можете изменить одну конкретную форму или все формы вашего проекта (например, создав собственный базовый класс **FormHelper**).

**Переопределение каталога шаблонов**: это работает так же, как описано в разделе [Переопределение шаблонов объектов макета](layouts-makety.md#pereopredelenie-shablonov-obektov-maketa). Если вы адаптируете шаблоны **crispy-forms** к используемому вами популярному пакету шаблонов с открытым исходным кодом, отправьте его, чтобы больше людей могли извлечь из него пользу.

**Создание TEMPLATE PACK**: Возможно, вы захотите использовать **crispy-forms** с вашей любимой структурой CSS или CSS вашей компании. Для этого вам нужно хорошо знать **crispy-forms**, объекты макета и их шаблоны. Вы, вероятно, захотите начать с одного из существующих пакетов шаблонов, возможно, **bootstrap**. Представьте, что ваш пакет шаблонов называется **chocolate**, это означает, что вы, вероятно, хотите, чтобы ваш корневой каталог назывался так же. Для использования вашего пакета шаблонов вам необходимо установить переменную `CRISPY_TEMPLATE_PACK = 'chocolate'` в вашем файле настроек, а также установить `CRISPY_ALLOWED_TEMPLATE_PACKS = ('bootstrap', 'chocolate')`. Таким образом, **crispy-forms** будет знать, что вы хотите использовать свой собственный пакет шаблонов, который является разрешенным, и где его искать.

## Создание собственных объектов макета

[Объекты универсального макета](layouts-makety.md#obekty-universalnogo-maketa), связанные с **django-crispy-forms**, представляют собой набор наиболее часто встречающихся компонентов, которые создают форму. Вы, вероятно, сможете сделать все, что вам нужно, комбинируя их. В любом случае, вы можете захотеть создать свои собственные компоненты, для этого вам понадобится хорошее понимание **django-crispy-forms**. Каждый объект макета должен иметь метод с именем **render**. Его прототип должен быть:

```python
def render(self, form, context):
```

Официальные объекты макета находятся в **layout.py** и **bootstrap.py**, вы можете взглянуть на них, чтобы полностью понять, как действовать дальше. Но в общих чертах объект макета — это шаблон, отображаемый с некоторыми переданными параметрами.

Если вы придумали хорошую идею и разработали объект макета, который, по вашему мнению, может быть полезен другим, пожалуйста, откройте проблему или отправьте запрос на включение, чтобы **django-crispy-forms** стал лучше.

## Составление (композиция) макетов

Представьте, что у вас есть несколько форм, которые используют один и тот же макет. Существует простой способ создать макет **Layout**, повторно использовать и расширять его. Вы можете иметь **Layout** как компонент другого **Layout**. Вы можете построить этот общий фрагмент разными способами. Как отдельный класс:

```python
class CommonLayout(Layout):
    def __init__(self, *args, **kwargs):
        super().__init__(
            MultiField("User data",
                'username',
                'lastname',
                'age'
            )
        )
```

Возможно, экземпляр объекта достаточно хорош:

```python
common_layout = Layout(
    MultiField("User data",
        'username',
        'lastname',
        'age'
    )
)
```

Затем вы можете сделать:

```python
helper.layout = Layout(
    CommonLayout(),
    Div(
        'favorite_food',
        'favorite_bread',
        css_id = 'favorite-stuff'
    )
)
```

Или:

```python
helper.layout = Layout(
    common_layout,
    Div(
        'professional_interests',
        'job_description',
    )
)
```

Мы определили макет и использовали его как фрагмент другого макета, что означает, что эти два макета будут начинаться одинаково, а затем расширять макет по-разному.
