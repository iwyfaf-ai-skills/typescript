# Примеры TSDoc

Before / after по типам символов. Весь текст — по-русски, теги — строго из TSDoc.

## Функция

```ts
// before
export function formatPrice(value: number, currency: string): string {
  return new Intl.NumberFormat('ru-RU', { style: 'currency', currency }).format(value);
}

// after
/**
 * Форматирует число как денежную сумму в локали ru-RU.
 *
 * @param value — Сумма для форматирования.
 *
 * @param currency — ISO-4217 код валюты (RUB, USD, EUR и т.д.).
 *
 * @returns Строка вида «1 234,56 ₽».
 */
export function formatPrice(value: number, currency: string): string {
  return new Intl.NumberFormat('ru-RU', { style: 'currency', currency }).format(value);
}
```

## Асинхронная функция с исключениями

```ts
// after
/**
 * Загружает пользователя по идентификатору.
 *
 * @remarks
 * Кэширует результат на 5 минут. После logout кэш инвалидируется автоматически.
 *
 * @param id — Идентификатор пользователя.
 *
 * @returns Сущность пользователя.
 *
 * @throws {NotFoundError} Если пользователь не найден.
 *
 * @throws {UnauthorizedError} Если сессия истекла.
 */
export async function fetchUser(id: number): Promise<User> {
  // ...
}
```

## Дженерик

```ts
// after
/**
 * Возвращает последний элемент коллекции или `undefined`, если она пуста.
 *
 * @typeParam T — Тип элемента.
 *
 * @param items — Произвольная коллекция.
 *
 * @returns Последний элемент либо `undefined`.
 */
export function last<T>(items: readonly T[]): T | undefined {
  return items[items.length - 1];
}
```

## Класс и метод

```ts
// after
/**
 * Очередь задач с ограничением на одновременное выполнение.
 *
 * @remarks
 * Порядок выполнения — FIFO. Задачи, добавленные после `dispose()`, сразу реджектятся.
 */
export class TaskQueue {
  /**
   * Добавляет задачу в очередь.
   *
   * @param task — Функция, возвращающая `Promise`.
   *
   * @returns Промис с результатом задачи.
   *
   * @throws {QueueDisposedError} Если очередь уже уничтожена.
   */
  public enqueue<T>(task: () => Promise<T>): Promise<T> {
    // ...
  }

  /**
   * Дожидается завершения всех активных задач и освобождает ресурсы.
   *
   * @remarks
   * После вызова повторно использовать инстанс нельзя.
   */
  public async dispose(): Promise<void> {
    // ...
  }
}
```

## Интерфейс

```ts
// after
/**
 * Контракт адаптера транспорта для отправки запросов.
 */
export interface TransportAdapter {
  /**
   * Отправляет запрос и ожидает ответ.
   *
   * @param request — Подготовленный HTTP-запрос.
   *
   * @returns Обёртка с телом ответа и мета-информацией.
   */
  send(request: HttpRequest): Promise<HttpResponse>;

  /**
   * Таймаут по умолчанию для запросов без явного значения.
   *
   * @defaultValue `30000`
   */
  defaultTimeoutMs?: number;
}
```

## Type alias

```ts
// after
/**
 * Набор поддерживаемых локалей интерфейса.
 */
export type Locale = 'ru' | 'en';
```

## Enum

```ts
// after
/**
 * Статус заявки в жизненном цикле обработки.
 */
export enum RequestStatus {
  /** Создана, но ещё не отправлена на обработку. */
  Draft = 'draft',

  /** В обработке. */
  Pending = 'pending',

  /** Успешно обработана. */
  Completed = 'completed',

  /** Отклонена оператором или системой. */
  Rejected = 'rejected',
}
```

## Depricated-пометка

```ts
// after
/**
 * Возвращает полное имя пользователя.
 *
 * @deprecated Используйте {@link User.displayName}. Будет удалено в v3.
 */
export function getFullName(user: User): string {
  // ...
}
```

## Const со сложным типом

```ts
// after
/**
 * Таблица маппинга HTTP-кодов на внутренние коды ошибок.
 */
export const HTTP_ERROR_MAP: Readonly<Record<number, ErrorCode>> = {
  400: ErrorCode.BadRequest,
  401: ErrorCode.Unauthorized,
  403: ErrorCode.Forbidden,
};
```

## Package-level

В корневом `index.ts` пакета:

```ts
/**
 * HTTP-клиент с поддержкой retry, circuit breaker и middleware-цепочки.
 *
 * @packageDocumentation
 */
```
