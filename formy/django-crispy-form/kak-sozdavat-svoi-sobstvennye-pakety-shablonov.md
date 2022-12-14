# Как создавать свои собственные пакеты шаблонов

Сначала вам нужно будет назвать свой пакет шаблонов, для этого вы **не можете** использовать имя одного из доступных пакетов шаблонов в **crispy-forms** из-за коллизии имен. Например, предположим, что в компании, в которой мы работаем, дизайнер придумал загрузочную CSS-bootstrap, известную как **chocolate**. У компании есть проект Django, который должен начать использовать **chocolate**, поэтому нам нужно создать папку с именем **chocolate** в нашем каталоге шаблонов. Проверьте настройку **TEMPLATE\_DIRS** в Django и выберите предпочтительный путь.

Как только мы создадим эту папку, нам нужно будет создать конкретную иерархию каталогов, чтобы crispy-forms могли ее подобрать. Вот как выглядит пакет шаблонов начальной загрузки (v2):

```bash
.
├── accordion-group.html
├── accordion.html
├── betterform.html
├── display_form.html
├── errors.html
├── errors_formset.html
├── * field.html
├── layout
│   ├── alert.html
│   ├── * baseinput.html
│   ├── button.html
│   ├── checkboxselectmultiple.html
│   ├── checkboxselectmultiple_inline.html
│   ├── div.html
│   ├── field_errors.html
│   ├── field_errors_block.html
│   ├── field_with_buttons.html
│   ├── fieldset.html
│   ├── formactions.html
│   ├── help_text.html
│   ├── help_text_and_errors.html
│   ├── multifield.html
│   ├── prepended_appended_text.html
│   ├── radioselect.html
│   ├── radioselect_inline.html
│   ├── tab-link.html
│   ├── tab.html
│   └── uneditable_input.html
├── table_inline_formset.html
├── * uni_form.html
├── uni_formset.html
├── * whole_uni_form.html
└── whole_uni_formset.html
```

Успокойтесь, не паникуйте, нам не понадобится столько шаблонов для нашего пакета шаблонов. Шаблоны также довольно просты в использовании, если вы понимаете, какую проблему решает crispy-forms. Минимумом будут шаблоны, отмеченные звездочкой.

## Основы

Во-первых, поскольку это **crispy-forms 1.5.0**, пакеты шаблонов являются автономными, вы не можете ссылаться на шаблон из другого пакета шаблонов.

**crispy-forms** имеет много функций, но, возможно, вам не нужен пакет шаблонов, чтобы охватить их все. Тег шаблонов `{% crispy %}` отображает формы, используя глобальную структуру, содержащуюся в файле **whole\_uni\_form.html**. Однако фильтр `|crispy` использует **uni\_form.html**. Как вы, наверное, догадались, название шаблонов происходит из старых времен **django-uni-form**. Во всяком случае, например, если мы не используем фильтр `|crispy`, нам не нужно поддерживать шаблон **uni\_form.html** в нашем пакете шаблонов.

Если мы планируем использовать наборы форм formsets + `{% crispy %}`, нам понадобится файл **whole\_uni\_formset.html**, вместо этого, если мы используем наборы форм formsets + `|crispy`, нам понадобится файл **uni\_formset.html**.

Все эти шаблоны используют тег с именем `{% crispy_field %}`, который загружается с помощью `{% load crispy_forms_field %}`, который генерирует HTML для `<input>` с использованием шаблона **field.html**, но предварительно выполняет подготовку Python. Если вам интересно, код этого тега находится в **crispy\_forms.templatetags.crispy\_forms\_field** вместе с некоторыми другими вещами.

Таким образом, пакет шаблонов для очень простого примера, охватывающего только формы и использование тега `{% crispy %}`, потребует 2 шаблона: **whole\_uni\_form.html** и **field.html**. Ну, это не совсем так, потому что к каждому объекту макета прикреплен шаблон. Итак, если мы хотим использовать **Div**, нам понадобится **div.html**. Некоторые не так очевидны, если вам нужен **Submit**, вам понадобится **baseinput.html**. Некоторые объекты макета на самом деле _**не имеют прикрепленного шаблона**_, например **HTML**.

В предыдущем дереве шаблонов есть несколько шаблонов, предназначенных для DRY целей, они на самом деле не являются обязательными или частью объекта макета, так что не беспокойтесь слишком сильно.

## Начинаем

Теперь лучше всего начать копировать некоторые или все шаблоны из существующего пакета шаблонов **crispy-forms**, такого как **bootstrap3**, а затем удалить те, которые вам не нужны. Следующий шаг — отредактировать эти шаблоны и настроить **HTML/CSS**, чтобы они соответствовали **chocolate**, что иногда означает удаление/добавление div, классов и других вещей. Вы всегда можете создать форму в своем приложении с помощником, прикрепленным к этому новому пакету шаблонов, и сразу же начать опробовать свою адаптацию.

В настоящее время существует пакет шаблонов для **crispy-forms**, который не находится в ядре, разработанный Дэвидом Теноном как внешнее подключаемое приложение с именем [crispy-forms-foundation](https://github.com/sveetch/crispy-forms-foundation), это также хороший справочник для ознакомления.

Помните, что **crispy-forms** развивается и добавляет новые атрибуты **FormHelper.attributes**, если вы хотите использовать их в будущем, вам придется адаптировать свои шаблоны, добавляя эти переменные и их обработку.
