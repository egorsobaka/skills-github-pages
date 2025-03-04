## **🔹 Тестирование в Node.js: Виды и Практики**

Тестирование — важный этап разработки, который помогает обеспечить стабильность, работоспособность и безопасность приложений. В Node.js существует несколько типов тестирования, которые применяются в зависимости от уровня, на котором необходимо проверять работу приложения. Рассмотрим основные виды тестирования в Node.js, а также инструменты и практики для их реализации.

### **1️⃣ Юнит-тестирование (Unit Testing)**

**Юнит-тестирование** — это проверка отдельных компонентов системы (например, функций, методов, классов) на корректность их работы в изоляции. В случае с Node.js юнит-тесты обычно проверяют функции, которые выполняют отдельные бизнес-логики или алгоритмы.

#### **Цель**:
- Проверить работу отдельных единиц кода.
- Убедиться, что каждая функция или метод работает правильно при различных входных данных.

#### **Пример**:
Для функции, которая суммирует два числа:

```javascript
function sum(a, b) {
  return a + b;
}

module.exports = sum;
```

Тест для этой функции:

```javascript
const sum = require('./sum');
const assert = require('assert');

describe('Sum function', function() {
  it('should return 3 when adding 1 and 2', function() {
    assert.strictEqual(sum(1, 2), 3);
  });

  it('should return 0 when adding 0 and 0', function() {
    assert.strictEqual(sum(0, 0), 0);
  });
});
```

#### **Инструменты**:
- **Mocha**: Популярный тестовый фреймворк, который предоставляет функции для описания тестов и организации их выполнения.
- **Chai**: Библиотека утверждений, которая работает в паре с Mocha и позволяет удобно проверять результаты.
- **Jest**: Один из самых популярных фреймворков для тестирования в Node.js, включает в себя инструменты для юнит-тестирования, мокирования и покрытия кода.
- **Jasmine**: Фреймворк для тестирования с синтаксисом, похожим на BDD (Behavior-Driven Development).

---

### **2️⃣ Интеграционное тестирование (Integration Testing)**

**Интеграционное тестирование** направлено на проверку взаимодействия между компонентами системы, такими как базы данных, API, внешние сервисы или микросервисы. Тесты проверяют, правильно ли работают эти компоненты в связке, и все ли данные корректно передаются между ними.

#### **Цель**:
- Проверить, как модули приложения взаимодействуют друг с другом.
- Убедиться, что компоненты системы правильно взаимодействуют, обрабатывают данные и обеспечивают нужный результат.

#### **Пример**:
Допустим, у вас есть контроллер, который вызывает базу данных для получения информации о пользователе:

```javascript
// controller.js
const { getUser } = require('./database');

async function getUserInfo(req, res) {
  const user = await getUser(req.params.id);
  res.json(user);
}

module.exports = { getUserInfo };
```

Для тестирования этого контроллера с подключением к базе данных можно использовать `sinon` или мокировать зависимости, чтобы протестировать логику без реальной базы данных.

#### **Инструменты**:
- **Mocha + Chai**: Можно использовать для написания интеграционных тестов, включая работу с внешними сервисами и базами данных.
- **Jest**: Встроенная поддержка мокирования и работы с асинхронными кодами.
- **Supertest**: Библиотека для тестирования HTTP-сервисов, полезна для тестирования RESTful API и взаимодействия с сервером.

---

### **3️⃣ Тестирование API (API Testing)**

**Тестирование API** направлено на проверку работы RESTful API или других интерфейсов, таких как GraphQL. Тестирование включает проверку правильности выполнения запросов, корректности ответов, работы с ошибками и производительности.

#### **Цель**:
- Проверить правильность обработки HTTP-запросов и возвращаемых ответов.
- Проверить, что API возвращает нужные данные в нужном формате.
- Проверить работу с ошибками (например, код 404 или 500).

#### **Пример**:
Для тестирования API можно использовать **Supertest** для выполнения HTTP-запросов и проверки ответа:

```javascript
const request = require('supertest');
const app = require('./app'); // Express app

describe('GET /users/:id', function() {
  it('should return user info', async function() {
    const response = await request(app).get('/users/1');
    assert.strictEqual(response.status, 200);
    assert.strictEqual(response.body.name, 'John Doe');
  });
});
```

#### **Инструменты**:
- **Supertest**: Позволяет удобно тестировать HTTP-запросы и взаимодействие с серверами.
- **Jest**: Можно использовать вместе с **Supertest** для тестирования API.
- **Chai HTTP**: Дополнительный плагин для библиотеки Chai, который помогает тестировать HTTP-запросы.

---

### **4️⃣ Тестирование производительности (Performance Testing)**

**Тестирование производительности** помогает определить, как приложение будет вести себя под нагрузкой. Это тестирование важно для сервисов, которые должны поддерживать большое количество пользователей и запросов.

#### **Цель**:
- Оценить, насколько хорошо приложение справляется с большими нагрузками.
- Проверить, как долго приложение будет работать под большой нагрузкой.

#### **Пример**:
Тестирование производительности можно проводить с использованием нагрузочных тестов, например, с помощью инструмента **Artillery**:

```bash
artillery quick --duration 30 --rate 5 -n 10 http://localhost:3000/
```

#### **Инструменты**:
- **Artillery**: Легковесный инструмент для тестирования производительности, который подходит для стресс-тестирования веб-приложений.
- **Apache JMeter**: Еще один популярный инструмент для тестирования нагрузки на сервер.

---

### **5️⃣ Сквозное тестирование (End-to-End Testing - E2E)**

**E2E-тестирование** проверяет приложение как целое, имитируя реальные сценарии использования. Эти тесты включают взаимодействие с интерфейсом (UI) или с полным стеком приложения, включая клиентскую и серверную части.

#### **Цель**:
- Проверить, как приложение работает в целом.
- Проверить функциональность всех компонентов системы при их взаимодействии.

#### **Пример**:
Для тестирования E2E можно использовать **Cypress** или **Puppeteer**. Например, тестирование, когда пользователь входит в систему:

```javascript
describe('User Login', function() {
  it('should login successfully', function() {
    cy.visit('http://localhost:3000/login');
    cy.get('input[name="username"]').type('testUser');
    cy.get('input[name="password"]').type('password123');
    cy.get('button[type="submit"]').click();
    cy.url().should('include', '/dashboard');
  });
});
```

#### **Инструменты**:
- **Cypress**: Один из самых популярных инструментов для тестирования UI, позволяет эмулировать действия пользователя в браузере.
- **Puppeteer**: Библиотека для управления браузером через API, используемая для автоматизации тестов пользовательского интерфейса.

---

### **6️⃣ Тестирование безопасности (Security Testing)**

**Тестирование безопасности** направлено на проверку уязвимостей приложения, таких как SQL-инъекции, XSS, CSRF и другие уязвимости. Это критически важный аспект для защиты данных пользователей и надежности приложения.

#### **Цель**:
- Проверить, защищено ли приложение от внешних атак.
- Обеспечить безопасность данных, которые обрабатываются приложением.

#### **Инструменты**:
- **OWASP ZAP**: Автоматизированный инструмент для тестирования безопасности веб-приложений.
- **Snyk**: Инструмент для поиска уязвимостей в зависимостях Node.js.
- **Node Security Platform**: Платформа для анализа уязвимостей в Node.js-приложениях.

---

### **7️⃣ Мокирование и стабы (Mocking and Stubbing)**

Мокирование (Mocking) и создание стабов (Stubbing) используются для изоляции компонентов и тестирования взаимодействий между ними. Это важно, когда тестируемые компоненты имеют внешние зависимости, такие как базы данных или API.

#### **Цель**:
- Изолировать компоненты для тестирования.
- Подменять реальные зависимости на фальшивые для проверки логики без взаимодействия с реальными сервисами.

#### **Инструменты**:
- **Sinon**: Библиотека для создания моков, стабов и шпионов.
- **Nock**: Для мокирования HTTP-запросов и тестирования взаимодействий с внешними API.

---

