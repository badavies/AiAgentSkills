---
name: CSS-Agent-Skill
description: This skill governs how an AI coding agent must generate, edit, review, and document CSS code.
---

# CSS Agent Skill

## Purpose

This skill governs how an AI coding agent must generate, edit, review, and document CSS code.

The CSS produced by the agent must be maintainable, modular, responsive, accessible, performant, and compatible with modern tooling. CSS must be kept out of HTML wherever practical. Inline CSS must be the exception, not the default.

The skill reflects best practices from the referenced CSS best-practices guide, including conventions, methodology, linting, avoiding inline styles, avoiding magic numbers and hard-coded values, avoiding dangerous selectors, avoiding reactive `!important`, avoiding ID selectors, using relative units, using CSS variables, modularising styles, separating global and local styles, and writing responsive/adaptive CSS.

---

## Core Principles

The agent must write CSS that follows these principles:

1. Separation of concerns: HTML is structure, CSS is presentation, JavaScript is behaviour.
2. Predictable specificity: avoid brittle selectors and specificity wars.
3. Reusability: prefer reusable classes and components.
4. Maintainability: organise code into logical sections or files.
5. Accessibility: preserve keyboard focus, contrast, readability, and user preferences.
6. Responsiveness: layouts must adapt to viewport and content.
7. Performance: avoid expensive selectors and unnecessary styles.
8. DRY: avoid duplicated rule sets and repeated magic values.
9. KISS: prefer simple CSS over complex hacks.
10. Security: avoid unsafe dynamic CSS generation and CSS injection.

---

## Source Standards Reflected

The agent must follow these best-practice themes:

- Follow project conventions.
- Use a CSS methodology such as BEM, ITCSS, OOCSS, SMACSS, SUIT CSS, or an agreed design-system approach.
- Lint and format CSS.
- Prefer CSS over JavaScript for visual states and layout where possible.
- Comment CSS sections where comments improve maintainability.
- Avoid undoing styles.
- Avoid magic numbers.
- Avoid qualified selectors.
- Avoid hard-coded values where variables or relative values are better.
- Avoid brute-force positioning.
- Avoid dangerous broad selectors.
- Avoid extra selectors.
- Avoid reactive `!important`.
- Avoid ID selectors for styling.
- Avoid loose class names.
- Avoid duplicated key selectors.
- Avoid inline styles.
- Use multiple classes and modifier classes.
- Use relative units.
- Use CSS custom properties.
- When Bootstrap is used, integrate with Bootstrap utilities, components, CSS variables, and Sass customisation patterns instead of duplicating or fighting the framework.
- Write descriptive media queries.
- Separate global and local/component styles.
- Minimise expensive properties.
- Let content define size where practical.
- Modularise styles.
- Remove unused CSS.
- Minify CSS in production builds.

---

## File Organisation

Use a clear file structure. For small projects, one stylesheet may be acceptable. For larger projects, split CSS by responsibility.

Recommended structure:

```text
assets/
  css/
    app.css
    base/
      reset.css
      typography.css
      variables.css
    layout/
      grid.css
      header.css
      sidebar.css
    components/
      button.css
      card.css
      modal.css
      form.css
    utilities/
      spacing.css
      visibility.css
```

For projects without a bundler, keep the structure simple and avoid excessive file fragmentation.

Each file should have a clear purpose. Do not create tiny files unless the project uses a build pipeline or component-based import system.

---

## Naming Methodology

Use a consistent naming methodology. BEM-style naming is the default unless the existing project uses another convention.

Pattern:

```css
.block {}
.block__element {}
.block--modifier {}
.block__element--modifier {}
```

Example:

```css
.entity-card {}
.entity-card__header {}
.entity-card__title {}
.entity-card--selected {}
```

Rules:

- Use kebab-case.
- Use semantic names.
- Avoid vague names such as `.box`, `.blue`, `.large`, `.thing`, `.left`.
- Avoid names tied only to visual appearance.
- Prefer names tied to component purpose or UI role.
- Do not use IDs for styling.

Good:

```css
.client-summary {}
.client-summary__metric {}
.client-summary__metric--warning {}
```

Bad:

```css
#summaryBox {}
.big-red-text {}
.left-panel-thing {}
```

