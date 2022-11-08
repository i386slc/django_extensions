# Тег \{% crispy %\} с формами

**django-crispy-forms** реализует класс **FormHelper**, который определяет поведение рендеринга формы. Помощники (helpers) дают вам возможность управлять атрибутами формы и ее макетом, делая это программным способом с использованием Python. Таким образом, вы пишете как можно меньше HTML, а вся ваша логика остается в файлах форм и представлений.

## Основы

В оставшейся части этого документа мы будем использовать следующий пример формы, показывающий, как использовать помощника **helper**. Эта форма отвечает за сбор некоторой информации о пользователе:

```python
class ExampleForm(forms.Form):
    like_website = forms.TypedChoiceField(
        label = "Do you like this website?",
        choices = ((1, "Yes"), (0, "No")),
        coerce = lambda x: bool(int(x)),
        widget = forms.RadioSelect,
        initial = '1',
        required = True,
    )

    favorite_food = forms.CharField(
        label = "What is your favorite food?",
        max_length = 80,
        required = True,
    )

    favorite_color = forms.CharField(
        label = "What is your favorite color?",
        max_length = 80,
        required = True,
    )

    favorite_number = forms.IntegerField(
        label = "Favorite number",
        required = False,
    )

    notes = forms.CharField(
        label = "Additional notes or feedback",
        required = False,
    )
```

Давайте посмотрим, как работают помощники, шаг за шагом, с объяснением некоторых примеров. Сначала вам нужно будет импортировать **FormHelper**:

```python
from crispy_forms.helper import FormHelper
```

