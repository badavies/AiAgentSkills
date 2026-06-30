---
name: JS-Agent-Skill
description: This skill governs how an AI coding agent must generate, edit, review, document, and test JavaScript code for browser-based web applications.
---

# JavaScript Agent Skill

## Purpose

This skill governs how an AI coding agent must generate, edit, review, document, and test JavaScript for browser-based web applications.

The JavaScript produced by the agent must be:

1. Secure by default.
2. Testable.
3. Maintainable.
4. DRY.
5. KISS.
6. Compatible with modern browser standards unless the project specifies otherwise.
7. Separated from HTML and CSS.
8. Clear enough for PhpStorm and modern IDEs to analyse.

This skill applies to plain JavaScript, ES modules, JavaScript used with PHP/Twig-rendered pages, DOM interaction scripts, AJAX/fetch code, and client-side components.

---

## Core Rule

JavaScript provides behaviour. It must not be embedded in HTML by default.

Do not use inline event handlers:

```html
<button onclick="saveChart()">Save</button>
```

Use external JavaScript:

```html
<button type="button" data-action="save-chart">Save</button>
<script src="/assets/js/chart-page.js" defer></script>
```

```js
document
  .querySelector('[data-action="save-chart"]')
  ?.addEventListener("click", handleSaveChart);
```

---

## Source Standards Reflected

The agent must follow current MDN JavaScript code-style guidance, including:

- Use a formatter such as Prettier.
- Use modern JavaScript features when supported by the target browsers.
- Use array literals rather than `new Array()`.
- Use `push()` rather than assigning to `array[array.length]`.
- Prefer `async`/`await` where it simplifies asynchronous code.
- Use comments to explain intent, not to restate obvious code.
- Prefer single-line `//` comments for normal implementation comments.
- Use camelCase for function and variable names.
- Prefer function declarations for named functions.
- Use arrow functions for short callbacks where `this` is not needed.
- Prefer `for...of` or array iteration methods for collections.
- Use template literals for string interpolation.
- Use `const` by default and `let` only where reassignment is required.
- Do not use `var`.
- Declare one variable per line.
- Avoid implicit type coercion.
- Avoid deprecated APIs.
- Use `fetch()` rather than `XMLHttpRequest`.
- Use `textContent` rather than `innerHTML` for plain text insertion.
- Use appropriate logging methods, such as `console.error()` for errors.

---

## File Organisation

Use external JavaScript files.

Recommended structure:

```text
assets/
  js/
    app.js
    modules/
      api-client.js
      dom.js
      validators.js
    pages/
      chart-page.js
      filings-page.js
    components/
      modal.js
      dropdown.js
      file-upload.js
    tests/
      validators.test.js
      api-client.test.js
```

For small projects, keep the structure simple. Do not over-engineer.

Each file should have a clear purpose.

---

## Module Design

Prefer ES modules where the project supports them.

```js
// assets/js/modules/validators.js
export function isNonEmptyString(value) {
  return typeof value === "string" && value.trim() !== "";
}
```

```js
// assets/js/pages/chart-page.js
import { isNonEmptyString } from "../modules/validators.js";
```

If the project does not use modules, wrap code to avoid global pollution:

```js
(() => {
  function initialisePage() {
    // ...
  }

  document.addEventListener("DOMContentLoaded", initialisePage);
})();
```

Do not create unnecessary globals. If a global is required for legacy integration, document it clearly.

---

## Naming

Use clear, semantic names.

### Functions

Use camelCase and verb-based names for actions.

Good:

```js
function saveChart() {}
function validateEmailAddress() {}
function createEntityCard() {}
```

Bad:

```js
function SaveChart() {}
function doIt() {}
function process() {}
```

### Variables

Use camelCase. Prefer names that describe the value’s meaning.

Good:

```js
const clientReference = "ABC123";
const selectedEntities = [];
```

Bad:

```js
const sClientReference = "ABC123";
const arr = [];
const data1 = {};
```

For collections, use plural names:

```js
const companies = [];
const selectedEntityIds = [];
```

Avoid Hungarian notation and type prefixes.

### Constants

For true constants, use screaming snake case:

```js
const MAX_FILE_SIZE_BYTES = 10 * 1024 * 1024;
```

