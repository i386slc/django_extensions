# Запуск набора тестов

Самый простой способ запустить тесты **django-filter** — проверить исходный код и создать виртуальную среду, в которой вы можете установить тестовые зависимости. Django-filter использует настраиваемый модуль запуска тестов для настройки среды, поэтому доступен скрипт-оболочка для настройки и запуска набора тестов.

{% hint style="info" %}
Далее предполагается, что у вас установлены [virtualenv](https://virtualenv.pypa.io/en/stable/) и [git](https://git-scm.com/).
{% endhint %}

## Клонируйте репозиторий

Получите исходный код с помощью следующей команды:

```bash
$ git clone https://github.com/carltongibson/django-filter.git
```

Перейдите в каталог **django-filter**:

```bash
$ cd django-filter
```

## Настройте виртуальную среду virtualenv

Создайте новую виртуальную среду для запуска набора тестов:

```bash
$ virtualenv venv
```

Затем активируйте **virtualenv** и установите тестовые требования **requirements**:

```bash
$ source venv/bin/activate
$ pip install -r requirements/test.txt
```

## Выполните test runner

Запустите тесты с помощью скрипта runner:

```bash
$ python runtests.py
```

## Протестируйте все поддерживаемые версии

Вы также можете использовать отличный инструмент для тестирования **tox** для запуска тестов со всеми поддерживаемыми версиями Python и Django. Установите **tox**, а затем просто запустите:

```bash
$ pip install tox
$ tox
```

## Уборка

Утилита **isort** используется для поддержки импорта модулей. Вы можете протестировать импорт модуля с помощью соответствующей среды **tox** или непосредственно с помощью **isort**.

```bash
$ pip install tox
$ tox -e isort

# или

$ pip install isort
$ isort --check --diff django_filters tests
```

Чтобы отсортировать импорт, просто удалите опцию `--check-only`.

```python
$ isort --recursive django_filters tests
```
