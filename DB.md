## **🔹 Реляционные (SQL) vs NoSQL базы данных**  

Реляционные (SQL) и NoSQL базы данных решают **разные задачи**, и выбор между ними зависит от **типа данных**, **нагрузки** и **требований к масштабируемости**.  

---

## **1️⃣ Основные различия между SQL и NoSQL**  

| 🔹 Характеристика      | 🔹 Реляционные (SQL) | 🔹 NoSQL |
|-----------------------|----------------------|---------|
| **Структура данных**  | Таблицы с жесткой схемой | Гибкая, может быть без схемы |
| **Язык запросов**     | SQL (SELECT, JOIN, WHERE) | Нет единого стандарта (MongoDB → BSON, Redis → ключ-значение) |
| **Целостность данных** | ACID (Atomicity, Consistency, Isolation, Durability) | BASE (Basically Available, Soft state, Eventually consistent) |
| **Масштабируемость**  | Вертикальная (мощнее сервер) | Горизонтальная (распределение данных по узлам) |
| **Поддержка сложных связей** | Да (JOIN, Foreign Keys) | Слабая поддержка или её нет |
| **Производительность** | Медленнее при больших нагрузках | Высокая при распределенных нагрузках |
| **Типы данных** | Четкая схема (int, varchar, json) | Гибкость (JSON, BSON, key-value, документ-ориентированная структура) |

---

## **2️⃣ Разбор на примерах**  

### **📌 Реляционные базы данных (SQL)**  
> Используют **жесткую схему** и хорошо подходят для **структурированных данных** с четкими связями.

#### **Пример таблиц в PostgreSQL/MySQL**  

📌 **Таблица `users`**
| id | name   | email            |
|----|--------|------------------|
| 1  | Alice  | alice@mail.com   |
| 2  | Bob    | bob@mail.com     |

📌 **Таблица `orders`**
| id | user_id | product | amount |
|----|---------|---------|--------|
| 1  | 1       | Laptop  | 1000   |
| 2  | 2       | Phone   | 500    |

📌 **SQL-запрос с `JOIN` (связь между таблицами)**  
```sql
SELECT users.name, orders.product, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```
🔹 **Преимущества SQL:**  
- Хорошо подходит для **финансовых систем, CRM, ERP**.  
- Гарантирует **целостность данных**.  
- Поддерживает сложные запросы (`JOIN`, `GROUP BY`).  

🔹 **Недостатки SQL:**  
- Сложнее масштабировать **горизонтально** (разделение на серверы).  
- Структура данных **жестко задана**, нельзя просто так добавить произвольное поле.  

---

### **📌 NoSQL базы данных (MongoDB, Redis, Cassandra)**  
> Гибкие, масштабируемые базы, хорошо подходящие для **Big Data**, **IoT**, **Social Networks**.

#### **Пример хранения данных в MongoDB (JSON-документ)**  
```json
{
  "_id": "1",
  "name": "Alice",
  "email": "alice@mail.com",
  "orders": [
    { "product": "Laptop", "amount": 1000 }
  ]
}
```
📌 **Запрос на поиск пользователя по email в MongoDB**  
```js
db.users.findOne({ email: "alice@mail.com" })
```

🔹 **Преимущества NoSQL:**  
- **Гибкость** → можно хранить разные структуры данных.  
- **Горизонтальное масштабирование** → легко работать с большими нагрузками.  
- Отлично подходит для **реального времени, логов, аналитики**.  

🔹 **Недостатки NoSQL:**  
- **Нет строгих связей** → сложнее делать аналитику.  
- **Нет ACID-гарантий (по умолчанию)** → могут быть **рассинхронизации** данных.  
- **Меньше инструментов для сложных отчетов**.  

---

## **3️⃣ Когда использовать SQL, а когда NoSQL?**  

