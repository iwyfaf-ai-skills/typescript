# Справочник по системе типов TypeScript

> **Загружать, когда:** Пользователь спрашивает об аннотациях типов, интерфейсах vs типах,
объединениях, пересечениях или основах системы типов

Полное руководство по структурной системе типов TypeScript.

## Аннотации типов

### Аннотации переменных

```typescript
// Явные аннотации типов
const name: string = "Alice";
const age: number = 30;
const active: boolean = true;

// Type inference (предпочтительно, когда очевидно)
const inferredName = "Bob";  // TypeScript infers string
const inferredAge = 25;      // TypeScript infers number

// Массивы
const numbers: number[] = [1, 2, 3];
const strings: Array<string> = ["a", "b", "c"];

// Кортежи (tuples) (массивы фиксированной длины с определёнными типами)
const pair: [string, number] = ["age", 30];
const triple: [string, number, boolean] = ["name", 1, true];
```

### Аннотации функций

```typescript
// Функция с типизированными параметрами и возвращаемым значением
function greet(name: string): string {
  return `Привет, ${name}!`;
}

// Стрелочная функция
const add = (a: number, b: number): number => a + b;

// Опциональные параметры
function greetOptional(name: string, greeting?: string): string {
  return `${greeting ?? "Привет"}, ${name}!`;
}

// Параметры по умолчанию
function greetDefault(name: string, greeting: string = "Привет"): string {
  return `${greeting}, ${name}!`;
}

// Остаточные параметры
function sum(...numbers: number[]): number {
  return numbers.reduce((a, b) => a + b, 0);
}

// Псевдоним типа функции
type Comparator<T> = (a: T, b: T) => number;
const numberCompare: Comparator<number> = (a, b) => a - b;
```

---

## Interfaces vs Type Aliases

### Когда использовать интерфейсы

```typescript
// Интерфейсы идеально подходят для форм объектов
interface User {
  id: string;
  name: string;
  email: string;
}

// Интерфейсы можно расширять
interface Employee extends User {
  employeeId: string;
  department: string;
}

// Интерфейсы могут быть реализованы классами
class Manager implements Employee {
  constructor(
    public id: string,
    public name: string,
    public email: string,
    public employeeId: string,
    public department: string
  ) {}
}

// Слияние объявлений (только для интерфейсов)
interface Config {
  apiUrl: string;
}

interface Config {
  timeout: number;
}

// Config теперь имеет и apiUrl, и timeout
```

### Когда использовать псевдонимы типов

```typescript
// Псевдонимы типов для объединений
type Status = "pending" | "approved" | "rejected";

// Псевдонимы типов для сложных типов
type Handler = (event: Event) => void;

// Псевдонимы типов для отображённых типов
type Nullable<T> = { [K in keyof T]: T[K] | null };

// Псевдонимы типов для условных типов
type NonNullable<T> = T extends null | undefined ? never : T;

// Псевдонимы типов для кортежей
type Point = [x: number, y: number];
type RGB = [red: number, green: number, blue: number];
```

### Руководство по выбору

| Use Case               | Prefer      |
|------------------------|-------------|
| Формы объектов         | `interface` |
| Расширение объектов    | `interface` |
| Контракты для классов  | `interface` |
| Union типы             | `type`      |
| Tuple типы             | `type`      |
| Mapped типы            | `type`      |
| Условные типы          | `type`      |
| Псевдонимы примитивов  | `type`      |

---

## Union and Intersection Types

### Типы-объединения (Union)

```typescript
// Значение может быть одним из нескольких типов
type StringOrNumber = string | number;

// Размеченные объединения (tagged unions)
interface Dog {
  kind: "dog";
  bark(): void;
}

interface Cat {
  kind: "cat";
  meow(): void;
}

type Pet = Dog | Cat;

function speak(pet: Pet): void {
  switch (pet.kind) {
    case "dog":
      pet.bark();
      break;
    case "cat":
      pet.meow();
      break;
  }
}
```

### Типы-пересечения (Intersection)

```typescript
// Значение должно удовлетворять всем типам
interface HasName {
  name: string;
}

interface HasAge {
  age: number;
}

type Person = HasName & HasAge;

const person: Person = {
  name: "Алиса",
  age: 30
};

// Практический пример: расширение дополнительными свойствами
type WithTimestamp<T> = T & { createdAt: Date; updatedAt: Date };

interface Article {
  title: string;
  content: string;
}

type TimestampedArticle = WithTimestamp<Article>;
```

---

## Литеральные типы

### Строковые литералы

```typescript
// Конкретные строковые значения
type Direction = "north" | "south" | "east" | "west";
type HttpMethod = "GET" | "POST" | "PUT" | "DELETE" | "PATCH";

function move(direction: Direction): void {
  console.log(`Движение ${direction}`);
}

move("north"); // OK
move("up");    // Ошибка: тип '"up"' не назначается
```

### Числовые литералы

```typescript
type DiceRoll = 1 | 2 | 3 | 4 | 5 | 6;
type BinaryDigit = 0 | 1;

function roll(): DiceRoll {
  return Math.ceil(Math.random() * 6) as DiceRoll;
}
```

### Шаблонные литеральные типы

