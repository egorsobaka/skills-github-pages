## **Сетевые протоколы и работа с HTTP в Node.js**

Node.js широко используется для работы с HTTP, WebSockets и микросервисами. Разберём ключевые протоколы и их реализацию.

---

## **1. HTTP/1.1, HTTP/2 и WebSockets в Node.js**
### **📌 HTTP/1.1**
**Как работает:**  
- **Текстовый протокол**, передающий запросы и ответы строками.  
- Каждый HTTP-запрос создаёт **новое соединение**, но в HTTP/1.1 появилась **Keep-Alive** – возможность **многократного использования одного соединения**.  

**Реализация HTTP/1.1 в Node.js:**
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, HTTP/1.1!');
});

server.listen(3000, () => console.log('Server running on port 3000'));
```
✅ Поддержка HTTP/1.1 встроена в `http`-модуль.  

---

### **📌 HTTP/2**
**Что нового по сравнению с HTTP/1.1?**
- **Бинарный протокол** (не текстовый, как HTTP/1.1)  
- **Мультиплексирование** – несколько запросов по **одному TCP-соединению**  
- **Server Push** – сервер может отправлять данные клиенту **без запроса**  
- **Сжатие заголовков (HPACK)**  

**Реализация HTTP/2 в Node.js:**
```javascript
const http2 = require('http2');
const fs = require('fs');

const server = http2.createSecureServer({
    key: fs.readFileSync('server-key.pem'),
    cert: fs.readFileSync('server-cert.pem')
});

server.on('stream', (stream, headers) => {
    stream.respond({ ':status': 200, 'content-type': 'text/html' });
    stream.end('<h1>Hello, HTTP/2!</h1>');
});

server.listen(3000, () => console.log('HTTP/2 server running on port 3000'));
```
✅ HTTP/2 **требует HTTPS**, поэтому нужен **SSL-сертификат**.  

---

### **📌 WebSockets**
**Отличие от HTTP:**  
- WebSocket – **постоянное двустороннее соединение** между клиентом и сервером.  
- Подходит для **чата, live-обновлений, финансовых данных**.  
- Работает поверх TCP, но не требует открытия нового соединения для каждого сообщения.  

**Реализация WebSocket в Node.js с `ws`:**
```javascript
const WebSocket = require('ws');

const server = new WebSocket.Server({ port: 8080 });

server.on('connection', ws => {
    console.log('Client connected');
    ws.send('Welcome to WebSocket server!');

    ws.on('message', message => {
        console.log(`Received: ${message}`);
        ws.send(`Echo: ${message}`);
    });

    ws.on('close', () => console.log('Client disconnected'));
});
```
✅ **WebSockets не зависят от HTTP/1.1 или HTTP/2** – они работают через **ws:// или wss://**.

---

## **2. Реализация и особенности HTTP/2 в Node.js**
### **Основные моменты HTTP/2 в Node.js:**
1. **Поддержка в `http2` модуле**  
2. **Работает через TLS (HTTPS)**  
3. **Server Push можно включить вручную**  
4. **Потоки вместо отдельных соединений**  

### **📌 Server Push в HTTP/2**
```javascript
server.on('stream', (stream, headers) => {
    if (headers[':path'] === '/') {
        stream.pushStream({ ':path': '/style.css' }, (err, pushStream) => {
            if (err) throw err;
            pushStream.respond({ ':status': 200, 'content-type': 'text/css' });
            pushStream.end('body { background: lightgray; }');
        });

        stream.respond({ ':status': 200, 'content-type': 'text/html' });
        stream.end('<link rel="stylesheet" href="/style.css"><h1>Server Push</h1>');
    }
});
```
✅ Сервер сам **"подталкивает"** файлы клиенту **до того, как он запросит их**.  

---

## **3. API Gateway для микросервисов**
В микросервисной архитектуре **API Gateway** помогает управлять маршрутизацией, балансировкой нагрузки и безопасностью.  

🔹 **Популярные решения:**
- **Express + http-proxy-middleware**
- **Kong API Gateway**
- **Traefik**
- **Nginx + Node.js**
- **Nest.js + GraphQL Gateway**  

**📌 Пример API Gateway на `http-proxy-middleware` (Node.js + Express)**  
```javascript
const express = require('express');
const { createProxyMiddleware } = require('http-proxy-middleware');

const app = express();

app.use('/users', createProxyMiddleware({ target: 'http://localhost:4001', changeOrigin: true }));
app.use('/orders', createProxyMiddleware({ target: 'http://localhost:4002', changeOrigin: true }));

app.listen(3000, () => console.log('API Gateway running on port 3000'));
```
✅ **Маршрутизирует запросы между микросервисами**  

---

## **4. Управление соединениями и отказоустойчивость**
### **🔹 Проблемы в распределённых системах:**
1. **Лик без соединений (connection leak)** – клиенты не закрывают соединения  
2. **Перегрузка сервера** – слишком много активных соединений  
3. **Сбой узла** – узел падает, но клиенты продолжают слать запросы  

### **Решения:**
#### **📌 1. Управление временем жизни соединений (Keep-Alive & Timeouts)**  
```javascript
const agent = new http.Agent({ keepAlive: true, timeout: 5000 });

const req = http.request({ hostname: 'example.com', agent }, res => {
    res.on('data', chunk => console.log(chunk.toString()));
});
req.end();
```
✅ **`keepAlive: true`** снижает нагрузку на сервер.  
✅ **`timeout: 5000`** закрывает "зависшие" соединения.  

---

#### **📌 2. Circuit Breaker (Отключение неработающих узлов)**
Можно использовать **`opossum`** – библиотеку Circuit Breaker для Node.js.  

```javascript
const CircuitBreaker = require('opossum');

const riskyCall = () => fetch('http://unreliable-service.com');

const breaker = new CircuitBreaker(riskyCall, { timeout: 3000, errorThresholdPercentage: 50 });

breaker.fallback(() => 'Service unavailable, try again later.');

breaker.fire()
    .then(console.log)
    .catch(console.error);
```
✅ **Если сервис падает – запросы временно блокируются.**  

---

#### **📌 3. Балансировка нагрузки (Load Balancing)**
Можно использовать **Nginx, HAProxy или Kubernetes Ingress**.

Пример конфигурации Nginx:
```nginx
upstream app_servers {
    server 192.168.1.2;
    server 192.168.1.3;
}

server {
    listen 80;
    location / {
        proxy_pass http://app_servers;
    }
}
```
✅ **Распределяет трафик между разными серверами.**  

---

### **Вывод**
| Протокол | Подходит для | Реализация в Node.js |
|----------|-------------|----------------------|
| HTTP/1.1 | Классические API | `http` |
| HTTP/2 | Высоконагруженные API | `http2` |
| WebSockets | Реальное время, чаты | `ws` |
| API Gateway | Микросервисы | `http-proxy-middleware`, Kong |
| Circuit Breaker | Защита от падений | `opossum` |
| Load Balancing | Масштабируемость | Nginx, Kubernetes |

🚀 **Какая тема интересует больше: API Gateway в Nest.js, WebSockets в микросервисах или балансировка нагрузки?** 😊