---

## Selectors and Specificity

Selectors must be as simple and stable as possible.

### Prefer Class Selectors

Good:

```css
.navigation-item {}
.button {}
.form-field {}
```

Avoid ID selectors:

```css
#navigationItem {}
```

IDs have high specificity and are not reusable. Use IDs for accessibility, form labels, fragment targets, and JavaScript hooks only where needed.

### Avoid Qualified Selectors

Do not unnecessarily prepend element names.

Bad:

```css
button.button {}
div.card {}
ul.nav-list {}
```

Good:

```css
.button {}
.card {}
.nav-list {}
```

### Avoid Dangerous Broad Selectors

Do not style broad elements in a way that leaks across the application.

Bad:

```css
div {
  background: #fff4cc;
}
```

Acceptable only in deliberate reset/base layers:

```css
body {
  margin: 0;
  font-family: var(--font-family-base);
}
```

### Avoid Deep Selector Chains

Bad:

```css
body .page .sidebar .menu ul li a span {}
```

Good:

```css
.menu-link__label {}
```

### Avoid Duplicate Key Selectors

Do not define the same component class in many unrelated places with conflicting rules.

Bad:

```css
.header .button { ... }
.modal .button { ... }
.sidebar .button { ... }
```

Good:

```css
.button { ... }
.button--compact { ... }
.button--danger { ... }
```

---

## Inline CSS Policy

Do not generate inline styles in HTML.

Bad:

```html
<div style="margin-top: 20px; color: red;">Error</div>
```

Good:

```html
<div class="form-alert form-alert--error">Error</div>
```

Inline styles are permitted only for controlled CSS custom properties where values are validated and genuinely dynamic:

```html
<div class="progress-meter" style="--progress-value: 72%;">
  ...
</div>
```

Do not place untrusted values in `style`.

---

## CSS Custom Properties

Use CSS custom properties for design tokens and repeated values.

```css
:root {
  --color-surface: #ffffff;
  --color-text: #1f2933;
  --color-border: #d9e2ec;
  --space-1: 0.25rem;
  --space-2: 0.5rem;
  --space-3: 1rem;
  --radius-sm: 0.25rem;
  --radius-md: 0.5rem;
}
```

Use variables for:

- Colours.
- Spacing.
- Font stacks.
- Font sizes.
- Border radii.
- Shadows.
- Layout widths.
- Z-index scale.
- Animation durations.
- Component-specific controlled values.

Avoid scattering repeated values:

```css
.card {
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
  padding: var(--space-3);
}
```

---

## Units and Values

### Use Relative Units

Prefer relative units for scalable layouts and text:

- `rem` for spacing and font sizes.
- `em` where sizing should depend on the current element.
- `%`, `fr`, `vw`, `vh`, `svh`, `lvh`, and `dvh` where appropriate for layout.
- Unitless `line-height`.

Good:

```css
.article-title {
  font-size: 1.5rem;
  line-height: 1.3;
}
```

Avoid hard-coded pixel values for typography and layout unless they represent a genuinely fixed asset, border, icon, or hairline.

### Avoid Magic Numbers

A magic number is a value used only because it appears to work.

Bad:

```css
.modal {
  margin-left: 37px;
  top: 113px;
}
```

Good:

```css
.modal {
  inset-block-start: var(--space-4);
  margin-inline: auto;
}
```

If a non-obvious value is required, name it as a custom property or explain it with a comment.

---

## Layout

Use modern CSS layout primitives:

- Flexbox for one-dimensional layout.
- Grid for two-dimensional layout.
- Logical properties for writing-mode friendliness where practical.
- `gap` for spacing between flex/grid items.
- `minmax()`, `clamp()`, and `fit-content()` where useful.
- Container queries if supported by the project baseline.

Prefer content-driven layout. Do not force fixed heights unless necessary.

Good:

```css
.card-grid {
  display: grid;
  gap: var(--space-3);
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
}
```

Avoid layout hacks and brute-force offsets.

Bad:

```css
.sidebar {
  position: relative;
  left: -7px;
}
```

---

## Responsive and Adaptive Design

All generated CSS must consider small screens, large screens, touch devices, keyboard users, and zoomed text.

Use mobile-first CSS by default:

