# PHP / JavaScript Documentation Skill

## Purpose

This skill defines how an AI coding agent must document PHP and JavaScript code in this project. It is intended to produce code that is easy to understand, easy to maintain, compatible with PhpStorm, and suitable for generated API documentation where applicable.

The agent must document intent, contracts, edge cases, side effects, and integration points. It must not generate noisy, repetitive, or misleading comments that merely restate the code.

## Scope

Apply this skill whenever creating, editing, refactoring, reviewing, or generating:

- PHP classes, interfaces, traits, enums, functions, methods, properties, constants, exceptions, DTOs, repositories, controllers, services, commands, event subscribers, form types, validators, migrations, tests, and scripts.
- JavaScript modules, classes, functions, exported utilities, event handlers, API clients, state-management code, UI components, configuration objects, and test helpers.
- Inline comments used to explain complex logic, business rules, workarounds, or non-obvious decisions.
- TODO, FIXME, deprecation, migration, and temporary compatibility notes.

## Documentation objectives

Documentation must help the next developer answer these questions quickly:

1. What does this code do?
2. Why does it exist?
3. What inputs does it accept?
4. What does it return or change?
5. What can go wrong?
6. What business rule, framework convention, or external contract does it rely on?
7. Is it safe to reuse, extend, or call from another part of the system?

## Non-negotiable rules

1. Prefer clear code and native type declarations first. Use comments to add information that cannot be expressed clearly in code.
2. Do not write comments that merely repeat the function name, class name, or obvious implementation.
3. Keep comments accurate after every code change. Outdated documentation is a bug.
4. Document public APIs more thoroughly than private helpers.
5. Document business logic and edge cases even when the code is short.
6. Document all non-obvious framework, database, security, permission, caching, queue, file-system, API, or browser behaviour.
7. Do not invent behaviour in comments. If behaviour is uncertain, inspect the code or state the uncertainty explicitly in a temporary TODO.
8. Use complete sentences for descriptive text.
9. Use UK English spelling unless matching an external API, library, database field, or framework term.
10. Never leave placeholder comments such as `TODO: document this`, `Description`, `No description`, or `TBD`.
11. Never add author/date boilerplate unless the repository already requires it.
12. Never use comments to hide bad code. Refactor where reasonable.

## Comment quality standard

A good comment explains intent, constraints, or consequences.

Bad:

```php
// Loop through users.
foreach ($users as $user) {
    // ...
}
```

Good:

```php
// Preserve the original import order so validation errors can be mapped back to the uploaded spreadsheet rows.
foreach ($users as $user) {
    // ...
}
```

Bad:

```js
// Set disabled to true.
button.disabled = true;
```

Good:

```js
// Prevent duplicate submissions while the request is in flight; the button is re-enabled in finally().
button.disabled = true;
```

## PHP documentation standard

PHP documentation must use PHPDoc / phpDocumentor-style DocBlocks.

Use `/** ... */` for PHPDoc blocks that describe PHP symbols. Use `//` for ordinary inline implementation comments.

### PHPDoc block structure

Use this structure:

```php
/**
 * Short summary written as a complete sentence.
 *
 * Optional longer description explaining business context, edge cases,
 * side effects, persistence, permissions, external APIs, or framework behaviour.
 *
 * @param Type $name Description of the parameter where useful.
 * @return Type Description of the returned value where useful.
 * @throws ExceptionType When this exception is thrown.
 */
```

Rules:

- Start with a short summary.
- Add a blank line before the longer description.
- Add a blank line before tags.
- Align tags naturally; do not force excessive spacing.
- Keep line length readable. Prefer 100-120 characters unless the repository has a stricter standard.
- Use fully meaningful descriptions. Avoid repeating the variable name in prose.
- Do not include `@param` or `@return` descriptions if they add no information, unless required for IDE clarity or documentation generation.

### Native PHP types first

Use native PHP type declarations wherever possible:

```php
public function findActiveBoard(int $boardId): ?Board
```

Do not duplicate obvious native types with useless PHPDoc:

