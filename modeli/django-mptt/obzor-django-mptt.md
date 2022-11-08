# Обзор django-mptt

## Что такое модифицированный обход дерева предзаказов?

MPTT — это метод хранения иерархических данных в базе данных. Цель состоит в том, чтобы сделать поисковые операции очень эффективными.

Компромисс за эту эффективность заключается в том, что выполнение вставок и перемещение элементов по дереву более сложно, поскольку требуется дополнительная работа, чтобы постоянно поддерживать структуру дерева в хорошем состоянии.

Вот хорошая статья о MPTT, чтобы подогреть ваш аппетит и предоставить подробную информацию о том, как работает сама техника:

> * [Trees in SQL](https://www.ibase.ru/files/articles/programming/dbmstrees/sqltrees.html)
> * [Storing Hierarchical Data in a Database](https://www.sitepoint.com/hierarchical-data-database/)
> * [Managing Hierarchical Data in MySQL](http://mikehillyer.com/articles/managing-hierarchical-data-in-mysql/)

## Что такое django-mptt?

**django-mptt** — это многоразовое приложение Django, цель которого — упростить использование MPTT с вашими собственными моделями Django.

Он заботится о деталях управления таблицей базы данных как древовидной структурой и предоставляет инструменты для работы с деревьями экземпляров модели.

### Обзор функций

* Простая регистрация моделей - поля, необходимые для древовидной структуры, будут добавлены автоматически.
* Древовидная структура автоматически обновляется при создании или удалении экземпляров модели или при изменении родителя экземпляра.
* Каждый уровень дерева автоматически сортируется по выбранному вами полю (или полям).
* Новые методы модели добавляются к каждой зарегистрированной модели для:
  * изменение положения в дереве
  * поиск предков, братьев и сестер, потомков
  * подсчет потомков
  * другие операции с деревом
* Менеджер **TreeManager** добавляется ко всем зарегистрированным моделям. Это предоставляет методы для:
  * перемещения узлов вокруг дерева или в другое дерево
  * вставки узла в любом месте дерева
  * перестройки поля MPTT для дерева (полезно, когда вы делаете массовые обновления вне django)
* Поля формы для моделей дерева.
* Вспомогательные функции для моделей деревьев.
* Теги шаблонов и фильтры для рендеринга деревьев.
* Переводы для:
  * датского
  * французского
  * немецкого
  * польского