```css
.dashboard {
  display: grid;
  gap: var(--space-3);
}

@media (min-width: 48rem) {
  .dashboard {
    grid-template-columns: 18rem minmax(0, 1fr);
  }
}
```

Media queries must be descriptive and based on layout needs, not arbitrary device names.

Prefer:

```css
@media (min-width: 64rem) {
  ...
}
```

Avoid:

```css
@media (min-width: 1024px) {
  /* iPad */
}
```

---

## Accessibility in CSS

CSS must not harm accessibility.

### Focus

Never remove outlines without replacing them.

Bad:

```css
button:focus {
  outline: none;
}
```

Good:

```css
.button:focus-visible {
  outline: 0.1875rem solid var(--color-focus);
  outline-offset: 0.125rem;
}
```

### Motion

Respect reduced-motion preferences.

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms;
    animation-iteration-count: 1;
    scroll-behavior: auto;
    transition-duration: 0.01ms;
  }
}
```

Use the above pattern carefully. In a component file, prefer limiting it to the component rather than applying it globally unless this is the project’s agreed base style.

### Colour

Do not rely on colour alone to indicate state.

Use visual affordances such as icons, text, borders, or labels in addition to colour.

### Hidden Content

Use correct hiding patterns.

Visually hidden but accessible:

```css
.visually-hidden {
  block-size: 1px;
  clip: rect(0 0 0 0);
  clip-path: inset(50%);
  inline-size: 1px;
  overflow: hidden;
  position: absolute;
  white-space: nowrap;
}
```

Fully hidden:

```css
[hidden] {
  display: none;
}
```

Do not hide focusable content from view while leaving it keyboard-focusable.

---

## `!important` Policy

Do not use `!important` reactively to escape specificity problems.

Bad:

```css
.card-title {
  color: red !important;
}
```

Permitted only for controlled utility classes or accessibility overrides where the project convention allows it:

```css
.u-hidden {
  display: none !important;
}
```

When `!important` is used, include a short comment explaining why unless the utility class is part of an established utility system.

---

## Comments

Use CSS comments to explain structure, intent, constraints, or non-obvious decisions.

Good:

```css
/* Component: Entity toolbar
   Used by the chart builder and the filings viewer. */
.entity-toolbar {
  ...
}
```

Good:

```css
/* Keeps resize handles above the Konva canvas interaction layer. */
.chart-resize-handle {
  z-index: var(--z-chart-controls);
}
```

Bad:

```css
/* Make text red */
.error {
  color: var(--color-danger);
}
```

Do not leave commented-out CSS in final output unless explicitly requested.

---

## Preferred Ordering

Follow the project’s formatter/linter. If no project standard exists, use this ordering inside rule sets:

1. Custom properties.
2. Positioning.
3. Display and layout.
4. Box model.
5. Typography.
6. Visual styles.
7. Animation/transition.
8. Miscellaneous.

Example:

```css
.card {
  --card-padding: var(--space-3);

  position: relative;

  display: grid;
  gap: var(--space-2);

  padding: var(--card-padding);

  color: var(--color-text);

  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-md);
}
```

If the project uses alphabetical ordering, follow that instead.

---

## Global Styles

Global styles must be limited and deliberate.

Allowed global areas:

- Reset/normalisation.
- Typography baseline.
- Root design tokens.
- Body layout.
- Accessibility utilities.
- Framework integration.

Avoid global component-like selectors:

```css
div > p > a {
  ...
}
```

Use component classes instead.

---

## Component Styles

Each reusable UI element should have a clear root class.

Example:

```css
.file-upload {
  display: grid;
  gap: var(--space-2);
}

.file-upload__drop-zone {
  border: 1px dashed var(--color-border);
  border-radius: var(--radius-md);
  padding: var(--space-4);
}