📌 **Когда SQL (PostgreSQL, MySQL, MariaDB, etc.)?**  
✅ Структурированные данные, которые **не часто меняются**.  
✅ Нужно гарантировать **целостность (ACID)**.  
✅ Приложение с **отчетностью и сложными запросами**.  
✅ **Финансовые системы, банковские приложения, CRM, ERP**.  

📌 **Когда NoSQL (MongoDB, Redis, Cassandra, etc.)?**  
✅ Неструктурированные **или часто изменяющиеся данные**.  
✅ Нужно **масштабирование** (миллионы пользователей).  
✅ Высокая скорость записи/чтения **(Real-time данные, кэши, логирование)**.  
✅ **Чаты, социальные сети, IoT, big data, machine learning**.  

---

## **🔹 Итог**  
| Выбор базы | Когда использовать? |
|------------|----------------------|
| **SQL (PostgreSQL, MySQL)** | Строгая структура, связи, финансы, отчетность |
| **NoSQL (MongoDB, Redis, etc.)** | Гибкость, высокая нагрузка, real-time данные |

💡 **Если важна целостность данных → SQL**  
💡 **Если важна скорость и масштабируемость → NoSQL**  

🔹 **Гибридный подход** тоже возможен (например, PostgreSQL + Redis для кэша). 🚀

## **🔹 Как правильно проектировать схему базы данных?**  
Грамотное проектирование базы данных **повышает производительность**, **снижает дублирование данных** и **облегчает поддержку**.  

---

## **1️⃣ Анализ требований**  
Перед проектированием схемы нужно понять:  
✅ **Какие данные будут храниться?**  
✅ **Какие операции будут выполняться чаще всего?** (чтение/запись/аналитика)  
✅ **Какой уровень нагрузки?** (1000 пользователей или 1 млн?)  
✅ **Как данные будут обновляться и масштабироваться?**  

**Пример:**  
📌 Разрабатываем **интернет-магазин** → храним **пользователей, заказы, товары**.  

---

## **2️⃣ Нормализация vs Денормализация**  
🔹 **Нормализация** → устраняет дублирование данных и делает структуру логичной.  
🔹 **Денормализация** → ускоряет сложные запросы, но увеличивает объем данных.  

### **Пример нормализации (SQL)**  
📌 **Плохо (денормализованные данные)**  
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_name VARCHAR(255),
    user_email VARCHAR(255),
    product_name VARCHAR(255),
    amount INT
);
```
🔻 **Проблемы**:  
- Если **user_name/email** изменится, придется менять все записи.  
- Дублирование данных.  

📌 **Хорошо (нормализованная структура)**  
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT REFERENCES users(id),
    product_id INT REFERENCES products(id),
    amount INT
);
```
🔹 **Преимущества**:  
✅ **Нет дублирования данных**.  
✅ **Изменения пользователя применяются ко всем заказам**.  
✅ **Более эффективные обновления**.  

---

## **3️⃣ Выбор типов данных**  
### **📌 Оптимальный выбор типов данных уменьшает размер БД и ускоряет запросы.**  

✅ **Для чисел:**  
- `INT` → если число до 2 млрд  
- `BIGINT` → если нужно больше  
- `DECIMAL(10,2)` → для денег  

✅ **Для текста:**  
- `VARCHAR(255)` → если длина ограничена  
- `TEXT` → если не знаем длину  

✅ **Для даты и времени:**  
- `TIMESTAMP` → с учетом временной зоны  
- `DATETIME` → если без часового пояса  

📌 **Ошибка** → использовать `VARCHAR(255)` для чисел (неоптимально по памяти и скорости).  

---

## **4️⃣ Индексы для ускорения поиска**  
📌 **Индексы нужны, если часто ищем по определенным колонкам.**  

```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```
🔹 **Когда использовать индексы?**  
✅ Поля, по которым часто **фильтруем (`WHERE`)**.  
✅ Поля, по которым часто **сортируем (`ORDER BY`)**.  
✅ Поля, по которым часто **соединяем таблицы (`JOIN`)**.  

