### **Event-Driven архитектура (EDA)**
Event-Driven Architecture (EDA) — это архитектурный подход, в котором компоненты системы взаимодействуют между собой с помощью событий. Вместо того чтобы работать в строгом синхронном порядке, система реагирует на события, когда они происходят.

В **EDA** есть три ключевых элемента:
1. **Производитель событий (Event Producer)** – создает событие.
2. **Брокер событий (Event Broker)** – может передавать события подписчикам.
3. **Подписчик на события (Event Consumer)** – получает событие и выполняет соответствующие действия.

### **EDA в Node.js**
Node.js изначально основан на **событийно-ориентированной модели**. Основной механизм обработки событий в Node.js — это `EventEmitter` из встроенного модуля `events`.

#### **Пример работы с EventEmitter**
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// Подписываемся на событие
myEmitter.on('data_received', (message) => {
    console.log(`Received: ${message}`);
});

// Генерируем событие
myEmitter.emit('data_received', 'Hello, Event-Driven World!');
```

---

### **Применение Event-Driven архитектуры в Node.js**
#### **1. Обработка I/O-операций**
Node.js обрабатывает асинхронные задачи через события. Например, чтение файла:
```javascript
const fs = require('fs');

fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data); // Событие вызовет callback, когда данные будут доступны.
});
```

#### **2. WebSockets и реальное время**
EDA активно используется в WebSocket-приложениях:
```javascript
const WebSocket = require('ws');

const server = new WebSocket.Server({ port: 8080 });

server.on('connection', (ws) => {
    ws.on('message', (message) => {
        console.log(`Received: ${message}`);
        ws.send(`Echo: ${message}`);
    });
});
```

#### **3. Взаимодействие микросервисов через брокеры событий**
Для связи микросервисов можно использовать **Kafka, RabbitMQ, Redis Pub/Sub**:
```javascript
const redis = require('redis');

const pub = redis.createClient();
const sub = redis.createClient();

sub.subscribe('channel');

sub.on('message', (channel, message) => {
    console.log(`Received: ${message} from ${channel}`);
});

pub.publish('channel', 'Hello from Redis!');
```

#### **4. Использование событий в Nest.js**
В Nest.js можно использовать **EventEmitter2**:
```javascript
import { Injectable } from '@nestjs/common';
import { EventEmitter2 } from '@nestjs/event-emitter';

@Injectable()
export class AppService {
    constructor(private eventEmitter: EventEmitter2) {}

    triggerEvent() {
        this.eventEmitter.emit('user.created', { name: 'John Doe' });
    }
}
```

---

### **Преимущества Event-Driven архитектуры**
✅ **Высокая производительность** – нет блокировки потоков.  
✅ **Гибкость** – подписчики могут динамически обрабатывать события.  
✅ **Масштабируемость** – легко добавлять новые компоненты.  

### **Недостатки**
❌ **Сложность отладки** – события могут приходить асинхронно.  
❌ **Неопределённость порядка выполнения** – сложно контролировать порядок обработки.  

---

### **Вывод**
Event-Driven Architecture идеально подходит для **асинхронных приложений**, особенно в **Node.js**. Ее используют в WebSockets, микросервисах, обработке событий и потоковых данных.

### **Масштабирование и синхронизация в Event-Driven архитектуре (EDA)**  

Когда система растёт, появляются проблемы:  
- Как обработать **большое количество событий**?  
- Как **синхронизировать** несколько экземпляров приложения?  
- Как **обеспечить надежность** и **упорядоченность** событий?  

Разберем **ключевые стратегии** решения этих проблем.  

---

## **1. Горизонтальное масштабирование**
Node.js работает в **однопоточном** режиме, поэтому масштабирование требует **разделения нагрузки**.

### **1.1. Cluster (fork процессов)**
Модуль `cluster` позволяет запустить несколько процессов, используя все ядра CPU.

```javascript
const cluster = require('cluster');
const http = require('http');
const os = require('os');

