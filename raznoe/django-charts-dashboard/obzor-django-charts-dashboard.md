# Обзор django-charts-dashboard

## Документация

Полная документация находится на [https://django-charts-dashboard.readthedocs.io/en/latest/](https://django-charts-dashboard.readthedocs.io/en/latest/)

## Быстрый старт

Установите django-charts-dashboard:

```bash
pip install django-charts-dashboard
```

или:

```bash
pipenv install django-charts-dashboard
```

Добавьте его в свои **INSTALLED\_APPS**:

```python
INSTALLED_APPS = (
    ...
    'charts_dashboard',
    ...
)
```

**PS: вам нужно определить библиотеки jquery и chartjs в вашем скрипте раздела html.**

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.5.0/jquery.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.9.3/Chart.bundle.js"></script>
```

## Запуск тестов

Код действительно работает?

```bash
source <YOURVIRTUALENV>/bin/activate
(myenv) $ pip install tox
(myenv) $ tox
```