For ordinary values that are not reassigned, use `const` with normal camelCase:

```js
const selectedEntity = getSelectedEntity();
```

---

## Variables and Declarations

Use `const` by default.

```js
const endpoint = "/api/charts";
```

Use `let` only where reassignment is required.

```js
let retryCount = 0;
retryCount += 1;
```

Never use `var`.

Declare one variable per line.

Good:

```js
const chartId = getChartId();
const endpoint = `/api/charts/${chartId}`;
```

Bad:

```js
const chartId = getChartId(), endpoint = `/api/charts/${chartId}`;
```

---

## Functions

Prefer small functions that do one thing.

Good:

```js
function normaliseCompanyNumber(value) {
  return value.trim().toUpperCase();
}
```

Avoid functions that mix validation, DOM updates, network calls, and state changes in one place.

### Function Declarations

Prefer function declarations for named functions:

```js
function calculateTotal(values) {
  return values.reduce((total, value) => total + value, 0);
}
```

Avoid assigning named functions to variables unless there is a specific reason:

```js
const calculateTotal = (values) => {
  return values.reduce((total, value) => total + value, 0);
};
```

### Arrow Functions

Use arrow functions for concise callbacks where `this` is not needed:

```js
const selectedIds = entities.map((entity) => entity.id);
```

Do not use arrow functions for object methods or code that relies on `this`.

---

## DRY and KISS

### DRY

Do not duplicate business logic, selectors, validation rules, endpoint construction, or DOM update patterns.

Bad:

```js
document.querySelector("#name").value.trim();
document.querySelector("#email").value.trim();
document.querySelector("#phone").value.trim();
```

Better:

```js
function getInputValue(form, fieldName) {
  return form.elements[fieldName]?.value.trim() ?? "";
}
```

### KISS

Prefer straightforward code over clever abstractions.

Do not introduce classes, factories, event buses, or complex state managers for simple behaviours.

Good:

```js
function togglePanel(button, panel) {
  const isExpanded = button.getAttribute("aria-expanded") === "true";

  button.setAttribute("aria-expanded", String(!isExpanded));
  panel.hidden = isExpanded;
}
```

Avoid excessive indirection unless it improves clarity or testability.

---

## Testability Requirements

JavaScript must be written so important logic can be tested without a browser where practical.

### Separate Pure Logic from DOM Code

Good:

```js
export function calculateVat(amount, rate) {
  return amount * rate;
}
```

DOM integration:

```js
import { calculateVat } from "./tax.js";

function updateVatOutput(form, output) {
  const amount = Number(form.elements.amount.value);
  const vat = calculateVat(amount, 0.2);

  output.textContent = vat.toFixed(2);
}
```

The calculation can be unit-tested independently.

### Dependency Injection

Do not hard-code dependencies where passing them makes testing easier.

Good:

```js
export async function loadChart(fetchJson, chartId) {
  return fetchJson(`/api/charts/${encodeURIComponent(chartId)}`);
}
```

Less testable:

```js
export async function loadChart(chartId) {
  const response = await fetch(`/api/charts/${chartId}`);
  return response.json();
}
```

### Return Values

Functions should return useful values rather than only mutating hidden state.

### Avoid Hidden Globals

Do not rely on global mutable state unless it is explicitly part of the design.

### Suggested Test Types

For generated JavaScript, the agent should recommend tests where appropriate:

- Unit tests for pure functions.
- Integration tests for API wrappers.
- DOM tests for component behaviour.
- End-to-end tests only for critical flows.

Test examples may use Vitest, Jest, Playwright, or the project’s existing test runner. Do not introduce a new test framework without reason.

Example unit test:

```js
import { describe, expect, it } from "vitest";
import { normaliseCompanyNumber } from "./company-number.js";

describe("normaliseCompanyNumber", () => {
  it("trims and uppercases company numbers", () => {
    expect(normaliseCompanyNumber(" abc123 ")).toBe("ABC123");
  });
});
```

---

## DOM Interaction

Centralise DOM lookups where possible.

Use `data-*` attributes for JavaScript hooks:

```html
<button type="button" data-action="delete-entity" data-entity-id="123">
  Delete
</button>
```

