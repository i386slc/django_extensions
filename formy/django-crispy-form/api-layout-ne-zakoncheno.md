# API Layout (не закончено)

## _class_ layout.BaseInput(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#BaseInput).

Базовый класс для уменьшения объема кода в классах ввода.

#### Параметры

* **name** (_str_) - Атрибут name кнопки.
* **value** (_str_) - Атрибут value кнопки.
* **css\_id** (_str_) (опционально) - Пользовательский идентификатор DOM для объекта макета. Если он не указан, аргумент имени зашифровывается и превращается в идентификатор кнопки отправки. По умолчанию `None`.
* **css\_class** (_str_) (опционально) - Дополнительные классы CSS для применения к `<input>`. По умолчанию `None`.
* **template** (_str_) (опционально) - Переопределяет шаблон по умолчанию, если он предоставлен. По умолчанию `None`.
* **\*\*kwargs** (_dict_) (опционально) - Дополнительные атрибуты передаются в **flatatt** и преобразуются в пары `key=”value”`. Эти атрибуты добавляются в файл `<input>`.

#### Атрибуты

* **template** (_str_) - Шаблон по умолчанию, с которым будет отображаться этот объект макета.

#### Методы

**render(**_**form**_**, **_**context**_**, **_**template\_pack=\<SimpleLazyObject: 'bootstrap4'>**_**, **_**\*\*kwargs**_**)**

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#BaseInput.render).

Отображает `<input />` если контейнер используется как объект **Layout**. Значение кнопки ввода может быть переменной в контексте.

## _class_ layout.Button(_name_, _value_, _\*_, _css\_id=None_, _css\_class=None_, _template=None_, _\*\*kwargs_)

[Исходник](https://django-crispy-forms.readthedocs.io/en/latest/\_modules/layout.html#Button).

Используется для создания дескриптора кнопки для тега шаблона `{% crispy %}` .

#### Параметры

*