🔻 **Когда НЕ использовать индексы?**  
🚫 Поля, которые **часто обновляются** (индексы замедляют `INSERT/UPDATE`).  
🚫 Маленькие таблицы (до 1000 строк).  

---

## **5️⃣ Масштабируемость (SQL vs NoSQL)**  
📌 Если данных **очень много**, может понадобиться **шардирование** или **репликация**.  

### **SQL (PostgreSQL, MySQL)**  
✅ **Горизонтальное масштабирование** (шардирование)  
✅ **Репликация для отказоустойчивости**  

```sql
CREATE TABLE orders_2023 AS TABLE orders;
```
📌 **Хранение заказов за разные годы в разных таблицах (партицирование).**  

### **NoSQL (MongoDB, Cassandra)**  
✅ Данные автоматически распределяются по кластерам.  
✅ Отлично работает с **большими нагрузками**.  

---

## **6️⃣ Денормализация для NoSQL**  
📌 **В NoSQL часто дублируем данные ради скорости**.  

📌 **SQL (нормализованное хранение)**  
```sql
users:
{ "_id": 1, "name": "Alice" }

orders:
{ "_id": 1, "user_id": 1, "product": "Laptop" }
```
📌 **MongoDB (денормализованное хранение)**  
```json
{
  "_id": 1,
  "name": "Alice",
  "orders": [
    { "product": "Laptop" }
  ]
}
```
🔹 **Плюсы**: Запрос быстрее (нет `JOIN`).  
🔻 **Минусы**: Если имя изменится, надо менять везде.  

---

## **7️⃣ Кэширование для ускорения работы**  
📌 **Redis/Memcached** для хранения часто используемых данных.  

🔹 **Пример кэша в Redis**  
```bash
SET user:1 "{'name': 'Alice'}"
```
📌 **Запрос в Redis быстрее, чем в SQL!**  

---

## **🔹 Итог**  
✅ **Нормализуем данные в SQL для целостности**.  
✅ **Используем индексы для ускорения запросов**.  
✅ **Продумываем масштабируемость (шардирование, репликация)**.  
✅ **В NoSQL денормализуем для скорости**.  
✅ **Используем кэш Redis для быстрого доступа**.  

🔥 **Есть вопросы? Готов разобрать конкретный кейс!** 🚀

## **🔹 Виды масштабирования в PostgreSQL**  

Масштабирование PostgreSQL бывает **вертикальное** (мощнее сервер) и **горизонтальное** (несколько серверов). Рассмотрим все подходы подробно.  

---

## **1️⃣ Вертикальное масштабирование (Scaling Up)**  
📌 **Увеличение мощности одного сервера (CPU, RAM, SSD).**  

### **Плюсы**  
✅ Простая реализация (не требует переработки архитектуры).  
✅ Хорошо работает для небольших и средних нагрузок.  

### **Минусы**  
❌ Ограничено железом (невозможно бесконечно увеличивать).  
❌ Дорогой сервер стоит намного дороже, чем несколько средних.  

### **Когда использовать?**  
✅ Когда еще можно увеличить мощность сервера без проблем.  
✅ Когда горизонтальное масштабирование слишком сложное или не требуется.  

---

## **2️⃣ Репликация (Master-Slave, Master-Replica)**  
📌 **Создание копий базы (реплик) для разгрузки чтения.**  

🔹 **Как работает?**  
- **Master** → основной сервер (принимает `INSERT`, `UPDATE`, `DELETE`).  
- **Slaves (реплики)** → только чтение (`SELECT`).  

### **Типы репликации в PostgreSQL:**  
1. **Физическая (Streaming Replication)** – передает бинарные WAL-журналы.  
2. **Логическая (Logical Replication)** – передает только изменения данных.  
3. **Синхронная и асинхронная** – баланс между скоростью и целостностью данных.  

