---
name: typescript
description: Освойте enterprise-разработку на TypeScript с типобезопасными паттернами, современными инструментами и интеграцией с фреймворками. Этот навык предоставляет всестороннее руководство по TypeScript 5.9+, охватывая основы системы типов (дженерики, маппинг типов, условные типы, оператор satisfies), enterprise-паттерны (обработка ошибок, валидация с помощью Zod), NestJS для масштабируемых API и LangChain.js для AI-приложений. Используйте при создании типобезопасных приложений, миграции кодовых баз с JavaScript, настройке современных инструментов (Vite 7, pnpm, ESLint, Vitest), внедрении продвинутых паттернов типизации или сравнении TypeScript с подходами Java/Python.
license: MIT
metadata:
  author: iWatchYouFromAfar
  version: "2026.4.22"
  source: Generated from https://github.com/SpillwaveSolutions/mastering-typescript-skill/tree/main/mastering-typescript, Scripts at https://github.com/iwyfaf-ai-skills/vue-3
---

# Освоение современного TypeScript

Создавайте enterprise-приложения с типобезопасностью на TypeScript 5.9+.

> **Совместимость:** TypeScript 5.9+, Node.js 22 LTS, Vite 7, NestJS 11

## Быстрый старт

```bash
```bash
# Инициализация TypeScript проекта с ESM
pnpm create vite@latest my-app --template vanilla-ts
cd my-app && pnpm install

# Настройка строгого TypeScript
cat > tsconfig.json << 'EOF'
{
  "compilerOptions": {
    "target": "ES2024",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "esModuleInterop": true,
    "skipLibCheck": true
  }
}
EOF
```

## Когда использовать этот навык

Используйте когда:

- Создаете типобезопасные приложения на NestJS или Node.js
- Мигрируете кодовую базу с JavaScript на TypeScript
- Внедряете продвинутые паттерны типизации (дженерики, маппинг типов, условные типы)
- Настраиваете современные инструменты TypeScript (Vite, pnpm, ESLint)
- Проектируете типобезопасные контракты API с валидацией Zod
- Сравниваете подходы TypeScript с Java или Python

## Чек-лист настройки проекта

Перед началом любого TypeScript проекта:

```
- [ ] Используйте pnpm для управления пакетами (быстрее, экономит место)
- [ ] Настройте ESM как основной модуль ("type": "module" в package.json)
- [ ] Включите строгий режим в tsconfig.json
- [ ] Настройте ESLint с @typescript-eslint
- [ ] Добавьте Prettier для единообразного форматирования
- [ ] Настройте Vitest для тестирования
```

## Соглашения об именовании

- **Псевдонимы типов** (`type X = ...`) должны иметь префикс `T` — например, `TStatus`, `TResult`, `TUnwrapPromise`.
- **Interfaces** (`interface X {}`) должны иметь префикс `I` — e.g. `IUser`, `IApiResponse`.
- **Enums** (`enum X {}`) должны иметь префикс `E` — e.g. `E`, `EPlanet`, `EFilms`.
- Применяется к каждому объявлению в этом навыке и в генерируемом коде.

## Краткий справочник по системе типов

### Примитивные типы

```typescript
const name: string = "Алиса";
const age: number = 30;
const active: boolean = true;
const id: bigint = 9007199254740991n;
const key: symbol = Symbol("уникальный");
```

### Union and Intersection Types

```typescript
// Union: значение может быть одного из нескольких типов
type TStatus = "pending" | "approved" | "rejected";

// Intersection: значение должно удовлетворять всем типам
type TEmployee = IPerson & { employeeId: string };

// Размеченное объединение для типобезопасной обработки
type TResult<T> =
  | { success: true; data: T }
  | { success: false; error: string };