```typescript
// Конструирование строковых литеральных типов
type EventName = "click" | "hover" | "focus";
type HandlerName = `on${Capitalize<EventName>}`;
// "onClick" | "onHover" | "onFocus"

// Типы CSS-единиц
type CSSUnit = "px" | "em" | "rem" | "%";
type CSSValue = `${number}${CSSUnit}`;

const width: CSSValue = "100px";   // OK
const height: CSSValue = "50%";    // OK
const bad: CSSValue = "100";       // Ошибка
```

---

## Type Guards and Narrowing

### Встроенные Type Guards

```typescript
function process(value: string | number | null): string {
  // Страж typeof
  if (typeof value === "string") {
    return value.toUpperCase();
  }

  // Страж typeof для number
  if (typeof value === "number") {
    return value.toFixed(2);
  }

  // Сужение null/undefined
  if (value === null) {
    return "null";
  }

  // Проверка исчерпываемости
  const _exhaustive: never = value;
  throw new Error(`Необработанный случай: ${_exhaustive}`);
}
```

### instanceof Guard

```typescript
class ApiError extends Error {
  constructor(public statusCode: number, message: string) {
    super(message);
  }
}

class ValidationError extends Error {
  constructor(public fields: string[]) {
    super("Validation failed");
  }
}

function handleError(error: Error): void {
  if (error instanceof ApiError) {
    console.log(`API Error ${error.statusCode}: ${error.message}`);
  } else if (error instanceof ValidationError) {
    console.log(`Validation Error on fields: ${error.fields.join(", ")}`);
  } else {
    console.log(`Unknown error: ${error.message}`);
  }
}
```

### Пользовательские стражи типов

```typescript
interface User {
  type: "user";
  name: string;
}

interface Admin {
  type: "admin";
  name: string;
  permissions: string[];
}

type Account = User | Admin;

// Type predicate: возвращает boolean, но сужает тип
function isAdmin(account: Account): account is Admin {
  return account.type === "admin";
}

function getPermissions(account: Account): string[] {
  if (isAdmin(account)) {
    return account.permissions; // TypeScript знает, что это Admin
  }
  return [];
}
```

### Assertion Functions

```typescript
function assertDefined<T>(value: T | undefined, message: string): asserts value is T {
  if (value === undefined) {
    throw new Error(message);
  }
}

function processUser(user: User | undefined): void {
  assertDefined(user, "Пользователь обязателен");
  // После утверждения user сужен до User
  console.log(user.name);
}
```

---

## Оператор satisfies

### Проблема: утверждения типов скрывают ошибки

```typescript
// Использование 'as' может скрыть ошибки типов
const config = {
  port: 3000,
  host: "localhost"
} as Record<string, string | number>;

// Нет ошибки, но port теперь string | number
const portString = config.port.toFixed(2); // Ошибка во время выполнения, если port — строка!
```

### Решение: satisfies проверяет соответствие без расширения

```typescript
// satisfies проверяет соответствие, но сохраняет литеральные типы
const config = {
  port: 3000,
  host: "localhost"
} satisfies Record<string, string | number>;

// TypeScript знает, что port — number, host — string
config.port.toFixed(2);      // OK — port это number
config.host.toUpperCase();   // OK — host это string
```

### Практические примеры использования

```typescript
// Цветовая палитра с ограниченными значениями
const palette = {
  primary: "#007bff",
  secondary: "#6c757d",
  success: "#28a745"
} satisfies Record<string, `#${string}`>;

// TypeScript знает, что каждое свойство существует и является hex-строкой
palette.primary.startsWith("#"); // OK

// Конфигурация маршрутов
type RouteConfig = {
  path: string;
  method: "GET" | "POST";
  handler: () => void;
};

const routes = {
  home: { path: "/", method: "GET", handler: () => {} },
  login: { path: "/login", method: "POST", handler: () => {} }
} satisfies Record<string, RouteConfig>;

// TypeScript сохраняет литеральные типы для каждого маршрута
routes.home.method; // "GET" (а не "GET" | "POST")
```

---

## Специальные типы

### any против unknown

```typescript
// any: отказ от проверки типов (избегайте)
let anyValue: any = "hello";
anyValue.toFixed(2); // Нет ошибки, но падает во время выполнения

// unknown: типобезопасный any (предпочтительнее)
let unknownValue: unknown = "hello";
unknownValue.toFixed(2); // Ошибка: объект типа 'unknown'

// Перед использованием unknown необходимо сузить
if (typeof unknownValue === "string") {
  unknownValue.toUpperCase(); // OK после сужения
}
```

### never

```typescript
// never: представляет невозможные значения
function fail(message: string): never {
  throw new Error(message);
}

// Проверка исчерпываемости с never
type Shape = "circle" | "square";

function getArea(shape: Shape): number {
  switch (shape) {
    case "circle":
      return Math.PI;
    case "square":
      return 1;
    default:
      // Если добавим новую фигуру, здесь будет ошибка
      const _exhaustive: never = shape;
      throw new Error(`Неизвестная фигура: ${_exhaustive}`);
  }
}
```

### void против undefined

```typescript
// void: функция не возвращает значимого значения
function log(message: string): void {
  console.log(message);
}

// undefined: явное значение undefined
function findUser(id: string): User | undefined {
  return users.get(id);
}
```