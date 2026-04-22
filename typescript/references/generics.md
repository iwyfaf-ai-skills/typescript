# Справочник по дженерикам

> **Загружать, когда:** Пользователь спрашивает о дженериках, маппинге типов, условных типах, шаблонных строковых типах или паттернах переиспользуемых типов.

Продвинутые дженерики и паттерны типобезопасного программирования.

## Основы дженериков

### Базовая дженерик-функция

```typescript
// Параметр типа T может быть любым типом
function identity<T>(value: T): T {
  return value;
}

const str = identity("hello");   // string
const num = identity(42);        // number
const obj = identity({ x: 1 });  // { x: number }

// Явное указание типа (редко требуется)
const explicit = identity<string>("hello");
```

### Дженерик-интерфейсы

```typescript
interface Container<T> {
  value: T;
  getValue(): T;
  setValue(value: T): void;
}

interface Repository<T, ID = string> {
  findById(id: ID): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<T>;
  delete(id: ID): Promise<boolean>;
}

// Реализация
class UserRepository implements Repository<User> {
  async findById(id: string): Promise<User | null> {
    // реализация
  }
  // ... остальные методы
}
```

### Дженерик-классы

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
const top = numberStack.pop(); // number | undefined
```

---

## Ограничения дженериков

### Ограничение extends

```typescript
// T должен иметь свойство length
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): T {
  console.log(`Длина: ${item.length}`);
  return item;
}

logLength("hello");        // OK: у строки есть length
logLength([1, 2, 3]);      // OK: у массива есть length
logLength({ length: 10 }); // OK: у объекта есть length
logLength(42);             // Ошибка: у числа нет length
```

### Ограничение keyof

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

interface Person {
  name: string;
  age: number;
}

const person: Person = { name: "Алиса", age: 30 };

const name = getProperty(person, "name"); // string
const age = getProperty(person, "age");   // number
const bad = getProperty(person, "email"); // Ошибка: "email" нет в Person
```

### Несколько ограничений

```typescript
interface Printable {
  print(): void;
}

interface Loggable {
  log(): string;
}

// T должен удовлетворять обоим интерфейсам
function process<T extends Printable & Loggable>(item: T): void {
  item.print();
  console.log(item.log());
}
```

### Параметры типа по умолчанию

```typescript
interface ApiResponse<T = unknown, E = Error> {
  data?: T;
  error?: E;
  status: number;
}

// Используются значения по умолчанию
const response1: ApiResponse = { status: 200 };

// Переопределён только тип данных
const response2: ApiResponse<User> = { data: user, status: 200 };

// Переопределены оба
const response3: ApiResponse<User, ValidationError> = {
  error: new ValidationError(),
  status: 400
};
```

---

## Маппинг типов (Mapped Types)

### Базовый маппинг

```typescript
// Преобразовать все свойства в опциональные
type Partial<T> = {
  [K in keyof T]?: T[K];
};

// Преобразовать все свойства в обязательные
type Required<T> = {
  [K in keyof T]-?: T[K];
};

// Преобразовать все свойства в readonly
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// Убрать модификатор readonly
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

### Практические маппинги

```typescript
// Сделать все свойства nullable
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Сделать все свойства асинхронными геттерами
type AsyncGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => Promise<T[K]>;
};

interface User {
  name: string;
  email: string;
}

type UserGetters = AsyncGetters<User>;
// {
//   getName: () => Promise<string>;
//   getEmail: () => Promise<string>;
// }
```

### Переопределение ключей (as clause)

```typescript
// Фильтрация ключей по типу
type FilterByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K];
};

interface Mixed {
  name: string;
  age: number;
  active: boolean;
  score: number;
}

type StringProps = FilterByType<Mixed, string>;
// { name: string }

type NumberProps = FilterByType<Mixed, number>;
// { age: number; score: number }

// Добавить префикс ко всем ключам
type Prefixed<T, P extends string> = {
  [K in keyof T as `${P}${Capitalize<string & K>}`]: T[K];
};

