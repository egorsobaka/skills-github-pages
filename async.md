Вот подробные вопросы и ответы по теме **"Асинхронность и Event Loop"**, которые могут встретиться на собеседовании **Senior Node.js Developer**.  

---

### **1. Как работает Event Loop в Node.js?**  
**Ответ:**  
Event Loop — это механизм, который позволяет Node.js обрабатывать асинхронные операции, выполняя код неблокирующим образом. Он состоит из нескольких фаз:  
1. **Timers** – обработка `setTimeout()` и `setInterval()`.  
2. **I/O callbacks** – обработка асинхронных операций (например, файловый ввод-вывод).  
3. **Idle, prepare** – используется для внутренних процессов Node.js.  
4. **Poll** – получение новых I/O событий и выполнение коллбеков.  
5. **Check** – обработка `setImmediate()`.  
6. **Close callbacks** – обработка закрывающих событий (например, `socket.on('close')`).  

**Пример кода с Event Loop:**  
```javascript
setTimeout(() => console.log('Timeout'), 0);
setImmediate(() => console.log('Immediate'));
process.nextTick(() => console.log('NextTick'));
```
**Вывод:**  
```
NextTick
Immediate (или Timeout, зависит от нагрузки)
Timeout (или Immediate)
```
`process.nextTick()` всегда выполняется **до следующего тика Event Loop**, поэтому он выполнится первым.  

---

### **2. В чем разница между process.nextTick(), setImmediate() и setTimeout()?**  
**Ответ:**  
- `process.nextTick()` выполняется **до начала следующего тика Event Loop**, в фазе текущего выполнения.  
- `setImmediate()` выполняется **в фазе Check** (после Poll).  
- `setTimeout(fn, 0)` выполняется **в фазе Timers**, но после минимальной задержки.  

**Пример:**  
```javascript
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
process.nextTick(() => console.log('process.nextTick'));
```
**Вывод:**  
```
process.nextTick
setImmediate (или setTimeout, зависит от нагрузки)
setTimeout (или setImmediate)
```

---

### **3. Как работает асинхронность в Node.js (callbacks, promises, async/await)?**  
**Ответ:**  
В Node.js есть три основных способа работы с асинхронностью:  
1. **Callbacks** – передача функции обратного вызова.  
2. **Promises** – удобный способ обработки асинхронных операций с `.then()` и `.catch()`.  
3. **Async/Await** – синтаксический сахар над Promises.  

**Пример работы с callback:**  
```javascript
function fetchData(callback) {
  setTimeout(() => {
    callback(null, "Data loaded");
  }, 1000);
}

fetchData((err, data) => {
  if (err) console.error(err);
  else console.log(data);
});
```

**Пример с Promises:**  
```javascript
function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("Data loaded"), 1000);
  });
}

fetchData().then(console.log);
```

**Пример с Async/Await:**  
```javascript
async function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => resolve("Data loaded"), 1000);
  });
}

async function main() {
  const data = await fetchData();
  console.log(data);
}

main();
```
Async/Await делает код более читабельным и понятным.  

---

### **4. Что такое "callback hell" и как его избежать?**  
**Ответ:**  
"Callback Hell" – это ситуация, когда код становится трудно читаемым из-за вложенных коллбеков.  

**Пример callback hell:**  
```javascript
getUser(userId, (user) => {
  getOrders(user, (orders) => {
    getOrderDetails(orders[0], (details) => {
      console.log(details);
    });
  });
});
```
**Способы избежать callback hell:**  
1. Использование **Promises**  
2. Использование **Async/Await**  
3. Разбивка кода на функции  

**Использование Promises:**  
```javascript
getUser(userId)
  .then(getOrders)
  .then((orders) => getOrderDetails(orders[0]))
  .then(console.log)
  .catch(console.error);
```

**Использование Async/Await:**  
```javascript
async function main() {
  try {
    const user = await getUser(userId);
    const orders = await getOrders(user);
    const details = await getOrderDetails(orders[0]);
    console.log(details);
  } catch (error) {
    console.error(error);
  }
}

main();
```

---

### **5. Чем отличаются microtask и macrotask в Node.js?**  
**Ответ:**  
- **Microtasks** (`process.nextTick()` и `Promise.then()`) выполняются **до следующего тика Event Loop**.  
- **Macrotasks** (`setTimeout()`, `setImmediate()`, I/O) выполняются в соответствующих фазах Event Loop.  

**Пример:**  
```javascript
setTimeout(() => console.log('setTimeout'), 0);
setImmediate(() => console.log('setImmediate'));
Promise.resolve().then(() => console.log('Promise'));
process.nextTick(() => console.log('nextTick'));
```
**Вывод:**  
```
nextTick
Promise
setTimeout (или setImmediate)
setImmediate (или setTimeout)
```
Сначала выполняются **microtasks**, затем **macrotasks**.

---

### **6. Как работает setTimeout(fn, 0)?**  
**Ответ:**  
`setTimeout(fn, 0)` не выполняется **сразу**, а откладывается до следующего тика Event Loop, так как он попадает в фазу **Timers**.

**Пример:**  
```javascript
console.log('Start');
setTimeout(() => console.log('Timeout'), 0);
console.log('End');
```
**Вывод:**  
```
Start
End
Timeout
```
`setTimeout()` выполняется **только после завершения текущего кода**.

---

### **7. Как использовать Worker Threads в Node.js?**  
**Ответ:**  
Worker Threads позволяют выполнять тяжелые вычисления в отдельных потоках, не блокируя Event Loop.  

**Пример:**  
```javascript
const { Worker } = require('worker_threads');

const worker = new Worker(`
  const { parentPort } = require('worker_threads');
  let count = 0;
  for (let i = 0; i < 1e9; i++) count++;
  parentPort.postMessage(count);
`, { eval: true });

worker.on('message', console.log);
```
Такой подход полезен для CPU-интенсивных задач.

---

### **8. Что такое backpressure и как с ним бороться?**  
**Ответ:**  
Backpressure – это ситуация, когда поток данных (Stream) генерирует данные быстрее, чем их может обработать потребитель.  

**Решения:**  
1. Использовать **pause/resume** в Streams.  
2. Применять **async iteration**.  
3. Использовать **pipeline()**.  

**Пример использования pipeline():**  
```javascript
const { pipeline } = require('stream');
pipeline(
  fs.createReadStream('largeFile.txt'),
  fs.createWriteStream('output.txt'),
  (err) => { if (err) console.error('Pipeline failed', err); }
);
```
`pipeline()` автоматически управляет потоком, предотвращая перегрузку.

---
