# Пример BOOTSTRAP\_DATEPICKER\_PLUS

Чтобы настроить внешний вид и функции, скопируйте [блок настроек](https://github.com/monim67/django-bootstrap-datepicker-plus/blob/5.0.0/dev/mysite/settings.py#L140-L250) в файл **settings.py** и настройте его, следуя комментариям к инструкции. Настройки применяются глобально ко всем виджетам, используемым на вашем сайте.

Это пример разработчиков из Github:

````python
BOOTSTRAP_DATEPICKER_PLUS = {
    # Параметры для ВСЕХ объявленных виджетов ввода
    # Больше параметров: https://getdatepicker.com/4/Options/
    "options": {
        # "locale": "bn",
        "showClose": True,
        "showClear": True,
        "showTodayButton": True,
        "allowInputToggle": True,
    },
    # Вы можете установить параметры перехвата даты и события с помощью JavaScript,
    # использование в README.
    # Вы также можете установить параметры только для определенных вариантов виджетов,
    # которые переопределяют указанные выше параметры.
    "variant_options": {
        # "date": {
        #     "format": "MM/DD/YYYY",
        # },
        # "datetime": {
        #     "format": "MM/DD/YYYY HH:mm",
        # },
        "month": {
            "format": "MMMM, YYYY",
        },
    },
    #
    # HTML атрибуты для элемента <input> виджета
    # "attrs": {
    #     "class": "input",
    # },
    #
    # Переопределить классы иконок надстроек ввода
    "addon_icon_classes": {
        "month": "bi-calendar-month",
    },
    #
    # HTML шаблон для отображения ввода html
    # пример: https://github.com/monim67/django-bootstrap-datepicker-plus/blob/5.0.0/dev/myapp/templates/myapp/custom-input.html
    #
    # "template_name": "your-app/custom-input.html",
    #
    # Дополнительно: выберите, где будут обслуживаться статические файлы JS/CSS.
    # по умолчанию: https://github.com/monim67/django-bootstrap-datepicker-plus/blob/5.0.0/src/bootstrap_datepicker_plus/settings.py#L16
    # Чтобы обслуживать из любого другого предпочитаемого CDN,
    # просто измените параметры ниже.
    # Вы также можете установить для них значение None,
    # если у вас уже есть следующие ресурсы, включенные в ваш шаблон.
    #
    # "datetimepicker_js_url": "https://..",
    # "datetimepicker_css_url": "https://..",
    # "momentjs_url": None,  # Если вы уже добавили momentjs в свой шаблон
    # "bootstrap_icon_css_url": None,  # Если вам не нужны иконки Bootstrap
    #
    # Если вы хотите обслуживать статические файлы самостоятельно без CDN
    # (из staticfiles) и знаете, как обслуживать статические файлы django
    # на рабочем сервере (DEBUG=False)
    # Затем загрузите файлы js/css в любой из ваших статических каталогов,
    # обновите URL-адреса js/css выше и установите следующую опцию.
    #
    # "app_static_url": "bootstrap_datepicker_plus/",
}
```
````