type PrefixedUser = Prefixed<User, "user">;
// { userName: string; userEmail: string }
```

---

## Условные типы (Conditional Types)

### Базовые условные типы

```typescript
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type A = IsString<string>;  // true
type B = IsString<number>;  // false
type C = IsString<"hello">; // true

// Практика: извлечение ненулевого типа
type NonNullable<T> = T extends null | undefined ? never : T;

type D = NonNullable<string | null>;     // string
type E = NonNullable<number | undefined>; // number
```

### Дистрибутивные условные типы

```typescript
// Условные типы распределяются по объединениям
type ToArray<T> = T extends unknown ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;
// string[] | number[] (не (string | number)[])

// Предотвратить распределение с помощью кортежа
type ToArrayNonDist<T> = [T] extends [unknown] ? T[] : never;

type Mixed = ToArrayNonDist<string | number>;
// (string | number)[]
```

### Ключевое слово infer

```typescript
// Извлечь возвращаемый тип
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type FnReturn = ReturnType<() => string>; // string

// Извлечь тип элемента массива
type ArrayElement<T> = T extends (infer E)[] ? E : never;

type Element = ArrayElement<number[]>; // number

// Извлечь результат Promise
type Awaited<T> = T extends Promise<infer R> ? Awaited<R> : T;

type Result = Awaited<Promise<Promise<string>>>; // string

// Извлечь первый параметр функции
type FirstParam<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;

type First = FirstParam<(name: string, age: number) => void>; // string
```

### Практические условные типы

```typescript
// Вспомогательный тип для API-ответа
type ApiResult<T> =
  | { success: true; data: T }
  | { success: false; error: string };

// Извлечь тип данных из результата
type ExtractData<T> = T extends { success: true; data: infer D } ? D : never;

type UserResult = ApiResult<User>;
type UserData = ExtractData<UserResult>; // User

// Типобезопасные обработчики событий
type EventHandler<T> = T extends `on${infer Event}`
  ? (event: Event) => void
  : never;

type ClickHandler = EventHandler<"onClick">; // (event: "Click") => void
```

---

## Шаблонные строковые типы (Template Literal Types)

### Базовые шаблонные строки

```typescript
type Greeting = `Привет, ${string}!`;

const valid: Greeting = "Привет, Мир!";          // OK
const invalid: Greeting = "Здравствуй, Мир!";    // Ошибка

// Комбинация с объединениями
type Size = "small" | "medium" | "large";
type Color = "red" | "blue" | "green";

type ColoredSize = `${Color}-${Size}`;
// "red-small" | "red-medium" | "red-large" |
// "blue-small" | "blue-medium" | "blue-large" |
// "green-small" | "green-medium" | "green-large"
```

### Типы для манипуляции строками

```typescript
// Встроенные типы для манипуляции строками
type Upper = Uppercase<"hello">;     // "HELLO"
type Lower = Lowercase<"HELLO">;     // "hello"
type Cap = Capitalize<"hello">;      // "Hello"
type Uncap = Uncapitalize<"Hello">; // "hello"

// Практика: генерация имён событий
type Event = "click" | "hover" | "focus";
type EventHandler = `on${Capitalize<Event>}`;
// "onClick" | "onHover" | "onFocus"

// CSS-свойства с вендорными префиксами
type CSSProp = "transform" | "transition";
type Prefixed = `-webkit-${CSSProp}` | `-moz-${CSSProp}` | CSSProp;
```

### Продвинутые шаблонные паттерны

```typescript
// Разбор путей в нотации с точкой
type PathSegment<T> = T extends `${infer Head}.${infer Tail}`
  ? Head | PathSegment<Tail>
  : T;

type Segments = PathSegment<"user.profile.name">;
// "user" | "profile" | "name"

// HTTP-методы с путями
type Method = "GET" | "POST" | "PUT" | "DELETE";
type Endpoint = "/users" | "/posts" | "/comments";

type Route = `${Method} ${Endpoint}`;
// "GET /users" | "GET /posts" | "GET /comments" |
// "POST /users" | ... и т.д.

