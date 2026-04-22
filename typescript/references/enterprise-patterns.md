# Справочник корпоративных паттернов

> **Загружать когда:** Пользователь спрашивает об обработке ошибок, валидации, архитектуре проекта, стратегиях миграции или масштабных паттернах TypeScript.

Проверенные паттерны для создания поддерживаемых TypeScript-приложений.

## Обработка ошибок

### Паттерн Result Type

Вместо выбрасывания исключений возвращайте типизированные результаты:

```typescript
// Определение типа Result
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Вспомогательные функции
function ok<T>(data: T): Result<T, never> {
  return { success: true, data };
}

function err<E>(error: E): Result<never, E> {
  return { success: false, error };
}

// Использование
interface ValidationError {
  field: string;
  message: string;
}

function parseEmail(input: string): Result<string, ValidationError> {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  if (!emailRegex.test(input)) {
    return err({ field: "email", message: "Неверный формат email" });
  }

  return ok(input.toLowerCase());
}

// Потребление Result
const result = parseEmail(userInput);

if (result.success) {
  console.log(`Действительный email: ${result.data}`);
} else {
  console.error(`Ошибка в поле ${result.error.field}: ${result.error.message}`);
}
```

### Типизированные классы ошибок

```typescript
// Бзовая ошибка приложения
abstract class AppError extends Error {
  abstract readonly code: string;
  abstract readonly statusCode: number;

  constructor(message: string) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Конкретные типы ошибок
class NotFoundError extends AppError {
  readonly code = "NOT_FOUND";
  readonly statusCode = 404;

  constructor(resource: string, id: string) {
    super(`${resource} с id ${id} не найден`);
  }
}

class ValidationError extends AppError {
  readonly code = "VALIDATION_ERROR";
  readonly statusCode = 400;

  constructor(
    message: string,
    public readonly fields: Record<string, string[]>
  ) {
    super(message);
  }
}

class UnauthorizedError extends AppError {
  readonly code = "UNAUTHORIZED";
  readonly statusCode = 401;

  constructor(message = "Требуется аутентификация") {
    super(message);
  }
}

// Type guard для ошибок приложения
function isAppError(error: unknown): error is AppError {
  return error instanceof AppError;
}

// Обработчик ошибок
function handleError(error: unknown): { status: number; body: object } {
  if (isAppError(error)) {
    return {
      status: error.statusCode,
      body: {
        code: error.code,
        message: error.message,
        ...(error instanceof ValidationError && { fields: error.fields })
      }
    };
  }

  console.error("Неожиданная ошибка:", error);
  return {
    status: 500,
    body: { code: "INTERNAL_ERROR", message: "Внутренняя ошибка сервера" }
  };
}
```

---

## Паттерны валидации

### Валидация схем Zod

```typescript
import { z } from 'zod';

// Определение схем
const UserSchema = z.object({
  id: z.string().uuid(),
  name: z.string().min(1).max(100),
  email: z.string().email(),
  age: z.number().int().min(0).max(150).optional(),
  role: z.enum(["user", "admin", "moderator"]),
  metadata: z.record(z.string()).optional()
});

// Вывод типа TypeScript
type User = z.infer<typeof UserSchema>;

// Создание схем DTO
const CreateUserSchema = UserSchema.omit({ id: true });
type CreateUserDto = z.infer<typeof CreateUserSchema>;

const UpdateUserSchema = UserSchema.partial().omit({ id: true });
type UpdateUserDto = z.infer<typeof UpdateUserSchema>;

// Функции валидации
function validateCreateUser(data: unknown): Result<CreateUserDto, z.ZodError> {
  const result = CreateUserSchema.safeParse(data);

  if (result.success) {
    return ok(result.data);
  }

  return err(result.error);
}

// Преобразование ошибок Zod в удобный для пользователя формат
function formatZodError(error: z.ZodError): Record<string, string[]> {
  const formatted: Record<string, string[]> = {};

  for (const issue of error.issues) {
    const path = issue.path.join(".");
    if (!formatted[path]) {
      formatted[path] = [];
    }
    formatted[path].push(issue.message);
  }

  return formatted;
}
```

### Branded типы для валидации

```typescript
// Branded/Nominal типы
declare const EmailBrand: unique symbol;
type Email = string & { readonly [EmailBrand]: true };

declare const UserIdBrand: unique symbol;
type UserId = string & { readonly [UserIdBrand]: true };

// Функции валидации, возвращающие branded типы
function validateEmail(input: string): Email {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!emailRegex.test(input)) {
    throw new ValidationError("Неверный email", { email: ["Неверный формат"] });
  }
  return input as Email;
}

function validateUserId(input: string): UserId {
  const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;
  if (!uuidRegex.test(input)) {
    throw new ValidationError("Неверный ID пользователя", { id: ["Должен быть UUID"] });
  }
  return input as UserId;
}

// Использование: функции требуют валидированные типы
function sendEmail(to: Email, subject: string): void {
  // to гарантированно является действительным email
}

function getUser(id: UserId): Promise<User> {
  // id гарантированно является действительным UUID
}

// Компилятор требует валидации
sendEmail("invalid", "Привет");                // Ошибка: string не присваивается Email
sendEmail(validateEmail("a@b.com"), "Привет"); // OK
```

---

## Организация проекта

### Структура на основе фич

