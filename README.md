<p align="center">
    <a>
        <img src="https://raw.githubusercontent.com/gri-gus/alchemica/main/assets/cover.jpg" alt="alchemica">
    </a>
</p>


<p align="center">
    <a href="https://pypi.org/project/alchemica" target="_blank">
        <img src="https://img.shields.io/pypi/v/alchemica" alt="PyPI">
    </a>
    <a href="https://pypi.org/project/alchemica" target="_blank">
        <img src="https://static.pepy.tech/badge/alchemica" alt="PyPI">
    </a>
    <a href="https://opensource.org/licenses/Apache-2.0" target="_blank">
        <img src="https://img.shields.io/badge/License-Apache_2.0-blue.svg" alt="Apache">
    </a>
</p>

# Alchemica - SQLAlchemy Logger

**Alchemica** — это мощный и гибкий инструмент для логирования SQL-запросов в Python-приложениях, работающих с
SQLAlchemy.
Он поддерживает как синхронный, так и асинхронный режимы работы, а также корректно функционирует в многопоточных и
многопроцессорных средах. Alchemica позволяет легко отслеживать выполнение SQL-запросов, анализировать их
производительность и выявлять потенциальные проблемы.

---

## Основные возможности

1. **Логирование SQL-запросов**
    * Логирует все SQL-запросы, выполняемые через SQLAlchemy, в контексте задекорированных функций.

    * Поддерживает логирование как параметризованных запросов, так и запросов с подставленными значениями (через
      `compile_sql=True`).

    * Каждый запрос сопровождается уникальным идентификатором (query_uuid), что упрощает отслеживание.

2. **Поддержка синхронного и асинхронного режимов**
    * Работает как с синхронными, так и с асинхронными приложениями.

    * Для синхронного режима используется декоратор `sync_sql_logger`.

    * Для асинхронного режима используется декоратор `async_sql_logger`.

3. **Многопоточная и многопроцессорная поддержка**
    * Корректно работает в многопоточных и многопроцессорных средах.

    * Каждый поток или процесс имеет изолированный контекст выполнения, что исключает пересечение данных.

4. **Логирование времени выполнения**
    * Замеряет время выполнения каждого SQL-запроса.

    * Предупреждает, если время выполнения превышает заданный порог (`critical_time`).

5. **План выполнения запросов** (`EXPLAIN`)
    * Поддерживает логирование плана выполнения запросов для анализа производительности.

    * Полезно для оптимизации сложных запросов.

6. **Гибкая настройка**
    * Позволяет настраивать уровень логирования (`log_level`), формат сообщений и другие параметры.

    * Поддерживает логирование дополнительной информации о выполнении (`log_execution_info`), такой как имя функции и
      UUID запроса.

7. **Изоляция контекста**
    * Логирует только те SQL-запросы, которые выполняются в контексте задекорированной функции.

    * Если внутри функции вызывается другая задекорированная функция, для нее создается новый контекст логирования.

## Установка

Для установки Alchemica выполните следующую команду:

```shell
pip install alchemica
```

## Быстрый старт

### Синхронный режим

```python
from alchemica import sync_sql_logger
from sqlalchemy import create_engine, text

# Создаем подключение к базе данных
engine = create_engine("sqlite:///:memory:")


# Декорируем функцию для логирования SQL-запросов
@sync_sql_logger(explain_plan=True)
def execute_query():
    with engine.connect() as conn:
        conn.execute(text("SELECT 1"))


if __name__ == '__main__':
    # Выполняем запрос
    execute_query()

```

Вывод в консоль:

```text
2025-03-06 02:39:53,565 - INFO - 🚀 [FUNC](func_name: execute_query; func_uuid: 318df3) запущена
2025-03-06 02:39:53,566 - INFO - 🟢 [SQL](func_name: execute_query; func_uuid: 318df3; query_uuid: 353cfd) SELECT 1
2025-03-06 02:39:53,566 - INFO - 📝 [SQL PLAN](func_name: execute_query; func_uuid: 318df3; query_uuid: 353cfd) 
0 Init 0 1 0 None 0 None
1 Integer 1 1 0 None 0 None
2 ResultRow 1 1 0 None 0 None
3 Halt 0 0 0 None 0 None
2025-03-06 02:39:53,566 - INFO - 🕑️ [SQL](func_name: execute_query; func_uuid: 318df3; query_uuid: 353cfd) Выполнено за 0.0001 сек
2025-03-06 02:39:53,566 - INFO - ✅ [FUNC](func_name: execute_query; func_uuid: 318df3) выполнена за 0.0009 сек
```

### Асинхронный режим

Для примера потребуется установить библиотеку `aiosqlite`:

```shell
pip install aiosqlite
```

```python
import asyncio

from alchemica import async_sql_logger
from sqlalchemy import text
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

# Создаем асинхронное подключение к базе данных
async_engine = create_async_engine("sqlite+aiosqlite:///:memory:")


# Декорируем асинхронную функцию для логирования SQL-запросов
@async_sql_logger()
async def execute_async_query():
    async with AsyncSession(async_engine) as session:
        await session.execute(text("SELECT 1"))


if __name__ == '__main__':
    # Выполняем асинхронный запрос
    asyncio.run(execute_async_query())

```

Вывод в консоль:

```text
2025-03-06 02:41:03,244 - INFO - 🚀 [FUNC](func_name: execute_async_query; func_uuid: f01219) запущена
2025-03-06 02:41:03,251 - INFO - 🟢 [SQL](func_name: execute_async_query; func_uuid: f01219; query_uuid: 2080d4) SELECT 1
2025-03-06 02:41:03,251 - INFO - 🕑️ [SQL](func_name: execute_async_query; func_uuid: f01219; query_uuid: 2080d4) Выполнено за 0.0005 сек
2025-03-06 02:41:03,252 - INFO - ✅ [FUNC](func_name: execute_async_query; func_uuid: f01219) выполнена за 0.0072 сек
```

## Общие параметры для sync_sql_logger и async_sql_logger

| Параметр          | Тип               | По умолчанию      | Описание                                                        |
|-------------------|-------------------|-------------------|-----------------------------------------------------------------|
| logger            | logging.Logger    | None              | Логгер для записи сообщений. Если не указан, используется стандартный логгер. |
| log_level         | int               | logging.INFO      | Уровень логирования (например, logging.INFO, logging.DEBUG).     |
| log_execution_info| bool              | True              | Логировать ли информацию о выполнении (имя функции, UUID запроса и т.д.). |
| compile_sql       | bool              | False             | Компилировать ли SQL-запросы перед логированием.                |
| critical_time     | float             | 1.0               | Время в секундах, после которого запрос считается медленным.    |
| explain_plan      | bool              | False             | Логировать ли план выполнения запроса (EXPLAIN).                |
| uuid_len          | int               | 6                 | Длина UUID для идентификации запросов.                          |

### Примеры использования

#### Логирование с компиляцией SQL

```python
@sync_sql_logger(compile_sql=True)
def execute_query():
    with engine.connect() as conn:
        conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
```

#### Логирование с планом выполнения

```python

@sync_sql_logger(explain_plan=True)
def execute_query():
    with engine.connect() as conn:
        conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
```

#### Логирование с критическим временем выполнения

```python
@sync_sql_logger(critical_time=0.5)
def execute_query():
    with engine.connect() as conn:
        conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": 1})
```

## Многопоточная поддержка

Alchemica корректно работает в многопоточных и многопроцессорных средах. Каждый поток или процесс имеет изолированный
контекст выполнения, что исключает пересечение данных между ними.

### Пример многопоточного использования

```python
import threading

from alchemica import sync_sql_logger
from sqlalchemy import create_engine, text

engine = create_engine("sqlite:///:memory:")


@sync_sql_logger()
def execute_query():
    with engine.connect() as conn:
        conn.execute(text("SELECT 1"))


def main():
    # Запускаем несколько потоков
    threads = []
    for _ in range(3):
        thread = threading.Thread(target=execute_query)
        threads.append(thread)
        thread.start()

    for thread in threads:
        thread.join()


if __name__ == '__main__':
    main()

```

