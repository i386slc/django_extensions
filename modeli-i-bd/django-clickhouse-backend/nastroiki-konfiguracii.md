# Настройки конфигурации

Чтобы настроить ваш проект **django** для использования бэкенда **clickhouse**, необходимо изменить только несколько собственных элементов конфигурации django. Этот проект не вводит никаких дополнительных элементов конфигурации.

## DATABASES

В конфигурации **DATABASES** требуется только **ENGINE**, остальные параметры имеют значения по умолчанию.

* **ENGINE**: требуется, установите значение `clickhouse_backend.backend`.
* **NAME**: имя базы данных, значение по умолчанию **default**.
* **HOST**: хост базы данных, по умолчанию **localhost**.
* **PORT**: порт базы данных, по умолчанию **9000**.
* **USER**: пользователь базы данных, по умолчанию **default**.
* **PASSWORD**: пароль базы данных, по умолчанию пусто.

```python
DATABASES = {
    'default': {
        'ENGINE': 'clickhouse_backend.backend',
        'NAME': 'default',
        'HOST': 'localhost',
        'USER': 'DB_USER',
        'PASSWORD': 'DB_PASSWORD',
        'OPTIONS': {
            'settings': {'mutations_sync': 1}
        },
        'TEST': {
            'fake_transaction': True
        }
    }
}
```

Действительные ключи **OPTIONS**:

* **connections\_min** — это минимальное количество соединений, которое может храниться в пуле соединений, по умолчанию **10**. Установите это значение на 0, чтобы отключить пул соединений.
* **connections\_max** — это максимальное количество подключений, которое можно использовать, по умолчанию **100**. Фактически, **connections\_max** — это максимальное количество запросов, которые можно выполнять одновременно. Потому что [исходный код соединения DBAPI](https://github.com/mymarilyn/clickhouse-driver/blob/0.2.5/clickhouse\_driver/dbapi/connection.py#L46) показывает, что каждый курсор создает новое соединение.
* **dsn** предоставляет URL-адрес подключения, например `clickhouse://localhost/test?param1=value1&...`. Если указан **dsn**, все остальные параметры подключения игнорируются.
* Все остальные параметры [clickhouse\_driver.connection.Connection](https://clickhouse-driver.readthedocs.io/en/latest/api.html#connection).
* **settings** может содержать настройки [clickhouse\_driver.Client](https://clickhouse-driver.readthedocs.io/en/latest/api.html?highlight=client#clickhouse\_driver.Client).
* **settings** может содержать [настройки clickhouse](https://clickhouse.com/docs/en/operations/settings/settings).

Действительные ключи TEST:

* `'fake_transaction'` заставляет **clickhouse** притворяться транзакционным. В случае нескольких баз данных, если другая база данных поддерживает транзакции, используйте **TransactionTestCase** или унаследованные классы (включая **pytest\_django**) для тестирования, все тестовые данные (включая различные настройки и фикстуры **pytest**) для каждой базы данных будут автоматически очищаться после каждого тестового примера путем вызова django команда сброса. Чтобы использовать транзакции для изоляции данных **postgresql** каждого тестового примера (ускорение теста), все базы данных должны поддерживать транзакции. Вы можете сделать так, чтобы соединения **clickhouse** поддерживали поддельные транзакции. Установив `'TEST'` в базе данных: `{'fake_transaction': True}`. Но это будет иметь побочный эффект, то есть данные **clickhouse** каждого тестового примера не будут изолированы. Поэтому в целом не рекомендуется использовать эту функцию, если вы не очень хорошо знаете, в чем заключается проблема.

## DEFAULT\_AUTO\_FIELD

`DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'` НЕОБХОДИМО ДЛЯ РАБОТЫ С DJANGO MIGRATION.

Django ORM сильно зависит от первичного ключа с одним столбцом, этот первичный ключ является уникальным идентификатором объекта ORM. Все действия **get**, **save** и **delete** зависят от первичного ключа.

Но в ClickHouse [первичный ключ](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree#primary-keys-and-indexes-in-queries) имеет другое значение, чем первичный ключ django. ClickHouse не требует уникального первичного ключа. Вы можете вставить несколько строк с одним и тем же первичным ключом.

В clickhouse [нет уникального ограничения](https://github.com/ClickHouse/ClickHouse/issues/3386#issuecomment-429874647) или автоматического увеличения столбца.

По умолчанию django добавит поле с именем **id** в качестве автоматического увеличения первичного ключа.

* **AutoField** - Сопоставлен с типом данных clickhouse **Int32**. Вы должны сгенерировать этот уникальный идентификатор самостоятельно.
* **BigAutoField** - Сопоставлен с типом данных clickhouse **Int64**. Если первичный ключ не указан при вставке данных, то для генерации этого уникального идентификатора используется `clickhouse_driver.idworker.id_worker`. **id\_worker** по умолчанию — это экземпляр `clickhouse.idworker.snowflake.SnowflakeIDWorker`, который реализует идентификатор [снежинки в твиттере](https://en.wikipedia.org/wiki/Snowflake\_ID). Если вставки данных происходят в нескольких центрах обработки данных, серверах, процессах или потоках, необходимо обеспечить уникальность переменной среды (**CLICKHOUSE\_WORKER\_ID**, **CLICKHOUSE\_DATACENTER\_ID**). Поскольку **work\_id** и **datacenter\_id** являются 5-битными, они должны быть целыми числами от 0 до 31. **CLICKHOUSE\_WORKER\_ID** по умолчанию равен 0, **CLICKHOUSE\_DATACENTER\_ID** будет генерироваться случайным образом, если он не указан. `clickhouse.idworker.snowflake.SnowflakeIDWorker` не является потокобезопасным. Вы можете унаследовать `clickhouse.idworker.base.BaseIDWorker` и реализовать его, а затем установить **CLICKHOUSE\_ID\_WORKER** в `settings.py` на любимый путь импорта вашего экземпляра **IDWorker**.

Django использует таблицу с именем **django\_migrations** для отслеживания файлов миграции. Поле идентификатора должно быть **BigAutoField**, чтобы **IDWorker** мог сгенерировать для вас уникальный идентификатор. После Django 3.2 появилась новая [конфигурация DEFAULT\_AUTO\_FIELD](https://docs.djangoproject.com/en/4.1/releases/3.2/#customizing-type-of-auto-created-primary-keys) для управления типом поля первичного ключа по умолчанию. Таким образом, `DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'` требуется, если вы хотите использовать миграции с бэкэндом django clickhouse.
