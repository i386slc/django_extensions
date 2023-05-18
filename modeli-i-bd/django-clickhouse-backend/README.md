# django-clickhouse-backend

Бэкэнд Django clickhouse — это [бэкэнд баз данных django](https://docs.djangoproject.com/en/4.1/ref/databases/) для базы данных [clickhouse](https://clickhouse.com/docs/en/home/). Этот проект позволяет использовать django ORM для взаимодействия с clickhouse, целью проекта является управление clickhouse, как при работе с mysql, postgresql в django.

Благодаря [драйверу clickhouse](https://github.com/mymarilyn/clickhouse-driver), бэкенд django clickhouse использует его как [DBAPI](https://peps.python.org/pep-0249/). Благодаря [пулу clickhouse](https://github.com/ericmccarthy7/clickhouse-pool) он создает пул соединений clickhouse.

Подробнее читайте в [документации](./#temy).

**Функции**:

* Повторно использует большинство существующих средств django ORM, минимизирует затраты на обучение.
* Эффективно подключается к clickhouse через [собственный интерфейс clickhouse](https://clickhouse.com/docs/en/interfaces/tcp/) и пул соединений.
* Нет другого промежуточного хранилища, нет необходимости синхронизировать данные, просто напрямую взаимодействуйте с clickhouse.
* Поддержка конкретных функций схемы clickhouse, таких как [Engine](https://clickhouse.com/docs/en/engines/table-engines/) и [Index](https://clickhouse.com/docs/en/guides/improving-query-performance/skipping-indexes).
* Поддержка большинства типов миграций таблиц.
* Поддержка создания тестовой базы данных и таблицы, работа с django **TestCase** и **pytest-django**.
* Поддержка большинства типов данных clickhouse.
* Поддержка [SETTINGS в SELECT Query](https://clickhouse.com/docs/en/sql-reference/statements/select/#settings-in-select-query).

**Примечания**:

* Не тестировалось на всех версиях **clickhouse-server**, рекомендуется `clickhouse-server 22.x.y.z` или выше.
* Функции агрегации возвращают **0** или **nan** (не `NULL`), когда набор данных пуст. `max/min/sum/count` равно 0, `avg/STDDEV_POP/VAR_POP` равно **nan**.
* Во внешнем соединении clickhouse установит для отсутствующих столбцов пустые значения (0 для числа, пустая строка для текста, эпоха unix для даты/времени данных) вместо **NULL**. Таким образом, `Count("book")` разрешается в 1 в отсутствующем совпадении **LEFT OUTER JOIN**, а не в 0. В выражении агрегации `Avg("book__rating", default=2.5)`, `default=2.5` не влияет на отсутствующее совпадение.

**Требования**:

* [Python](https://www.python.org/) >= 3.6
* [Django](https://docs.djangoproject.com/) >= 3.2
* [clickhouse driver](https://github.com/mymarilyn/clickhouse-driver)
* [clickhouse pool](https://github.com/ericmccarthy7/clickhouse-pool)

## Приступаем к работе

### Установка

```bash
$ pip install django-clickhouse-backend
```

или

```bash
$ git clone https://github.com/jayvynl/django-clickhouse-backend
$ cd django-clickhouse-backend
$ python setup.py install
```

### Настройка конфигурации

В настройках базы данных требуется только **ENGINE**, остальные параметры имеют значения по умолчанию.

* **ENGINE**: требуется, установите значение `clickhouse_backend.backend`.
* **NAME**: имя базы данных, значение по умолчанию **default**.
* **HOST**: хост базы данных, по умолчанию **localhost**.
* **PORT**: порт базы данных, по умолчанию **9000**.
* **USER**: пользователь базы данных, по умолчанию **default**.
* **PASSWORD**: пароль базы данных, по умолчанию пусто.

В большинстве случаев вы можете просто использовать clickhouse для хранения некоторых больших таблиц событий и использовать некоторые СУБД для хранения других таблиц. Здесь я привожу пример настройки для clickhouse и postgresql.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': 'localhost',
        'USER': 'postgres',
        'PASSWORD': '123456',
        'NAME': 'postgres',
    },
    'clickhouse': {
        'ENGINE': 'clickhouse_backend.backend',
        'NAME': 'default',
        'HOST': 'localhost',
        'USER': 'DB_USER',
        'PASSWORD': 'DB_PASSWORD',
        'TEST': {
            'fake_transaction': True
        }
    }
}
DATABASE_ROUTERS = ['dbrouters.ClickHouseRouter']
DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'
```

```python
# dbrouters.py
from clickhouse_backend.models import ClickhouseModel

def get_subclasses(class_):
    classes = class_.__subclasses__()

    index = 0
    while index < len(classes):
        classes.extend(classes[index].__subclasses__())
        index += 1

    return list(set(classes))


class ClickHouseRouter:
    def __init__(self):
        self.route_model_names = set()
        for model in get_subclasses(ClickhouseModel):
            if model._meta.abstract:
                continue
            self.route_model_names.add(model._meta.label_lower)

    def db_for_read(self, model, **hints):
        if (model._meta.label_lower in self.route_model_names
                or hints.get('clickhouse')):
            return 'clickhouse'
        return None

    def db_for_write(self, model, **hints):
        if (model._meta.label_lower in self.route_model_names
                or hints.get('clickhouse')):
            return 'clickhouse'
        return None

    def allow_migrate(self, db, app_label, model_name=None, **hints):
        if (f'{app_label}.{model_name}' in self.route_model_names
                or hints.get('clickhouse')):
            return db == 'clickhouse'
        elif db == 'clickhouse':
            return False
        return None
```

Вы должны использовать [маршрутизатор базы данных](https://docs.djangoproject.com/en/4.1/topics/db/multi-db/#automatic-database-routing), чтобы автоматически направлять ваши запросы в нужную базу данных. В предыдущем примере я пишу маршрутизатор базы данных, который направляет все запросы из подклассов `clickhouse_backend.models.ClickhouseModel` или пользовательских миграций с ключом подсказки `clickhouse` в clickhouse. Все остальные запросы направляются в базу данных по умолчанию (postgresql).

`DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'` требуется для работы с миграцией django. Дополнительные сведения будут описаны в [DEFAULT\_AUTO\_FIELD](nastroiki-konfiguracii.md#default\_auto\_field).

## Определение модели

Бэкэнд Clickhouse поддерживает встроенные поля django и специальные поля clickhouse.

Прочтите [документацию по полям](polya.md) для получения дополнительной информации.

Уведомления об определении модели:

* импортируйте модели из **clickhouse\_backend**, а не из django.db
* добавляйте **low\_cardinality** для **StringFiled**, когда кардинальность поля данных относительно невелика, эта конфигурация может значительно повысить производительность запросов
* нельзя использовать `db_index=True` в поле, но мы можем добавить в индексы Meta
* необходимо указать порядок в **Meta** только для порядка запросов по умолчанию
* нужно указать движок для clickhouse, указать order\_by для заказа clickhouse и аргумент **partition\_by**

```python
from django.db.models import CheckConstraint, Func, Q, IntegerChoices
from django.utils import timezone

from clickhouse_backend import models


class Event(models.ClickhouseModel):
    class Action(IntegerChoices):
        PASS = 1
        DROP = 2
        ALERT = 3
    ip = models.GenericIPAddressField(default='::')
    ipv4 = models.GenericIPAddressField(default='127.0.0.1')
    ip_nullable = models.GenericIPAddressField(null=True)
    port = models.UInt16Field(default=0)
    protocol = models.StringField(default='', low_cardinality=True)
    content = models.StringField(default='')
    timestamp = models.DateTime64Field(default=timezone.now)
    created_at = models.DateTime64Field(auto_now_add=True)
    action = models.EnumField(choices=Action.choices, default=Action.PASS)

    class Meta:
        verbose_name = 'Network event'
        ordering = ['-id']
        db_table = 'event'
        engine = models.ReplacingMergeTree(
            order_by=['id'],
            partition_by=Func('timestamp', function='toYYYYMMDD')
        )
        indexes = [
            models.Index(
                fields=["ip"],
                name='ip_set_idx',
                type=models.Set(1000),
                granularity=4
            ),
            models.Index(
                fields=["ipv4"],
                name="ipv4_bloom_idx",
                type=models.BloomFilter(0.001),
                granularity=1
            )
        ]
        constraints = (
            CheckConstraint(
                name='port_range',
                check=Q(port__gte=0, port__lte=65535),
            ),
        )
```

### Миграции

```bash
$ python manage.py makemigrations
```

Эта операция создаст файл миграции в папке `apps/migrations/`

Затем мы мигрируем.

```bash
$ python manage.py migrate
```

При первом запуске эта операция создаст таблицу **django\_migrations** с созданием таблицы sql, подобной этой

```sql
> show create table django_migrations;

CREATE TABLE other.django_migrations
(
    `id` Int64,
    `app` FixedString(255),
    `name` FixedString(255),
    `applied` DateTime64(6, 'UTC')
)
ENGINE = MergeTree
ORDER BY id
SETTINGS index_granularity = 8192
```

мы можем запросить его с такими результатами

```sql
> select * from django_migrations;

┌──────────────────id─┬─app─────┬─name─────────┬────────────────────applied─┐
│ 1626937818115211264 │ testapp │ 0001_initial │ 2023-02-18 13:32:57.538472 │
└─────────────────────┴─────────┴──────────────┴────────────────────────────┘
```

миграция создаст таблицу с именем события, как мы определили в моделях

```sql
> show create table event;

CREATE TABLE other.event
(
    `id` Int64,
    `ip` IPv6,
    `ipv4` IPv6,
    `ip_nullable` Nullable(IPv6),
    `port` UInt16,
    `protocol` LowCardinality(String),
    `content` String,
    `timestamp` DateTime64(6, 'UTC'),
    `created_at` DateTime64(6, 'UTC'),
    `action` Enum8('Pass' = 1, 'Drop' = 2, 'Alert' = 3),
    INDEX ip_set_idx ip TYPE set(1000) GRANULARITY 4,
    INDEX port_bloom_idx port TYPE bloom_filter(0.001) GRANULARITY 1,
    CONSTRAINT port_range CHECK (port >= 0) AND (port <= 65535)
)
ENGINE = ReplacingMergeTree
PARTITION BY toYYYYMMDD(timestamp)
ORDER BY id
SETTINGS index_granularity = 8192
```

### Работа с данными

#### create

```python
for i in range(10):
    Event.objects.create(ip_nullable=None, port=i,
                         protocol="HTTP", content="test", 
                         action=Event.Action.PASS.value)
assert Event.objects.count() == 10
```

#### query

```python
queryset = Event.objects.filter(content="test")
for i in queryset:
    print(i)
```

#### update

```python
Event.objects.filter(port__in=[1, 2, 3]).update(protocol="TCP")
time.sleep(1)
assert Event.objects.filter(protocol="TCP").count() == 3
```

#### delete

```python
Event.objects.filter(protocol="TCP").delete()
time.sleep(1)
assert not Event.objects.filter(protocol="TCP").exists()
```

За исключением определения модели, все остальные операции аналогичны операциям с реляционными базами данных, такими как mysql и postgresql.

### Тестирование

Написание тестового примера такое же, как и в обычном проекте django. Вы можете использовать django TestCase или pytest-django. Примечание: clickhouse использует мутации для [удаления или обновления](https://clickhouse.com/docs/en/guides/developer/mutations). По умолчанию мутации данных обрабатываются асинхронно. То есть, когда вы обновляете или удаляете строку, clickhouse выполнит действие через некоторое время. Поэтому вам следует изменить это поведение по умолчанию при тестировании на удаление или обновление. Есть 2 способа сделать это:

* Настройте механизм базы данных следующим образом, установив [mutations\_sync=1](https://clickhouse.com/docs/en/operations/settings/settings#mutations\_sync) в области сеанса.

```python
DATABASES = {
    'default': {
        'ENGINE': 'clickhouse_backend.backend',
        'OPTIONS': {
            'settings': {
                'mutations_sync': 1,
            }
        }
    }
}
```

* Используйте [SETTINGS в SELECT Query](https://clickhouse.com/docs/en/sql-reference/statements/select/#settings-in-select-query).

```python
Event.objects.filter(protocol='UDP').settings(mutations_sync=1).delete()
```

Пример тестового случая.

```python
from django.test import TestCase

class TestEvent(TestCase):
    def test_spam(self):
        assert Event.objects.count() == 0
```

## Тест

Чтобы запустить тест для этого проекта:

```bash
$ git clone https://github.com/jayvynl/django-clickhouse-backend
$ cd django-clickhouse-backend
# docker and docker-compose are required.
$ docker-compose up -d
$ python tests/runtests.py
# run test for every python version and django version
$ pip install tox
$ tox
```

## Список изменений

[Все журналы изменений](https://github.com/jayvynl/django-clickhouse-backend/blob/main/CHANGELOG.md).

## Лицензия

Django clickhouse backend распространяется под [лицензией MIT](http://www.opensource.org/licenses/mit-license.php).

## Темы

* [Настройки конфигурации](nastroiki-konfiguracii.md)
* [Миграции](migracii.md)
* [Поля](polya.md)