Вывод в консоль:

```text
2025-03-06 02:51:15,422 - INFO - 🚀 [FUNC](func_name: execute_query; func_uuid: fa1278) запущена
2025-03-06 02:51:15,422 - INFO - 🚀 [FUNC](func_name: execute_query; func_uuid: 0a4fea) запущена
2025-03-06 02:51:15,422 - INFO - 🚀 [FUNC](func_name: execute_query; func_uuid: 4f27a3) запущена
2025-03-06 02:51:15,423 - INFO - 🟢 [SQL](func_name: execute_query; func_uuid: 4f27a3; query_uuid: 505aac) SELECT 1
2025-03-06 02:51:15,423 - INFO - 🕑️ [SQL](func_name: execute_query; func_uuid: 4f27a3; query_uuid: 505aac) Выполнено за 0.0003 сек
2025-03-06 02:51:15,424 - INFO - ✅ [FUNC](func_name: execute_query; func_uuid: 4f27a3) выполнена за 0.0013 сек
2025-03-06 02:51:15,424 - INFO - 🟢 [SQL](func_name: execute_query; func_uuid: 0a4fea; query_uuid: 16e6c1) SELECT 1
2025-03-06 02:51:15,424 - INFO - 🟢 [SQL](func_name: execute_query; func_uuid: fa1278; query_uuid: 571638) SELECT 1
2025-03-06 02:51:15,424 - INFO - 🕑️ [SQL](func_name: execute_query; func_uuid: fa1278; query_uuid: 571638) Выполнено за 0.0001 сек
2025-03-06 02:51:15,424 - INFO - ✅ [FUNC](func_name: execute_query; func_uuid: fa1278) выполнена за 0.0023 сек
2025-03-06 02:51:15,424 - INFO - 🕑️ [SQL](func_name: execute_query; func_uuid: 0a4fea; query_uuid: 16e6c1) Выполнено за 0.0005 сек
2025-03-06 02:51:15,424 - INFO - ✅ [FUNC](func_name: execute_query; func_uuid: 0a4fea) выполнена за 0.0022 сек
```

## Логирование в асинхронном режиме

Alchemica поддерживает асинхронные приложения, работающие с SQLAlchemy и asyncio.

### Пример асинхронного использования

```python
import asyncio

from alchemica import async_sql_logger
from sqlalchemy import Table, Column, Integer, String, MetaData
from sqlalchemy import insert
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession

# Создаем асинхронное подключение к базе данных
async_engine = create_async_engine("sqlite+aiosqlite:///:memory:")

# Определяем метаданные и таблицу
metadata = MetaData()
users_table = Table(
    "users",
    metadata,
    Column("id", Integer, primary_key=True),
    Column("name", String),
)


@async_sql_logger(compile_sql=True, explain_plan=True)
async def create_user(name: str):
    """
    Функция для создания нового пользователя.
    """
    async with AsyncSession(async_engine) as session:
        # Используем SQLAlchemy Core для создания запроса
        stmt = insert(users_table).values(name=name)
        await session.execute(stmt)
        await session.commit()


# Основная асинхронная функция
@async_sql_logger(compile_sql=True)
async def main():
    # Создаем таблицу users
    async with async_engine.begin() as conn:
        await conn.run_sync(metadata.create_all)

    # Создаем пользователя
    await create_user("Alice")


# Запускаем асинхронный код
if __name__ == '__main__':
    asyncio.run(main())


```

Вывод в консоль:

```text
2025-03-06 03:11:34,493 - INFO - 🚀 [FUNC](func_name: main; func_uuid: 6c1054) запущена
2025-03-06 03:11:34,495 - INFO - 🟢 [SQL](func_name: main; func_uuid: 6c1054; query_uuid: 155ef2) 
CREATE TABLE users (
	id INTEGER NOT NULL, 
	name VARCHAR, 
	PRIMARY KEY (id)
)


2025-03-06 03:11:34,495 - INFO - 🕑️ [SQL](func_name: main; func_uuid: 6c1054; query_uuid: 155ef2) Выполнено за 0.0003 сек
2025-03-06 03:11:34,496 - INFO - 🚀 [FUNC](func_name: create_user; func_uuid: ddb888) запущена
2025-03-06 03:11:34,496 - INFO - 🟢 [SQL](func_name: create_user; func_uuid: ddb888; query_uuid: fb157e) INSERT INTO users (name) VALUES ('Alice')
2025-03-06 03:11:34,496 - INFO - 📝 [SQL PLAN](func_name: create_user; func_uuid: ddb888; query_uuid: fb157e) 
0 Init 0 8 0 None 0 None
1 OpenWrite 0 2 0 2 0 None
2 SoftNull 2 0 0 None 0 None
3 String8 0 3 0 Alice 0 None
4 NewRowid 0 1 0 None 0 None
5 MakeRecord 2 2 4 DB 0 None
6 Insert 0 4 1 users 57 None
7 Halt 0 0 0 None 0 None
8 Transaction 0 1 1 0 1 None
9 Goto 0 1 0 None 0 None
2025-03-06 03:11:34,497 - INFO - 🕑️ [SQL](func_name: create_user; func_uuid: ddb888; query_uuid: fb157e) Выполнено за 0.0005 сек
2025-03-06 03:11:34,497 - INFO - ✅ [FUNC](func_name: create_user; func_uuid: ddb888) выполнена за 0.0015 сек
2025-03-06 03:11:34,497 - INFO - ✅ [FUNC](func_name: main; func_uuid: 6c1054) выполнена за 0.0039 сек
```

## Логирование контекста в Alchemica

Одной из ключевых особенностей Alchemica является изоляция контекста выполнения. Это означает, что декоратор
sync_sql_logger или async_sql_logger логирует только те SQL-запросы, которые выполняются в контексте функции, к которой
он применен. Если внутри этой функции вызывается другая функция, также декорированная sync_sql_logger или
async_sql_logger, то логироваться будут только запросы в контексте этой вложенной функции, а не в контексте родительской
функции.

### Как это работает

1. **Контекст выполнения:**
    * Каждый вызов декорированной функции создает уникальный контекст выполнения, который идентифицируется с помощью
      UUID.

    * Логируются только те SQL-запросы, которые выполняются в этом контексте.

2. **Вложенные функции:**
    * Если внутри декорированной функции вызывается другая функция, также декорированная `sync_sql_logger` или
      `async_sql_logger`, то для нее создается новый контекст выполнения.

    * Запросы внутри вложенной функции логируются в ее контексте, а не в контексте родительской функции.

### Пример

Рассмотрим пример, где есть две функции: `outer_function` и `inner_function`.
Обе функции декорированы `sync_sql_logger`.

```python
from sqlalchemy import create_engine, text
from alchemica import sync_sql_logger

# Создаем подключение к базе данных
engine = create_engine("sqlite:///:memory:")


@sync_sql_logger()
def outer_function():
    """
    Внешняя функция, которая вызывает внутреннюю функцию.
    """
    with engine.connect() as conn:
        conn.execute(text("SELECT 1"))  # Запрос 1
        inner_function()  # Вызов внутренней функции
        conn.execute(text("SELECT 2"))  # Запрос 2


@sync_sql_logger()
def inner_function():
    """
    Внутренняя функция, которая выполняет свой SQL-запрос.
    """
    with engine.connect() as conn:
        conn.execute(text("SELECT 3"))  # Запрос 3


# Вызов внешней функции
outer_function()
```

Вывод в консоль:

```text
2025-03-06 03:18:04,659 - INFO - 🚀 [FUNC](func_name: outer_function; func_uuid: 77afc9) запущена
2025-03-06 03:18:04,660 - INFO - 🟢 [SQL](func_name: outer_function; func_uuid: 77afc9; query_uuid: 07d89d) SELECT 1
2025-03-06 03:18:04,660 - INFO - 🕑️ [SQL](func_name: outer_function; func_uuid: 77afc9; query_uuid: 07d89d) Выполнено за 0.0002 сек
2025-03-06 03:18:04,660 - INFO - 🚀 [FUNC](func_name: inner_function; func_uuid: ef9eed) запущена
2025-03-06 03:18:04,660 - INFO - 🟢 [SQL](func_name: inner_function; func_uuid: ef9eed; query_uuid: 86e750) SELECT 3
2025-03-06 03:18:04,660 - INFO - 🕑️ [SQL](func_name: inner_function; func_uuid: ef9eed; query_uuid: 86e750) Выполнено за 0.0001 сек
2025-03-06 03:18:04,660 - INFO - ✅ [FUNC](func_name: inner_function; func_uuid: ef9eed) выполнена за 0.0002 сек
2025-03-06 03:18:04,660 - INFO - 🟢 [SQL](func_name: outer_function; func_uuid: 77afc9; query_uuid: db39e7) SELECT 2
2025-03-06 03:18:04,660 - INFO - 🕑️ [SQL](func_name: outer_function; func_uuid: 77afc9; query_uuid: db39e7) Выполнено за 0.0001 сек
2025-03-06 03:18:04,660 - INFO - ✅ [FUNC](func_name: outer_function; func_uuid: 77afc9) выполнена за 0.0014 сек
```

### Ключевые моменты

1. **Изоляция контекста:**

    * Каждая функция (outer_function и inner_function) имеет свой уникальный контекст выполнения, который
      идентифицируется с помощью func_uuid.

    * Логируются только те SQL-запросы, которые выполняются в контексте текущей функции.

2. **Уникальные идентификаторы:**

    * Каждый SQL-запрос имеет свой уникальный идентификатор (query_uuid), что позволяет отслеживать выполнение отдельных
      запросов.

3. **Время выполнения:**

    * Для каждого SQL-запроса и функции логируется время выполнения, что помогает анализировать производительность.

4. **Иерархия вызовов:**

    * Логи четко показывают, какая функция вызывает другую, и какие запросы выполняются в каждом контексте.

> Важно отметить, что если оставить декоратор только на `outer_function`, то все запросы будут добавлены в лог в
> контексте этой функции. А если оставить декоратор только на `inner_function`, то запросы из `outer_function` не
> будут добавлены в лог.

### Визуализация

Для наглядности можно представить выполнение программы в виде дерева:

```text
outer_function (func_uuid: 77afc9)
├── SQL: SELECT 1 (query_uuid: 07d89d)
├── inner_function (func_uuid: ef9eed)
│   └── SQL: SELECT 3 (query_uuid: 86e750)
└── SQL: SELECT 2 (query_uuid: db39e7)
```

## Логируемые сообщения

Alchemica логирует следующие события:

Запуск функции:

```text
🚀 [FUNC](func_name: execute_query; func_uuid: abc123) запущена
```

Выполнение SQL-запроса:

```text
🟢 [SQL](func_name: execute_query; func_uuid: abc123; query_uuid: def456) SELECT * FROM users WHERE id = :id
```

Завершение функции:

```text
✅ [FUNC](func_name: execute_query; func_uuid: abc123) выполнена за 0.1234 сек
```

Медленный запрос:

```text
⚠️ [SQL](func_name: execute_query; func_uuid: abc123) Запрос выполнялся дольше 1 секунд: 1.2345 сек
```

План выполнения (EXPLAIN):

```text
📝 [SQL PLAN](func_name: execute_query; func_uuid: abc123)
SCAN TABLE users
```

Ошибки:

```text
❌ [SQL ERROR](func_name: execute_query; func_uuid: abc123) Ошибка при выполнении запроса: ...
```

---

Alchemica — ваш надежный помощник в логировании SQL-запросов! 🚀
