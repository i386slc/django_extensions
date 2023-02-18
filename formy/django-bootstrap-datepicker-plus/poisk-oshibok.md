# Поиск ошибок

Если календарь выбора даты **date-picker** не отображается, возможно, вы что-то пропустили в процессе установки. Проверьте следующие ошибки, и вы можете найти свое решение.

## Ошибки, отображаемые на экране браузера

{% hint style="danger" %}
**TemplateSyntaxError: bootstrap3/4 is not a registered tag library.**

TemplateSyntaxError: bootstrap3/4 не является зарегистрированной библиотекой тегов.
{% endhint %}

Это означает, что вы не установили **django-bootstrap3** и не добавили его в список **INSTALLED\_APPS**. Ознакомьтесь с нашими [инструкциями по настройке](kratkoe-rukovodstvo.md), чтобы узнать, как это сделать.

{% hint style="danger" %}
**TemplateDoesNotExist bootstrap\_datepicker\_plus/date-picker.html**
{% endhint %}

Это означает, что вы не установили **django-bootstrap-datepicker-plus** и не добавили его в список **INSTALLED\_APPS**. Ознакомьтесь с нашими [инструкциями по настройке](kratkoe-rukovodstvo.md), чтобы узнать, как это сделать.

{% hint style="danger" %}
**TemplateSyntaxError: Invalid block tag ‘bootstrap\_form’.**

TemplateSyntaxError: Неверный тег блока ‘bootstrap\_form’.
{% endhint %}

Вы не загрузили тег **bootstrap3/bootstrap4**. Ознакомьтесь с нашими [инструкциями по настройке](kratkoe-rukovodstvo.md), чтобы узнать, как это сделать.

## Ошибки, отображаемые в консоли браузера

Иногда страница загружается нормально, но ошибки регистрируются в консоли разработчика браузера.

{% hint style="danger" %}
**Uncaught TypeError: Cannot read property ‘fn’ of undefined**

Неперехваченная TypeError: Невозможно прочитать свойство «fn»; не определено
{% endhint %}

{% hint style="danger" %}
**Uncaught Error: Bootstrap’s JavaScript requires jQuery**

Неперехваченная ошибка: для JavaScript Bootstrap требуется jQuery
{% endhint %}

{% hint style="danger" %}
**Uncaught ReferenceError: jQuery is not defined**

Неперехваченная ReferenceError: jQuery не определен
{% endhint %}

{% hint style="danger" %}
**Uncaught bootstrap-datetimepicker requires jQuery to be loaded first**

Неперехваченный bootstrap-datetimepicker требует, чтобы сначала был загружен jQuery
{% endhint %}

Вышеупомянутые ошибки отображаются в консоли, если вы забыли добавить **jQuery** в свой шаблон. **JavaScript** в **Bootstrap** должен предшествовать **jQuery**, иначе **Bootstrap** выдаст ошибку. Ознакомьтесь с нашими инструкциями по настройке, чтобы увидеть различные варианты для этого.

{% hint style="danger" %}
**Uncaught TypeError: Cannot read property ‘Constructor’ of undefined**

Неперехваченная TypeError: Не удается прочитать свойство «Constructor»;  не определено
{% endhint %}

Вы забыли добавить файл bootstrap **JavaScript** в свой шаблон, убедитесь, что вы включили в свой шаблон файлы **Bootstrap JavasScript** и **CSS**. Ознакомьтесь с нашими инструкциями по настройке, чтобы увидеть различные варианты для этого.

## Исправление ошибки 404 (не найдено)

{% hint style="danger" %}
GET `http://.../datepicker-widget.js` net::ERR\_ABORTED 404 (Not Found)
{% endhint %}

{% hint style="danger" %}
GET `http://.../datepicker-widget.css` net::ERR\_ABORTED 404 (Not Found)
{% endhint %}

В некоторых рабочих средах вам необходимо собрать статические ресурсы **JS/CSS** из всех пакетов в одно место, указанное как **STATIC\_ROOT** в файле **settings.py**. Обратитесь к руководству вашего хостинг-провайдера, чтобы узнать, как это сделать правильно.

В противном случае, если ваш интерфейс хостинга предоставляет доступ к консоли, войдите в консоль сервера и выполните следующую команду, а затем перезагрузите сервер из интерфейса хостинга.

```bash
python3 manage.py collectstatic
```

{% hint style="danger" %}
**You’re using the staticfiles app without having set the STATIC\_ROOT setting to a filesystem path**

Вы используете приложение staticfiles, не установив для параметра STATIC\_ROOT путь к файловой системе.
{% endhint %}

Если вы столкнулись с вышеуказанной ошибкой, значит, у вас не установлен каталог **STATIC\_ROOT**. Обратитесь к руководству по эксплуатации вашего хостинг-провайдера, чтобы определить каталог, из которого он обслуживает статические файлы, обычно это статический каталог в корне вашего проекта. Если это так, добавьте следующую строку внизу файла **settings.py**.

```python
STATIC_ROOT = os.path.join(BASE_DIR, "static")
```

Теперь вы можете выполнить команду **collectstatic** и перезагрузить сервер с вашего интерфейса хостинга.

## Ошибок нигде нет, но календарь не отображается!

Вы забыли добавить `{{ form.media }}` в свой шаблон. Ознакомьтесь с нашими инструкциями по настройке, чтобы узнать больше об этой проблеме.

## Моя ошибка здесь не указана

Пожалуйста, создайте задачу в [репозитории проекта на GitHub](https://github.com/monim67/django-bootstrap-datepicker-plus/issues/new/choose), предоставив как можно больше информации.