function handleResult<T>(result: TResult<T>): T | null {
  if (result.success) {
    return result.data; // Здесь TypeScript знает, что data существует
  }
  console.error(result.error);
  return null;
}
```

### Type Guards

```typescript
// Защитник типа typeof
function process(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

// Пользовательский защитник типа
interface IUser { type: "user"; name: string }
interface IAdmin { type: "admin"; permissions: string[] }

function isAdmin(person: IUser | IAdmin): person is IAdmin {
  return person.type === "admin";
}
```

### Оператор `satisfies` (TS 5.0+)

Проверяет соответствие типу, сохраняя вывод типов:

```typescript
// Проблема: утверждение типа теряет конкретную информацию о типе
const colors1 = {
  red: "#ff0000",
  green: "#00ff00"
} as Record<string, string>;

colors1.red.toUpperCase(); // OK, но red может быть undefined

// Решение: satisfies сохраняет литеральные типы
const colors2 = {
  red: "#ff0000",
  green: "#00ff00"
} satisfies Record<string, string>;

colors2.red.toUpperCase(); // OK, и TypeScript знает, что red существует
```

## Паттерны дженериков

### Базовая дженерик-функция

```typescript
function first<T>(items: T[]): T | undefined {
  return items[0];
}

const num = first([1, 2, 3]);     // number | undefined
const str = first(["a", "b"]);   // string | undefined
```

### Ограниченные дженерики

```typescript
interface IHasLength {
  length: number;
}

function logLength<T extends IHasLength>(item: T): T {
  console.log(item.length);
  return item;
}

logLength("hello");     // OK: у строки есть length
logLength([1, 2, 3]);   // OK: у массива есть length
logLength(42);          // Ошибка: у числа нет length
```

### Дженерик-обертка для API ответов

```typescript
interface IApiResponse<T> {
  data: T;
  status: number;
  timestamp: Date;
}

async function fetchUser(id: string): Promise<IApiResponse<IUser>> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return {
    data,
    status: response.status,
    timestamp: new Date()
  };
}
```

## Справочник вспомогательных типов

| Тип            | Назначение                                  | Пример                      |
|-----------------|---------------------------------------------|------------------------------|
| `Partial<T>`    | Все свойства опциональны                    | `Partial<User>`              |
| `Required<T>`   | Все свойства обязательны                    | `Required<Config>`           |
| `Pick<T, K>`    | Выбрать определенные свойства               | `Pick<User, "id" \| "name">` |
| `Omit<T, K>`    | Исключить определенные свойства             | `Omit<User, "password">`     |
| `Record<K, V>`  | Объект с типизированными ключами/значениями | `Record<string, number>`     |
| `ReturnType<F>` | Извлечь тип возвращаемого значения функции  | `ReturnType<typeof fn>`      |
| `Parameters<F>` | Извлечь типы параметров функции             | `Parameters<typeof fn>`      |
| `Awaited<T>`    | Развернуть тип Promise                      | `Awaited<Promise<User>>`     |

## Условные типы (Conditional Types)

```typescript
// Базовый условный тип
type TIsString<T> = T extends string ? true : false;

// Извлечение типа элемента массива
type TArrayElement<T> = T extends (infer E)[] ? E : never;

type TNumbers = TArrayElement<number[]>; // number
type TStrings = TArrayElement<string[]>; // string

// Практический пример: извлечение типа результата Promise
type TUnwrapPromise<T> = T extends Promise<infer R> ? R : T;
```

## Маппинг типов

```typescript
// Сделать все свойства доступными только для чтения
type TImmutable<T> = {
  readonly [K in keyof T]: T[K];
};

// Сделать все свойства nullable
type TNullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Создать функции-геттеры для каждого свойства
type TGetters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface IPerson { name: string; age: number }
type TPersonGetters = TGetters<IPerson>;
// { getName: () => string; getAge: () => number }
```

## Интеграция с фреймворками

### NestJS с TypeScript

```typescript
// Типобезопасный DTO с class-validator
import { IsString, IsEmail, MinLength } from 'class-validator';

class CreateUserDto {
  @IsString()
  @MinLength(2)
  name: string;

  @IsEmail()
  email: string;
}

// Или с Zod (современный подход)
import { z } from 'zod';

const CreateUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email()
});

