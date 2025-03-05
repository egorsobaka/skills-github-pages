### **Функциональное программирование (FP) в Node.js**  

**Функциональное программирование (FP)** – это стиль программирования, в котором функции являются **первоклассными объектами**, а код пишется декларативно и без побочных эффектов.  

Node.js позволяет применять FP-подход благодаря **JavaScript**, который поддерживает **функции высшего порядка, иммутабельность, каррирование и композицию**.  

---

## **1. Основные принципы функционального программирования**  

1️⃣ **Чистые функции (Pure Functions)**  
2️⃣ **Функции высшего порядка (Higher-Order Functions)**  
3️⃣ **Иммутабельность данных (Immutability)**  
4️⃣ **Каррирование (Currying)**  
5️⃣ **Композиция функций (Function Composition)**  

---

## **2. Чистые функции (Pure Functions)**
🔹 Функция **не изменяет внешние данные** и всегда **возвращает одинаковый результат** для одних и тех же входных данных.  

❌ **Императивный (нечистый) код:**  
```javascript
let multiplier = 2;
function multiply(x) {
    return x * multiplier; // Использует внешнюю переменную (побочный эффект)
}
```
✅ **Чистая функция:**  
```javascript
function multiply(x, y) {
    return x * y; // Нет побочных эффектов
}
console.log(multiply(2, 3)); // 6
```

**Преимущества чистых функций:**  
✔ Легко тестировать  
✔ Нет неожиданных изменений состояния  
✔ Подходят для параллельных вычислений  

---

## **3. Функции высшего порядка (Higher-Order Functions)**
🔹 Функция, которая **принимает другую функцию в качестве аргумента** или **возвращает функцию**.  

### **Пример: `map`, `filter`, `reduce`**
```javascript
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

const evens = numbers.filter(x => x % 2 === 0);
console.log(evens); // [2, 4]

const sum = numbers.reduce((acc, x) => acc + x, 0);
console.log(sum); // 15
```

**Почему это круто?**  
✔ Код короче и читаемее  
✔ Уменьшает использование циклов  

---

## **4. Иммутабельность данных (Immutability)**
🔹 Изменение состояния **запрещено** – вместо этого создаётся **новая копия объекта**.  

❌ **Изменение объекта (нефункциональный подход):**  
```javascript
const user = { name: "Alice", age: 25 };
user.age = 26; // Изменение исходного объекта
```
✅ **Создание нового объекта (функциональный подход):**  
```javascript
const updatedUser = { ...user, age: 26 }; // Создаём новый объект
```

📌 **Библиотеки для работы с иммутабельными структурами**:  
- **Immutable.js**  
- **Ramda.js**  

---

## **5. Каррирование (Currying)**
🔹 Разделение функции на несколько функций, каждая из которых принимает **один аргумент**.  

📌 **Пример без каррирования:**  
```javascript
function add(a, b) {
    return a + b;
}
console.log(add(2, 3)); // 5
```
📌 **С использованием каррирования:**  
```javascript
const curriedAdd = a => b => a + b;
console.log(curriedAdd(2)(3)); // 5

const add2 = curriedAdd(2);
console.log(add2(3)); // 5
```

**Когда это полезно?**  
✔ Позволяет **создавать частично применённые функции**  
✔ Удобно в **функциональных цепочках**  

---

## **6. Композиция функций (Function Composition)**
🔹 Объединение нескольких функций **в одну** для поэтапной обработки данных.  

📌 **Пример без композиции:**  
```javascript
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);
const exclaim = str => str + "!";
const repeat = str => str + " " + str;

console.log(repeat(exclaim(capitalize("hello")))); // Hello! Hello!
```
📌 **С использованием композиции:**  
```javascript
const compose = (...funcs) => x => funcs.reduceRight((acc, fn) => fn(acc), x);

const shout = compose(repeat, exclaim, capitalize);
console.log(shout("hello")); // Hello! Hello!
```
✅ **Код чище и легче для понимания**.  

📌 **Библиотека `Ramda` делает это удобнее**  
```javascript
const R = require('ramda');
const shout = R.pipe(capitalize, exclaim, repeat);
console.log(shout("hello")); // Hello! Hello!
```

---

## **7. Функциональное программирование в Node.js**
Node.js отлично поддерживает **FP**, так как:
- Поддерживает **асинхронные функции (Promise, Async/Await)**
- Имеет **стримы (Streams)**, которые работают как **чистые функции**  
- Использует **Middleware** в Express, который работает как композиция функций  

📌 **Пример FP в Express.js**  
```javascript
const express = require('express');
const app = express();

const logger = fn => (req, res, next) => {
    console.log(`Request to ${req.url}`);
    fn(req, res, next);
};

app.get('/', logger((req, res) => res.send('Hello, Functional World!')));

app.listen(3000, () => console.log('Server running'));
```

---

## **Вывод**  
Функциональное программирование делает код **читаемым, предсказуемым и удобным для тестирования**.  

| FP-концепция | Что даёт? |
|-------------|-----------|
| Чистые функции | Код без побочных эффектов |
| Функции высшего порядка | Гибкость и переиспользуемость |
| Иммутабельность | Нет неожиданных изменений данных |
| Каррирование | Удобная частичная передача аргументов |
| Композиция функций | Чистый, декларативный код |

📌 **Хотите углубиться?**  
- `Ramda.js` – мощная FP-библиотека  
- `Lodash/fp` – функциональный вариант Lodash  
- `Functional-Light JavaScript` – отличная книга по FP в JS  

🚀 Вам интересно применить FP в **Nest.js** или **асинхронных задачах**? 😊
