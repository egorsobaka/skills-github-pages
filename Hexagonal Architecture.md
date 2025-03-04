## **Hexagonal Architecture (Ports & Adapters) в Node.js**  

**Hexagonal Architecture**, также известная как **Ports & Adapters**, — это архитектурный стиль, который делает приложение **гибким, тестируемым и легко расширяемым**. Основная идея заключается в **изоляции бизнес-логики от деталей реализации** (база данных, API, UI и т. д.).  

---

### 🔹 **Основные принципы Hexagonal Architecture**  

1. **Приложение не зависит от внешних сервисов**  
   - Бизнес-логика не привязана к конкретной базе данных, API или UI.  
   - Можно легко заменить базу данных или внешний API без изменения основной логики.  

2. **Разделение на "Порты" и "Адаптеры"**  
   - **Порты (Ports)** – это **интерфейсы**, через которые взаимодействуют внешние системы.  
   - **Адаптеры (Adapters)** – это **конкретные реализации**, работающие с базой данных, API и т. д.  

3. **Чистая архитектура без зависимости от инфраструктуры**  
   - Вся бизнес-логика изолирована внутри приложения.  
   - Данные могут приходить откуда угодно (REST, GraphQL, CLI, WebSockets и т. д.), но логика остается неизменной.  

---

## **🔹 Структура проекта в Hexagonal Architecture**  
Проект разделен на **ядро (Core)** и **инфраструктуру (Adapters)**.  

```
/src
  ├── application/  # Бизнес-логика (Core)
  │   ├── ports/  # Интерфейсы (Ports)
  │   │   ├── UserRepository.js
  │   │   ├── NotificationService.js
  │   ├── services/  # Основная логика
  │       ├── UserService.js
  │
  ├── infrastructure/  # Внешние адаптеры
  │   ├── database/  # Работа с БД
  │   │   ├── MongoUserRepository.js
  │   │   ├── PostgresUserRepository.js
  │   ├── api/  # Входные адаптеры (контроллеры)
  │   │   ├── ExpressUserController.js
  │   ├── notifications/  # Отправка сообщений
  │       ├── EmailNotificationService.js
  │       ├── SmsNotificationService.js
  │
  ├── config/  # Конфигурация приложения
  ├── index.js  # Точка входа
```

---

## **🔹 Реализация Hexagonal Architecture в Node.js**

### **1️⃣ Определяем "Порт" (Интерфейс)**
Порт определяет, **какие методы должен реализовать адаптер**.  

📌 **Файл: `src/application/ports/UserRepository.js`**  
```javascript
class UserRepository {
  async findById(id) {
    throw new Error('Метод findById() должен быть реализован');
  }

  async save(user) {
    throw new Error('Метод save() должен быть реализован');
  }
}

module.exports = UserRepository;
```
💡 **Этот интерфейс не зависит от конкретной БД (MongoDB, PostgreSQL и т. д.).**

---

### **2️⃣ Реализуем адаптеры для базы данных**
Адаптеры реализуют интерфейс `UserRepository`, но с разными базами данных.

📌 **Файл: `src/infrastructure/database/MongoUserRepository.js`**  
```javascript
const UserRepository = require('../../application/ports/UserRepository');
const mongoose = require('mongoose');
const UserModel = require('./UserModel'); // Модель пользователя в MongoDB

class MongoUserRepository extends UserRepository {
  async findById(id) {
    return UserModel.findById(id);
  }

  async save(user) {
    return UserModel.create(user);
  }
}

module.exports = MongoUserRepository;
```

📌 **Файл: `src/infrastructure/database/PostgresUserRepository.js`**  
```javascript
const UserRepository = require('../../application/ports/UserRepository');
const db = require('./db'); // Подключение к PostgreSQL

class PostgresUserRepository extends UserRepository {
  async findById(id) {
    return db.query('SELECT * FROM users WHERE id = $1', [id]);
  }

  async save(user) {
    return db.query('INSERT INTO users (name, email) VALUES ($1, $2)', [user.name, user.email]);
  }
}

module.exports = PostgresUserRepository;
```

💡 Теперь можно **легко переключаться между MongoDB и PostgreSQL** без изменения бизнес-логики.

---

### **3️⃣ Определяем бизнес-логику (Core)**
Вся логика работы с пользователями находится в **UserService**.

📌 **Файл: `src/application/services/UserService.js`**  
```javascript
class UserService {
  constructor(userRepository) {
    this.userRepository = userRepository;
  }

  async getUser(id) {
    return this.userRepository.findById(id);
  }

  async createUser(name, email) {
    const user = { name, email };
    return this.userRepository.save(user);
  }
}

module.exports = UserService;
```
💡 `UserService` **не знает**, какая база данных используется. Она работает через порт `UserRepository`.

---

### **4️⃣ Подключаем адаптер API (Express)**
Контроллер принимает запросы и передает их в `UserService`.

📌 **Файл: `src/infrastructure/api/ExpressUserController.js`**  
```javascript
const express = require('express');
const router = express.Router();

module.exports = (userService) => {
  router.get('/:id', async (req, res) => {
    const user = await userService.getUser(req.params.id);
    res.json(user);
  });

  router.post('/', async (req, res) => {
    const user = await userService.createUser(req.body.name, req.body.email);
    res.status(201).json(user);
  });

  return router;
};
```

---

### **5️⃣ Подключаем адаптеры в точке входа**
Теперь можно легко менять адаптеры без изменения бизнес-логики.

📌 **Файл: `src/index.js`**  
```javascript
const express = require('express');
const MongoUserRepository = require('./infrastructure/database/MongoUserRepository');
const UserService = require('./application/services/UserService');
const createUserController = require('./infrastructure/api/ExpressUserController');

const app = express();
app.use(express.json());

// Подключаем MongoDB адаптер
const userRepository = new MongoUserRepository();
const userService = new UserService(userRepository);

// Подключаем контроллер
app.use('/users', createUserController(userService));

app.listen(3000, () => console.log('Server running on port 3000'));
```

💡 Если захочется **переключиться на PostgreSQL**, нужно просто заменить:  
```javascript
const PostgresUserRepository = require('./infrastructure/database/PostgresUserRepository');
const userRepository = new PostgresUserRepository();
```
**И ВСЁ! Бизнес-логика останется неизменной!** 🎉

---

## **🔹 Преимущества Hexagonal Architecture**
✅ **Гибкость** – легко менять базы данных и API.  
✅ **Тестируемость** – можно подменять адаптеры моками при тестировании.  
✅ **Чистая архитектура** – бизнес-логика изолирована.  
✅ **Простота в масштабировании** – можно легко добавлять новые адаптеры (например, GraphQL).  

---

## **🔹 Итог**
**Hexagonal Architecture (Ports & Adapters)** помогает строить **гибкие и поддерживаемые приложения**. Она идеально подходит для **крупных проектов**, где важно **разделение ответственности**.  

🔹 **Если нужно больше деталей или примеров — спрашивай!** 🚀