Вашим помощником (helper) может быть переменная уровня класса или переменная уровня экземпляра, если вы не знаете, что это значит, вы можете прочитать статью «[Будьте осторожны при использовании статических переменных в формах](https://tothinkornottothink.com/post/7157151391/be-careful-how-you-use-static-variables-in-forms)». Как правило, если вы не собираетесь манипулировать **FormHelper** в своем коде, например, в представлении, вам следует использовать _статический помощник_, в противном случае вам следует использовать _помощник уровня экземпляра_. Если вы все еще не понимаете тонкие различия между ними, используйте помощник уровня экземпляра, потому что вы не столкнетесь с побочными эффектами. Как и в следующих шагах, я покажу вам, как манипулировать хелпером формы, поэтому мы создадим хелпер уровня экземпляра. Вот как вы это сделаете:

```python
from crispy_forms.helper import FormHelper

class ExampleForm(forms.Form):
    [...]
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
```

Как видите, вам нужно вызвать конструктор базового класса с помощью **super** и переопределить конструктор. Этот помощник не устанавливает никаких атрибутов формы, поэтому он бесполезен. Давайте посмотрим, как настроить некоторые основные атрибуты **FormHelper**:

```python
from crispy_forms.helper import FormHelper
from crispy_forms.layout import Submit

class ExampleForm(forms.Form):
    [...]
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.helper = FormHelper()
        self.helper.form_id = 'id-exampleForm'
        self.helper.form_class = 'blueForms'
        self.helper.form_method = 'post'
        self.helper.form_action = 'submit_survey'

        self.helper.add_input(Submit('submit', 'Submit'))
```

Обратите внимание, что мы импортируем класс **Submit**, который является объектом макета (layout). Позже мы подробно увидим, что представляют собой объекты макета, а сейчас просто скажем, что это добавляет кнопку отправки в нашу форму, чтобы люди могли отправить свой опрос.

Мы также использовали вспомогательную магию. **FormHelper** имеет список атрибутов, которые можно установить, которые _влияют в основном на атрибуты формы_. Наша форма будет иметь <mark style="color:orange;">идентификатор DOM</mark> **id-exampleForm**, она будет иметь <mark style="color:orange;">класс DOM CSS</mark> **blueForms**, она будет использовать <mark style="color:orange;">http POST</mark> для отправки информации, и ее <mark style="color:orange;">действие action</mark> будет установлено на `reverse(submit_survey)`.

Давайте посмотрим, как отобразить форму в шаблоне. Предположим, что у нас есть форма в контексте шаблона как **example\_form**, мы бы отрендерили ее следующим образом:

```django
{% raw %}
{% load crispy_forms_tags %}
{% crispy example_form example_form.helper %}
{% endraw %}
```

Обратите внимание, что теги `{% crispy %}` ожидают _**два параметра**_: сначала _переменную формы_, а затем _вспомогательную функцию_. В этом случае мы используем **FormHelper**, прикрепленный к форме, но вы также можете создать экземпляр **FormHelper** и передать его как контекстную переменную. В большинстве случаев вы захотите использовать _прикрепленный помощник_. Обратите внимание, что если вы назовете свой помощник helper'ом как атрибут **FormHelper**, вам нужно будет сделать только:

```django
{% raw %}
{% crispy form %}
{% endraw %}
```

С пакетом шаблонов **Bootstrap 4** по умолчанию это именно этот **html**, который вы получите:

```html
<form action="submit_survey" class="blueForms" id="id-exampleForm" method="post">
    <input name="csrfmiddlewaretoken" type="hidden" value="evU93ufHyzX5dP5h5hgOaq96zIj8c02X">
    <div id="div_id_like_website" class="form-group">
        <label class="requiredField"> Do you like this website?<span class="asteriskField">*</span> </label>
        <div action="submit_survey" class="blueForms" id="id-exampleForm">
            <div class="custom-control custom-radio">
                <input type="radio" class="custom-control-input" name="like_website" value="1" id="id_like_website_0" required checked /> <label class="custom-control-label" for="id_like_website_0"> Yes </label>
            </div>
            <div class="custom-control custom-radio">
                <input type="radio" class="custom-control-input" name="like_website" value="0" id="id_like_website_1" required /> <label class="custom-control-label" for="id_like_website_1"> No </label>
            </div>
        </div>
    </div>
    <div id="div_id_favorite_food" class="form-group">
        <label for="id_favorite_food" class="requiredField"> What is your favorite food?<span class="asteriskField">*</span> </label>
        <div><input type="text" name="favorite_food" maxlength="80" class="textinput inputtext form-control" required id="id_favorite_food" /></div>
    </div>
    <div id="div_id_favorite_color" class="form-group">
        <label for="id_favorite_color" class="requiredField"> What is your favorite color?<span class="asteriskField">*</span> </label>
        <div><input type="text" name="favorite_color" maxlength="80" class="textinput inputtext form-control" required id="id_favorite_color" /></div>
    </div>
    <div id="div_id_favorite_number" class="form-group">
        <label for="id_favorite_number" class=""> Favorite number </label>
        <div><input type="number" name="favorite_number" class="numberinput form-control" id="id_favorite_number" /></div>
    </div>
    <div id="div_id_notes" class="form-group">
        <label for="id_notes" class=""> Additional notes or feedback </label>
        <div><input type="text" name="notes" class="textinput inputtext form-control" id="id_notes" /></div>
    </div>
    <div class="form-group">
        <div class=""><input type="submit" name="submit" value="Submit" class="btn btn-primary" id="submit-id-submit" /></div>
    </div>
</form>
```

То, что вы получите, — это форма, представленная в виде HTML с потрясающими битами. Конкретно…

Открытие и закрытие тегов формы с идентификатором, классом, действием и методом, установленными как в помощнике:

```html
<form action="submit_survey" class="blueForms" id="id-exampleForm" method="post">
    [...]
</form>
```

CSRF Django контроль:

```html
<input name="csrfmiddlewaretoken" type="hidden" value="evU93ufHyzX5dP5h5hgOaq96zIj8c02X">
```

Кнопка отправки:

```html
<div class="form-group">
    <div class=""><input type="submit" name="submit" value="Submit" class="btn btn-primary" id="submit-id-submit" /></div>
</div>
```

## Управление помощником helper в представлении view

Давайте посмотрим, как мы можем изменить любое вспомогательное свойство в представлении:

```python
@login_required()
def inbox(request, template_name):
    example_form = ExampleForm()
    redirect_url = request.GET.get('next')

    # Логика работы с формой
    [...]

    if redirect_url is not None:
        example_form.helper.form_action = reverse('submit_survey') + '?next=' + redirectUrl

    return render_to_response(
        template_name,
        {'example_form': example_form},
        context_instance=RequestContext(request)
    )
```

Мы меняем вспомогательное свойство **form\_action** на случай, если представление было вызвано со параметром GET **next**.

## Отрисовка нескольких форм с хелперами

Часто нас спрашивают: «Как сделать рендеринг двух или более форм с соответствующими хелперами, используя тег `{% crispy %}`, без двойного рендеринга `<form>` тегов?» Легко, вам нужно установить свойство помощника **form\_tag** в `False` в каждом помощнике:

```python
self.helper.form_tag = False
```

Затем вам нужно будет написать небольшой html-код, окружающий формы:

```django
<form action="{% raw %}
{% url 'submit_survey' %}" class="my-class" method="post">
    {% crispy first_form %}
    {% crispy second_form %}
{% endraw %}
</form>
```

Вы можете прочитать [список атрибутов помощника, которые вы можете установить](https://django-crispy-forms.readthedocs.io/en/latest/form\_helper.html#helper-attributes), и для чего они нужны.

## Измените `«*»` обязательные поля

Если вам не нравится использование `'*'` (звездочка) для обозначения обязательных полей, у вас есть два варианта:

Звездочки имеют набор классов **asteriskField**. Таким образом, вы можете скрыть его с помощью правила CSS:

```css
.asteriskField {
    display: none;
}
```

Или замените шаблон `field.html` на собственный.

## Сделайте так, чтобы crispy формы терпели неудачу громко

## Изменить `<input>` классы по умолчанию для crispy-forms

## Визуализация формы в коде Python

## Рецепт проверки AJAX

## Bootstrap горизонтальные формы

## Встроенные формы Bootstrap
