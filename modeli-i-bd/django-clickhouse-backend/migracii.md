# Миграции

Миграции трудны и сложны. Наиболее сложным является изменение схемы, например изменение типа поля.

Поскольку ограничения **clickhouse**, такие как отсутствие поддержки транзакций и ограничения уникальности, а не сортировка на уровне таблиц, миграции clickhouse немного отличаются от СУБД, таких как postgresql.

## Ограничения

* Все операции, связанные с уникальным ограничением.
* Хотя вы можете использовать **ForeignKey** в модели (поскольку django может обрабатывать это в python), но ограничения **fk** на уровне базы данных не будут созданы.
* Миграция данных не откатывается при возникновении исключения.
* Анонимные индексы и ограничения (db\_index, unique, index\_together, unique\_together) не поддерживаются, вы должны указать имена явно.
* Изменение типа поля, с которым несовместимо, вызовет исключение, когда поле используется в первичном ключе или в порядке. Совместимость означает, что при изменении длина значения не изменяется, а порядок сохраняется.

```
DB::Exception: ALTER of key column id from type FixedString(100)
               to type FixedString(99) is not safe because it can change
               the representation of primary key.
```

* Изменение типа поля, с которым несовместимо, вызовет исключение, когда поле используется в индексе. Поддерживается тип данных от не нулевого до нулевого, но не реверс.

```
DB::Exception: ALTER of key column pink is forbidden.
```

## Различия

* Используйте `clickhouse_backend.models.indexes.Index` вместо `django.db.models.Index`, чтобы добавить индекс.
* При использовании `migrations.RunSQL` для выполнения необработанного SQL-запроса многострочный SQL, разделенный точкой с запятой, не поддерживается.
* При использовании `migrations.RunSQL` для выполнения запроса **INSERT INTO** значения должны быть предоставлены с использованием параметров и заканчиваться предложением **VALUES**.

```python
migrations.RunSQL(
    # forwards
    (
        [
            "INSERT INTO i_love_ponies (id, special_thing) VALUES;",
            [(1, "Django"), (2, "Ponies")],
        ],
        (
            "INSERT INTO i_love_ponies (id, special_thing) VALUES;",
            [(3, "Python")],
        ),
    ),
    # backwards
    [
        "ALTER TABLE i_love_ponies DELETE WHERE special_thing = 'Django';",
        ["ALTER TABLE i_love_ponies DELETE WHERE special_thing = 'Ponies';", None],
        (
            "ALTER TABLE i_love_ponies DELETE WHERE id = %s OR special_thing = %s;",
            [3, "Python"],
        ),
    ],
)
```