```js
const deleteButton = document.querySelector('[data-action="delete-entity"]');
```

Do not depend on visual styling classes for JavaScript selectors where a dedicated data attribute is clearer.

### Event Listeners

Use `addEventListener`.

Do not use inline event attributes.

Prefer event delegation for repeated dynamic elements:

```js
document.addEventListener("click", (event) => {
  const button = event.target.closest('[data-action="delete-entity"]');

  if (!button) {
    return;
  }

  handleDeleteEntity(button);
});
```

### DOM Creation

Use DOM APIs for dynamic content.

Good:

```js
function createErrorMessage(message) {
  const paragraph = document.createElement("p");
  paragraph.classList.add("form-error");
  paragraph.textContent = message;

  return paragraph;
}
```

Avoid:

```js
function createErrorMessage(message) {
  return `<p class="form-error">${message}</p>`;
}
```

---

## Security Requirements

Security is a primary requirement.

### HTML Injection and XSS

Do not use `innerHTML` for plain text.

Bad:

```js
messageContainer.innerHTML = userSuppliedMessage;
```

Good:

```js
messageContainer.textContent = userSuppliedMessage;
```

Only use `innerHTML`, `insertAdjacentHTML`, or template HTML when:

1. The HTML is static and controlled by the application; or
2. The content has been sanitised by a vetted sanitisation library; and
3. The reason is documented; and
4. There is no safer DOM API alternative.

### Dangerous APIs

Do not use:

- `eval()`
- `new Function()`
- string-based `setTimeout()` or `setInterval()`
- inline event handlers
- unsafe `innerHTML`
- `document.write()`
- `javascript:` URLs
- unvalidated dynamic import paths
- untrusted CSS selectors without escaping
- untrusted URL construction without validation

### URLs

Validate and encode URL components.

```js
const url = `/api/charts/${encodeURIComponent(chartId)}`;
```

For query strings, use `URLSearchParams`:

```js
const params = new URLSearchParams({
  status: "open",
  page: String(page),
});

const url = `/api/jobs?${params.toString()}`;
```

### Fetch and CSRF

When making mutating requests in a web app:

- Include the project’s CSRF token where required.
- Use appropriate HTTP methods.
- Set `Content-Type: application/json` for JSON bodies.
- Handle non-2xx responses.
- Avoid leaking sensitive data in URLs.
- Do not store secrets in JavaScript.

Example:

```js
async function postJson(url, payload, csrfToken) {
  const response = await fetch(url, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-CSRF-Token": csrfToken,
    },
    credentials: "same-origin",
    body: JSON.stringify(payload),
  });

  if (!response.ok) {
    throw new Error(`Request failed with status ${response.status}`);
  }

  return response.json();
}
```

### Data Attributes

Treat values read from the DOM as untrusted. Validate and parse before use.

```js
function getPositiveIntegerAttribute(element, name) {
  const value = Number(element.dataset[name]);

  if (!Number.isInteger(value) || value <= 0) {
    throw new Error(`Invalid ${name} value`);
  }

  return value;
}
```

### Logging

Do not log secrets, access tokens, personal data, tax data, client data, or large confidential payloads.

Use `console.error()` for errors during development, but avoid noisy production logs unless the project has a logging policy.

---

## Asynchronous Code

Prefer `async`/`await` when it improves readability.

```js
async function loadEntities(apiClient) {
  try {
    const entities = await apiClient.getEntities();
    renderEntities(entities);
  } catch (error) {
    showError("Unable to load entities.");
    console.error(error);
  }
}
```

Always handle errors.

Do not leave unhandled promises:

```js
saveChart().catch((error) => {
  console.error(error);
  showError("Unable to save chart.");
});
```

Avoid top-level `await` unless the script is an ES module and the project supports it.

---

## Arrays and Objects

Use array literals.

Good:

```js
const selectedEntities = [];
```

Bad:

```js
const selectedEntities = new Array();
```

Use `push()` to append:

```js
selectedEntities.push(entity);
```

Use object literals:

```js
const entity = {
  id: 123,
  name: "Acme Ltd",
};
```

Use destructuring where it improves clarity:

```js
const { id, name } = entity;
```