.file-upload__drop-zone--active {
  border-color: var(--color-accent);
}
```

Do not let one component reach deeply into another component’s internals.

Bad:

```css
.sidebar .file-upload .button span {
  ...
}
```

Good:

```css
.file-upload__button {
  ...
}
```

---

## Utility Classes

Use utility classes sparingly and consistently.

Acceptable utilities:

```css
.u-visually-hidden {}
.u-text-center {}
.u-stack {}
.u-cluster {}
```

Do not create one-off utilities to avoid designing a component.

If many utility classes are needed, consider whether the project should use an existing utility framework or a formal design system.

---

## State Classes

Use clear state classes or ARIA/data attributes.

Examples:

```css
.is-active {}
.is-disabled {}
.has-error {}
[aria-expanded="true"] {}
[data-state="loading"] {}
```

Do not encode state only in JavaScript variables when CSS can reflect it through classes or attributes.

---

## JavaScript Integration

Use classes for styling and `data-*` attributes for JavaScript hooks.

Preferred HTML:

```html
<button class="button button--primary" data-action="save-chart">
  Save chart
</button>
```

CSS:

```css
.button {}
.button--primary {}
```

JavaScript:

```js
document.querySelector('[data-action="save-chart"]');
```

Avoid using styling classes as fragile JavaScript selectors where a `data-*` hook is clearer.

Do not build class names by unsafe string concatenation. Prefer `classList.add`, `classList.remove`, `classList.toggle`, and explicit allowlists.

---

## Security

CSS can be a security concern when values are generated dynamically.

The agent must:

- Never place untrusted input directly into CSS.
- Never place untrusted input directly into a `style` attribute.
- Validate dynamic CSS values against strict allowlists.
- Avoid CSS `url()` values derived from user input.
- Avoid exposing sensitive data in generated CSS or custom properties.
- Avoid creating selectors from untrusted user input.
- Avoid unsafe HTML/CSS injection patterns in generated templates.

Example of controlled dynamic value:

```php
$progress = max(0, min(100, (int) $progress));
```

```html
<div class="progress-meter" style="--progress-value: <?= $progress ?>%;">
```

The value must be numeric, constrained, and escaped according to context.

---

## Performance

The agent must avoid CSS that creates unnecessary rendering cost.

Prefer:

- Simple selectors.
- Class selectors.
- Transform and opacity for simple animations.
- Modern layout primitives.
- Small, modular CSS.
- Removing unused rules.

Avoid:

- Deep descendant selectors.
- Repeated expensive shadows/filters.
- Animating layout properties like `width`, `height`, `top`, `left`, or `margin` when `transform` would work.
- Huge universal selector rules outside reset layers.
- Overly broad transitions such as `transition: all`.

Good:

```css
.panel {
  transition: transform 150ms ease;
}
```

Bad:

```css
.panel {
  transition: all 150ms ease;
}
```

---

## Frameworks and Existing Projects

When editing an existing project, match its established conventions unless they are unsafe or clearly broken.

### Bootstrap CSS Library

When Bootstrap is present in a project, the agent must treat Bootstrap as the base CSS framework and must not generate competing framework-style CSS unless expressly requested.

Bootstrap-specific rules:

- Confirm which Bootstrap major version the project uses before relying on version-specific classes or features.
- Use Bootstrap's documented layout, grid, spacing, display, flex, colour, typography, sizing, border, position, and visibility utilities where they already fit the design.
- Use Bootstrap components before rebuilding common UI patterns such as buttons, alerts, cards, modals, navs, tabs, dropdowns, forms, badges, tables, pagination, accordions, and offcanvas panels.
- Use Bootstrap's base-plus-modifier class model as intended, for example `.btn` with `.btn-primary`, rather than inventing parallel button systems.
- Do not duplicate Bootstrap utilities or components in custom CSS.
- Do not globally override Bootstrap selectors such as `.btn`, `.card`, `.modal`, `.row`, `.container`, `.form-control`, or `.dropdown-menu` unless the project has deliberately chosen to theme Bootstrap at that level.
- Scope custom Bootstrap adjustments with a project or component class.
- Prefer Bootstrap CSS custom properties, Sass variables, maps, mixins, and utility API where the project build pipeline supports them.
- Do not use `!important` to fight Bootstrap except for deliberate utility-class behaviour or where Bootstrap itself requires a utility override pattern.
- Avoid excessive utility-class markup where a reusable component class would be clearer and easier to maintain.
- Keep project-specific design rules in custom CSS files loaded after Bootstrap.
- Do not edit Bootstrap vendor files directly. Override through project CSS, Sass configuration, or the established build process.
- Preserve Bootstrap's accessibility behaviours, required markup structure, ARIA attributes, JavaScript data attributes, and focus states.

Preferred Bootstrap customisation pattern:

```html
<div class="card client-card">
  <div class="card-body client-card__body">
    ...
  </div>
