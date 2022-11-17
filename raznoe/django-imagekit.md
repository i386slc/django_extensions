# django-imagekit

**ImageKit** — это приложение Django для обработки изображений. Нужна миниатюра? Черно-белая версия изображения, загруженного пользователем? **ImageKit** сделает их для вас. Если вам нужно программно сгенерировать одно изображение из другого, вам понадобится **ImageKit**.

**ImageKit** поставляется с набором процессоров изображений для обычных задач, таких как изменение размера и обрезка, но вы также можете создать свой собственный. Чтобы получить представление о том, что возможно, ознакомьтесь с проектом [Instakit](https://github.com/fish2000/instakit).

Полную документацию по последней стабильной версии **ImageKit** см. в разделе [ImageKit на RTD](http://django-imagekit.readthedocs.org/).

## Установка

1. Установите [PIL](http://pypi.python.org/pypi/PIL) или [Pillow](http://pypi.python.org/pypi/Pillow). (Если вы используете **ImageField** в Django, вы уже должны были это сделать.)
2. `pip install django-imagekit`
3. Добавьте `'imagekit'` в список **INSTALLED\_APPS** в файле `settings.py` вашего проекта.

{% hint style="info" %}
Если вы никогда раньше не видели **Pillow**, она считается более часто обновляемой «дружественной» вилкой **PIL**, совместимой с **setuptools**. Таким образом, она использует то же пространство имен, что и **PIL**, и является заменой.
{% endhint %}

## Обзор использования

### Спецификации

У вас есть одно изображение, и вы хотите что-то с ним сделать, чтобы создать другое изображение. Но как сказать **ImageKit**, что делать? Определив спецификацию изображения.

_**Спецификация изображения**_ — это тип _**генератора изображений**_, который генерирует новое изображение из исходного изображения.

### Определение спецификаций в моделях

Самый простой способ определить спецификацию изображения — использовать **ImageSpecField** в классе модели:

```python
from django.db import models
from imagekit.models import ImageSpecField
from imagekit.processors import ResizeToFill

class Profile(models.Model):
    avatar = models.ImageField(upload_to='avatars')
    avatar_thumbnail = ImageSpecField(
        source='avatar',
        processors=[ResizeToFill(100, 50)],
        format='JPEG',
        options={'quality': 60}
    )

profile = Profile.objects.all()[0]
print profile.avatar_thumbnail.url
# > /media/CACHE/images/982d5af84cddddfd0fbf70892b4431e4.jpg
print profile.avatar_thumbnail.width
# > 100
```

Как вы, наверное, заметили, **ImageSpecFields** во многом похожи на **ImageFields** Django. Разница в том, что они автоматически генерируются **ImageKit** на основе ваших инструкций. В приведенном выше примере миниатюра **avatar\_thumbnail** представляет собой версию изображения **avatar** с измененным размером, сохраненную в формате **JPEG** с качеством **60**.

Однако иногда вам не нужно сохранять исходное изображение (**avatar** в приведенном выше примере); когда пользователь загружает изображение, вы просто хотите его обработать и сохранить результат. В этих случаях вы можете использовать класс **ProcessedImageField**:

```python
from django.db import models
from imagekit.models import ProcessedImageField

class Profile(models.Model):
    avatar_thumbnail = ProcessedImageField(
        upload_to='avatars',
        processors=[ResizeToFill(100, 50)],
        format='JPEG',
        options={'quality': 60}
    )

profile = Profile.objects.all()[0]
print profile.avatar_thumbnail.url
# > /media/avatars/MY-avatar.jpg
print profile.avatar_thumbnail.width
# > 100
```

Это очень похоже на наш предыдущий пример. Нам больше не нужно указывать **source**, так как мы не обрабатываем другое поле изображения, но нам нужно передать аргумент **upload\_to**. Этот  пример ведет себя точно так же, как и для Django **ImageField**.

{% hint style="info" %}
Вам может быть интересно, почему нам не понадобился аргумент **upload\_to** для нашего **ImageSpecField**. Причина в том, что **ProcessedImageField** на самом деле такие же, как **ImageField** сохраняют путь к файлу в базе данных, и вам нужно запустить **syncdb** (или создать миграцию), когда вы добавляете его в свою модель.

**ImageSpecField**, с другой стороны, являются виртуальными — они не добавляют полей в вашу базу данных и не требуют базы данных. Это удобно по многим причинам, но это означает, что путь к файлу изображения должен быть создан программно на основе исходного изображения и спецификации.
{% endhint %}

### Определение спецификаций вне моделей

Определение спецификаций как полей модели — очень удобный способ обработки изображений, но не единственный. Иногда вы не можете (или не хотите) добавлять поля в свои модели, и это нормально. Вы можете определить классы спецификации изображения и использовать их напрямую. Это может быть особенно полезно для обработки изображений в представлениях, особенно когда выполняемая обработка зависит от ввода пользователя.

```python
from imagekit import ImageSpec
from imagekit.processors import ResizeToFill

class Thumbnail(ImageSpec):
    processors = [ResizeToFill(100, 50)]
    format = 'JPEG'
    options = {'quality': 60}
```

Вероятно, неудивительно, что этот класс способен обрабатывать изображение точно так же, как наш **ImageSpecField** выше. Однако, в отличие от поля модели спецификации изображения, этот класс не определяет, на какой источник воздействует спецификация или что следует делать с результатом; это зависит от вас:

```python
source_file = open('/path/to/myimage.jpg')
image_generator = Thumbnail(source=source_file)
result = image_generator.generate()
```

{% hint style="info" %}
Не обязательно использовать **open**! Вы можете использовать любой файлоподобный объект, включая **ImageField** модели.
{% endhint %}

Результатом вызова `generate()` для спецификации изображения является файлоподобный объект, содержащий изображение с измененным размером, с которым вы можете делать все, что хотите. Например, если вы хотите сохранить его на диск:

```python
dest = open('/path/to/dest.jpg', 'w')
dest.write(result.read())
dest.close()
```

### Использование спецификаций в шаблонах

Если у вас есть модель с **ImageSpecField** или **ProcessedImageField**, вы можете легко использовать это обработанное изображение так же, как обычное поле изображения:

```django
<img src="{{ profile.avatar_thumbnail.url }}" />
```

{% hint style="info" %}
Предполагается, что у вас есть представление, которое устанавливает контекстную переменную с именем **profile** для экземпляра нашей модели **Profile**.
{% endhint %}

Но вы также можете создавать обработанные файлы изображений непосредственно в своем шаблоне — из любого изображения — без добавления чего-либо к своей модели. Чтобы сделать это, вам сначала нужно определить класс генератора изображений (помните, спецификации — это тип генератора) где-то в вашем приложении, как мы это сделали в предыдущем разделе. Вам также понадобится способ ссылки на генератор в вашем шаблоне, поэтому вам нужно будет зарегистрировать его.

```python
from imagekit import ImageSpec, register
from imagekit.processors import ResizeToFill

class Thumbnail(ImageSpec):
    processors = [ResizeToFill(100, 50)]
    format = 'JPEG'
    options = {'quality': 60}

register.generator('myapp:thumbnail', Thumbnail)
```

{% hint style="info" %}
Вы можете зарегистрировать свой генератор под любым идентификатором, но выбирайте с умом! Если вы выберете что-то слишком общее, у вас может возникнуть конфликт с другим сторонним приложением, которое вы используете. По этой причине хорошей идеей является префикс идентификаторов генератора с именем вашего приложения. Кроме того, **ImageKit** распознает двоеточие как разделитель при сопоставлении с шаблоном (например, в команде управления **generateimages**), поэтому рекомендуется использовать и их!
{% endhint %}

{% hint style="warning" %}
Этот код может быть в любом файле, который вы хотите, но вы должны убедиться, что он загружен! Для простоты **ImageKit** автоматически попытается загрузить модуль с именем **imagegenerators** в каждое из ваших установленных приложений. Так почему бы вам просто не избавить себя от головной боли и не поместить туда свои характеристики изображения?
{% endhint %}

Теперь, когда мы создали класс генератора изображений и зарегистрировали его в **ImageKit**, мы можем использовать его в наших шаблонах!

#### generateimage

Самый общий тег шаблона, который дает вам **ImageKit**, называется **generateimage**. Требуется хотя бы один аргумент: идентификатор **id** зарегистрированного генератора изображений. Дополнительные аргументы в стиле ключевых слов передаются зарегистрированному классу генератора. Как мы видели выше, конструкторы спецификации изображения ожидают аргумент ключевого слова источника, так что это то, что нам нужно передать, чтобы использовать нашу спецификацию эскиза:

```django
{% raw %}
{% load imagekit %}

{% generateimage 'myapp:thumbnail' source=source_file %}
{% endraw %}
```

Это выведет следующий HTML:

```html
<img src="/media/CACHE/images/982d5af84cddddfd0fbf70892b4431e4.jpg" width="100" height="50" />
```

Вы также можете добавить дополнительные атрибуты HTML; просто отделите их от аргументов ключевого слова двумя дефисами:

```django
{% raw %}
{% load imagekit %}

{% generateimage 'myapp:thumbnail' source=source_file -- alt="A picture of Me" id="mypicture" %}
{% endraw %}
```

Не генерируются HTML-теги изображений? Без проблем. Тег также функционирует как тег назначения, предоставляя доступ к базовому файловому объекту:

```django
{% raw %}
{% load imagekit %}

{% generateimage 'myapp:thumbnail' source=source_file as th %}
{% endraw %}
<a href="{{ th.url }}">Click to download a cool {{ th.width }} x {{ th.height }} image!</a>
```

#### thumbnail

Поскольку это такой распространенный вариант использования, **ImageKit** также предоставляет тег шаблона `«thumbnail»`:

```django
{% raw %}
{% load imagekit %}

{% thumbnail '100x50' source_file %}
{% endraw %}
```

Как и тег **generateimage**, тег **thumbnail** выводит тег `<img>`:

```html
<img src="/media/CACHE/images/982d5af84cddddfd0fbf70892b4431e4.jpg" width="100" height="50" />
```

Сравнив этот синтаксис с приведенным выше тегом **generateimage**, вы заметите несколько отличий.

Во-первых, нам не нужно было указывать идентификатор генератора изображений; если не указано иное, тег **thumbnail** использует генератор, зарегистрированный с идентификатором `«imagekit:thumbnail»`. _**Важно отметить, что этот тег не использует класс спецификации Thumbnail, который мы определили ранее**_; он использует генератор, зарегистрированный с идентификатором `«imagekit:thumbnail»`, который по умолчанию является `imagekit.generatorlibrary.Thumbnail`.

Во-вторых, мы передаем два позиционных аргумента (размеры и исходное изображение) в отличие от аргументов ключевого слова, которые мы использовали с тегом **generateimage**.

Как и в случае с тегом **generateimage**, вы также можете указать дополнительные HTML-атрибуты для тега эскиза или использовать его в качестве тега назначения:

```django
{% raw %}
{% load imagekit %}

{% thumbnail '100x50' source_file -- alt="A picture of Me" id="mypicture" %}
{% thumbnail '100x50' source_file as th %}
{% endraw %}
```

### Использование спецификаций в формах

В дополнение к полю модели, описанному выше, существует также версия поля формы класса **ProcessedImageField**. Функционал в принципе такой же (однократно обрабатывает изображение и сохраняет результат), но используется в классе формы:

```python
from django import forms
from imagekit.forms import ProcessedImageField
from imagekit.processors import ResizeToFill

class ProfileForm(forms.Form):
    avatar_thumbnail = ProcessedImageField(
        spec_id='myapp:profile:avatar_thumbnail',
        processors=[ResizeToFill(100, 50)],
        format='JPEG',
        options={'quality': 60}
    )
```

Преимущество использования `imagekit.forms.ProcessedImageField` (в отличие от `imagekit.models.ProcessedImageField` выше) заключается в том, что он сохраняет логику создания изображения вне вашей модели (в которой вы использовали бы обычный Django **ImageField**). Вы даже можете создать несколько форм, каждая со своим собственным **ProcessedImageField**, которые сохранят свои результаты в одном и том же поле изображения.

### Процессоры (processors)

Пока мы видели только один процессор: `imagekit.processors.ResizeToFill`. Но **ImageKit** способен на гораздо большее, чем просто изменение размера изображений, и эта мощь исходит от его процессоров.

Процессоры берут объект изображения **PIL**, что-то делают с ним и возвращают новый. Спецификация может использовать столько процессоров, сколько вам нужно, и все они будут запускаться по порядку.

```python
from imagekit import ImageSpec
from imagekit.processors import TrimBorderColor, Adjust

class MySpec(ImageSpec):
    processors = [
        TrimBorderColor(),
        Adjust(contrast=1.2, sharpness=1.1),
    ]
    format = 'JPEG'
    options = {'quality': 60}
```

Модуль `imagekit.processors` содержит процессоры для многих распространенных операций с изображениями, таких как изменение размера, поворот и корректировка цвета. Однако, если они не справляются с задачей, вы можете создать свой собственный. Все, что вам нужно сделать, это определить класс, реализующий метод `process()`:

```python
class Watermark(object):
    def process(self, image):
        # Код для добавления водяного знака находится здесь.
        return image
```

Вот и все! Чтобы использовать свой модный новый пользовательский процессор, просто включите его в список процессоров вашей спецификации:

```python
from imagekit import ImageSpec
from imagekit.processors import TrimBorderColor, Adjust
from myapp.processors import Watermark

class MySpec(ImageSpec):
    processors = [
        TrimBorderColor(),
        Adjust(contrast=1.2, sharpness=1.1),
        Watermark(),
    ]
    format = 'JPEG'
    options = {'quality': 60}
```

{% hint style="info" %}
Обратите внимание, что когда вы импортируете процессор из `imagekit.processors`, **imagekit**, в свою очередь, импортирует процессор из [PILKit](https://github.com/matthewwithanm/pilkit). Поэтому, если вы ищете доступные процессоры, посмотрите на PILKit.
{% endhint %}

### Админка (Admin)

**ImageKit** также содержит класс с именем `imagekit.admin.AdminThumbnail` для отображения спецификаций (или даже обычных полей **ImageField**) в [списке изменений администратора Django](https://docs.djangoproject.com/en/dev/intro/tutorial02/#customize-the-admin-change-list). **AdminThumbnail** используется как свойство в классах администрирования Django:

```python
from django.contrib import admin
from imagekit.admin import AdminThumbnail
from .models import Photo

class PhotoAdmin(admin.ModelAdmin):
    list_display = ('__str__', 'admin_thumbnail')
    admin_thumbnail = AdminThumbnail(image_field='thumbnail')

admin.site.register(Photo, PhotoAdmin)
```

**AdminThumbnail** может даже использовать собственный шаблон. Дополнительные сведения см. в разделе `imagekit.admin.AdminThumbnail`.

### Команды управления

В **ImageKit** есть одна команда управления — **generateimages** — которая создает файлы кэша для всех ваших зарегистрированных генераторов изображений. Вы также можете передать ему список идентификаторов генераторов, чтобы выборочно генерировать изображения.

## Сообщество

Пожалуйста, используйте [систему отслеживания ошибок GitHub](https://github.com/matthewwithanm/django-imagekit/issues), чтобы сообщать об ошибках с **django-imagekit**. Также существует [список рассылки](https://groups.google.com/forum/#!forum/django-imagekit) для обсуждения проекта и вопросов, а также официальный канал [#imagekit](irc://irc.freenode.net/imagekit) на Freenode.

## Содействие

Мы любим содействие! И вам не нужно быть экспертом в библиотеке или даже в Django, чтобы внести свой вклад: процессоры **ImageKit** — это автономные классы, которые полностью отделены от более пугающих внутренних компонентов ORM Django. Если вы написали процессор, который, по вашему мнению, может быть полезен другим людям, откройте запрос на извлечение, чтобы мы могли его посмотреть!

Вы также можете ознакомиться с нашим списком [открытых, удобных для участников вопросов](https://github.com/matthewwithanm/django-imagekit/issues?labels=contributor-friendly\&state=open) для идей.

Ознакомьтесь с нашими [рекомендациями по участию](https://github.com/matthewwithanm/django-imagekit/blob/develop/CONTRIBUTING.rst), чтобы получить дополнительную информацию о том, как заявить о себе с помощью **ImageKit**.

## Авторы

**ImageKit** изначально был написан [Justin Driscoll](http://github.com/jdriscoll).

Полевой API и другие вещи после 1.0 были написаны умными людьми из [HZDG](http://hzdg.com/).

### Сопровождающие

* [Matthew Tretter](http://github.com/matthewwithanm)
* [Bryan Veloso](http://github.com/bryanveloso)
* [Chris Drackett](http://github.com/chrisdrackett)
* [Greg Newman](http://github.com/gregnewman)

### Разработчики

* [Josh Ourisman](http://github.com/joshourisman)
* [Jonathan Slenders](http://github.com/jonathanslenders)
* [Eric Eldredge](http://github.com/lettertwo)
* [Chris McKenzie](http://github.com/kenzic)
* [Markus Kaiserswerth](http://github.com/mkai)
* [Ryan Bagwell](http://github.com/ryanbagwell)
* [Alexander Bohn](http://github.com/fish2000)
* [Timothée Peignier](http://github.com/cyberdelia)
* [Madis Väin](http://github.com/madisvain)
* [Jan Sagemüller](https://github.com/version2)
* [Clay McClure](https://github.com/claymation)
* [Jannis Leidel](https://github.com/jezdez)
* [Sean Bell](https://github.com/seanbell)
* [Saul Shanabrook](https://github.com/saulshanabrook)
* [Venelin Stoykov](https://github.com/vstoykov)

## Индексы и таблицы

<mark style="color:red;">Этот раздел еще не заполнен !!!</mark>

* Индексация
* Индексация модуля
* Страница поиска
* Конфигурация
  * Настройки
* Расширенное использование
  * Модели
  * Исходные группы
* Кэширование
  * Серверный рабочий процесс по умолчанию
  * Оптимизация
* Обновление с 2.x
  * Характеристики модели
  * Серверные части кэша изображений
  * Процессоры условной модели
  * Условные имена файлов cache\_to
  * Процессоры перешли на PILKit
