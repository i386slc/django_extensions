# Установка django-crispy-forms

## Процесс установки django-crispy-forms

Установите последнюю стабильную версию в свою среду Python, используя **pip**:

```bash
pip install django-crispy-forms
```

Если вы хотите установить разрабатываемую версию (нестабильную), вы можете сделать это следующим образом:

```bash
pip install git+git://github.com/django-crispy-forms/django-crispy-forms.git@main#egg=django-crispy-forms
```

Или, если вы хотите установить разрабатываемую версию как репозиторий git (чтобы вы могли получать обновления `git pull`), используйте флаг `-e` с установкой `pip install`, например:

```bash
pip install -e git+git://github.com/django-crispy-forms/django-crispy-forms.git@main#egg=django-crispy-forms
```

После установки добавьте **crispy\_forms** в **INSTALLED\_APPS** в `settings.py`:

```python
INSTALLED_APPS = (
    ...
    'crispy_forms',
)
```

В рабочих средах всегда активируйте загрузчик кеша шаблонов Django. Это доступно, начиная с **Django 1.2**, и то, что он делает, в основном загружает шаблоны один раз, а затем кэширует результат для каждого последующего рендеринга. Это приводит к значительному улучшению производительности. Чтобы узнать, как его настроить, обратитесь к потрясающей [странице документации Django](https://docs.djangoproject.com/en/2.2/ref/templates/api/#django.template.loaders.cached.Loader).

## Пакеты шаблонов

Начиная с версии **2.0**, **django-crispy-forms** имеет встроенную поддержку версий 3 и 4 CSS-фреймворка **Bootstrap**.

Версия **1.x** также предоставляла пакеты шаблонов для:

* `bootstrap` [Bootstrap](https://getbootstrap.com/) — это пакет шаблонов по умолчанию для **crispy-forms**, версия 2 популярного простого и гибкого HTML, CSS и Javascript для пользовательских интерфейсов из Twitter.
* `uni-form` [Uni-form](https://github.com/draganbabic/uni-form/tree/uni-form-v-1-5) — это красиво выглядящие, хорошо структурированные, легко настраиваемые, доступные и удобные формы.

Кроме того, в отдельно поддерживаемых проектах доступны следующие пакеты шаблонов.

* `foundation` [Foundation](https://get.foundation/) По словам создателя, «Самая продвинутая адаптивная интерфейсная среда в мире». Этот пакет шаблонов доступен на сайте [crispy-forms-foundation](https://github.com/sveetch/crispy-forms-foundation).
* `tailwind` [Tailwind](https://tailwindcss.com/) Утилита первого фреймворка. Этот пакет шаблонов доступен через [crispy-tailwind](https://github.com/django-crispy-forms/crispy-tailwind).
* `Bootstrap 5` Поддержка более новых версий Bootstrap будет осуществляться в отдельных пакетах шаблонов. Это начинается с версии 5 и доступно через [crispy-bootstrap5](https://github.com/django-crispy-forms/crispy-bootstrap5).
* `Bulma` [Bulma](https://bulma.io/): современный CSS-фреймворк, который просто работает. Этот пакет шаблонов доступен через [crispy-bulma](https://github.com/ckrybus/crispy-bulma).

Если ваша структура CSS формы не поддерживается и у нее открытый исходный код, вы можете создать проект `crispy-forms-templatePackName`. Пожалуйста, дайте мне знать, чтобы я мог дать ссылку на него. Предоставляется документация о том,[ как создавать собственные пакеты шаблонов](https://django-crispy-forms.readthedocs.io/en/latest/template\_packs.html#template-packs).

Вы можете установить пакет шаблонов по умолчанию для своего проекта, используя переменную настроек **CRISPY\_TEMPLATE\_PACK** Django `settings.py`:

```python
CRISPY_TEMPLATE_PACK = 'uni_form'
```

Пожалуйста, проверьте документацию вашего пакета шаблонов, чтобы узнать правильное значение параметра **CRISPY\_TEMPLATE\_PACK** (существуют пакеты, которые предоставляют более одного пакета шаблонов).

## Настройка media файлов

**crispy-forms** не включает статические файлы. Вам нужно будет самостоятельно включить соответствующие медиафайлы в зависимости от того, какую структуру CSS (пакет шаблонов) вы используете. Это может включать один или несколько файлов **CSS** и **JS**. Прочтите документацию по фреймворку CSS, чтобы узнать, как его настроить.