```php
/**
 * @param int $boardId
 * @return Board|null
 */
public function findActiveBoard(int $boardId): ?Board
```

Use PHPDoc to add information native PHP cannot express clearly:

```php
/**
 * @return list<BoardColumn>
 */
public function getVisibleColumns(): array
```

```php
/**
 * @param non-empty-string $slug
 * @return array{label: string, colour: string, isDefault: bool}
 */
public function getStatusConfig(string $slug): array
```

```php
/**
 * @param callable(Item): bool $predicate
 */
public function filterItems(callable $predicate): ItemCollection
```

### PHP type notation

Use phpDocumentor-compatible type syntax, and only use static-analysis-specific syntax where the project already supports PHPStan or Psalm.

Preferred examples:

```php
/** @var list<string> $codes */
```

```php
/** @var array<string, mixed> $payload */
```

```php
/** @var array<int, BoardColumn> $columnsById */
```

```php
/**
 * @return iterable<int, ItemValue>
 */
```

Use these conventions:

- `list<Type>` for zero-indexed sequential arrays.
- `array<TKey, TValue>` for associative arrays.
- `array{key: Type, other?: Type}` for shaped arrays when supported by the project tooling.
- `class-string<Foo>` where a class name string is required and tooling supports it.
- `non-empty-string`, `positive-int`, and similar refined types only when the analyser supports them.
- `mixed` only where the value is genuinely unconstrained.
- `object` only where any object is valid.
- `resource` only for actual PHP resources.
- Avoid `array` alone on public APIs unless no better type is possible.

### PHP tag order

Use this order where relevant:

1. Summary and description.
2. `@template` / generic declarations.
3. `@extends`, `@implements`, `@use`.
4. `@param`.
5. `@return`.
6. `@throws`.
7. `@var`.
8. `@property`, `@property-read`, `@property-write`.
9. `@method`.
10. `@see`.
11. `@deprecated`.
12. `@internal`.
13. Tool-specific tags such as `@phpstan-*` or `@psalm-*` if already used in the project.

Do not add tags that are irrelevant.

### Class, interface, trait, and enum documentation

Every non-trivial PHP class, interface, trait, and enum must have a DocBlock unless its purpose is completely obvious from the name and namespace.

Document:

- The responsibility of the class.
- The layer it belongs to, such as controller, application service, domain service, repository, DTO, entity, command, event, listener, or view model.
- Important invariants.
- External systems or framework conventions it depends on.
- Whether the class is intended for extension or internal use.

Example:

```php
/**
 * Imports contact responsibility data from the practice management export.
 *
 * The importer is deliberately idempotent: running the same file twice must not
 * create duplicate contacts, companies, or responsibility links. Differences are
 * detected before writes so the import log can report meaningful changes.
 */
final class ContactImportService
{
}
```

Interfaces must document the contract, not the likely implementation:

```php
/**
 * Provides read-only access to board configuration visible to the current user.
 *
 * Implementations must apply board-level permissions before returning results.
 */
interface BoardConfigurationProvider
{
}
```

Traits must document what they assume about the consuming class:

```php
/**
 * Adds audit-log helper methods to services that already expose a current user.
 *
 * Consuming classes must provide getCurrentUserId().
 */
trait AuditLogAwareTrait
{
}
```

Enums must document the business meaning if values are persisted, exposed in APIs, or mapped to UI labels:

```php
/**
 * Persisted automation trigger types.
 *
 * Backed values are stored in the database and must not be renamed without a migration.
 */
enum AutomationTrigger: string
{
    case ItemCreated = 'item_created';
    case ColumnChanged = 'column_changed';
}
```

### Method and function documentation

Document public and protected methods when:

- They are part of a service, repository, interface, controller, command, or reusable utility.
- They contain business rules.
- They accept arrays, callables, variadic values, or generic collections.
- They perform I/O, persistence, transactions, queue dispatch, cache mutation, API calls, or security checks.
- They throw meaningful exceptions.
- The return type needs additional explanation.

Private methods usually need comments only when the intent is not obvious.