🔹 **Настройка физической репликации (Streaming Replication)**  
📌 В `postgresql.conf` на **мастере**:  
```ini
wal_level = replica
max_wal_senders = 5
synchronous_commit = on
```
📌 На **реплике** (в `recovery.conf`):  
```ini
standby_mode = on
primary_conninfo = 'host=master_ip port=5432 user=replicator password=secret'
```

### **Плюсы**  
✅ Уменьшает нагрузку на основной сервер (чтение уходит на реплики).  
✅ Защищает от потери данных (если мастер упадет, можно переключиться на реплику).  

### **Минусы**  
❌ **Не решает проблему записи** (все `INSERT`/`UPDATE` всё равно идут на мастер).  
❌ **Реплики отстают** (если нагрузка большая).  

### **Когда использовать?**  
✅ Когда **нагрузка на чтение высокая**.  
✅ Когда нужна отказоустойчивость.  

---

## **3️⃣ Шардирование (Sharding, Horizontal Scaling)**  
📌 **Разбивает данные по разным серверам (шардам).**  

🔹 **Как работает?**  
- **Каждый сервер хранит только часть данных**.  
- Запросы делятся между серверами.  
- Например, **шардирование по пользователям**:  
  - `users_A-M` → сервер 1  
  - `users_N-Z` → сервер 2  

🔹 **Методы шардирования:**  
1. **По ID (`RANGE` или `HASH`)**  
   - `user_id % 2 = 0` → сервер 1  
   - `user_id % 2 = 1` → сервер 2  
2. **По географии (Geo-Sharding)**  
   - США → сервер 1  
   - Европа → сервер 2  
3. **По типу данных**  
   - Транзакции → сервер 1  
   - Логи → сервер 2  

🔹 **Реализация в PostgreSQL:**  
- Использование **pg_shard**, **Citus** (расширения для шардинга).  
- Пример `FOREIGN DATA WRAPPER` (FDW) для шардирования:  
```sql
CREATE SERVER shard1 FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host 'shard1_ip', dbname 'mydb');
CREATE SERVER shard2 FOREIGN DATA WRAPPER postgres_fdw OPTIONS (host 'shard2_ip', dbname 'mydb');
```

### **Плюсы**  
✅ Позволяет **обрабатывать большие объемы данных**.  
✅ Горизонтально **масштабируется** (можно добавлять новые сервера).  
✅ Можно **разделять нагрузку** по типу данных.  

### **Минусы**  
❌ **Сложнее запросы (`JOIN` между серверами неудобен)**.  
❌ Требует **дополнительного уровня логики** в коде.  

### **Когда использовать?**  
✅ Когда база **слишком большая для одного сервера**.  
✅ Когда **нужно распределять нагрузку на запись**.  

---

## **4️⃣ Партиционирование (Partitioning)**  
📌 **Разделение данных внутри одного сервера** (разные таблицы для разных диапазонов данных).  

🔹 **Пример партиционирования заказов по году:**  
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    created_at DATE NOT NULL
) PARTITION BY RANGE (created_at);

