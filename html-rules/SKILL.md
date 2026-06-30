---
name: HTML-Agent-Skill
description: This skill governs how an AI coding agent must generate, edit, review, and document HTML code. It is intended for maintainable web application code where HTML provides semantic structure, CSS provides presentation, and JavaScript provides behaviour.
---

# HTML Agent Skill

## Purpose

This skill governs how an AI coding agent must generate, edit, review, and document HTML. It is intended for maintainable web application code where HTML provides semantic structure, CSS provides presentation, and JavaScript provides behaviour.

The agent must prioritise:

1. Semantic, accessible HTML.
2. Separation of concerns.
3. Minimal inline CSS and minimal inline JavaScript.
4. Secure rendering of untrusted or dynamic content.
5. Maintainable templates that are easy to style, test, and refactor.
6. Compatibility with IDE tooling, linters, validators, and code review.

This skill applies to standalone HTML, PHP-rendered templates, Twig templates, partials, components, modals, forms, and markup generated for use with JavaScript.

---

## Core Rule

HTML must describe document structure and meaning. It must not be used as the primary place for styling or behaviour.

The agent must not place CSS or JavaScript directly in HTML unless there is a specific, documented reason.

Preferred structure:

```html
<!doctype html>
<html lang="en-GB">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Page title</title>
    <link rel="stylesheet" href="/assets/css/app.css">
    <script src="/assets/js/app.js" defer></script>
  </head>
  <body>
    <main>
      <!-- Page content -->
    </main>
  </body>
</html>
```

---

## Source Standards Reflected

The agent must follow current mainstream HTML standards and the following conventions:

- Use HTML5 doctype: `<!doctype html>`.
- Set a meaningful document language with `lang`, usually `en-GB` for UK projects unless instructed otherwise.
- Include `<meta charset="utf-8">`.
- Include a mobile viewport meta tag.
- Use lowercase element names and lowercase attribute names.
- Quote all attribute values with double quotes.
- Use semantic class and ID names in kebab-case.
- Do not include unnecessary values for boolean attributes.
- Prefer literal characters where safe and valid, rather than unnecessary character references.

Examples:

```html
<input type="email" name="email" id="email" required>
```

Do not write:

```html
<input TYPE=email required="required">
```

---

## Separation of Concerns

### CSS

Do not use inline styles.

Avoid:

```html
<div style="display: flex; gap: 1rem; color: red;">
  ...
</div>
```

Prefer:

```html
<div class="notice notice--error">
  ...
</div>
```

CSS belongs in an external stylesheet or a framework/component stylesheet.

Inline `style` attributes are permitted only when all of the following are true:

1. The value is genuinely dynamic and cannot sensibly be represented by a class or CSS custom property.
2. The value is generated from trusted, validated data.
3. The reason is documented in a nearby comment or implementation note.
4. The value cannot introduce CSS injection or layout breakage.

Even then, prefer CSS custom properties:

```html
<div class="progress" style="--progress-value: 72%;">
  ...
</div>
```

Only use the above pattern for controlled values. Validate and constrain dynamic values before output.

### JavaScript

Do not use inline event handlers.

Avoid:

```html
<button onclick="saveItem()">Save</button>
```

Prefer:

```html
<button type="button" class="button" data-action="save-item">
  Save
</button>
```

Then attach behaviour in JavaScript:

```js
document
  .querySelector('[data-action="save-item"]')
  ?.addEventListener("click", handleSaveItem);
```

Do not use inline `<script>` blocks for application logic. Use external JavaScript files loaded with `defer` or ES modules.

Acceptable exceptions:

- Small server-rendered configuration blocks where no safer architecture is practical.
- JSON data islands using `<script type="application/json">`.
- One-off prototype code explicitly marked as throwaway.

For server data, prefer data attributes or JSON script tags over executable inline JavaScript:

```html
<section
  class="chart"
  data-chart-id="679"
  data-chart-endpoint="/api/charts/679"
>
</section>

<script type="application/json" id="chart-config">
  {
    "readOnly": false,
    "maxEntities": 500
  }
</script>
```

The JavaScript must parse and validate the data before use.

---

## Semantic HTML

Use the most meaningful element available.

Preferred examples:

- Use `<main>` for the main page content.
- Use `<header>` for introductory/header content.
- Use `<nav>` for navigation.
- Use `<section>` for a thematic section with a heading.
- Use `<article>` for standalone content.
- Use `<aside>` for tangential/supporting content.
- Use `<button>` for actions.
- Use `<a>` for navigation.
- Use lists (`<ul>`, `<ol>`) for lists.
- Use tables only for tabular data, not for layout.
- Use headings in a logical hierarchy.

Avoid generic `<div>` and `<span>` elements where a semantic element would be more accurate.

Bad:

```html
<div class="button" onclick="submitForm()">Submit</div>
```

Good:

```html
<button type="submit" class="button button--primary">Submit</button>
```

---

## Accessibility Requirements

The agent must build accessibility into the markup rather than adding it as an afterthought.

### General Accessibility

