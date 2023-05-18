# Поля

Бэкэнд Clickhouse поддерживает встроенные поля **django** и специальные поля **clickhouse**.

{% hint style="info" %}
Вы всегда должны использовать специальные поля **clickhouse** в новых проектах. Поддержка встроенных полей **django** предназначена только для совместимости с существующими сторонними приложениями.
{% endhint %}

{% hint style="info" %}
[ForeignKey](https://docs.djangoproject.com/en/4.1/ref/models/fields/#foreignkey), [ManyToManyField](https://docs.djangoproject.com/en/4.1/ref/models/fields/#manytomanyfield) или даже [OneToOneField](https://docs.djangoproject.com/en/4.1/ref/models/fields/#onetoonefield) можно использовать с бэкендом **clickhouse**. Но ограничения на уровне базы данных добавляться не будут, поэтому могут возникнуть некоторые проблемы с согласованностью.
{% endhint %}

## Поля Django

Поддерживаются следующие поля django:

| Класс                              | Тип БД      | Тип Python            | Комментарии                                                                                                                                                                                     |
| ---------------------------------- | ----------- | --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| django.db.models.SmallAutoField    | Int16       | int                   | Нет автоматического значения, необходимо указать значение самостоятельно                                                                                                                        |
| django.db.models.AutoField         | Int32       | int                   | Нет автоматического значения, необходимо указать значение самостоятельно                                                                                                                        |
| django.db.models.BigAutoField      | Int64       | int                   | `clickhouse.idworker.snowflake.SnowflakeIDWorker` будет генерировать значение автоматически.                                                                                                    |
| django.db.models.CharField         | FixedString | str                   | Кодируется как UTF-8 при записи в ClickHouse                                                                                                                                                    |
| django.db.models.TextField         | String      | str                   | Кодируется как UTF-8 при записи в ClickHouse                                                                                                                                                    |
| django.db.models.BinaryField       | String      | bytes or str          |                                                                                                                                                                                                 |
| django.db.models.SlugField         | String      |                       |                                                                                                                                                                                                 |
| django.db.models.FileField         | String      |                       |                                                                                                                                                                                                 |
| django.db.models.FilePathField     | String      |                       |                                                                                                                                                                                                 |
| django.db.models.DateField         | Date32      | datetime.date         | Диапазон от 1900-01-01 до 2299-12-31; Дата превышения этого диапазона будет сохранена как минимальное или максимальное значение.                                                                |
| django.db.models.DateTimeField     | DateTime64  | datetime.datetime     | Диапазон от 1900-01-01 00:00:00 до 2299-12-31 23:59:59.999999; знает часовой пояс; Дата и время, превышающие этот диапазон, будут сохранены как минимальное значение или максимальное значение. |
| django.db.models.SmallIntegerField | Int16       | int                   | Диапазон от -32768 до 32767                                                                                                                                                                     |
| django.db.models.IntegerField      | Int32       | int                   | Диапазон от -2147483648 до 2147483647                                                                                                                                                           |
| django.db.models.BigIntegerField   | Int64       | int                   | Диапазон от -9223372036854775808 до 9223372036854775807                                                                                                                                         |
| django.db.models.SmallIntegerField | UInt16      | int                   | Диапазон от 0 до 32767                                                                                                                                                                          |
| django.db.models.IntegerField      | UInt32      | int                   | Диапазон от 0 до 2147483647                                                                                                                                                                     |
| django.db.models.BigIntegerField   | UInt64      | int                   | Диапазон от 0 до 9223372036854775807                                                                                                                                                            |
| django.db.models.FloatField        | Float64     | float                 |                                                                                                                                                                                                 |
| django.db.models.DecimalField      | Decimal     | Decimal               | Значения Python округляются, чтобы соответствовать масштабу поля базы данных.                                                                                                                   |
| django.db.models.UUIDField         | UUID        | uuid.UUID             |                                                                                                                                                                                                 |
| django.db.models.IPAddressField    | IPv4        | ipaddress.IPv4Address |                                                                                                                                                                                                 |
| django.db.models.BooleanField      | Bool        |                       |                                                                                                                                                                                                 |
| NullableField                      | Nullable    |                       | Когда null=True                                                                                                                                                                                 |

## Поля Clickhouse

Если тип данных **clickhouse** поддерживает **LowCardinality**, в соответствующем поле модели будет параметр **low\_cardinality**.

Если тип данных **clickhouse** не поддерживает **Nullable**, при выполнении проверок django возникнет ошибка, если вы установите `null = True` в поле модели.

Передача `null=True` сделает поле [Nullable](https://clickhouse.com/docs/en/sql-reference/data-types/nullable/).

Передача `low_cardinality=True` сделает поле [LowCardinality](https://clickhouse.com/docs/en/sql-reference/data-types/lowcardinality).

Имя класса поля всегда связано с именем типа даты clickhouse с **Field**. Например, поле [DateTime64](https://clickhouse.com/docs/en/sql-reference/data-types/datetime64) называется **DateTime64Field**. Все поля clickhouse импортированы из `clickhouse_backend.models`

Поддерживаемые типы данных:

* Float32/Float64
* Int8/Int16/Int32/Int64/Int128/Int256
* UInt8/UInt16/UInt32/UInt64/UInt128/UInt256
* Date/Date32
* DateTime/DateTime64
* String/FixedString(N)
* Enum/Enum8/Enum16
* Array(T)
* Bool
* UUID
* Decimal
* IPv4/IPv6
* LowCardinality(T)
* Tuple(T1, T2, ...)
* Map(key, value)

### \[U]Int(8|16|32|64|128|256)

Путь импорта полей: `clickhouse_backend.models.[U]Int(8|16|32|64|128|256)Field`

Например, тип **UInt8** импортируется из `clickhouse_backend.models.UInt8Field`.

И **Nullable**, и **LowCardinality** поддерживаются для всех типов **int**.

Все типы **UInt** будут иметь корректные валидаторы диапазона.

Например, `clickhouse_backend.models.UInt16Field` имеет диапазон от 0 до 65535. Для сравнения, `django.db.models.SmallIntegerField` имеет диапазон от 0 до 32767, что приводит к сужению половины диапазона.

### Float(32|64)

Путь импорта полей: `clickhouse_backend.models.Float(32|64)Field`

Например, тип **Float32** импортируется из `clickhouse_backend.models.Float32Field`.

И **Nullable**, и **LowCardinality** поддерживаются для **Float32Field** и **Float64Field**.

### Decimal

Путь импорта поля: `clickhouse_backend.models.DecimalField`

**Nullable** поддерживается, но **LowCardinality** не поддерживается.

### Bool

Путь импорта поля: `clickhouse_backend.models.BoolField`

Поддерживаются как **Nullable**, так и **LowCardinality**.

### String

Путь импорта поля: `clickhouse_backend.models.StringField`

Поддерживаются как **Nullable**, так и **LowCardinality**.

Тип **Clickhouse String** больше похож на байтовый тип в Python. **StringField** поддерживает **bytes** или тип **str** при присвоении значения. Строка Python будет закодирована в кодировке **UTF-8** при сохранении в clickhouse.

{% hint style="info" %}
**max\_length** будет иметь странное поведение, когда вы укажете тип байтов. Например, если **max\_length=4**, `'世界'` является допустимым значением, но соответствующие закодированные байты `b'\xe4\xb8\x96\xe7\x95\x8c'` являются недопустимым значением, поскольку его длина равна **6**.
{% endhint %}

### FixedString

Путь импорта поля: `clickhouse_backend.models.FixedStringField`

Параметр **max\_bytes** требуется в качестве длины **FixedString** в байтах.

Поддерживаются как **Nullable**, так и **LowCardinality**.

Тип Clickhouse **FixedString** больше похож на байтовый тип в Python. **FixedStringField** поддерживает **bytes** или тип **str** при присвоении значения. Строка Python будет закодирована в кодировке **UTF-8** при сохранении в clickhouse.

{% hint style="info" %}
**max\_length** будет иметь странное поведение, когда вы укажете тип байтов. Например, если **max\_length=4**, `'世界'` является допустимым значением, но соответствующие закодированные байты `b'\xe4\xb8\x96\xe7\x95\x8c'` являются недопустимым значением, поскольку его длина равна **6**.
{% endhint %}

### UUID

Путь импорта поля: `clickhouse_backend.models.UUIDField`

Поддерживаются как **Nullable**, так и **LowCardinality**.

Но создание столбцов типа `LowCardinality(Nullable(UUID))` по умолчанию запрещено из-за ожидаемого негативного влияния на производительность. Его можно включить с помощью параметра `«allow_supicious_low_cardinality_types»`.

```python
DATABASES = {
    'default': {
        'ENGINE': 'clickhouse_backend.backend',
        'OPTIONS': {
            'settings': {
                'allow_suspicious_low_cardinality_types': 1,
            }
        }
    }
}
```

{% hint style="info" %}
Из-за [ошибки](https://github.com/mymarilyn/clickhouse-driver/issues/363) `clickhouse-driver 0.2.5`, когда для **UUIDField** одновременно установлено значение `null=True` и `low_cardinality=True`, при вставке строки будет вызвано исключение. Та же ошибка существует и в **DateField** и **Date32Field**.
{% endhint %}

### Date и Date32

Путь импорта полей: `clickhouse_backend.models.Date[32]Field`

[Запрос дат](https://docs.djangoproject.com/en/4.1/ref/models/querysets/#dates) поддерживается **DateField** и **Date32Field**.

Все [операции поиска даты](https://docs.djangoproject.com/en/4.1/ref/models/querysets/#date) поддерживаются **DateField** и **Date32Field**.

Поддерживаются как **Nullable**, так и **LowCardinality**.

Но создание столбцов типа `LowCardinality(DateTime)` или `LowCardinality(DateTime32)` по умолчанию запрещено из-за ожидаемого негативного влияния на производительность. Его можно включить с помощью параметра `«allow_supicious_low_cardinality_types»`.

```python
DATABASES = {
    'default': {
        'ENGINE': 'clickhouse_backend.backend',
        'OPTIONS': {
            'settings': {
                'allow_suspicious_low_cardinality_types': 1,
            }
        }
    }
}
```

Clickhouse поддерживает целое число или число с плавающей запятой при назначении столбца **Date** или **Date32**. Но поддержка целого числа или числа с плавающей запятой может привести к странному поведению, поскольку целое число и число с плавающей запятой обрабатываются как временные метки unix, clickhouse сохраняет их как дату в [часовом поясе сервера](https://clickhouse.com/docs/en/operations/server-configuration-parameters/settings#server\_configuration\_parameters-timezone), что может быть не тем, что вам нужно. Таким образом, эта функция не реализована в настоящее время.

{% hint style="info" %}
Из-за [ошибки](https://github.com/mymarilyn/clickhouse-driver/issues/363) `clickhouse-driver 0.2.5`, когда для **DateField** и **Date32Field** одновременно установлено значение `null=True` и `low_cardinality=True`, при вставке строки будет вызвано исключение. Та же ошибка существует и в **UUIDField**.
{% endhint %}

### DateTime и DateTime64

Путь импорта полей: `clickhouse_backend.models.DateTime[64]Field`

**DateTime64Field** имеет параметр точности [precision](https://clickhouse.com/docs/en/sql-reference/data-types/datetime64), который по умолчанию равен **6**.

[Запрос даты](https://docs.djangoproject.com/en/4.1/ref/models/querysets/#dates) и [запрос даты и времени](https://docs.djangoproject.com/en/4.1/ref/models/querysets/#datetimes) поддерживаются DateTimeField и DateTime64Field.

Все [операции поиска даты](https://docs.djangoproject.com/en/4.1/ref/models/querysets/#date) поддерживаются **DateTimeField** и **DateTime64Field**.

И **Nullable**, и **LowCardinality** поддерживаются **DateTime**. Но **LowCardinality** не поддерживается **DateTime64**.

Clickhouse поддерживает целое число или число с плавающей запятой при назначении столбца **DateTime** или **DateTime64**. Эта функция также реализована бэкендом clickhouse.

Плавающее значение обрабатывается как временная метка unix в **DateTime** и **DateTime64**.

Значение **int** обрабатывается как временная метка unix в **DateTime**.

Но значение значения **int** зависит от точности **DateTime64**. Например, `DateTime64(3)` рассматривает значение **int** как миллисекунды из эпохи Unix, `DateTime64(6)` рассматривает значение **int** как микросекунды из эпохи Unix.

### Enum\[8|16]

Путь импорта полей: `clickhouse_backend.models.Enum[8|16]Field`

**Nullable** поддерживается, но **LowCardinality** не поддерживается.

Параметр выбора **choices** требуется при определении поля **Enum**, а **choices** должен быть итерируемым, содержащим только кортежи `(int, str)` , и **int** в вариантах выбора не должны превышать диапазон. **EnumField** и **Enum16Field** находятся в диапазоне от -32768 до 32767, **Enum8Field** — в диапазоне от -128 до 127.

Используйте **return\_int** (по умолчанию `True`), чтобы указать, следует ли получать значение **int** или **str** при запросе из базы данных.

Если предоставлен недопустимый выбор, возникает ошибка при выполнении проверок **django**.

При присвоении значения **EnumField** поддерживаются как значение выбора **choices**, так и метка выбора **choices label**.

При запросе из базы данных возвращается значение **str**.

Пример использования:

```python
from clickhouse_backend import models
from django.db.models import IntegerChoices

class EnumTest(models.ClickhouseModel):
    class Fruit(IntegerChoices):
        banana = 1, 'banana'
        pear = 2, 'pear'
        apple = 3, 'apple'

    enum = models.EnumField(choices=Fruit.choices, return_int=False)
    
    def __str__(self):
        return str(self.enum)

EnumTest.objects.bulk_create([
    EnumTest(enum=1),
    EnumTest(enum='pear'),
    EnumTest(enum=b'apple')
])
# [<EnumTest: 1>, <EnumTest: pear>, <EnumTest: b'apple'>]

EnumTest.objects.filter(enum=1)
# <QuerySet [<EnumTest: banana>]>
EnumTest.objects.filter(enum='pear')
# <QuerySet [<EnumTest: pear>]>
EnumTest.objects.filter(enum=b'apple')
# <QuerySet [<EnumTest: apple>]>
EnumTest.objects.filter(enum__gt=1)
# <QuerySet [<EnumTest: pear>, <EnumTest: apple>]>
EnumTest.objects.filter(enum__gte='pear')
# <QuerySet [<EnumTest: pear>, <EnumTest: apple>]>
EnumTest.objects.filter(enum__contains='ana')
# <QuerySet [<EnumTest: banana>]>
```

### IPv4 IPv6

Путь импорта полей: `clickhouse_backend.models.IPv(4|6)Field`

Поддерживаются как **Nullable**, так и **LowCardinality**.

**IPv4Field** поддерживает `ipaddress.IPv4Address` или **str** при присвоении значения. **IPv6Field** поддерживает `ipaddress.IPv4Address`, `ipaddress.IPv6Address` или **str** при присвоении значения.

При запросе из базы данных возвращается **str**.

### GenericIPAddressField

**GenericIPAddressField** существует для обеспечения того же поведения, что и встроенный в django **GenericIPAddressField**.

В базе данных **GenericIPAddressField** хранится как **IPv6**. Таким образом, он ведет себя в основном как **IPv6Field**, за исключением того, что **GenericIPAddressField** попытается вернуть сопоставленный адрес **ipv4**, когда `unpack_ipv4=True`.

Пример использования:

```python
from clickhouse_backend import models

class IPTest(models.ClickhouseModel):
    ipv4 = models.IPv4Field()
    ipv6 = models.IPv6Field()
    ip = models.GenericIPAddressField(unpack_ipv4=True)

    def __str__(self):
        return str(self.ip)

IPTest.objects.create([
    IPTest(ipv4='1.2.3.4',ipv6='1.2.3.4',ip='1.2.3.4'),
    IPTest(ipv4='2.3.4.5',ipv6='2.3.4.5',ip='2.3.4.5'),
])
# [<IPTest: 1.2.3.4>, <IPTest: 2.3.4.5>]
IPTest.objects.filter(ipv4='1.2.3.4')
# <QuerySet [<IPTest: 1.2.3.4>]>
IPTest.objects.filter(ipv6='1.2.3.4')
# <QuerySet [<IPTest: 1.2.3.4>]>
IPTest.objects.filter(ip='1.2.3.4')
# <QuerySet [<IPTest: 1.2.3.4>]>

IPTest.objects.filter(ipv6__gt='1.2.3.4')
# <QuerySet [<IPTest: 2.3.4.5>]>

ip = IPTest.objects.get(ipv6__contains='4.5')
ip.ipv4, ip.ipv6, ip.ip
# ('2.3.4.5', '::ffff:203:405', '2.3.4.5')
```
