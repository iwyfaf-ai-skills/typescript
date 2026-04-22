# Современный справочник по инструментам

> **Загружать, когда:** Пользователь спрашивает о Vite, pnpm, ESLint, Vitest, tsconfig, инструментах сборки или конфигурации проекта.

Конфигурация современного TypeScript-инструментария для 2025 года.

## Конфигурация TypeScript

### Строгая корпоративная конфигурация

```json
// tsconfig.json
{
  "compilerOptions": {
    // Язык и окружение
    "target": "ES2024",
    "lib": ["ES2024"],
    "module": "ESNext",
    "moduleResolution": "bundler",

    // Строгая проверка типов
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noPropertyAccessFromIndexSignature": true,

    // Обработка модулей
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "isolatedModules": true,
    "verbatimModuleSyntax": true,

    // Выходные данные
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",

    // Алиасы путей
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    },

    // Производительность
    "skipLibCheck": true,
    "incremental": true,
    "tsBuildInfoFile": "./node_modules/.cache/tsbuildinfo"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Конфигурация для React-проекта

```json
// tsconfig.json for React
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,

    "esModuleInterop": true,
    "isolatedModules": true,
    "skipLibCheck": true,

    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@hooks/*": ["./src/hooks/*"]
    }
  },
  "include": ["src"],
  "references": [{ "path": "./tsconfig.node.json" }]
}
```

### Конфигурация для Node.js бэкенда

```json
// tsconfig.json for Node.js/NestJS
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "lib": ["ES2022"],

    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,

    "esModuleInterop": true,
    "isolatedModules": true,
    "skipLibCheck": true,

    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "sourceMap": true,

    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

---

## Менеджер пакетов (pnpm)

### Почему pnpm

| Возможность                 | npm                             | pnpm                      |
|-----------------------------|---------------------------------|---------------------------|
| Использование диска         | Дублирует пакеты                | Общее хранилище, симлинки |
| Скорость установки          | Медленнее                       | В 2-3 раза быстрее        |
| Строгость                   | Разрешает фантомные зависимости | Строгий по умолчанию      |
| Поддержка монорепозиториев  | Базовые workspaces              | Первоклассная поддержка   |

### Основные команды

```bash
# Установка зависимостей
pnpm install

# Добавление пакетов
pnpm add typescript
pnpm add -D vitest @types/node

# Запуск скриптов
pnpm run build
pnpm test

# Обновление пакетов
pnpm update
pnpm update --interactive

# Список пакетов
pnpm list
pnpm why lodash

# Чистая установка
pnpm install --frozen-lockfile
```

### Конфигурация рабочей области (workspace)

```yaml
# pnpm-workspace.yaml
packages:
  - 'packages/*'
  - 'apps/*'
```

```json
// package.json (корневой)
{
  "name": "my-monorepo",
  "private": true,
  "scripts": {
    "build": "pnpm -r run build",
    "test": "pnpm -r run test",
    "lint": "pnpm -r run lint"
  }
}
```

---

## Инструмент сборки (Vite)

### Конфигурация Vite

```typescript
// vite.config.ts
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [
    react(),
    tsconfigPaths()
  ],
  server: {
    port: 3000,
    host: true
  },
  build: {
    target: 'es2022',
    sourcemap: true,
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          utils: ['lodash-es', 'date-fns']
        }
      }
    }
  },
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts']
  }
});
```

### Режим библиотеки

```typescript
// vite.config.ts for library
import { defineConfig } from 'vite';
import dts from 'vite-plugin-dts';

export default defineConfig({
  build: {
    lib: {
      entry: './src/index.ts',
      name: 'MyLibrary',
      fileName: 'my-library',
      formats: ['es', 'cjs']
    },
    rollupOptions: {
      external: ['react', 'react-dom'],
      output: {
        globals: {
          react: 'React',
          'react-dom': 'ReactDOM'
        }
      }
    }
  },
  plugins: [
    dts({ insertTypesEntry: true })
  ]
});
```

---

## Линтинг (ESLint 9)

### Формат Flat Config

```javascript
// eslint.config.js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';
import reactPlugin from 'eslint-plugin-react';
import reactHooksPlugin from 'eslint-plugin-react-hooks';

export default tseslint.config(
  // Базовые рекомендации ESLint
  eslint.configs.recommended,

  // Строгая проверка типов TypeScript
  ...tseslint.configs.strictTypeChecked,
  ...tseslint.configs.stylisticTypeChecked,

  // Опции парсера TypeScript
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname
      }
    }
  },

  // Конфигурация React
  {
    files: ['**/*.tsx'],
    plugins: {
      react: reactPlugin,
      'react-hooks': reactHooksPlugin
    },
    rules: {
      ...reactPlugin.configs.recommended.rules,
      ...reactHooksPlugin.configs.recommended.rules,
      'react/react-in-jsx-scope': 'off'
    },
    settings: {
      react: { version: 'detect' }
    }
  },

  // Пользовательские правила
  {
    rules: {
      '@typescript-eslint/no-unused-vars': ['error', {
        argsIgnorePattern: '^_',
        varsIgnorePattern: '^_'
      }],
      '@typescript-eslint/consistent-type-imports': ['error', {
        prefer: 'type-imports'
      }],
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/await-thenable': 'error'
    }
  },

  // Шаблоны игнорирования
  {
    ignores: ['dist/**', 'node_modules/**', '*.config.js']
  }
);
```

### Объяснение распространенных правил

```javascript
// Важные правила TypeScript ESLint
{
  rules: {
    // Требовать импорты типов для лучшей tree-shaking
    '@typescript-eslint/consistent-type-imports': 'error',

      // Предотвращать необработанные промисы
      '@typescript-eslint/no-floating-promises': 'error',
      '@typescript-eslint/no-misused-promises': 'error',

      // Предотвращать await не-промисов
      '@typescript-eslint/await-thenable': 'error',

      // Требовать типы возврата для функций
      '@typescript-eslint/explicit-function-return-type': ['error', {
      allowExpressions: true
    }],

      // Предпочитать оператор нулевого слияния
      '@typescript-eslint/prefer-nullish-coalescing': 'error',

      // Предпочитать опциональную цепочку
      '@typescript-eslint/prefer-optional-chain': 'error',

      // Запретить тип any
      '@typescript-eslint/no-explicit-any': 'error',

      // Требовать строгие булевы выражения
      '@typescript-eslint/strict-boolean-expressions': 'error'
  }
}
```

---

## Тестирование (Vitest)

### Конфигурация Vitest

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

export default defineConfig({
  plugins: [react(), tsconfigPaths()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./src/test/setup.ts'],
    include: ['src/**/*.{test,spec}.{ts,tsx}'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['**/*.d.ts', '**/*.config.*', '**/test/**']
    },
    typecheck: {
      enabled: true
    }
  }
});
```