- Every page must have one primary `<main>` landmark.
- Heading levels must be logical and must not skip levels for visual reasons.
- Interactive controls must be keyboard-accessible.
- Do not create clickable `<div>` or `<span>` elements.
- Do not remove focus outlines unless replacing them with an equally visible focus style.
- Use visible text labels where possible.
- Use ARIA only where native HTML is insufficient.
- Do not use ARIA to override correct native semantics.
- Do not add `tabindex` unless necessary.
- Avoid positive `tabindex` values.
- Provide meaningful link text.
- Do not rely on colour alone to communicate meaning.

### Forms

Every form control must have an associated label.

Preferred:

```html
<label for="client-reference">Client reference</label>
<input id="client-reference" name="clientReference" type="text" autocomplete="off">
```

For helper and error text:

```html
<label for="email">Email address</label>
<input
  id="email"
  name="email"
  type="email"
  autocomplete="email"
  aria-describedby="email-help email-error"
>
<p id="email-help" class="form-help">Use the client’s primary email address.</p>
<p id="email-error" class="form-error" hidden>Please enter a valid email address.</p>
```

When an error is visible, use appropriate state:

```html
<input
  id="email"
  name="email"
  type="email"
  aria-invalid="true"
  aria-describedby="email-error"
>
```

### Images

Every image must have an `alt` attribute.

- Informative images need useful alternative text.
- Decorative images must use `alt=""`.
- Do not start alternative text with “image of” or “picture of” unless that distinction matters.

```html
<img src="/assets/img/company-chart.png" alt="Corporate ownership chart for the Smith group">
```

Decorative:

```html
<img src="/assets/img/divider.svg" alt="" aria-hidden="true">
```

### Icons

Icon-only buttons must have accessible names.

```html
<button type="button" class="icon-button" aria-label="Delete entity">
  <svg aria-hidden="true" focusable="false">...</svg>
</button>
```

### Tables

Use tables only for tabular data.

- Use `<caption>` where useful.
- Use `<th>` for headers.
- Use `scope="col"` or `scope="row"` where appropriate.
- Do not use tables for layout.

```html
<table class="data-table">
  <caption>Share classes</caption>
  <thead>
    <tr>
      <th scope="col">Class</th>
      <th scope="col">Nominal value</th>
      <th scope="col">Voting rights</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Ordinary</th>
      <td>£1.00</td>
      <td>One vote per share</td>
    </tr>
  </tbody>
</table>
```

---

## Security Requirements

HTML generation is security-sensitive. The agent must assume that any value from a database, API, URL, form, upload, email, OCR output, or external register may be untrusted.

### Escaping

- Escape output according to context.
- HTML text, HTML attributes, URLs, JavaScript, and CSS each have different escaping rules.
- Do not concatenate raw user input into HTML.
- In server-rendered templates, use the framework’s escaping mechanism.
- Do not mark values as raw/safe unless they have been sanitised and the reason is documented.

### Dynamic Attributes

Validate dynamic values before placing them in attributes.

Do not allow untrusted values in:

- `href`
- `src`
- `srcdoc`
- `style`
- event handler attributes
- `target`
- `formaction`
- `data-*` that will later be treated as executable input

For links created from external data:

```html
<a href="https://example.com" rel="noopener noreferrer">External site</a>
```

Use `rel="noopener noreferrer"` for links with `target="_blank"`:

```html
<a href="https://example.com" target="_blank" rel="noopener noreferrer">
  Open external site
</a>
```

### Inline Script and CSP

Avoid inline scripts and inline event handlers because they make a strong Content Security Policy harder to implement.

Prefer:

```html
<script src="/assets/js/page.js" defer></script>
```

Avoid:

```html
<script>
  initialisePage();
</script>
```

### HTML Injection

Do not insert user-controlled HTML unless using a vetted sanitisation library and an explicit allowlist.

User-generated or OCR-generated text must be rendered as text, not HTML.

---

## Attributes and Naming

### Classes

Use semantic, stable class names.

Recommended:

```html
<section class="client-card client-card--archived">
```

Avoid presentation-only class names unless they are framework utility classes used consistently:

```html
<section class="red-box big-left-thing">
```

Class names must be kebab-case.

Good:

```html
<div class="entity-sidebar">
```

Bad:

```html
<div class="entitySidebar">
```

### IDs

Use IDs only for:

- Form label associations.
- ARIA references.
- Fragment targets.
- Unique JavaScript hooks where necessary.
- Unique document landmarks.

Do not use IDs as primary CSS selectors.

### Data Attributes

Use `data-*` attributes for JavaScript hooks and small amounts of non-sensitive configuration.

Good:

```html
<button type="button" data-action="delete-entity" data-entity-id="123">
  Delete
</button>
```

Rules:

- Do not store secrets in data attributes.
- Do not store large JSON payloads in data attributes.
- Do not treat data attributes as trusted just because they came from the DOM.
- Use kebab-case after `data-`.

---

## Forms and Buttons

### Buttons

Every button must have an explicit `type`.

```html
<button type="button">Cancel</button>
<button type="submit">Save</button>
```

Do not rely on the browser’s default button type.

### Inputs

Use the most appropriate input type.