Example:

```php
/**
 * Rebuilds derived item values after a column definition changes.
 *
 * This is intentionally performed inside the caller's transaction so item values
 * and item logs remain consistent if validation fails later in the update flow.
 *
 * @param list<Item> $items Items belonging to the affected board.
 *
 * @throws InvalidColumnConfigurationException If the new column settings cannot be applied safely.
 */
private function rebuildValuesForColumnChange(BoardColumn $column, array $items): void
{
}
```

### Parameter documentation

Add `@param` when at least one of these is true:

- The parameter is an array, iterable, callable, or object shape.
- The acceptable range or format is constrained.
- The value has security or permission implications.
- The value is passed to an external API.
- The native type is broader than the actual accepted domain.
- The parameter name is not enough to explain the intent.

Examples:

```php
/**
 * @param non-empty-string $apiKey Plain API key received from the request header.
 */
```

```php
/**
 * @param array{page: positive-int, perPage: positive-int, query?: string} $criteria
 */
```

```php
/**
 * @param callable(BoardColumn): bool $isVisible
 */
```

Do not add obvious comments:

```php
/**
 * @param User $user The user.
 */
```

### Return documentation

Add `@return` when:

- Returning an array, iterable, generator, or collection with useful element types.
- Returning a shape.
- Returning a scalar with a specific format or unit.
- Returning `mixed` or `object` cannot be avoided.
- Returning `self`, `static`, or fluent values where behaviour may be unclear.
- Returning `null` has important meaning beyond “not found”.

Examples:

```php
/**
 * @return list<array{id: int, label: string, selected: bool}>
 */
```

```php
/**
 * @return string ISO 8601 date-time in the user's timezone.
 */
```

```php
/**
 * @return Generator<int, ImportRow>
 */
```

### Exception documentation

Use `@throws` for exceptions the caller can reasonably handle or needs to know about.

Document:

- Validation failures.
- Permission failures.
- Persistence or transaction failures that are deliberately surfaced.
- External API failures.
- File-system failures.
- Domain exceptions.

Example:

```php
/**
 * @throws BoardPermissionException If the current user cannot edit the board.
 * @throws ColumnValidationException If the submitted value is incompatible with the column type.
 */
public function updateValue(Item $item, BoardColumn $column, mixed $value): void
{
}
```

Do not document every incidental low-level exception unless it is part of the contract.

### Property documentation

Use native property types first.

Use property DocBlocks for:

- Arrays, collections, and shapes.
- Doctrine collections.
- Values populated by magic methods or framework hydration.
- Public DTO properties where generated docs or IDE help matters.
- Read-only or write-only virtual properties.

Example:

```php
/**
 * @var Collection<int, BoardColumn>
 */
private Collection $columns;
```

```php
/**
 * @var array<string, mixed>
 */
private array $settings = [];
```

Avoid redundant property docs:

```php
/** @var string */
private string $name;
```

### Inline `@var` documentation

Use inline `@var` comments sparingly to help PhpStorm and static analysers where inference is weak.

Valid uses:

```php
/** @var BoardColumn $column */
$column = $this->columns->first();
```

```php
/** @var array{token: string, expiresAt: string} $authPayload */
$authPayload = json_decode($response->getBody()->getContents(), true, flags: JSON_THROW_ON_ERROR);
```

Avoid inline `@var` comments that fight the real type. If the type is uncertain, fix the source type where possible.

### Generics and templates

Use generics when documenting reusable collections, repositories, factories, or providers.

Example:

```php
/**
 * @template TEntity of object
 */
interface RepositoryInterface
{
    /**
     * @param class-string<TEntity> $className
     * @return TEntity|null
     */
    public function find(string $className, int $id): ?object;
}
```

If the project uses PHPStan or Psalm, tool-specific tags may be used where necessary:

```php
/**
 * @phpstan-type ColumnPayload array{id: int, type: non-empty-string, settings: array<string, mixed>}
 */
```

Do not introduce tool-specific annotations unless the project already supports that analyser.

### Magic methods and dynamic properties