CREATE TABLE orders_2023 PARTITION OF orders FOR VALUES FROM ('2023-01-01') TO ('2023-12-31');
CREATE TABLE orders_2024 PARTITION OF orders FOR VALUES FROM ('2024-01-01') TO ('2024-12-31');
```
🔹 **Как работает?**  
- Запросы автоматически идут **только в нужную таблицу**.  
- `SELECT * FROM orders WHERE created_at > '2023-01-01'` → ищет только в `orders_2023`.  

### **Плюсы**  
✅ Ускоряет запросы (`SELECT` обращается только к нужному разделу).  
✅ Уменьшает размер индексов (каждая партиция индексируется отдельно).  

### **Минусы**  
❌ **Не помогает распределить нагрузку по серверам**.  
❌ Требует **правильной настройки ключей**.  

### **Когда использовать?**  
✅ Когда есть **огромные таблицы (миллионы записей)**.  
✅ Когда данные удобно **разделить по диапазону (даты, регионы)**.  

---

## **5️⃣ Балансировка нагрузки (Pgpool-II, ProxySQL)**  
📌 **Распределяет запросы между репликами или шардами.**  

🔹 **Пример с Pgpool-II**  
- `SELECT` → идет на реплику.  
- `INSERT` → идет на мастер.  

```ini
load_balance_mode = on
backend_hostname0 = 'master_ip'
backend_hostname1 = 'replica1_ip'
backend_hostname2 = 'replica2_ip'
```
📌 **Pgpool-II умеет:**  
✅ Автоматически **переключаться на реплику** при сбое.  
✅ Разгружать сервер от запросов на **чтение**.  

### **Плюсы**  
✅ Уменьшает нагрузку на мастер.  
✅ Улучшает **отказоустойчивость**.  

### **Минусы**  
❌ Не решает проблему нагрузки на запись.  
❌ Требует **дополнительного сервера**.  

### **Когда использовать?**  
✅ Когда **много запросов на чтение**.  
✅ Когда нужна **автоматическая балансировка нагрузки**.  

---

## **🔹 Итог**  
| Метод | Что делает? | Когда использовать? |
|--------|------------|----------------------|
| **Вертикальное** | Увеличивает мощность сервера | Пока не достигли предела |
| **Репликация** | Копии для чтения | Когда много `SELECT` |
| **Шардирование** | Данные на разных серверах | Когда база **больше 1 TB** |
| **Партиционирование** | Разделяет таблицу на части | Когда **миллионы записей** |
| **Балансировка** | Разделяет запросы | Когда **много пользователей** |

💡 **Комбинированные стратегии** (например, репликация + партиционирование) дают наилучший эффект. 🚀


## **🔹 Транзакции и связи между таблицами в PostgreSQL**

### **1️⃣ Транзакции в PostgreSQL**  
**Транзакция** — это последовательность операций, которая выполняется как единое целое. Она либо **коммитится** (завершается успешно), либо **откатывается** (в случае ошибки). Транзакции позволяют гарантировать **целостность данных** и обеспечивают **атомарность операций**.

#### **Основные принципы транзакций (ACID):**
1. **A**tomacity (Атомарность): Операции внутри транзакции выполняются полностью или не выполняются вообще.
2. **C**onsistency (Согласованность): Транзакция переводит базу данных из одного согласованного состояния в другое.
3. **I**solation (Изоляция): Каждая транзакция изолирована от других.
4. **D**urability (Долговечность): После коммита транзакции изменения сохраняются.

---

### **Как транзакции работают в PostgreSQL:**

1. **Начало транзакции:**  
   Для начала транзакции используется команда `BEGIN` или `START TRANSACTION`.
   ```sql
   BEGIN;
   ```

2. **Выполнение операций:**  
   В рамках транзакции можно выполнять любые SQL-операции: `INSERT`, `UPDATE`, `DELETE`, `SELECT`. Эти изменения не будут видны другим транзакциям, пока не произойдет коммит.

3. **Коммит транзакции (commit):**  
   После выполнения всех операций и проверки их правильности, транзакция может быть зафиксирована с помощью команды `COMMIT`, что приведет к постоянному сохранению изменений.
   ```sql
   COMMIT;
   ```

4. **Откат транзакции (rollback):**  
   Если во время выполнения транзакции произошла ошибка или нужно отменить изменения, можно откатить все операции с помощью команды `ROLLBACK`.
   ```sql
   ROLLBACK;
   ```

---

### **Пример транзакции:**
Предположим, что мы создаем заказ в интернет-магазине, и для этого нужно уменьшить количество товара на складе.

```sql
BEGIN;

-- Уменьшаем количество товара
UPDATE products SET stock = stock - 1 WHERE id = 123;

-- Создаем новый заказ
INSERT INTO orders (user_id, product_id, quantity, order_date) 
VALUES (1, 123, 1, CURRENT_DATE);

