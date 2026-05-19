# Accessibility — Drupal Coding Standards Reference

## Source authority

This file consolidates three layers of authority. Treat findings differently depending on which layer they come from:

| Layer | Source | What it carries | Maximum severity |
|---|---|---|---|
| **Drupal official** | [Drupal Accessibility coding standards (official, minimal page)](https://project.pages.drupalcode.org/coding_standards/accessibility/accessibility/) | Drupal's binding commitment: **WCAG 2.2 Level AA** is the current accessibility target for Drupal core and the broader project. Very short page — it intentionally delegates concrete rules to the layers below. | 🔴 Blocker (when WCAG 2.2 AA criterion fails) |
| **Drupal.org complementary** | [Drupal.org Accessibility Documentation](https://www.drupal.org/about/features/accessibility) and [Drupal accessibility coding standards](https://www.drupal.org/docs/develop/standards/accessibility-coding-standards) (last updated 2026-03-04) | Drupal-specific patterns and conventions (form labels, focus management, `Drupal.announce`, Olivero/Claro patterns) and the explicit WCAG 2.2 AA target. | 🔴 Blocker or 🟡 Standards violation depending on whether a WCAG 2.2 AA criterion is breached. |
| **External best practice** | WCAG 2.2 AAA, WAI-ARIA Authoring Practices, HTML Living Standard, MDN. | Underlying web standards Drupal points to but does not bind itself to (anything above AA, or guidance Drupal docs don't explicitly adopt). Use as the **reason** for a finding, not as a citation of "official Drupal standard". | 🔴 Blocker only when a WCAG 2.2 **AA** criterion fails; otherwise 🟢 Suggestion. |

> When reporting a finding, **name the layer**: "Drupal official", "Drupal.org complementary", or "external best practice (WCAG x.y.z)". Do not pass an external recommendation off as a Drupal coding-standards rule.

The accessibility page on `project.pages.drupalcode.org` is deliberately concise; everything below either cites a specific WCAG 2.2 AA criterion (treat as the normative target Drupal binds itself to) or a Drupal.org accessibility doc (treat as Drupal complementary). WCAG 2.2 criteria **above AA** (i.e. AAA) remain external best practice and cap at 🟢.

---

## Table of contents

1. [Mandatory principles](#1-mandatory-principles)
2. [HTML semantics](#2-html-semantics)
3. [Alternative text and media](#3-alternative-text-and-media)
4. [Accessible forms](#4-accessible-forms)
5. [Keyboard navigation](#5-keyboard-navigation)
6. [ARIA: use with judgment](#6-aria-use-with-judgment)
7. [Color and contrast](#7-color-and-contrast)
8. [Skip links and landmarks](#8-skip-links-and-landmarks)
9. [Messages and dynamic feedback](#9-messages-and-dynamic-feedback)
10. [Accessible tables](#10-accessible-tables)
11. [Drupal-specific patterns](#11-drupal-specific-patterns)
12. [Testing and validation](#12-testing-and-validation)
13. [Accessibility review checklist](#accessibility-review-checklist)

---

## 1. Mandatory principles

Drupal core commits to **WCAG 2.2 Level AA** (per the [Drupal Accessibility Coding Standards](https://www.drupal.org/docs/develop/standards/accessibility-coding-standards), updated 2026-03-04). The four POUR principles apply to every module/theme:

1. **Perceivable**: information must be presented in a way that any user can perceive it. Alternative text, transcripts, sufficient contrast.
2. **Operable**: all functionality must be usable with a keyboard. Sufficient time to interact. No content causing seizures.
3. **Understandable**: readable text, predictable behavior, assistance to avoid errors.
4. **Robust**: compatible with assistive technologies (screen readers, voice control, switch devices).

---

## 2. HTML semantics

### Critical rules

- **Use correct semantic elements**. `<div>` and `<span>` are neutral, not semantic.
- **Don't reinvent native widgets**. A `<button>` is accessible by default; a `<div onclick="...">` is not.
- **Heading hierarchy**: use `<h1>` → `<h6>` in logical order, without skipping levels. A single `<h1>` per page.
- **Lists use `<ul>`, `<ol>`, `<dl>`**. Never simulate lists with consecutive `<br>` or `<p>`.

### Examples

❌ Antipattern:
```html
<div class="button" onclick="doSomething()">Save</div>
```

✅ Correct:
```html
<button type="button" onclick="doSomething()">Save</button>
```

❌ Antipattern (empty heading for styling):
```html
<h2></h2>
<div class="big-text">Big text</div>
```

✅ Correct:
```html
<h2>Products section</h2>
```

---

## 3. Alternative text and media

### Images

- **`alt` mandatory on all `<img>`**. If the image is decorative, `alt=""` (empty attribute, NOT absent).
- **`alt` describes the function or content**, not appearance. E.g.: `alt="Company logo"`, not `alt="Image of a blue logo"`.
- Linked images: the `alt` describes the link destination.
- **Decorative SVGs**: `aria-hidden="true"` + `focusable="false"`.
- **Meaningful SVGs**: `<title>` inside the SVG or `aria-label` on the wrapper.

### Video and audio

- Videos require **captions** and, when applicable, **audio description**.
- Audio requires **transcript**.
- **No autoplay** with sound unless there is a clear control to stop it.

### Icons

- Font or SVG icons without text: use `<span class="visually-hidden">` with description for screen readers, and `aria-hidden="true"` on the visual icon.

```html
<button type="button" class="close">
  <span aria-hidden="true">×</span>
  <span class="visually-hidden">Close dialog</span>
</button>
```

---

## 4. Accessible forms

### Labels

- **Each input requires an associated `<label>`** via `for="input-id"`.
- Visible labels, not just placeholders (placeholders disappear when typing).
- For inputs without visible label (e.g. search), use `aria-label` or a `<label class="visually-hidden">`.

```html
<label for="search-field">Search products</label>
<input type="search" id="search-field" name="search">
```

### Grouping

- **Radio/checkbox groups**: wrap in `<fieldset>` with `<legend>`.

```html
<fieldset>
  <legend>Shipping type</legend>
  <label><input type="radio" name="ship" value="express"> Express</label>
  <label><input type="radio" name="ship" value="standard"> Standard</label>
</fieldset>
```

### Errors and validation

- **Error messages associated with the input** via `aria-describedby`.
- **`aria-invalid="true"` only while an error is present** on that input. The attribute should reflect the current validity state — remove it (or set to `"false"`) once the user corrects the input. Never author it on a pristine field.
- **Clear and specific messages**: "Enter a valid email" better than "Error".
- **Required**: `required` attribute + visual indicator + explicit text (not just an asterisk).

```html
<label for="email">Email address <span aria-hidden="true">*</span>
  <span class="visually-hidden">required</span>
</label>
<input type="email" id="email" required aria-describedby="email-error" aria-invalid="true">
<div id="email-error" class="form-error">Enter a valid email (e.g. name@domain.com).</div>
```

---

## 5. Keyboard navigation

### Critical rules

- **All functionality accessible via keyboard**. Tab, Shift+Tab, Enter, Space, arrows.
- **Visible focus**: the browser outline is NOT removed. If customized, it must be visible at ALL times.
- **Logical tab order**: follows the visual flow of the page.
- **`tabindex` only when necessary**:
  - `tabindex="0"`: makes a non-focusable element focusable, maintains DOM order.
  - `tabindex="-1"`: removable from tab order, but focusable programmatically (useful for modals).
  - Positive `tabindex` values (`1`, `2`, ...) **FORBIDDEN**: they break the natural order.

❌ Antipattern:
```css
*:focus { outline: none; }
```

✅ Correct (stylize but keep visible):
```css
*:focus {
  outline: 2px solid #0066cc;
  outline-offset: 2px;
}
```

### Focus traps

- **Modals/dialogs**: focus is trapped inside the modal until closed.
- **On close**, focus returns to the trigger that opened the modal.

---

## 6. ARIA: use with judgment

> **Golden rule**: "No ARIA is better than bad ARIA". The first Rule of ARIA is: **if you can use a native HTML element, use it**.

### When to use ARIA

- **Only when there is no native HTML equivalent**.
- To enhance custom widgets (tabs, accordions, complex comboboxes).
- To describe relationships (`aria-describedby`, `aria-labelledby`).
- To announce dynamic changes (`aria-live`, `role="status"`, `role="alert"`).

### Most used ARIA roles

- `role="banner"` → main site header.
- `role="navigation"` → nav blocks (or use `<nav>`).
- `role="main"` → main content (or use `<main>`).
- `role="complementary"` → sidebars (or use `<aside>`).
- `role="contentinfo"` → footer (or use `<footer>`).
- `role="search"` → search region.
- `role="dialog"` / `role="alertdialog"` → modals.
- `role="tablist"` / `role="tab"` / `role="tabpanel"` → tabs.
- `role="button"` → only if you CANNOT use `<button>`.

### Common ARIA states/properties

- `aria-expanded="true|false"` → state of collapsible elements.
- `aria-current="page|step|location|date|time|true"` → indicates current element.
- `aria-controls="id"` → relationship between control and controlled region.
- `aria-hidden="true"` → hidden from screen readers (do NOT use on focusable elements).
- `aria-label="text"` → visually inaccessible label.
- `aria-labelledby="id"` → label by reference.
- `aria-describedby="id"` → description by reference.

---

## 7. Color and contrast

### WCAG 2.2 AA — required for Drupal

Contrast (carried over from 2.1 AA, still required under 2.2 AA):

- **Normal text (<18.66px or non-bold <24px)**: ratio **4.5:1** minimum (SC 1.4.3).
- **Large text (≥18.66px bold or ≥24px)**: ratio **3:1** minimum (SC 1.4.3).
- **UI components and graphic elements**: ratio **3:1** minimum (icons, input borders, etc.) (SC 1.4.11).

Added by WCAG 2.2 A/AA (treat as required, **not optional**):

- **Focus visible / not obscured**: keyboard focus must have a visible
  indicator (SC 2.4.7 *Focus Visible*) and the focused component must not be
  entirely hidden by author-created content (SC 2.4.11 *Focus Not Obscured
  (Minimum)*).
- **Target size**: pointer targets ≥ **24×24 CSS px**, with documented
  exceptions (inline, user-agent default, equivalent control elsewhere,
  essential) (SC 2.5.8 *Target Size (Minimum)*).
- **Dragging movements**: any dragging interaction must have a single-pointer alternative (SC 2.5.7).
- **Consistent help**: if help mechanisms are present on multiple pages, they appear in the same relative order (SC 3.2.6).
- **Redundant entry / accessible authentication**: don't force the user to
  re-enter info already provided, and don't gate auth behind a cognitive
  function test without an alternative (SC 3.3.7, 3.3.8).

Above-AA guidance (cap at 🟢 unless the project has adopted it locally):

- **Focus appearance area / contrast**: SC 2.4.13 *Focus Appearance* is
  **WCAG 2.2 AAA**, not AA. A focus indicator with 3:1 contrast and adequate
  area is a strong recommendation, but do not cite it as a Drupal WCAG 2.2 AA
  blocker.

### Rules

- **Don't convey information by color alone**. Combine with text, icons, patterns, or underlines.
- Validate with tools: WebAIM Contrast Checker, axe DevTools, Lighthouse.

❌ Antipattern:
```html
<p>Fields in <span style="color:red">red</span> are required.</p>
```

✅ Correct:
```html
<p>Fields marked with <span class="required-marker">*</span> are required.</p>
```

---

## 8. Skip links and landmarks

### Skip links

Drupal core implements skip links by default. Verify that:

- **First tabbable link** is "Skip to main content" (or translated equivalent).
- Visible when receiving focus (not permanently hidden).
- Points to `<main>` or `<div id="main-content">`.

```html
<a href="#main-content" class="visually-hidden focusable">
  Skip to main content
</a>
```

Associated CSS — these are **two additive classes on the same element** (`.visually-hidden.focusable`, i.e. `class="visually-hidden focusable"`). Below is the verbatim Drupal 11 core stylesheet from [core/modules/system/css/components/hidden.module.css](https://git.drupalcode.org/project/drupal/-/blob/11.x/core/modules/system/css/components/hidden.module.css):

```css
.visually-hidden {
  position: absolute !important;
  overflow: hidden;
  clip: rect(1px, 1px, 1px, 1px);
  width: 1px;
  height: 1px;
  word-wrap: normal;
}

.visually-hidden.focusable:active,
.visually-hidden.focusable:focus-within {
  position: static !important;
  overflow: visible;
  clip: auto;
  width: auto;
  height: auto;
}
```

### Landmarks

Use HTML5 landmarks (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`) or equivalent ARIA roles so screen readers can jump between sections.

---

## 9. Messages and dynamic feedback

### Drupal Messages

Drupal uses `\Drupal::messenger()->addMessage()` for messages (status, warning, error). The rendered `[data-drupal-messages]` region is a **landmark** (`role="contentinfo"`) — that's how screen-reader users *navigate to* the messages region, not how individual message severities are announced. Per-message severity uses the live-region roles below:

| Severity | Live-region role | `aria-live` | When to use |
|---|---|---|---|
| Status | `role="status"` | `polite` | Confirmation, success, neutral info |
| Warning | `role="alert"` | `assertive` | The action partially failed or has caveats |
| Error | `role="alert"` | `assertive` | The action failed; user must act |

`role="contentinfo"` is a **landmark** — typically applied to the footer or to the messages region wrapper. **Don't** use it as a severity indicator on an individual message.

### Live regions

For dynamic feedback (Ajax, autocomplete, inline validation):

- `aria-live="polite"` → announces when the user is inactive.
- `aria-live="assertive"` → interrupts immediately (use sparingly).
- `aria-atomic="true|false"` → whether the whole region or just changes are announced.

```html
<div id="status-message" role="status" aria-live="polite" aria-atomic="true">
  <!-- Drupal.announce() will put messages here. -->
</div>
```

### `Drupal.announce()` (JS)

In Drupal 8+, use `Drupal.announce(text, priority)` for announcements to screen readers from JS:

```javascript
Drupal.announce('Filter applied: 5 results.');
Drupal.announce('Save error.', 'assertive');
```

---

## 10. Accessible tables

### Data tables

- **`<table>` only for tabular data**, not for layout.
- **`<th>` for headers** with `scope="col"` or `scope="row"`.
- **`<caption>`** describes the purpose of the table.
- Complex tables with multiple headers: use `headers` + `id`.

```html
<table>
  <caption>Q1 2026 quarterly sales</caption>
  <thead>
    <tr>
      <th scope="col">Product</th>
      <th scope="col">Units</th>
      <th scope="col">Revenue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Widget A</th>
      <td>1,250</td>
      <td>$12,500</td>
    </tr>
  </tbody>
</table>
```

---

## 11. Drupal-specific patterns

### Render arrays and `#attributes`

When building render arrays in custom modules, add accessible attributes:

```php
$build['contact'] = [
  '#type' => 'link',
  '#title' => $this->t('Contact us'),
  '#url' => Url::fromRoute('contact.site_page'),
  '#attributes' => [
    'aria-label' => $this->t('Contact us via the contact form'),
  ],
];
```

### Form API

Drupal Form API generates accessible labels and attributes automatically if you use it correctly:

```php
$form['email'] = [
  '#type' => 'email',
  '#title' => $this->t('Your email'),
  '#required' => TRUE,
  '#description' => $this->t('We will never share this email.'),
];
```

HTML result: correct `<label>` + `aria-describedby` pointing to the description.

### `t()` for UI strings

**Every user-visible string** must pass through `t()` (in `.module`) or `$this->t()` (in classes). This includes accessible text:

```php
'#attributes' => [
  'aria-label' => $this->t('Close the dialog'),
],
```

### Theme `visually-hidden` class

Drupal provides the `.visually-hidden` class (canonical source: [`core/modules/system/css/components/hidden.module.css`](https://git.drupalcode.org/project/drupal/-/blob/11.x/core/modules/system/css/components/hidden.module.css), attached via the `system/base` library; a copy lives in `core/themes/stable9/css/system/components/hidden.module.css` for the legacy Stable9 base theme). Use it instead of inventing your own:

```html
<span class="visually-hidden">{{ 'New tab'|t }}</span>
```

### Library `core/drupal.dialog`

For modals, use the core dialog (`core/drupal.dialog` or `core/drupal.dialog.ajax`). It already has focus trap, escape to close, and screen reader announcement.

### `Drupal.tabbingManager` — keyboard focus traps

When you need a focus trap outside the standard dialog (custom modals, mega-menus, off-canvas panels), use Drupal's [`tabbing manager`](https://api.drupal.org/api/drupal/core%21misc%21tabbingmanager.es6.js/11.x) (`core/drupal.tabbingmanager`). It constrains Tab/Shift+Tab focus to the elements you nominate and releases it cleanly when the panel closes — so screen-reader users don't accidentally tab out into the page beneath the overlay.

```js
// Declare core/drupal.tabbingmanager in *.libraries.yml dependencies.

// Constrain focus to the panel:
const tabset = Drupal.tabbingManager.constrain(document.querySelectorAll('#my-panel, #my-panel *'));

// When closing the panel:
tabset.release();
```

Use `release()` (or the auto-release on outside click) — never leave a constrained set active, or keyboard users will be locked in.

### `prefers-reduced-motion` — respect user motion preferences

Any custom animation, transition, or scroll behavior must honor the `prefers-reduced-motion` media query. The Drupal core convention is to gate motion behind `(prefers-reduced-motion: no-preference)` so the default state is "no motion":

```css
/* CSS — only animate when the user has not requested reduced motion. */
.card {
  transition: none;
}

@media (prefers-reduced-motion: no-preference) {
  .card {
    transition: transform 200ms ease-out;
  }
}
```

```js
// JS — same idea via matchMedia.
const allowMotion = window.matchMedia('(prefers-reduced-motion: no-preference)').matches;
if (allowMotion) {
  element.scrollIntoView({ behavior: 'smooth' });
} else {
  element.scrollIntoView();
}
```

This is required by WCAG SC 2.3.3 *Animation from Interactions* (AAA) and is also baseline Drupal core practice (Olivero, Claro).

> **Severity carve-out (cross-reference with [SKILL.md § Severity criteria](../SKILL.md#severity-criteria)):** the general rule is "WCAG criteria outside the AA conformance target are at most 🟢". SC 2.3.3 is formally AAA and therefore would normally cap at 🟢, but Drupal core implements it as baseline (Olivero / Claro respect `prefers-reduced-motion` out of the box). Escalate violations that produce **actual vestibular harm** — unconstrained parallax, sustained rapid flashing, motion that ignores the user's reduced-motion preference on a core/contrib-themed page — to 🔴 on accessibility-harm grounds. Cosmetic AAA deviations that don't cause harm remain 🟢.

---

## 12. Testing and validation

### Recommended tools

- **axe DevTools** (Deque): Chrome/Firefox extension.
- **WAVE** (WebAIM): extension and web tool.
- **Lighthouse Accessibility audit** (Chrome DevTools).
- **Pa11y CI**: integration in CI/CD pipelines.
- **NVDA** (Windows) / **VoiceOver** (Mac) / **Orca** (Linux): real screen readers.

### Minimum manual testing

1. **Navigate everything with Tab/Shift+Tab/Enter/Space** without touching the mouse.
2. **Verify visible focus** on every interactive element.
3. **Enable screen reader** and traverse the page.
4. **Zoom to 200%** and verify the layout doesn't break.
5. **Test with `prefers-reduced-motion`** active: animations must respect it.

### Drupal accessibility module

For automated analysis on Drupal sites: contrib module [Editoria11y](https://www.drupal.org/project/editoria11y) offers accessibility checks for content authors.

---

## Accessibility review checklist

When reviewing Twig templates, render arrays, or markup in Drupal, validate:

### Semantics
- [ ] Correct HTML5 semantic elements (`<button>`, `<nav>`, `<main>`, `<header>`, `<footer>`)
- [ ] Logical heading hierarchy (single `<h1>`, no skipped levels)
- [ ] Lists use `<ul>`/`<ol>`/`<dl>`, not `<br>` or repeated `<p>`
- [ ] No reinventing native widgets with `<div>`/`<span>` + onclick

### Images and media
- [ ] All `<img>` have `alt` (empty if decorative)
- [ ] `alt` describes content/function, not appearance
- [ ] Decorative SVGs: `aria-hidden="true"` + `focusable="false"`
- [ ] Videos with captions, audio with transcript
- [ ] No autoplay with sound

### Forms
- [ ] Each input with associated `<label>` (or `aria-label`/`aria-labelledby`)
- [ ] Visible labels (not just placeholders)
- [ ] Groups in `<fieldset>` + `<legend>`
- [ ] Errors associated with `aria-describedby` + `aria-invalid="true"`
- [ ] Required: `required` + visual indicator + explicit text

### Keyboard
- [ ] All functionality accessible via keyboard
- [ ] Visible focus (no `outline: none` without replacement)
- [ ] Logical tab order
- [ ] No positive `tabindex`
- [ ] Modals with focus trap and focus return

### ARIA
- [ ] Only when no native HTML available
- [ ] Correct roles (`role="banner"`, `"navigation"`, `"main"`, etc.)
- [ ] Updated states (`aria-expanded`, `aria-current`)
- [ ] `aria-hidden="true"` only on non-focusables
- [ ] `aria-label`/`aria-labelledby`/`aria-describedby` with valid values

### Color, contrast, and A/AA accessibility checks
- [ ] Normal text: ratio 4.5:1 minimum (SC 1.4.3)
- [ ] Large text: ratio 3:1 minimum (SC 1.4.3)
- [ ] UI components: ratio 3:1 minimum (SC 1.4.11)
- [ ] Information not conveyed by color alone (SC 1.4.1)
- [ ] Visible keyboard focus indicator (SC 2.4.7)
- [ ] Focused component not entirely obscured by author-created content (SC 2.4.11)
- [ ] Focus indicator with 3:1 contrast and adequate area only as 🟢 AAA guidance (SC 2.4.13)
- [ ] Target size ≥ 24×24 CSS px or documented exception (SC 2.5.8)
- [ ] Single-pointer alternative for any drag interaction (SC 2.5.7)
- [ ] Consistent placement of help mechanisms across pages (SC 3.2.6)
- [ ] No redundant data entry; accessible authentication path exists (SC 3.3.7, 3.3.8)

### Landmarks and skip links
- [ ] Skip link as first focusable
- [ ] Correct landmarks (`<main>`, `<nav>`, `<aside>`, etc.)
- [ ] `.visually-hidden` with `.focusable` for skip links

### Dynamic messages
- [ ] Live regions (`aria-live`) on dynamic feedback
- [ ] `Drupal.announce()` for JS announcements
- [ ] Status messages with appropriate role

### Tables
- [ ] `<table>` only for data
- [ ] `<caption>` describes purpose
- [ ] `<th scope="col|row">` correct

### Drupal patterns
- [ ] Render arrays with accessible `#attributes`
- [ ] Form API: `#title`, `#description`, `#required` used correctly
- [ ] UI strings pass through `t()` or `$this->t()`
- [ ] Reuse core classes (`.visually-hidden`, `.visually-hidden.focusable`)
- [ ] Modals use `core/drupal.dialog`

---

## Official resources

- [WCAG 2.2 Guidelines](https://www.w3.org/TR/WCAG22/)
- [Drupal Accessibility](https://www.drupal.org/about/features/accessibility)
- [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)
- [Inclusive Components (Heydon Pickering)](https://inclusive-components.design/)
- [a11y Project](https://www.a11yproject.com/)