Use `@property`, `@property-read`, `@property-write`, and `@method` only for deliberate dynamic APIs, such as framework models, proxies, or magic forwarding.

Example:

```php
/**
 * @property-read int $id
 * @property-read string $displayName
 * @method bool canEditBoard(Board $board)
 */
final class UserPermissionProxy
{
}
```

Do not use magic-property annotations to compensate for avoidable unclear code.

### Deprecated PHP code

Use `@deprecated` when code remains temporarily for compatibility.

Include:

- The version or date of deprecation if known.
- What to use instead.
- When removal is expected if known.

Example:

```php
/**
 * @deprecated Since 2026-06. Use BoardValueNormaliser::normalise() instead.
 */
public function normaliseLegacyValue(mixed $value): mixed
{
}
```

### Security-sensitive PHP comments

Document security decisions clearly.

Add comments where code handles:

- Authentication or authorisation.
- CSRF, CORS, session, cookie, or token behaviour.
- File uploads, downloads, or temporary storage.
- Escaping, sanitisation, SQL, HTML, JavaScript, shell commands, or path handling.
- Tenant, client, board, or row-level permissions.
- Secrets, API keys, or credentials.

Example:

```php
// Do not trust the board id in the submitted form. The item relationship is the source of truth
// because a malicious user could tamper with hidden inputs.
$board = $item->getBoard();
```

Never include actual secrets, credentials, tokens, customer data, or private identifiers in comments.

### Database and migration comments

Document database-impacting code when intent is not obvious.

For migrations:

- Explain destructive changes.
- Explain backfills.
- Explain compatibility windows.
- Explain why indexes are added.
- Explain any manual follow-up required.

Example:

```php
// Backfill existing text values before adding the NOT NULL constraint. Empty strings are used
// deliberately because legacy rows did not distinguish between “unset” and “blank”.
$this->addSql("UPDATE item_values SET value_text = '' WHERE value_text IS NULL");
```

### Framework-specific PHP comments

Add comments when framework magic is not obvious from the code.

Examples:

```php
// Symfony calls this method before validation, so normalisation must not rely on a valid entity state.
public function preSubmit(FormEvent $event): void
{
}
```

```php
// Doctrine orphanRemoval handles deleted options after flush(); do not call remove() manually here
// or the unit of work may process the same entity twice.
$column->removeOption($option);
```

### PhpStorm-specific guidance

The agent must write documentation that works well with PhpStorm.

Use PhpStorm-friendly comments for:

- PHPDoc generation from `/**` above classes, methods, functions, and properties.
- Inline `@var` hints where PhpStorm cannot infer array shapes, collection element types, or dynamic return types.
- `@see` references to related classes, methods, issues, or documentation.
- `@deprecated` to trigger IDE warnings.
- `@method` and `@property-read` for deliberate dynamic APIs.
- `@template` and shaped arrays where supported by the project tooling.

Use `@noinspection` only when all of these apply:

1. PhpStorm reports a false positive.
2. The code is intentionally correct.
3. There is no better code-level fix.
4. The suppression is placed as narrowly as possible.
5. A short reason is provided if the reason is not obvious.

Example:

```php
// noinspection PhpUnhandledExceptionInspection
// The command runner catches JsonException at the process boundary and reports it as an import failure.
$payload = json_decode($json, true, flags: JSON_THROW_ON_ERROR);
```

Do not scatter broad IDE suppressions over files or classes.

### PHP TODO and FIXME comments

Use TODO comments only for specific, actionable future work.

Format:

```php
// TODO: Replace the temporary CSV parser with the shared import normaliser after contact import v2 ships.
```

Use FIXME where there is a known defect:

```php
// FIXME: This does not handle duplicate option labels; preserve IDs before enabling option renames.
```

Rules:

- Do not write vague TODOs.
- Include issue IDs only if the project uses issue tracking.
- Do not use TODOs for work that should be completed in the current change.
- Do not leave commented-out code. Delete it or move it to version control history.

## JavaScript documentation standard

