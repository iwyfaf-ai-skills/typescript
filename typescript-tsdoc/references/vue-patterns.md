# TSDoc в Vue 3

Правила для `<script setup lang="ts">` и composables.

## Composable

Composable — это функция. Документируется как обычная функция + описываются поля возвращаемого
объекта.

**Рекомендуемый паттерн** — отдельный interface для return-типа, поля документируются в нём:

```ts
// useLayout.types.ts
import type { Component, ShallowRef } from 'vue';

/**
 * Результат вызова composable {@link useLayout}.
 */
export interface UseLayoutReturn {
  /**
   * Текущий layout-компонент, соответствующий `route.meta.layout`.
   *
   * @remarks
   * До первого успешного импорта равен `null`. Обновляется реактивно при смене маршрута.
   */
  layout: ShallowRef<Component | null>;
}
```

```ts
// useLayout.ts
/**
 * Динамически загружает layout-компонент в зависимости от мета-поля текущего маршрута.
 *
 * @remarks
 * При ошибке импорта или неизвестном имени layout'а откатывается к `layout-default.vue`.
 *
 * @returns Реактивную ссылку на текущий layout.
 */
export function useLayout(): UseLayoutReturn {
  // ...
}
```

Если интерфейса нет — допустимо описать поля прямо в `@returns` через `{@link}`-ссылки или
отдельными `@remarks`, но interface-вариант читабельнее.

## `defineProps` с типовым дженериком

Каждый prop документируется **перед своим полем в типе**:

```ts
const props = defineProps<{
  /**
   * Заголовок карточки.
   */
  title: string;

  /**
   * Количество элементов для отображения счётчика.
   *
   * @defaultValue `0`
   */
  count?: number;

  /**
   * Обработчик закрытия.
   *
   * @remarks
   * Вызывается синхронно при клике на крестик.
   */
  onClose?: () => void;
}>();
```

Не использовать `@param` — это не параметры функции, а поля интерфейса.

## `defineEmits` с типовым дженериком

Каждое событие документируется **перед своим полем**. Описание payload — в комментарии поля,
не отдельным `@param`:

```ts
const emit = defineEmits<{
  /**
   * Эмитится при успешной отправке формы. Payload — серверный ответ.
   */
  submit: [response: SubmitResponse];

  /**
   * Эмитится при отмене. Payload отсутствует.
   */
  cancel: [];

  /**
   * Эмитится при изменении выбранного элемента.
   */
  'update:selected': [id: number];
}>();
```

## `defineModel`

```ts
/**
 * Двусторонняя модель выбранного идентификатора.
 *
 * @defaultValue `null`
 */
const selectedId = defineModel<number | null>({ default: null });
```

## `defineSlots`

```ts
const slots = defineSlots<{
  /**
   * Основное содержимое карточки.
   */
  default(props: { item: Item }): unknown;

  /**
   * Кастомный футер. Если не передан — используется дефолтный.
   */
  footer?(): unknown;
}>();
```

## `defineExpose`

Документируется сам **компонент** (через комментарий к `<script setup>` — редко) или каждый
экспортируемый метод индивидуально. Поля `defineExpose` документировать не нужно: они уже имеют
свои TSDoc-блоки.

## Когда НЕ добавлять TSDoc в Vue

- На локальных `ref`/`computed`, используемых только внутри компонента, — доки не нужны
(TS видит тип).
- На inline-обработчики (`const onClick = () => ...`) — документируются только если реализация
нетривиальна.
- Template-only-значения, созданные для шаблона, — описываются кодом, не комментарием.

## Запрещено

- `@type`, `@prop`, `@event`, `@emits` — не TSDoc.
- Описание реактивности словами «реактивная ссылка на X» для каждого `ref` — шум. Достаточно
описания смысла.