Examples:

- `type="email"` for email addresses.
- `type="tel"` for telephone numbers.
- `type="url"` for URLs.
- `type="number"` only where numeric spinner behaviour is appropriate.
- `inputmode` for mobile keyboard hints.
- `autocomplete` where appropriate.

### Validation

Client-side validation is a usability enhancement only. Server-side validation remains mandatory.

Use HTML validation attributes where helpful:

```html
<input
  id="share-count"
  name="shareCount"
  type="number"
  min="0"
  step="1"
  required
>
```

---

## Template and Component Rules

When generating HTML inside PHP/Twig or another server template:

1. Keep markup readable.
2. Avoid deeply nested conditionals.
3. Extract repeated blocks into partials/components.
4. Escape output by default.
5. Keep business logic out of templates.
6. Keep loops small and obvious.
7. Use meaningful variable names.
8. Avoid generating different structures for minor visual variations; prefer modifier classes.

Bad:

```twig
<div class="{% if error %}red big bold{% else %}green{% endif %}">
```

Good:

```twig
<div class="notice {{ error ? 'notice--error' : 'notice--success' }}">
```

---

## Progressive Enhancement

HTML should remain understandable and minimally functional without JavaScript wherever practical.

- Use real forms for form submissions unless an SPA pattern is explicitly required.
- Use links for navigation.
- Add JavaScript to enhance behaviour, not to replace core semantics unnecessarily.
- Do not hide critical content behind JavaScript-only rendering unless that is a deliberate application architecture decision.

---

## Performance

- Use semantic, concise markup.
- Avoid unnecessary wrapper elements.
- Use lazy loading for non-critical images where appropriate:

```html
<img src="/assets/img/report-preview.png" alt="Report preview" loading="lazy">
```

- Provide dimensions for images where known to reduce layout shift:

```html
<img src="/assets/img/logo.svg" alt="Company logo" width="160" height="48">
```

- Load scripts with `defer` unless a module or specific loading strategy is required.
- Avoid blocking scripts in the `<head>`.
- Avoid large inline SVG or embedded base64 assets unless justified.

---

## Comments in HTML

HTML comments should be rare and useful.

Use comments to explain:

- Non-obvious template boundaries.
- Accessibility decisions.
- Security-sensitive escaping/sanitisation decisions.
- Integration hooks required by external systems.

Do not use comments to restate obvious markup.

Good:

```html
<!-- Required by legacy import script; do not rename data-contact-ref without updating import-map.js. -->
<tr data-contact-ref="ABC123">
```

Bad:

```html
<!-- Button -->
<button type="button">Save</button>
```

Remove commented-out markup before returning code unless the user specifically asks to keep it.

---

## Agent Output Rules

When generating HTML, the agent must:

1. Prefer complete, valid, copy-pasteable snippets.
2. Include linked CSS and JS file references where relevant.
3. Avoid inline CSS and inline JavaScript.
4. Include accessibility attributes where needed.
5. Include secure defaults for external links, dynamic content, forms, and data attributes.
6. Use semantic elements before generic containers.
7. Keep indentation consistent.
8. Avoid placeholder text that looks final.
9. State any assumptions outside the code block, not as noisy comments in the code.
10. Provide separate HTML, CSS, and JS blocks or files where a feature requires all three.

---

## Review Checklist

Before returning HTML, the agent must verify:

- [ ] The markup uses `<!doctype html>` for complete documents.
- [ ] The `<html>` element has a correct `lang` attribute.
- [ ] UTF-8 charset is declared.
- [ ] Viewport meta tag is included for complete pages.
- [ ] Attribute values are quoted.
- [ ] Elements and attributes are lowercase.
- [ ] CSS is external or class-based, not inline.
- [ ] JavaScript behaviour is external, not inline.
- [ ] No inline event handlers are used.
- [ ] Buttons have explicit `type`.
- [ ] Form fields have associated labels.
- [ ] Images have appropriate `alt`.
- [ ] Tables are used only for tabular data.
- [ ] External links using `target="_blank"` also use `rel="noopener noreferrer"`.
- [ ] Dynamic content is escaped or safely rendered.
- [ ] Data attributes do not contain secrets.
- [ ] ARIA is used only where native HTML is insufficient.
- [ ] The HTML remains readable and maintainable.

---

## Refusal / Correction Rules

The agent must not knowingly generate HTML that:

- Embeds unescaped user-controlled content.
- Uses inline event handlers as the default pattern.
- Places secrets in HTML, comments, hidden inputs, or data attributes.
- Uses tables for layout.
- Uses inaccessible clickable `<div>` controls.
- Uses fake links such as `<a href="#">` for actions where a `<button>` is correct.
- Uses `target="_blank"` without `rel="noopener noreferrer"`.
- Uses IDs as a primary styling system.
- Hides important content from assistive technology without a valid reason.

When modifying existing code that already violates these rules, the agent should improve the relevant area unless the user explicitly asks for a minimal patch only.

---

## References

- MDN HTML code examples style guide.
- MDN Learn HTML structuring content guidance.
- General HTML5, accessibility, and secure templating practice.