JavaScript documentation must use JSDoc-style comments for exported or reusable symbols.

Use `/** ... */` for JSDoc blocks. Use `//` for ordinary implementation comments.

### JSDoc block structure

Use this structure:

```js
/**
 * Short summary written as a complete sentence.
 *
 * Optional longer description explaining UI behaviour, event flow, API contracts,
 * browser constraints, side effects, or failure modes.
 *
 * @param {Type} name Description where useful.
 * @returns {Type} Description where useful.
 * @throws {Error} When this error is thrown.
 */
```

### JavaScript documentation targets

Document:

- Exported functions, classes, constants, and modules.
- Public methods on classes.
- Reusable utilities.
- API clients and request/response mapping.
- Event handlers with non-obvious behaviour.
- Complex DOM manipulation.
- State transitions.
- Configuration objects and object shapes.
- Callbacks and promises.
- Browser compatibility workarounds.
- Security-sensitive code involving HTML, URLs, file uploads, cross-origin requests, or user-generated content.

Private local functions need comments only when intent is not obvious.

### JavaScript types

Use JSDoc types to make PhpStorm/WebStorm-style IDE inference useful.

Examples:

```js
/**
 * @param {HTMLTableElement} table
 * @param {Array<{id: number, label: string, type: string}>} columns
 * @returns {void}
 */
export function renderBoardHeader(table, columns) {
}
```

```js
/**
 * @typedef {Object} BoardColumnConfig
 * @property {number} id Persistent column id.
 * @property {string} label Display label shown in the table header.
 * @property {'text'|'status'|'date'|'number'|'user'} type Column type key returned by the API.
 * @property {boolean} editable Whether the current user can edit values in this column.
 */
```

```js
/**
 * @callback ValueFormatter
 * @param {unknown} value Raw API value.
 * @param {BoardColumnConfig} column
 * @returns {string} Safe text for display; HTML must be escaped by the caller.
 */
```

Use these conventions:

- `{string}`, `{number}`, `{boolean}`, `{bigint}`, `{symbol}`, `{undefined}`, `{null}`, `{unknown}`, `{object}` for primitive/common values.
- `{Array<Type>}` for arrays.
- `{Object<string, Type>}` for plain keyed objects.
- `{{key: Type, other: Type}}` for inline object shapes.
- `{'a'|'b'|'c'}` for string literal unions.
- `{Promise<Type>}` for async functions.
- `{Type|null}` for nullable values.
- `{Type|undefined}` for optional values where the property may be absent.
- `{?Type}` only if the existing project style uses nullable shorthand.
- `{*}` should be avoided; use `{unknown}` where the value must be narrowed before use.

### JavaScript imports and modules

At the top of modules with significant responsibilities, add a short module comment when helpful:

```js
/**
 * Board grid rendering and interaction helpers.
 *
 * This module owns DOM updates only. It must not perform API writes directly;
 * callers should use boardApi.js so request handling remains centralised.
 */
```

Do not add file headers to every JavaScript file unless they communicate meaningful architecture or ownership.

### JavaScript functions

Document exported functions with parameters and return values unless they are trivial.

Example:

```js
/**
 * Moves selected Konva nodes by a fixed delta while preserving their relative positions.
 *
 * The function updates the stage immediately but does not persist the chart. The caller
 * is responsible for saving the changed JSON after the drag operation completes.
 *
 * @param {Array<Konva.Node>} nodes Selected nodes to move.
 * @param {{x: number, y: number}} delta Movement in stage coordinates.
 * @returns {void}
 */
export function moveSelectedNodes(nodes, delta) {
}
```

### JavaScript classes

Document classes that represent services, controllers, API clients, stores, or reusable UI components.

Example:

```js
/**
 * Coordinates inline editing for a single board grid.
 *
 * The editor owns focus management, optimistic UI state, and error display. It does
 * not decide whether a user has permission to edit; that must be supplied in the
 * column configuration from the server.
 */
export class BoardCellEditor {
}
```

### JavaScript object shapes

Use `@typedef` for repeated object shapes rather than duplicating inline types.