### **Заключение**

Тестирование в Node.js играет важную роль в обеспечении качества и безопасности приложения. В зависимости от уровня, на котором необходимо проводить проверку, можно использовать различные виды тестов, такие как **юнит-тесты**, **интеграционные тесты**, **E2E-тесты**, **тестирование производительности** и другие. Важно применять лучшие практики тестирования, использовать современные инструменты и подходы, чтобы минимизировать риски ошибок в продакшн-среде.


Тестирование **Nest.js** приложения включает несколько этапов, таких как юнит-тестирование, интеграционное тестирование, тестирование API и тестирование производительности. Nest.js предоставляет удобную инфраструктуру для тестирования через встроенные инструменты и поддержку популярных библиотек для тестирования, таких как **Jest** и **Supertest**.

## **1. Настройка тестирования в Nest.js**

По умолчанию **Nest.js** использует **Jest** для тестирования, так как это популярный и мощный фреймворк для Node.js. Когда вы создаете проект с помощью **Nest CLI**, он автоматически добавляет конфигурацию для **Jest**.

### **1.1 Установка и настройка Jest**

Если по каким-то причинам Jest не установлен, вы можете установить его с помощью следующей команды:

```bash
npm install --save-dev jest @nestjs/testing ts-jest
```

Конфигурация Jest будет добавлена в `jest.config.js`. Важно, чтобы в проекте был правильно настроен трансформер TypeScript для работы с тестами, например:

```js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  transform: {
    '^.+\\.tsx?$': 'ts-jest',
  },
};
```

---

## **2. Юнит-тестирование (Unit Testing)**

**Юнит-тестирование** в Nest.js включает в себя тестирование отдельных компонентов, таких как сервисы, контроллеры и другие классы, которые не зависят от внешних систем (например, базы данных).

### **2.1 Тестирование сервисов**

Пример сервиса, который мы будем тестировать:

```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class UserService {
  private users = [{ id: 1, name: 'John Doe' }];

  findOne(id: number) {
    return this.users.find(user => user.id === id);
  }
}
```

Тестирование сервиса:

```typescript
// user.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserService } from './user.service';

describe('UserService', () => {
  let service: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [UserService],
    }).compile();

    service = module.get<UserService>(UserService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  it('should find a user by id', () => {
    const user = service.findOne(1);
    expect(user).toEqual({ id: 1, name: 'John Doe' });
  });
});
```

В этом примере:
- Мы используем метод `beforeEach()` для создания экземпляра сервиса перед каждым тестом.
- Затем проверяем, что сервис правильно работает (например, находим пользователя по `id`).

### **2.2 Тестирование контроллеров**

Теперь рассмотрим тестирование **контроллера**:

```typescript
// user.controller.ts
import { Controller, Get, Param } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  getUser(@Param('id') id: string) {
    return this.userService.findOne(Number(id));
  }
}
```

Тестирование контроллера:

```typescript
// user.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserController } from './user.controller';
import { UserService } from './user.service';

describe('UserController', () => {
  let controller: UserController;
  let service: UserService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [UserService],
    }).compile();

    controller = module.get<UserController>(UserController);
    service = module.get<UserService>(UserService);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });

  it('should return a user by id', () => {
    jest.spyOn(service, 'findOne').mockReturnValue({ id: 1, name: 'John Doe' });
    
    expect(controller.getUser('1')).toEqual({ id: 1, name: 'John Doe' });
  });
});
```

Здесь:
- Мы мокируем вызов метода `findOne` с помощью **Jest spy**.
- Используем `jest.spyOn(service, 'findOne').mockReturnValue(...)`, чтобы подменить поведение реального метода в тестах.
- Контроллер проверяется через вызов метода `getUser`.

---

## **3. Интеграционное тестирование (Integration Testing)**

**Интеграционное тестирование** проверяет взаимодействие между компонентами приложения, такими как контроллеры и базы данных. Здесь мы будем использовать **Supertest** для тестирования HTTP-запросов.

### **3.1 Пример интеграционного теста API**