</div>
```

```css
.client-card {
  --bs-card-border-color: var(--color-client-card-border);
  --bs-card-border-radius: var(--radius-md);
}

.client-card__body {
  display: grid;
  gap: var(--space-2);
}
```

Acceptable use of Bootstrap utilities:

```html
<div class="d-flex align-items-center justify-content-between gap-2">
  <h2 class="h5 mb-0">Client summary</h2>
  <button class="btn btn-primary" type="button">Save</button>
</div>
```

Prefer a component class when utilities become repetitive or obscure intent:

```html
<div class="client-summary-toolbar">
  <h2 class="client-summary-toolbar__title">Client summary</h2>
  <button class="btn btn-primary" type="button">Save</button>
</div>
```

```css
.client-summary-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-2);
}

.client-summary-toolbar__title {
  margin-block-end: 0;
}
```

Bad Bootstrap override:

```css
.card {
  border-radius: 2rem;
}
```

Good scoped override:

```css
.client-card {
  --bs-card-border-radius: var(--radius-lg);
}
```

If Bootstrap is used only for its grid or utility layer, do not assume the full component library is available. Check the project imports before generating classes.

---

## Linting and Formatting

Where possible, the agent should recommend or use:

- Stylelint for linting.
- Prettier or the project formatter for formatting.
- Autoprefixer/PostCSS where legacy browser prefixing is needed.
- Browserslist to define supported browsers.
- A CSS minification step for production.

The agent must not manually add vendor prefixes for modern properties unless the project’s browser support requires it and no build tool handles it.

---

## Agent Output Rules

When generating CSS, the agent must:

1. Keep CSS out of HTML unless a constrained custom property is justified.
2. Use semantic class names.
3. Avoid ID selectors.
4. Avoid broad dangerous selectors.
5. Avoid deep selector chains.
6. Avoid `!important` unless clearly justified.
7. Use design tokens or CSS custom properties for repeated values.
8. Use relative units where practical.
9. Provide responsive behaviour.
10. Preserve accessibility.
11. Keep rules modular and easy to delete.
12. Avoid excessive cleverness.
13. Include comments only where useful.
14. Match the existing project’s conventions where known.

---

## Review Checklist

Before returning CSS, the agent must verify:

- [ ] No unnecessary inline CSS is required.
- [ ] Selectors are simple and class-based.
- [ ] No ID selectors are used for styling.
- [ ] No dangerous broad selectors leak styles globally.
- [ ] No unnecessary qualified selectors are used.
- [ ] No reactive `!important` is used.
- [ ] Repeated values are variables or design tokens.
- [ ] Magic numbers are removed or explained.
- [ ] Units are relative where practical.
- [ ] Layout works responsively.
- [ ] Focus states are visible.
- [ ] Reduced-motion preferences are respected where animation is used.
- [ ] Colour is not the only state indicator.
- [ ] Component styles are modular.
- [ ] Global styles are deliberate and limited.
- [ ] JavaScript hooks are separated from styling classes where appropriate.
- [ ] Dynamic CSS values are validated and safe.
- [ ] The CSS is formatted consistently.

---

## Correction Rules

When modifying existing CSS, the agent should improve the touched area by:

- Removing duplicated rules.
- Reducing specificity.
- Replacing inline style patterns with classes.
- Extracting repeated values into variables.
- Replacing magic numbers with layout primitives.
- Making focus states accessible.
- Making selectors more stable.
- Keeping changes within the requested scope unless a security or accessibility issue is present.

If a requested change would require brittle CSS, the agent should produce the safer alternative and explain the reason briefly outside the code.

---

## References

- https://github.com/andredesousa/css-best-practices
- https://getbootstrap.com/docs/5.3/customize/css-variables/
- https://getbootstrap.com/docs/5.3/customize/components/
- https://getbootstrap.com/docs/5.3/utilities/api/
- MDN CSS styling guidance.
- Modern CSS accessibility, maintainability, and security practice.