Example:

```js
/**
 * @typedef {Object} ApiErrorPayload
 * @property {string} message Human-readable error message.
 * @property {Object<string, Array<string>>} [fieldErrors] Validation errors keyed by field name.
 * @property {string} [traceId] Server trace id for support logs.
 */
```

Use inline shapes only for one-off simple values.

### Async JavaScript

Document promises, cancellation behaviour, retry behaviour, and loading state where relevant.

Example:

```js
/**
 * Fetches the next page of board items.
 *
 * The request can be cancelled by passing an AbortSignal. A cancelled request must
 * not display an error banner because cancellation is expected during rapid filter changes.
 *
 * @param {number} boardId
 * @param {string|null} cursor Pagination cursor from the previous response.
 * @param {AbortSignal} [signal]
 * @returns {Promise<{items: Array<BoardItem>, nextCursor: string|null}>}
 */
export async function fetchNextItems(boardId, cursor, signal) {
}
```

### DOM and browser comments

Add comments where browser behaviour is non-obvious.

Examples:

```js
// Use requestAnimationFrame so the sticky column width is measured after the browser
// has applied the latest column-resize styles.
requestAnimationFrame(syncStickyColumnWidths);
```

```js
// Use textContent rather than innerHTML because option labels can be edited by users.
label.textContent = option.label;
```

### JavaScript security comments

Document deliberate security choices.

Add comments when code handles:

- User-generated HTML or text.
- URLs and redirects.
- File uploads and downloads.
- Authentication headers or CSRF tokens.
- Cross-origin requests.
- Local storage or session storage.
- Clipboard access.
- Dynamic script/style injection.

Example:

```js
// Do not use innerHTML here. Filing names are supplied by the API and may contain
// characters that would be interpreted as markup.
fileNameCell.textContent = filing.name;
```

### JavaScript TODO and FIXME comments

Use the same standards as PHP.

Examples:

```js
// TODO: Replace this polling loop with Mercure once reconnection handling is stable.
```

```js
// FIXME: Column resize handles are not keyboard-accessible yet.
```

## HTML, Twig, and template comments

Use template comments sparingly.

For Twig:

```twig
{# The first column is sticky; keep this wrapper in sync with board-grid.css. #}
```

For HTML:

```html
<!-- The hidden input is read by the legacy PHP endpoint. Remove after the AJAX route is mandatory. -->
```

Do not leave commented-out HTML, Twig, CSS, or JavaScript in templates.

## CSS comments

Use CSS comments for layout constraints and browser workarounds only.

Example:

```css
/* Keep the first board column above cells but below modal overlays. */
.board-grid__first-column {
    z-index: 20;
}
```

Avoid comments that merely restate selectors or declarations.

## Documentation for AI-generated changes

When an AI agent changes existing code, it must:

1. Preserve accurate existing comments.
2. Remove comments that are no longer true.
3. Update PHPDoc/JSDoc signatures after changing parameters, return types, thrown exceptions, or side effects.
4. Add comments for any newly introduced non-obvious behaviour.
5. Avoid broad rewrites of unrelated comments.
6. Keep project style consistent with nearby code.
7. Never invent issue numbers, ticket references, authors, dates, or business reasons.
8. Prefer one clear explanatory comment over several small obvious comments.

## Required documentation matrix

| Code element | Documentation requirement |
| --- | --- |
| Public PHP class/interface/trait/enum | DocBlock unless completely obvious. |
| Public PHP method/function | DocBlock where contract, array types, side effects, exceptions, or business logic need explanation. |
| Private PHP method/function | Comment only when intent is non-obvious. |
| PHP property | DocBlock for arrays, collections, shapes, magic/framework values, or public DTO properties. |
| PHP array return | Use `@return` with element type or shape. |
| PHP callable parameter | Use `@param callable(...)` with signature. |
| PHP exception contract | Use `@throws` for meaningful caller-visible exceptions. |
| Exported JS function/class/constant | JSDoc unless trivial. |
| Reused JS object shape | `@typedef`. |
| JS async function | Document resolved type, cancellation, retry, and error behaviour where relevant. |
| Security-sensitive code | Explain the security decision. |
| Framework magic | Explain lifecycle assumptions or conventions. |
| Temporary workaround | Use actionable TODO/FIXME with reason. |