```typescript
// user.controller.ts
import { Controller, Get, Param } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  getUser(@Param('id') id: string) {
    return this.userService.findOne(Number(id));
  }
}
```

Интеграционный тест с **Supertest**:

```typescript
// user.controller.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserController } from './user.controller';
import { UserService } from './user.service';
import * as request from 'supertest';
import { INestApplication } from '@nestjs/common';

describe('UserController (e2e)', () => {
  let app: INestApplication;

  beforeAll(async () => {
    const moduleFixture: TestingModule = await Test.createTestingModule({
      controllers: [UserController],
      providers: [UserService],
    }).compile();

    app = moduleFixture.createNestApplication();
    await app.init();
  });

  it('/users/:id (GET)', async () => {
    const response = await request(app.getHttpServer()).get('/users/1');
    expect(response.status).toBe(200);
    expect(response.body).toEqual({ id: 1, name: 'John Doe' });
  });

  afterAll(async () => {
    await app.close();
  });
});
```

Здесь:
- Мы используем `supertest` для отправки HTTP-запросов на API.
- Проверяем, что возвращаемый ответ соответствует ожиданиям.
- Проверяем статус ответа и структуру данных.

---

## **4. Мокирование зависимостей и тестирование с помощью DI**

Nest.js использует **Dependency Injection (DI)** для управления зависимостями. Чтобы тестировать компоненты с зависимостями, можно использовать моки или фальшивые реализации.

### **4.1 Мокирование зависимостей**

Например, если ваш сервис зависит от базы данных или внешнего API, вы можете мокировать эти зависимости для юнит-тестов:

```typescript
// user.service.ts
import { Injectable } from '@nestjs/common';
import { UserRepository } from './user.repository';

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  findOne(id: number) {
    return this.userRepository.findById(id);
  }
}
```

Тест с мокированием `UserRepository`:

```typescript
// user.service.spec.ts
import { Test, TestingModule } from '@nestjs/testing';
import { UserService } from './user.service';
import { UserRepository } from './user.repository';

describe('UserService', () => {
  let service: UserService;
  let repository: UserRepository;

  beforeEach(async () => {
    const mockRepository = { findById: jest.fn().mockReturnValue({ id: 1, name: 'John Doe' }) };
    
    const module: TestingModule = await Test.createTestingModule({
      providers: [
        UserService,
        { provide: UserRepository, useValue: mockRepository },
      ],
    }).compile();

    service = module.get<UserService>(UserService);
    repository = module.get<UserRepository>(UserRepository);
  });

  it('should return a user by id', () => {
    expect(service.findOne(1)).toEqual({ id: 1, name: 'John Doe' });
    expect(repository.findById).toHaveBeenCalledWith(1);
  });
});
```

Здесь:
- Мы создаем **мок** для репозитория `UserRepository` с помощью `jest.fn()`.
- Мокаем метод `findById` и подставляем фиксированные данные.

---

## **5. Тестирование ошибок и исключений**

В Nest.js важно проверять, как система обрабатывает ошибки и исключения, такие как **404 Not Found**, **400 Bad Request** или **500 Internal Server Error**.

Пример теста для обработки ошибки:

```typescript
// user.controller.ts
import { Controller, Get, Param, NotFoundException } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get(':id')
  getUser(@Param('id') id: string) {
    const user = this.userService.findOne(Number(id));
    if (!user) {
      throw new NotFoundException('User not found');
    }
    return user;
  }
}
```

Тест на ошибку:

```typescript
// user.controller.spec.ts
it('should throw 404 if user not found', async () => {
  const response = await request(app.getHttpServer()).get('/users/999');
  expect(response.status).toBe(404);
  expect(response.body.message).toBe('User not found');
});
```

---

## **Заключение**

Тестирование в **Nest.js** может быть очень гибким и мощным благодаря встроенной поддержке Jest, моки и мокированию зависимостей, а также возможности тестировать различные уровни приложения (юнит-тесты, интеграционные тесты, E2E тесты). Главное — использовать правильные подходы и инструменты для каждого типа тестов, чтобы обеспечить качество вашего приложения.