```
src/
├── features/
│   ├── users/
│   │   ├── index.ts           # Публичный экспорт (бочка)
│   │   ├── user.types.ts      # Типы и интерфейсы
│   │   ├── user.schema.ts     # Схемы Zod
│   │   ├── user.service.ts    # Бизнес-логика
│   │   ├── user.repository.ts # Доступ к данным
│   │   ├── user.controller.ts # HTTP-обработчики
│   │   └── __tests__/
│   │       ├── user.service.test.ts
│   │       └── user.controller.test.ts
│   ├── auth/
│   │   ├── index.ts
│   │   ├── auth.types.ts
│   │   └── ...
│   └── posts/
│       └── ...
├── shared/
│   ├── types/
│   │   ├── result.ts
│   │   └── pagination.ts
│   ├── utils/
│   │   ├── validation.ts
│   │   └── date.ts
│   └── errors/
│       └── app-error.ts
├── infrastructure/
│   ├── database/
│   │   └── client.ts
│   ├── cache/
│   │   └── redis.ts
│   └── logging/
│       └── logger.ts
└── config/
    ├── index.ts
    └── env.ts
```

### Barrel экспорт

```typescript
// features/users/index.ts
export type { User, CreateUserDto, UpdateUserDto } from './user.types';
export { UserSchema, CreateUserSchema } from './user.schema';
export { UserService } from './user.service';
export { UserController } from './user.controller';

// Не экспортировать репозиторий (внутренняя деталь)
// Не экспортировать внутренние вспомогательные функции

// Использование в других модулях
import { User, UserService } from '@/features/users';
```

### Path Aliases

```json
// tsconfig.json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@features/*": ["src/features/*"],
      "@shared/*": ["src/shared/*"],
      "@config/*": ["src/config/*"]
    }
  }
}
```

---

## Стратегии миграции

### Инкрементальная миграция с JavaScript

**Фаза 1: Включение TypeScript рядом с JavaScript**

```json
// tsconfig.json
{
  "compilerOptions": {
    "allowJs": true,
    "checkJs": false,
    "outDir": "./dist",
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "strict": false,
    "noImplicitAny": false
  },
  "include": ["src/**/*"]
}
```

**Фаза 2: Переименование файлов постепенно**

```bash
# Конвертация одного файла за раз
mv src/utils/helpers.js src/utils/helpers.ts

# Добавление минимальных аннотаций типов
# Исправление ошибок типов
# Запуск тестов для проверки
```

**Фаза 3: Постепенное включение строгих проверок**

```json
// Прогрессия строгих опций
{
  "compilerOptions": {
    // Шаг 1: Базовая строгость
    "noImplicitAny": true,

    // Шаг 2: Безопасность null
    "strictNullChecks": true,

    // Шаг 3: Полный строгий режим
    "strict": true,

    // Шаг 4: Дополнительная безопасность (опционально)
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### JSDoc для постепенной типизации

```javascript
// До полной миграции используйте JSDoc
/**
 * @typedef {Object} User
 * @property {string} id
 * @property {string} name
 * @property {string} email
 */

/**
 * Поиск пользователя по ID
 * @param {string} id - ID пользователя
 * @returns {Promise<User | null>}
 */
async function findUser(id) {
  // реализация
}

/**
 * @template T
 * @param {T[]} items
 * @returns {T | undefined}
 */
function first(items) {
  return items[0];
}
```

### Миграция с CommonJS на ESM

```json
// package.json
{
  "type": "module"
}
```

```typescript
// До (CommonJS)
const express = require('express');
const { UserService } = require('./user.service');
module.exports = { router };

// После (ESM)
import express from 'express';
import { UserService } from './user.service.js'; // Обратите внимание на расширение .js
export { router };
```

---

## Паттерны безопасности

### Санитизация ввода

```typescript
import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';

// Санитизированная строка
const SanitizedString = z.string().transform((val) => {
  return DOMPurify.sanitize(val.trim());
});

// Схема HTML-контента (для форматированного текста)
const HtmlContentSchema = z.string().transform((val) => {
  return DOMPurify.sanitize(val, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p', 'br'],
    ALLOWED_ATTR: ['href', 'target']
  });
});

// SQL-безопасный идентификатор
const SafeIdentifierSchema = z.string().regex(
  /^[a-zA-Z_][a-zA-Z0-9_]*$/,
  "Неверный идентификатор"
);
```

### TypeSafe переменные окружения

```typescript
import { z } from 'zod';

const EnvSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  REDIS_URL: z.string().url().optional()
});

// Валидация при запуске
function loadEnv() {
  const result = EnvSchema.safeParse(process.env);

  if (!result.success) {
    console.error('Invalid environment variables:');
    console.error(result.error.format());
    process.exit(1);
  }

  return result.data;
}

export const env = loadEnv();

// Использование: полностью типизировано
env.PORT;        // number
env.NODE_ENV;    // "development" | "production" | "test"
env.REDIS_URL;   // string | undefined
```

### Безопасные типы ответов API

```typescript
// Никогда не раскрывать внутренние поля
interface InternalUser {
  id: string;
  name: string;
  email: string;
  passwordHash: string;
  internalNotes: string;
}

// Публичный ответ API (Pick только безопасных полей)
type PublicUser = Pick<InternalUser, 'id' | 'name' | 'email'>;

// Или явное определение
interface UserResponse {
  id: string;
  name: string;
  email: string;
}

// Функция трансформации
function toPublicUser(user: InternalUser): UserResponse {
  return {
    id: user.id,
    name: user.name,
    email: user.email
  };
}
```

### Rate Limiting Types

```typescript
interface RateLimitConfig {
  windowMs: number;
  maxRequests: number;
}

interface RateLimitResult {
  allowed: boolean;
  remaining: number;
  resetAt: Date;
}

const rateLimits: Record<string, RateLimitConfig> = {
  api: { windowMs: 60000, maxRequests: 100 },
  auth: { windowMs: 300000, maxRequests: 5 },
  upload: { windowMs: 3600000, maxRequests: 10 }
} as const satisfies Record<string, RateLimitConfig>;
```