-- Если все операции успешны, фиксируем изменения
COMMIT;
```

Если что-то пошло не так (например, товар в наличии меньше, чем указано в заказе), мы можем откатить все изменения:

```sql
ROLLBACK;
```

### **Изоляция транзакций:**
Транзакции могут работать с разными уровнями изоляции, которые определяют, как транзакции будут видеть друг друга. В PostgreSQL доступны следующие уровни изоляции:

- **Read Uncommitted** — транзакции могут видеть данные других транзакций, даже если они еще не были зафиксированы.
- **Read Committed** (по умолчанию) — транзакции видят только те данные, которые были зафиксированы до начала текущей транзакции.
- **Repeatable Read** — транзакция видит одну и ту же версию данных на протяжении всего своего выполнения.
- **Serializable** — обеспечивает самую строгую изоляцию, как если бы транзакции выполнялись поочередно.

Пример:
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

---

### **2️⃣ Связи между таблицами в PostgreSQL:**

Связи между таблицами позволяют эффективно структурировать данные и обеспечивать **целостность данных**. В PostgreSQL связи между таблицами реализуются через **внешние ключи (Foreign Keys)**.

#### **Типы связей:**

1. **Один к одному (One-to-One)**  
   Каждая запись в одной таблице связана с одной записью в другой таблице.
   
   📌 **Пример:**  
   Мы можем создать таблицы для **пользователей** и их **адресов**, где каждый пользователь имеет только один адрес.
   ```sql
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100)
   );

   CREATE TABLE addresses (
       id SERIAL PRIMARY KEY,
       user_id INT UNIQUE,
       address TEXT,
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

   В данном примере связь "Один к одному" между `users` и `addresses` реализована с помощью **внешнего ключа** (`user_id`), который является уникальным.

2. **Один ко многим (One-to-Many)**  
   Одна запись в таблице может быть связана с множеством записей в другой таблице.
   
   📌 **Пример:**  
   Например, один **пользователь** может иметь несколько **заказов** в интернет-магазине.
   ```sql
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100)
   );

   CREATE TABLE orders (
       id SERIAL PRIMARY KEY,
       user_id INT,
       total_amount DECIMAL,
       FOREIGN KEY (user_id) REFERENCES users(id)
   );
   ```

   Здесь связь "Один ко многим" между `users` и `orders` создается через внешний ключ `user_id`.

3. **Многие ко многим (Many-to-Many)**  
   Записи в обеих таблицах могут быть связаны с множеством записей в другой таблице.

   📌 **Пример:**  
   Например, один **пользователь** может быть связан с несколькими **группами**, а каждая группа может содержать нескольких пользователей. Для реализации такой связи используется промежуточная таблица.
   ```sql
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100)
   );

   CREATE TABLE groups (
       id SERIAL PRIMARY KEY,
       name VARCHAR(100)
   );

   CREATE TABLE user_groups (
       user_id INT,
       group_id INT,
       FOREIGN KEY (user_id) REFERENCES users(id),
       FOREIGN KEY (group_id) REFERENCES groups(id),
       PRIMARY KEY (user_id, group_id)
   );
   ```

   В этой структуре связь "Многие ко многим" реализована через промежуточную таблицу `user_groups`.

#### **Целостность данных с помощью внешних ключей:**
- Внешние ключи обеспечивают **целостность данных**, гарантируя, что в родительской таблице существует запись, на которую ссылается внешний ключ.
- Также можно определить **какую операцию выполнять при удалении или обновлении записей**:
  - `ON DELETE CASCADE` — удаляет все связанные записи в дочерней таблице.
  - `ON UPDATE CASCADE` — обновляет значения внешнего ключа в дочерней таблице.

Пример:
```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

---

### **3️⃣ Пример транзакций и связей между таблицами:**

Предположим, что мы выполняем транзакцию, в рамках которой создаем заказ и уменьшаем количество товара на складе. При этом связь между заказом и пользователем должна быть целостной.

```sql
BEGIN;

-- Добавляем заказ
INSERT INTO orders (user_id, product_id, quantity) 
VALUES (1, 123, 2);

-- Обновляем количество товара
UPDATE products SET stock = stock - 2 
WHERE id = 123;

-- Если все успешно, подтверждаем
COMMIT;
```

Если при уменьшении товара количество товара недостаточно, можно откатить транзакцию, чтобы избежать создания заказа без товара.

```sql
ROLLBACK;
```

---

## **🔹 Заключение:**
- **Транзакции** позволяют гарантировать **атомарность**, **целостность** и **изоляцию** операций.
- **Связи между таблицами** (через внешние ключи) обеспечивают **нормализацию** данных и гарантируют их целостность.
- **Транзакции и связи** работают вместе, чтобы обеспечить корректное и надежное выполнение операций с данными.

## **🔹 Изоляция транзакций в PostgreSQL:**

Изоляция транзакций — это один из принципов ACID, который определяет, как транзакции взаимодействуют друг с другом в многозадачной среде. Это важно для предотвращения различных проблем, таких как **непоследовательные чтения** и **потери данных**. В PostgreSQL есть несколько уровней изоляции, которые контролируют, как транзакции видят друг друга.

### **1️⃣ Уровни изоляции транзакций в PostgreSQL**

PostgreSQL поддерживает **4 уровня изоляции** транзакций:

1. **Read Uncommitted**  
2. **Read Committed** (по умолчанию)  
3. **Repeatable Read**  
4. **Serializable**  

### **1.1 Read Uncommitted**

🔹 **Описание:**  
Это самый низкий уровень изоляции. На этом уровне транзакции могут видеть **неподтвержденные изменения** других транзакций. Это может привести к **грязным чтениям** (dirty reads), когда транзакция читает данные, которые могут быть отменены другой транзакцией.

🔹 **Пример:**
- Транзакция A обновляет запись.
- Транзакция B читает обновленные данные, но до того, как транзакция A завершится (не будет зафиксирована).
- Транзакция A отменяет изменения.
- Транзакция B будет работать с несуществующими или неправильными данными.

🔹 **Использование в PostgreSQL:**  
PostgreSQL не поддерживает полный уровень изоляции `Read Uncommitted` (он всегда действует как `Read Committed`). Поэтому такой уровень изоляции в PostgreSQL невозможен, даже если запросит приложение.

---

### **1.2 Read Committed** (по умолчанию)

🔹 **Описание:**  
На уровне **Read Committed** транзакция может видеть только те данные, которые были **подтверждены** (зафиксированы) другими транзакциями на момент начала выполнения запроса. Это предотвращает **грязные чтения**, но не защищает от **фантомных чтений** или **неповторяющихся чтений**.

- **Грязное чтение:** Когда транзакция читает данные, которые могут быть откатаны.
- **Неповторяющееся чтение:** Когда транзакция читает одни и те же данные дважды, и они изменяются между чтениями.

🔹 **Пример:**

1. **Транзакция A** начинает читать данные, и в этот момент **Транзакция B** обновляет их.
2. Когда **Транзакция A** снова читает эти данные, она видит уже обновленные значения.

```sql
-- Транзакция A
BEGIN;
SELECT * FROM products WHERE id = 1;  -- Читаем данные (значения = 100)

-- Транзакция B
BEGIN;
UPDATE products SET stock = 50 WHERE id = 1;
COMMIT;

-- Транзакция A снова читает данные
SELECT * FROM products WHERE id = 1;  -- Читаем обновленные значения (значения = 50)
COMMIT;
```

🔹 **Когда использовать?**  
Этот уровень изоляции — хороший выбор, если система должна поддерживать конкурентный доступ и если вам не нужно избежать неповторяющихся чтений.

---

### **1.3 Repeatable Read**

🔹 **Описание:**  
На уровне **Repeatable Read** транзакция видит **те же данные на протяжении всей своей работы** (все данные, которые она прочитала, будут одинаковыми на протяжении всей транзакции). Это предотвращает **неповторяющиеся чтения**, но может возникнуть **фантомное чтение** (когда запись добавляется в таблицу, и она появляется в следующем чтении внутри той же транзакции).

🔹 **Пример:**

1. **Транзакция A** начинает читать записи.
2. **Транзакция B** добавляет или изменяет записи.
3. **Транзакция A** снова выполняет чтение. Она **не видит новых данных**, которые были добавлены после начала транзакции.

```sql
-- Транзакция A
BEGIN;
SELECT * FROM orders WHERE amount > 100;  -- Читаем записи с суммой больше 100

-- Транзакция B
BEGIN;
INSERT INTO orders (amount) VALUES (200);  -- Добавляем новый заказ

-- Транзакция A снова читает те же данные
SELECT * FROM orders WHERE amount > 100;  -- Не увидит добавленный заказ
COMMIT;
```

🔹 **Когда использовать?**  
Этот уровень подходит, если нужно обеспечить, чтобы данные, прочитанные транзакцией, не менялись в ходе выполнения, но при этом изменения данных другими транзакциями могут привести к фантомным чтениям.

---

### **1.4 Serializable**

🔹 **Описание:**  
**Serializable** — самый строгий уровень изоляции, который гарантирует, что транзакции будут работать как если бы они выполнялись **последовательно**, то есть одна за другой. Это полностью исключает **грязные чтения**, **неповторяющиеся чтения** и **фантомные чтения**.

Транзакции, работающие на уровне `Serializable`, будут блокировать строки или даже таблицы для чтения/записи, чтобы предотвратить любые изменения, которые могут повлиять на их работу.

🔹 **Пример:**

1. **Транзакция A** начинает выполнение.
2. **Транзакция B** пытается выполнить операцию, которая может повлиять на данные транзакции A.
3. **PostgreSQL** **заблокирует транзакцию B**, пока не завершится транзакция A, чтобы избежать конфликтов.

```sql
-- Транзакция A
BEGIN;
SELECT * FROM products WHERE stock > 100;  -- Читаем данные

-- Транзакция B
BEGIN;
UPDATE products SET stock = 50 WHERE id = 1;  -- Попытка изменения данных
COMMIT;

-- Транзакция A продолжает выполнение
SELECT * FROM products WHERE stock > 100;  -- Данные остаются прежними до COMMIT транзакции A
COMMIT;
```

🔹 **Когда использовать?**  
Этот уровень следует использовать в критически важных приложениях, где необходимо обеспечить **полную целостность данных** и избежать любых возможных аномалий, даже если это снижает производительность.

---

### **2️⃣ Как задать уровень изоляции в PostgreSQL?**

Для установки уровня изоляции в PostgreSQL можно использовать команду `SET TRANSACTION ISOLATION LEVEL`.

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;  -- Устанавливаем уровень изоляции SERIALIZABLE
BEGIN;  -- Начинаем транзакцию
```

---

### **3️⃣ Проблемы, возникающие без правильной изоляции**

- **Грязные чтения:** Когда одна транзакция читает данные, которые еще не были зафиксированы другой транзакцией.
- **Неповторяющиеся чтения:** Когда данные, прочитанные в транзакции, изменяются другой транзакцией.
- **Фантомные чтения:** Когда новые строки или записи появляются в таблице в ходе транзакции, что может повлиять на результаты запросов.

---

### **4️⃣ Заключение**

Правильный выбор уровня изоляции транзакций критичен для достижения нужной производительности и защиты от аномалий в многозадачной среде.  
- **Read Committed** — хороший компромисс для большинства приложений.
- **Repeatable Read** и **Serializable** дают более строгую изоляцию, но могут уменьшить производительность.

