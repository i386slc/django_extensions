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