## Comment anti-patterns to avoid

Do not write:

```php
// Increment i.
$i++;
```

```php
/**
 * Gets the name.
 */
public function getName(): string
```

```js
// Create variable.
const result = calculateTotal(items);
```

```js
/**
 * Function to handle click.
 */
function handleClick(event) {
}
```

```php
// This is complicated.
```

```php
// TODO: fix later.
```

```js
// Old code kept in case needed.
// api.save(oldPayload);
```

## Preferred examples

### PHP service example

```php
/**
 * Applies a submitted value to an item column after checking board permissions.
 *
 * The method normalises the value according to the column type before persistence.
 * It also writes an item log entry when the normalised value differs from the
 * previous value. Permission checks are performed here rather than in the
 * controller so API and form submissions share the same enforcement point.
 *
 * @throws BoardPermissionException If the current user cannot edit this column.
 * @throws ColumnValidationException If the submitted value is invalid for the column type.
 */
public function updateColumnValue(User $user, Item $item, BoardColumn $column, mixed $submittedValue): ItemValue
{
}
```

### PHP repository example

```php
/**
 * Finds active boards visible to the user, ordered by category and board name.
 *
 * Admin users can see all boards. Non-admin users only receive boards where they
 * have an explicit board permission record.
 *
 * @return list<Board>
 */
public function findVisibleBoardsForUser(User $user): array
{
}
```

### PHP array-shape example

```php
/**
 * @return array{
 *     id: int,
 *     label: string,
 *     type: non-empty-string,
 *     settings: array<string, mixed>,
 *     editable: bool
 * }
 */
public function toClientPayload(BoardColumn $column, User $user): array
{
}
```

### JavaScript API client example

```js
/**
 * Saves a single board cell value.
 *
 * Validation errors are returned by the server as field errors and should be
 * rendered beside the edited cell. Network errors are re-thrown so the caller can
 * display a board-level error banner.
 *
 * @param {number} itemId
 * @param {number} columnId
 * @param {unknown} value Raw UI value to submit.
 * @returns {Promise<{valueHtml: string, updatedAt: string}>}
 * @throws {Error} If the request fails before a validation response is received.
 */
export async function saveBoardCellValue(itemId, columnId, value) {
}
```

### JavaScript UI example

```js
/**
 * Opens the filter modal with a cloned copy of the current view filters.
 *
 * The clone prevents accidental mutation of the live board view while the user is
 * editing modal controls. Changes are applied only when the user confirms.
 *
 * @param {BoardViewState} viewState
 * @returns {void}
 */
export function openFilterModal(viewState) {
}
```

## Review checklist

Before returning code, the AI agent must check:

- Are all public contracts documented where necessary?
- Are native PHP types used before PHPDoc types?
- Are array shapes, lists, collections, callables, and iterables clear?
- Are thrown exceptions documented where callers need to know about them?
- Are side effects documented?
- Are security-sensitive choices explained?
- Are framework lifecycle assumptions explained?
- Are comments concise and useful?
- Did any comment become false after the code change?
- Are TODO/FIXME comments specific and actionable?
- Are PhpStorm and static-analysis hints helpful rather than noisy?
- Are there any commented-out blocks of dead code that should be deleted?

## Output expectations for AI agents

When generating code, the agent must:

1. Include PHPDoc/JSDoc directly in the generated code where required.
2. Avoid adding large prose explanations outside the code unless the user asked for them.
3. Explain any significant documentation decisions briefly after the code.
4. State any assumptions that affect the generated documentation.
5. Flag places where project-specific types, issue references, or analyser tags may need adjustment.

## Minimal documentation principle

More documentation is not automatically better. The target is accurate, useful, maintainable documentation.

Write the smallest amount of documentation that preserves the code's contract, intent, and important constraints.