### Настройка тестов

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest';
import { cleanup } from '@testing-library/react';
import { afterEach, beforeAll, afterAll, vi } from 'vitest';

// Cleanup after each test
afterEach(() => {
  cleanup();
});

// Mock environment variables
beforeAll(() => {
  vi.stubEnv('API_URL', 'http://localhost:3000');
});

afterAll(() => {
  vi.unstubAllEnvs();
});
```

### Примеры тестов

```typescript
// src/utils/format.test.ts
import { describe, it, expect } from 'vitest';
import { formatCurrency, formatDate } from './format';

describe('formatCurrency', () => {
  it('корректно форматирует USD', () => {
    expect(formatCurrency(1234.56, 'USD')).toBe('$1,234.56');
  });

  it('обрабатывает ноль', () => {
    expect(formatCurrency(0, 'USD')).toBe('$0.00');
  });
});

// src/components/Button.test.tsx
import { render, screen, fireEvent } from '@testing-library/react';
import { describe, it, expect, vi } from 'vitest';
import { Button } from './Button';

describe('Button', () => {
  it('отображается с меткой', () => {
    render(<Button label="Нажми меня" onClick={() => {}} />);
    expect(screen.getByRole('button')).toHaveTextContent('Нажми меня');
  });

  it('вызывает onClick при нажатии', () => {
    const handleClick = vi.fn();
    render(<Button label="Нажми" onClick={handleClick} />);

    fireEvent.click(screen.getByRole('button'));

    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

---

## Форматирование (Prettier)

### Конфигурация Prettier

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "bracketSpacing": true,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### Интеграция с ESLint

```javascript
// eslint.config.js
import eslintConfigPrettier from 'eslint-config-prettier';

export default tseslint.config(
  // ... другие конфигурации
  eslintConfigPrettier // Должен быть последним для отключения конфликтующих правил
);
```

### Скрипты в package.json

```json
// package.json
{
  "scripts": {
    "format": "prettier --write .",
    "format:check": "prettier --check .",
    "lint": "eslint .",
    "lint:fix": "eslint --fix .",
    "typecheck": "tsc --noEmit"
  }
}
```