type TCreateUserDto = z.infer<typeof CreateUserSchema>;
```

Смотрите [nestjs-integration.md](references/nestjs-integration.md) для детальных паттернов.

## Валидация с Zod

```typescript
import { z } from 'zod';

// Определение схемы
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(["user", "admin", "moderator"]),
  createdAt: z.coerce.date()
});

// Вывод типа TypeScript из схемы
type TUser = z.infer<typeof UserSchema>;

// Валидация во время выполнения
function parseUser(data: unknown): TUser {
  return UserSchema.parse(data); // Бросает ZodError, если невалидно
}

// Безопасный парсинг (возвращает объект результата)
const result = UserSchema.safeParse(data);
if (result.success) {
  console.log(result.data); // Типизирован как User
} else {
  console.error(result.error.issues);
}
```

## Modern Toolchain (2025)

| Инструмент  | Версия  | Назначение                     |
|-------------|---------|--------------------------------|
| TypeScript  | 5.9+    | Проверка типов и компиляция    |
| Node.js     | 22 LTS  | Среда выполнения               |
| Vite        | 7.x     | Инструмент сборки и dev-сервер |
| pnpm        | 9.x     | Менеджер пакетов               |
| ESLint      | 9.x     | Линтер с плоской конфигурацией |
| Vitest      | 3.x     | Фреймворк для тестирования     |
| Prettier    | 3.x     | Форматирование кода            |

### Плоская конфигурация ESLint (ESLint 9+)

```javascript
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  }
);
```

## Стратегии миграции

### Инкрементальная миграция

1. Добавьте `allowJs: true` и `checkJs: false` в `tsconfig.json`
2. Переименовывайте файлы с `.js` на `.ts` по одному
3. Постепенно добавляйте аннотации типов
4. Включайте более строгие опции инкрементально

### JSDoc для постепенной типизации

```javascript
// Перед полной миграцией используйте JSDoc
/**
 * @param {string} name
 * @param {number} age
 * @returns {User}
 */
function createUser(name, age) {
  return { name, age };
}
```

Смотрите [enterprise-patterns.md](references/enterprise-patterns.md) для всесторонних руководств по миграции.

## Распространенные ошибки

| Mistake                                     | Problem                                   | Fix                                              |
|---------------------------------------------|-------------------------------------------|--------------------------------------------------|
| Частое использование `any`                  | Уничтожает типобезопасность               | Используйте `unknown` и сужайте тип              |
| Игнорирование строгого режима               | Пропускает ошибки с `null`/`undefined`    | Включите все строгие опции                       |
| Утверждения типа (`as`)                     | Могут скрыть ошибки типов                 | Используйте `satisfies` или защитники типа       |
| Перечисления (enum) для простых объединений | Генерируют код во время выполнения        | Вместо этого используйте литеральные объединения |
| Отсутствие валидации данных API             | Несоответствие типов во время выполнения  | Используйте Zod на границах приложения           |

## Сравнение с другими языками

| Особенности       | TypeScript                           | Java                       | Python                            |
|-------------------|--------------------------------------|----------------------------|-----------------------------------|
| Система типов     | Структурная                          | Номинальная                | Постепенная (утиная типизация)    |
| Null-безопасность | Явная (`T` \| `null`)                | `@Nullable` аннотации      | Опционально через typing          |
| Дженерики         | На уровне типов, стираются           | На уровне типов, стираются | Во время выполнения через typing  |
| Interfaces        | Структурное соответствие             | Должны быть реализованы    | Протокол (3.8+)                   |
| Enums             | Избегайте (используйте объединения)  | Первоклассные              | Enum Класс                        |

## Reference Files

- [type-system.md](references/type-system.md) — Полное руководство по системе типов
- [generics.md](references/generics.md) — Продвинутые паттерны дженериков
- [enterprise-patterns.md](references/enterprise-patterns.md) — Обработка ошибок, валидация, архитектура
- [nestjs-integration.md](references/nestjs-integration.md) — Разработка API на NestJS
- [toolchain.md](references/toolchain.md) — Настройка современных инструментов сборки