// Типобезопасные ссылки на колонки SQL
type Table = "users" | "posts";
type Column<T extends Table> = T extends "users"
  ? "id" | "name" | "email"
  : T extends "posts"
    ? "id" | "title" | "content"
    : never;

type UserColumn = `users.${Column<"users">}`;
// "users.id" | "users.name" | "users.email"
```

---

## Вариативные Tuple типы

### Базовые вариативные Tuples

```typescript
// Распространение кортежных типов
type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];

type Combined = Concat<[1, 2], [3, 4]>;
// [1, 2, 3, 4]

// Добавление элемента в начало
type Prepend<T, U extends unknown[]> = [T, ...U];

type WithFirst = Prepend<0, [1, 2, 3]>;
// [0, 1, 2, 3]

// Добавление элемента в конец
type Append<T extends unknown[], U> = [...T, U];

type WithLast = Append<[1, 2, 3], 4>;
// [1, 2, 3, 4]
```

### Практические вариативные паттерны

```typescript
// Типизированная функция каррирования
type Curry<F> = F extends (...args: infer A) => infer R
  ? A extends [infer First, ...infer Rest]
    ? (arg: First) => Curry<(...args: Rest) => R>
    : R
  : never;

declare function curry<F extends (...args: any[]) => any>(fn: F): Curry<F>;

function add(a: number, b: number, c: number): number {
  return a + b + c;
}

const curriedAdd = curry(add);
const add1 = curriedAdd(1);     // (arg: number) => Curry<...>
const add1and2 = add1(2);       // (arg: number) => number
const result = add1and2(3);     // number (6)

// Типизированная функция pipe
type Pipe<T extends unknown[], R> = T extends [infer First, ...infer Rest]
  ? First extends (arg: R) => infer Next
    ? Pipe<Rest, Next>
    : never
  : R;

function pipe<T extends ((arg: any) => any)[]>(
  ...fns: T
): (arg: Parameters<T[0]>[0]) => Pipe<T, Parameters<T[0]>[0]> {
  return (arg) => fns.reduce((acc, fn) => fn(acc), arg);
}

const process = pipe(
  (n: number) => n * 2,
  (n: number) => n.toString(),
  (s: string) => s.length
);

const length = process(5); // number (2 — длина "10")
```

---

## Справочник встроенных утилитарных типов

| Утилита                    | Назначение                     | Пример                               |
|----------------------------|--------------------------------|--------------------------------------|
| `Partial<T>`               | Все свойства опциональны       | `Partial<User>`                      |
| `Required<T>`              | Все свойства обязательны       | `Required<Partial<User>>`            |
| `Readonly<T>`              | Все свойства только для чтения | `Readonly<User>`                     |
| `Pick<T, K>`               | Выбрать свойства               | `Pick<User, "id" \| "name">`         |
| `Omit<T, K>`               | Исключить свойства             | `Omit<User, "password">`             |
| `Record<K, V>`             | Создать тип объекта            | `Record<string, User>`               |
| `Exclude<T, U>`            | Удалить члены объединения      | `Exclude<"a" \| "b", "a">`           |
| `Extract<T, U>`            | Оставить члены объединения     | `Extract<"a" \| "b", "a">`           |
| `NonNullable<T>`           | Убрать null/undefined          | `NonNullable<string \| null>`        |
| `Parameters<F>`            | Параметры функции              | `Parameters<typeof fn>`              |
| `ReturnType<F>`            | Возвращаемый тип функции       | `ReturnType<typeof fn>`              |
| `ConstructorParameters<C>` | Параметры конструктора         | `ConstructorParameters<typeof Date>` |
| `InstanceType<C>`          | Тип экземпляра                 | `InstanceType<typeof Date>`          |
| `Awaited<T>`               | Распаковать Promise            | `Awaited<Promise<User>>`             |
| `NoInfer<T>`               | Предотвратить вывод типа       | `NoInfer<T>` (TS 5.4+)               |