Do not destructure so aggressively that code becomes harder to read.

---

## Loops and Collection Methods

Use the clearest construct for the task.

- Use `for...of` for simple iteration.
- Use `.map()` for transformation.
- Use `.filter()` for filtering.
- Use `.reduce()` only when it remains readable.
- Use `.some()` and `.every()` for boolean checks.
- Use `.find()` for finding one matching item.

Good:

```js
for (const entity of entities) {
  renderEntity(entity);
}
```

Avoid classical `for` loops unless index control is required.

---

## Strings

Use template literals for interpolation.

Good:

```js
const url = `/api/charts/${encodeURIComponent(chartId)}`;
```

Bad:

```js
const url = "/api/charts/" + chartId;
```

Do not overuse template literals when a normal string is enough.

---

## Type Conversion and Validation

Avoid implicit type coercion.

Good:

```js
const amount = Number(input.value);
```

Bad:

```js
const amount = +input.value;
```

Validate parsed values:

```js
function parseRequiredNumber(value, fieldName) {
  const number = Number(value);

  if (!Number.isFinite(number)) {
    throw new Error(`${fieldName} must be a valid number`);
  }

  return number;
}
```

---

## Error Handling

Errors should be actionable.

- Throw errors for invalid developer-facing states.
- Show user-friendly messages for user-facing failures.
- Do not expose sensitive technical details to users.
- Preserve the original error for logging where appropriate.

```js
try {
  await saveChart();
  showSuccess("Chart saved.");
} catch (error) {
  console.error(error);
  showError("The chart could not be saved. Please try again.");
}
```

---

## Accessibility in JavaScript

JavaScript must preserve or improve accessibility.

When changing visibility:

- Update `hidden`.
- Update `aria-expanded`.
- Manage focus where appropriate.
- Do not trap keyboard focus unless implementing a modal/dialog pattern correctly.
- Restore focus after closing modal dialogs.
- Ensure dynamic status messages use appropriate live regions.

Example:

```js
function toggleDisclosure(button, panel) {
  const isExpanded = button.getAttribute("aria-expanded") === "true";

  button.setAttribute("aria-expanded", String(!isExpanded));
  panel.hidden = isExpanded;
}
```

For status updates:

```html
<p data-status-message role="status" aria-live="polite"></p>
```

```js
statusMessage.textContent = "Chart saved.";
```

---

## Performance

The agent must avoid unnecessary work.

- Cache DOM references when reused.
- Avoid repeated layout reads/writes in tight loops.
- Use event delegation for many similar controls.
- Debounce or throttle high-frequency events.
- Use `requestAnimationFrame` for visual updates where appropriate.
- Avoid memory leaks by removing listeners when components are destroyed.
- Avoid large synchronous work on the main thread.
- Avoid unnecessary dependencies.

Do not prematurely optimise at the cost of clarity.

---

## Comments and Documentation

Comments should explain why, not what.

Good:

```js
// Preserve focus so keyboard users return to the control that opened the modal
previouslyFocusedElement?.focus();
```

Bad:

```js
// Set focus
previouslyFocusedElement?.focus();
```

Use `//` for normal comments. Use JSDoc for exported functions, public APIs, complex parameters, and IDE assistance.

Example:

```js
/**
 * Formats a company number for display.
 *
 * @param {string} companyNumber Raw company number.
 * @returns {string} Uppercase company number without surrounding whitespace.
 */
export function formatCompanyNumber(companyNumber) {
  return companyNumber.trim().toUpperCase();
}
```

Do not over-comment obvious private code.

---

## JSDoc and IDE Support

Use JSDoc where it improves PhpStorm/static analysis support.

Document:

- Exported functions.
- Complex object shapes.
- Callback signatures.
- Public module APIs.
- Non-obvious return values.
- Configuration objects.

Example object typedef:

```js
/**
 * @typedef {object} ChartEntity
 * @property {number} id Unique entity ID.
 * @property {string} name Display name.
 * @property {"company"|"person"|"trust"} type Entity type.
 * @property {number} x X-coordinate on canvas.
 * @property {number} y Y-coordinate on canvas.
 */
```

Function using the typedef:

```js
/**
 * Finds an entity by ID.
 *
 * @param {ChartEntity[]} entities Available entities.
 * @param {number} entityId Entity ID to find.
 * @returns {ChartEntity|null} Matching entity, or null when not found.
 */
export function findEntityById(entities, entityId) {
  return entities.find((entity) => entity.id === entityId) ?? null;
}
```

Keep JSDoc accurate. Incorrect documentation is worse than no documentation.

---

## Browser APIs

Prefer modern APIs:

- `fetch()` over `XMLHttpRequest`.
- `classList` over string class manipulation.
- `dataset` for simple data attributes.
- `URL` and `URLSearchParams` for URL construction.
- `AbortController` for cancellable fetch requests.
- `addEventListener` over event properties.
- `textContent` over `innerHTML` for text.
- `closest()` for event delegation.

Avoid deprecated APIs.

---

## Progressive Enhancement

Where practical:

- Let forms submit without JavaScript or provide a graceful fallback.
- Use JavaScript to enhance native controls.
- Preserve standard browser behaviours.
- Do not break back/forward navigation without reason.
- Avoid requiring JavaScript for content that can be server-rendered.

---

## Framework-Neutral Rules

Unless the user specifies a framework, do not introduce one.

Do not add React, Vue, Alpine, jQuery, Stimulus, or other libraries unless:

1. The project already uses it; or
2. The user explicitly asks; or
3. The complexity genuinely justifies it and the agent explains why.

For existing jQuery projects, do not rewrite everything to vanilla JavaScript unless requested. For new vanilla JavaScript code, do not introduce jQuery.

---

## Agent Output Rules

When generating JavaScript, the agent must:

1. Keep JavaScript in external files or modules.
2. Avoid inline event handlers.
3. Use `const` by default and `let` only for reassignment.
4. Never use `var`.
5. Use function declarations for named functions.
6. Use arrow functions for short callbacks where suitable.
7. Keep functions small and focused.
8. Separate pure logic from DOM code.
9. Make important logic unit-testable.
10. Validate untrusted data.
11. Use safe DOM APIs.
12. Avoid `innerHTML` unless controlled/sanitised and justified.
13. Avoid dangerous APIs such as `eval`.
14. Handle asynchronous errors.
15. Use accessible state updates.
16. Avoid leaking secrets or personal/client data into logs.
17. Match the project’s established style where known.

---

## Review Checklist

Before returning JavaScript, the agent must verify:

- [ ] No inline event handlers are required.
- [ ] Code can run from an external file or module.
- [ ] No unnecessary globals are introduced.
- [ ] `const` and `let` are used correctly.
- [ ] `var` is not used.
- [ ] Functions are named clearly.
- [ ] Functions are small and focused.
- [ ] Repeated logic has been extracted.
- [ ] The solution is not over-engineered.
- [ ] Pure logic is separated from DOM operations where practical.
- [ ] Important logic is testable.
- [ ] DOM insertion uses `textContent` or safe DOM APIs for text.
- [ ] No unsafe `innerHTML`, `eval`, `new Function`, or `document.write` is used.
- [ ] URLs are encoded and validated.
- [ ] Fetch calls handle non-2xx responses.
- [ ] Errors are handled.
- [ ] Accessibility states are updated correctly.
- [ ] Event listeners are attached safely.
- [ ] Secrets and confidential data are not logged.
- [ ] JSDoc is used where it helps IDE/static analysis support.
- [ ] The code is formatted consistently.

---

## Correction Rules

When modifying existing JavaScript, the agent should improve the touched area by:

- Removing inline event handlers if feasible.
- Extracting repeated code into small functions.
- Replacing unsafe DOM insertion with safe APIs.
- Replacing `var` with `const` or `let`.
- Replacing XHR with `fetch()` where appropriate.
- Adding error handling to asynchronous code.
- Improving testability by separating pure logic.
- Adding JSDoc for exported or complex functions.
- Preserving existing project conventions unless unsafe.

If the user asks for a quick patch, keep the scope narrow but do not introduce avoidable security vulnerabilities.

---

## References

- MDN JavaScript code examples style guide.
- MDN dynamic scripting guidance.
- General secure JavaScript and testable frontend architecture practice.