if (cluster.isMaster) {
    const numCPUs = os.cpus().length;
    for (let i = 0; i < numCPUs; i++) {
        cluster.fork(); // Создаём процесс для каждого ядра
    }
} else {
    http.createServer((req, res) => {
        res.writeHead(200);
        res.end(`Handled by process ${process.pid}`);
    }).listen(3000);
}
```
⚠️ **Проблема**: Worker-процессы не "видят" друг друга, нужна **синхронизация** через общий механизм (Redis, Kafka).

---

### **1.2. Балансировка нагрузки**
В реальной инфраструктуре можно использовать **NGINX, HAProxy** или **Kubernetes** для распределения трафика между узлами.  

Пример **NGINX WebSocket Proxy**:
```nginx
upstream websocket_backend {
    ip_hash;
    server 192.168.1.1:3000;
    server 192.168.1.2:3000;
}

server {
    listen 80;
    location / {
        proxy_pass http://websocket_backend;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
    }
}
```

---

## **2. Синхронизация событий между экземплярами**
Когда приложение масштабируется, события должны передаваться между процессами. Для этого используют **брокеры сообщений**.

### **2.1. Redis Pub/Sub (Быстрая синхронизация)**
Используется для **реального времени**, когда события должны мгновенно передаваться между процессами.

```javascript
const redis = require('redis');

const pub = redis.createClient();
const sub = redis.createClient();

sub.subscribe('events');

sub.on('message', (channel, message) => {
    console.log(`Received: ${message}`);
});

pub.publish('events', 'Hello, world!');
```
✅ **Плюсы**: низкая задержка, хорошая поддержка в Node.js.  
❌ **Минусы**: не гарантирует доставку сообщений (если процесс отключился).  

---

### **2.2. Kafka / RabbitMQ (Гарантированная доставка)**
Если нужно **гарантированное** и **отложенное** выполнение событий, лучше использовать **Kafka** или **RabbitMQ**.

#### Kafka + Node.js
```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({ clientId: 'my-app', brokers: ['localhost:9092'] });
const producer = kafka.producer();
const consumer = kafka.consumer({ groupId: 'test-group' });

async function run() {
    await producer.connect();
    await consumer.connect();
    
    await producer.send({ topic: 'events', messages: [{ value: 'Hello Kafka' }] });

    await consumer.subscribe({ topic: 'events' });

    await consumer.run({
        eachMessage: async ({ message }) => {
            console.log(`Received: ${message.value.toString()}`);
        },
    });
}

run();
```
✅ **Плюсы**: масштабируемость, отказоустойчивость.  
❌ **Минусы**: сложность настройки.  

---

## **3. Гарантированная доставка и порядок событий**
Если события критичны, нужна система **очередей**:

- **RabbitMQ**: гарантирует **"at least once"** доставку.
- **Kafka**: поддерживает **упорядоченные очереди**.
- **SQS (AWS)**: облачное хранилище сообщений.

Пример RabbitMQ с Node.js:
```javascript
const amqp = require('amqplib');

async function sendMessage() {
    const conn = await amqp.connect('amqp://localhost');
    const channel = await conn.createChannel();
    
    await channel.assertQueue('tasks', { durable: true });
    channel.sendToQueue('tasks', Buffer.from('New Task'), { persistent: true });

    console.log('Message sent');
    setTimeout(() => conn.close(), 500);
}

sendMessage();
```

---

## **4. Обработка "шторма событий" (Event Storming)**
Если клиент или сервис начинает отправлять **слишком много событий**, это может перегрузить систему.

### **4.1. Rate Limiting (Ограничение запросов)**
Ограничение частоты событий через `express-rate-limit`:
```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
    windowMs: 60 * 1000, // 1 минута
    max: 100, // Максимум 100 запросов
});

app.use('/api', limiter);
```

---

## **5. Event Sourcing и CQRS**
Event Sourcing – стратегия хранения **всех событий**, а не только конечного состояния.

Пример:
- В **базе данных** хранятся все изменения.
- Для "отката назад" можно **воспроизвести** события.

📌 Используется в **финансовых приложениях, CRM, системах учёта.**

---

## **Вывод**
✅ **Если мало соединений** → `ws`, `socket.io` + `Redis Pub/Sub`  
✅ **Если много событий** → Kafka / RabbitMQ  
✅ **Если критична надежность** → **Очереди сообщений**  
✅ **Если нужна полная история** → Event Sourcing  

Вам интересно углубиться в **Kafka, RabbitMQ** или **построение масштабируемой архитектуры в Nest.js**? 🚀
