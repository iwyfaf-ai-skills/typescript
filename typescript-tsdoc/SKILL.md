---
name: iwyfaf-typescript-tsdoc
description: MUST be used when user asks to document TypeScript or Vue code — add TSDoc/JSDoc blocks, docblocks, или «задокументировать» functions, classes, methods, interfaces, type aliases, enums, generics, Vue composables, defineProps/defineEmits/defineModel. Enforces strict TSDoc spec (tsdoc.org) and writes all descriptions in Russian. Triggers on phrases like «добавь tsdoc», «задокументируй», «напиши док», «add tsdoc», «document this», «generate jsdoc».
metadata:
   author: iWatchYouFromAfar
   version: "2026.4.19"
   source: Scripts at https://github.com/antfu/skills
---

# TSDoc Generator

Воркфлоу добавления TSDoc-комментариев в TypeScript / Vue-код. Применять при любой задаче
«добавь доки / задокументируй».

## Базовые правила

- **Стандарт:** строгий TSDoc ([tsdoc.org](https://tsdoc.org)). Теги вне спецификации не использовать:
`@type`, `@function`, `@kind`, `@property`, `@class`, `@method`, `@constructor`, `@module` — **запрещены**.
- **Язык описаний:** русский. Имена символов — латиница как есть.
- **Scope:** документировать **только символы без существующего блока**. Уже задокументированные
не трогать, пока пользователь явно не попросит переписать.
- **`@example`:** не добавлять автоматически. Только по явной просьбе пользователя.
- **Не дублировать типы:** TS уже знает сигнатуру. Не писать «возвращает `number`» — описывать
смысл, а не типы.
- **Без тавтологий:** если описание повторяет имя символа (`getUser` → «получает пользователя»)
и не добавляет информации — не писать summary вообще и переходить к тегам.
- **Private remarks:** внутренние заметки для разработчиков — через `@privateRemarks`, не
в `@remarks`.

## Формат блока

```ts
/**
 * Краткая суть в одном предложении с точкой.
 *
 * @remarks
 * Нетривиальные детали: инварианты, side-effects, ограничения.
 *
 * @typeParam T — Назначение параметра типа.
 *
 * @param name — Что передаётся и зачем.
 *
 * @returns Что возвращается (без указания типа).
 *
 * @throws {ErrorType} Когда бросается.
 *
 * @defaultValue Значение по умолчанию для опционального поля.
 *
 * @deprecated Причина + чем заменить.
 *
 * @see {@link OtherSymbol}
 */
```

Правила оформления:

- Открывающий `/**`, закрывающий ` */`, каждая промежуточная строка начинается с ` * `.
- Первая строка — summary: **одно** предложение, заканчивается точкой.
- Между summary и блочными тегами — пустая строка ` *`.
- **Между каждым блочным тегом (`@param`, `@returns`, `@throws`, `@typeParam`, `@defaultValue`,
`@deprecated`, `@see`) — пустая строка ` *`.** Теги не ставятся подряд без разделения.
- **Порядок тегов:** `@remarks` → `@typeParam` → `@param` → `@returns` → `@throws` →
`@defaultValue` → `@deprecated` → `@see` → `@example`.
- Разделитель между именем параметра и описанием — ` — ` (длинное тире с пробелами вокруг).
- **Описания внутри тегов начинаются с заглавной буквы** (в том числе после ` — ` в `@param` и
`@typeParam`) и заканчиваются точкой.

## Workflow

1. **Определить цель** — какие файлы/символы просил документировать пользователь. Если не указано
явно — уточнить.
2. **Отфильтровать** — пропустить все символы, у которых уже есть `/** */` блок.
3. **Для каждого нового блока:**
   - Прочитать реализацию, понять назначение и контракт.
   - Написать summary по сути (если не получается не-тавтологичное — пропустить summary,
оставить только теги).
   - Добавить `@param` для каждого параметра, `@returns` для функций с non-void возвратом,
`@typeParam` для дженериков.
   - `@throws` — только если функция действительно кидает (не ловит сама) или вызывает что-то
кидающее наружу.
   - `@remarks` — только при наличии нетривиальных деталей (side-effects, порядок инициализации,
race condition-ы, ограничения).
   - `@deprecated` — при пометке устаревшим, всегда с указанием чем заменить.
4. **`@example` не добавлять**, если пользователь не попросил.
5. **Проверить self-check** в конце.

## Детальные справочники

- [references/tags.md](references/tags.md) — полный перечень разрешённых TSDoc-тегов и когда применять.
- [references/examples.md](references/examples.md) — before/after по типам символов (function, class, interface,
type alias, enum, generic).
- [references/vue-patterns.md](references/vue-patterns.md) — Vue-специфика: composables, `defineProps`, `defineEmits`,
`defineModel`, `defineSlots`.

## Self-check

- [ ] Все описания на русском.
- [ ] Использованы только теги из TSDoc-спецификации.
- [ ] Нет дублирования типов из сигнатуры в тексте описаний.
- [ ] Нет тавтологий, повторяющих имя символа.
- [ ] Порядок тегов соответствует разделу «Формат блока».
- [ ] Существующие блоки не изменены (только новые добавлены).
- [ ] `@example` отсутствует, если его не просили.
- [ ] Для Vue-специфики (composables, `defineProps`/`defineEmits`) применены паттерны из
`references/vue-patterns.md`.
