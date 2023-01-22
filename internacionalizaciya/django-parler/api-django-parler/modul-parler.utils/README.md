# Модуль parler.utils

Служебные функции для обработки языковых кодов и настроек.

### parler.utils.normalize\_language\_code(code)

Отменить различия между обозначениями кода языка

### parler.utils.is\_supported\_django\_language (language\_code)

Возвращает, поддерживается ли код языка.

### parler.utils.get\_language\_title(language\_code)

Возвращает **verbose\_name** для кода языка.

Возврат к **language\_code**, если язык не найден в настройках.

### parler.utils.get\_language\_settings (language\_code, site\_id=None)

Возвращает настройки языка для текущего сайта

### parler.utils.get\_active\_language\_choices(language\_code=None)

Узнайте, какие переводы должны быть видны на сайте. Он возвращает кортеж либо с одним выбором (текущий язык), либо кортеж с текущим языком + резервным языком.

### parler.utils.is\_multilingual\_project (site\_id=None)

Настроен ли текущий проект Django для многоязычной поддержки.

Подмодули:

* [модуль parler.utils.conf](modul-parler.utils.conf.md)
* [модуль parler.utils.context](modul-parler.utils.context.md)
