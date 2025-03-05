### **Race Condition в Node.js: Как избежать?**  

**Race Condition (гонка данных)** – это ситуация, когда несколько процессов или потоков **одновременно** обращаются к общему ресурсу и **непредсказуемо изменяют его состояние**.  

В **Node.js** это может происходить даже несмотря на **однопоточную модель**, например:  
- При работе с **асинхронными операциями** (например, с БД, файлами).  
- При **конкурентных HTTP-запросах** к серверу.  
- В **кластерных** или **распределённых системах**.  

---

## **1. Использование транзакций в базе данных**  

Если несколько запросов одновременно модифицируют одну и ту же запись в БД, могут возникнуть конфликты.  

🔹 **Пример проблемы (без транзакций)**  
```javascript
const db = require('./db'); // Подключение к БД

async function withdraw(userId, amount) {
    const user = await db.getUser(userId); // Запрос текущего баланса
    if (user.balance >= amount) {
        await db.updateBalance(userId, user.balance - amount); // Обновление баланса
    } else {
        throw new Error('Insufficient funds');
    }
}
```
⚠ **Проблема:** если два запроса на `withdraw()` поступят одновременно, оба могут прочитать **старый баланс** и потратить **одинаковые деньги**.  

✅ **Решение: транзакции**  
Транзакции позволяют **гарантировать** выполнение всех операций последовательно.  

**Пример с PostgreSQL (через `node-postgres`)**
```javascript
const { Pool } = require('pg');
const pool = new Pool();

async function withdraw(userId, amount) {
    const client = await pool.connect();
    try {
        await client.query('BEGIN'); // Начало транзакции
        const { rows } = await client.query('SELECT balance FROM users WHERE id = $1 FOR UPDATE', [userId]); // Блокировка строки
        if (rows[0].balance >= amount) {
            await client.query('UPDATE users SET balance = balance - $1 WHERE id = $2', [amount, userId]);
            await client.query('COMMIT'); // Завершаем транзакцию
        } else {
            await client.query('ROLLBACK'); // Откат изменений при ошибке
            throw new Error('Insufficient funds');
        }
    } catch (err) {
        await client.query('ROLLBACK'); // Откат транзакции при ошибке
        throw err;
    } finally {
        client.release();
    }
}
```
✅ **`FOR UPDATE` блокирует строку** и предотвращает гонку состояний.  

---

## **2. Использование Redis Lock (Блокировки в распределённых системах)**  

Если несколько экземпляров сервиса обрабатывают одну и ту же сущность, **транзакции в БД уже не помогут**. Нужно **распределённое блокирование**.  

### **Redis-based Lock (Redlock)**
**Redlock** – это механизм блокировки, использующий **Redis**, который позволяет **гарантировать атомарность** операций.  

📌 **Пример с `redlock` в Node.js**  
```javascript
const Redis = require("ioredis");
const Redlock = require("redlock");

const redis = new Redis();
const redlock = new Redlock([redis], { retryCount: 3 });

async function processOrder(orderId) {
    const lock = await redlock.lock(`locks:order:${orderId}`, 1000); // Блокируем ресурс на 1 секунду
    try {
        // Критическая секция (без гонки данных)
        console.log(`Processing order ${orderId}`);
        await new Promise(resolve => setTimeout(resolve, 500)); // Симуляция работы
    } finally {
        await lock.unlock(); // Разблокируем
    }
}
```
✅ **Гарантирует, что один процесс работает с ресурсом в данный момент.**  
❌ **Важно!** Redlock требует точной настройки таймаутов, чтобы избежать deadlock'ов.  

---

## **3. Очереди сообщений (Message Queue)**
Вместо того чтобы несколько сервисов пытались одновременно изменить данные, можно **поставить задачи в очередь**.  

### **Пример с RabbitMQ (BullMQ)**
```javascript
const { Queue } = require('bullmq');
const myQueue = new Queue('orders');

async function processOrder(orderId) {
    await myQueue.add('process_order', { orderId });
}
```
➡ **Обработчик будет брать задачи по одной**, избегая гонки данных.

---

## **4. Версионирование записей (Optimistic Locking)**
Если два клиента читают и обновляют одну запись в базе **параллельно**, поможет **версионирование записей**.  

📌 **Пример с MongoDB и Mongoose**
```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
    balance: Number,
}, { versionKey: true }); // Включаем __v для контроля версий

const User = mongoose.model('User', userSchema);

async function updateBalance(userId, amount) {
    const user = await User.findOne({ _id: userId });
    user.balance += amount;
    await user.save(); // Ошибка, если другая транзакция уже изменила версию документа
}
```
✅ **MongoDB автоматически увеличивает `__v`** и предотвращает конфликтные обновления.  

---

## **5. Atomic операции (CAS - Compare and Swap)**  
Когда операция зависит от предыдущего состояния, можно использовать **атомарные операции**.  

📌 **Пример с Redis (Incr/Decr вместо GET/SET)**
```javascript
const redis = require('ioredis');
const client = new redis();

async function withdraw(userId, amount) {
    const balance = await client.decrby(`user:${userId}:balance`, amount);
    if (balance < 0) {
        await client.incrby(`user:${userId}:balance`, amount); // Откат
        throw new Error("Insufficient funds");
    }
}
```
✅ **Гарантирует, что баланс обновляется атомарно** без условий гонки.  

---

## **Вывод**  

| Проблема | Решение |
|----------|---------|
| Гонка при работе с БД | **Транзакции (`BEGIN ... COMMIT`)** |
| Гонка в распределённых системах | **Redis Lock (Redlock)** |
| Очередь задач (конкурентные операции) | **Message Queue (RabbitMQ, BullMQ)** |
| Обновления одной записи разными клиентами | **Optimistic Locking (версии записей)** |
| Многопоточная работа с памятью (кеш, счетчики) | **Atomic операции (Redis `INCR/DECR`)** |

🚀 Если приложение простое — достаточно **транзакций и Redis**.  
🏢 Если система распределённая — **Redlock, Kafka, RabbitMQ**.  

Что вас больше интересует: **реализация в Nest.js**, **масштабирование** или **какой-то конкретный сценарий**? 😊
