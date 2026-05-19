# Markup — Drupal Coding Standards Reference

## Source authority

| Layer | Source | What it carries | Maximum severity |
|---|---|---|---|
| **Drupal official** | [Drupal Markup Style Guide (official)](https://project.pages.drupalcode.org/coding_standards/markup/style/) | The official page is intentionally minimal ("a document in progress") and points to the canonical guide at [Drupal Markup Style Guide on groups.drupal.org](https://groups.drupal.org/node/6355). | 🟡 Standards violation (rare; the official page asserts very little) |
| **Drupal.org complementary** | [Drupal theming documentation](https://www.drupal.org/docs/develop/theming-drupal), Olivero/Claro theme conventions, Drupal core base templates. | Drupal-specific markup conventions (BEM-ish class names, render-array driven output, `attributes` helper). | 🟡 Standards violation |
| **External best practice** | HTML Living Standard, WHATWG, MDN, semantic-HTML community guidance. | Underlying web standards. | 🟢 Suggestion (or 🔴 Blocker if a real WCAG criterion is breached — see `ACCESSIBILITY.md`) |

> When reporting a finding, **name the layer**: "Drupal official", "Drupal.org complementary", or "external best practice". Because the official Drupal Markup page is so minimal, **most concrete rules in this file fall under the second or third layer**. Do not classify a finding as 🔴 just because it deviates from HTML Living Standard — promote to 🔴 only when it also breaks accessibility or runtime.

This reference consolidates the formal principles applied by the Drupal core team plus the complementary documentation needed to give actionable feedback.

---

## Table of contents

1. [General principles](#1-general-principles)
2. [HTML5 doctype and structure](#2-html5-doctype-and-structure)
3. [Semantic elements](#3-semantic-elements)
4. [Attributes: order and format](#4-attributes-order-and-format)
5. [Indentation and whitespace](#5-indentation-and-whitespace)
6. [CSS classes: naming in markup](#6-css-classes-naming-in-markup)
7. [IDs in markup](#7-ids-in-markup)
8. [Base templates and theme hooks](#8-base-templates-and-theme-hooks)
9. [Render arrays as markup output](#9-render-arrays-as-markup-output)
10. [Sanitization and output](#10-sanitization-and-output)
11. [Markup that must NOT be hardcoded](#11-markup-that-must-not-be-hardcoded)
12. [Markup review checklist](#markup-review-checklist)

---

## 1. General principles

The goals of the Drupal Markup Style Guide are:

1. **Create good HTML/markup**: semantic, valid, accessible.
2. **Create base templates that themers can style easily**: minimal but with all customization hooks.
3. **Create good CSS classes**: named with design semantics, reusable, predictable.

Markup in Drupal **travels** between modules (which define default structure) and themes (which override it). For this reason:

- **Default markup must be minimalist, semantic, and design-agnostic**.
- **Themes assume control over the markup** via template overrides.
- **Any class with visual purpose** must be overridable by the theme.

---

## 2. HTML5 doctype and structure

### Doctype — document-level templates only

The `<!DOCTYPE html>` declaration belongs at the start of a **full HTML document**, not at the start of every Twig template. In Drupal that means:

- ✅ **Document-level templates carry the doctype.** These are the templates that render the whole `<html>` document: `html.html.twig` (and any sub-theme override of it), the standalone `maintenance-page.html.twig` / `install-page.html.twig` chain, and any custom full-page template a theme registers.
- ❌ **Fragment templates do NOT carry a doctype.** `page.html.twig`, `region--*.html.twig`, `block--*.html.twig`, `node--*.html.twig`, `field--*.html.twig`, `paragraph--*.html.twig`, `views-view*.html.twig`, `menu--*.html.twig`, form/element templates, and every custom theme hook registered via `hook_theme()` render fragments that get composed *inside* `html.html.twig`. Adding `<!DOCTYPE html>` inside a fragment produces invalid output (a stray doctype mid-document).

Rule of thumb: if the template's outermost element is `<html>` (or it's responsible for `<head>`/`<body>`), it owns the doctype. Otherwise it doesn't.

### Document-level minimum structure (`html.html.twig`)

```html
<!DOCTYPE html>
<html lang="{{ language.language }}" dir="{{ language.dir }}">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>{{ head_title|safe_join(' | ') }}</title>
    {{ page.head }}
  </head>
  <body{{ attributes }}>
    {{ page_top }}
    {{ page }}
    {{ page_bottom }}
  </body>
</html>
```

### Fragment template shape (no doctype, no `<html>`/`<body>`)

```twig
{# block--my-block.html.twig — rendered inside the document by html.html.twig. #}
<div{{ attributes.addClass('my-block') }}>
  {{ title_prefix }}
  {% if label %}<h2{{ title_attributes }}>{{ label }}</h2>{% endif %}
  {{ title_suffix }}
  <div class="my-block__content">{{ content }}</div>
</div>
```

### `lang` and `dir` attributes

- **`lang` mandatory on `<html>`** — Drupal provides it via `{{ language.language }}` (in `html.html.twig`).
- **`dir` when RTL support exists** — Drupal provides it via `{{ language.dir }}` (also on `<html>`).
- Fragment templates may set `lang` on a wrapper element when the fragment's content is in a different language than the surrounding document (e.g. a Japanese quotation inside an English page) — but they never set it on a synthetic `<html>` they don't own.

---

## 3. Semantic elements

### Master rule

Use the most semantically appropriate HTML element **always**. Don't use `<div>` when a specific element exists.

### Landmark elements

| Element | Use |
|---|---|
| `<header>` | Page or section header |
| `<nav>` | Main navigation blocks |
| `<main>` | Unique main content (only once per page) |
| `<aside>` | Tangential content (sidebars, callouts) |
| `<section>` | Thematic section with its own heading |
| `<article>` | Independent content (post, comment, card) |
| `<footer>` | Page or section footer |

### Text elements

- `<h1>` to `<h6>`: hierarchical headings. One `<h1>` per page.
- `<p>`: paragraphs.
- `<blockquote>` with `<cite>`: long quotes.
- `<q>`: inline quotes.
- `<address>`: contact information.
- `<time datetime="ISO-8601">`: dates and times.
- `<abbr title="...">`: abbreviations.
- `<code>`, `<pre>`, `<kbd>`, `<samp>`, `<var>`: code and technical elements.

### Lists

- `<ul>`: unordered list.
- `<ol>`: ordered list.
- `<dl>` + `<dt>` + `<dd>`: definition list (terms/descriptions).

### Forms

- `<form>` with `action`, `method`, optionally `enctype`.
- `<fieldset>` + `<legend>` to group related fields.
- `<label for="id">` associated to each input.
- `<button type="button|submit|reset">` (always explicit `type`).

### Tables

- `<table>` ONLY for tabular data.
- `<caption>` to describe the purpose.
- `<thead>`, `<tbody>`, `<tfoot>` to separate sections.
- `<th scope="col|row">` for headers.

---

## 4. Attributes: order and format

### Quotes in attributes

- **Double quotes** for attribute values: `class="block"`, `id="main"`.
- **Never single quotes** in HTML attributes.

❌ Bad:
```html
<div class='block' id='main'>
```

✅ Good:
```html
<div class="block" id="main">
```

### Attribute order

The official Drupal Markup Style Guide does **not prescribe** attribute order, and Drupal core templates are mixed. The Idiomatic HTML convention used by most Drupal core templates (Olivero, Claro, system templates) groups attributes by **purpose**, with functional / structural attributes before presentational ones:

1. Functional / structural: `type`, `name`, `value`, `src`, `href`, `for`
2. Identification: `id`
3. Presentation: `class`
4. State / data: `data-*`
5. Accessibility: `title`, `alt`, `role`, `aria-*`

This is a **convention, not a coding standard**. Don't flag a deviation as a 🟡 violation — only call it out as a 🟢 suggestion when readability suffers.

### Boolean attributes

In HTML5, boolean attributes are written without a value:

```html
<input type="checkbox" required disabled>
<button type="button" hidden>Hidden button</button>
```

Do NOT write `required="required"` or `required="true"` (although valid, redundant).

### Self-closing tags

HTML5 does not require a trailing `/` on void elements (`<img>`, `<br>`, `<input>`, `<meta>`, etc.). The official Drupal Markup Style Guide does **not** prescribe one style — Drupal core templates are mixed. Either form is valid HTML5. Pick one and stay consistent within a file:

```html
<!-- Both are valid HTML5. -->
<img src="logo.svg" alt="Logo">
<img src="logo.svg" alt="Logo" />
```

---

## 5. Indentation and whitespace

- **2 spaces** per indentation level. **Never tabs**.
- No trailing whitespace.
- Line endings `\n` (Unix).
- Final newline in the file.
- One blank line between logical blocks (header/main/footer).

### Indentation of nested elements

```html
<article class="card">
  <header class="card__header">
    <h2 class="card__title">Title</h2>
  </header>
  <div class="card__body">
    <p>Card content.</p>
  </div>
</article>
```

### Long attributes

If an element has many attributes and the line exceeds 80 chars, you can wrap each attribute on its own line:

```html
<input
  type="email"
  id="user-email"
  name="email"
  required
  aria-describedby="email-help"
  class="form-input form-input--email">
```

---

## 6. CSS classes: naming in markup

> **See `references/CSS.md`** for full rules. Here we summarize how to apply them in markup.

> **Source authority note.** The BEM-style naming below is from the [Drupal CSS architecture standard](https://www.drupal.org/docs/develop/standards/css/css-architecture-for-drupal-9), **not** from the Markup Style Guide (which is intentionally minimal and prescribes only that "every div requires at least one identifying class"). When reporting a finding against these rules, cite *"Drupal CSS architecture standard"* — and treat deviation as a 🟡 *standards violation* on the CSS-architecture axis, not the markup axis.

### Drupal CSS architecture conventions (BEM-style)

- **Component**: `.component-name` (full words, hyphens between words).
- **Sub-object**: `.component-name__sub-object` (double underscore).
- **Variant**: `.component-name--variant` (double dash).
- **State**: `.is-state` (e.g. `.is-active`, `.is-expanded`).
- **JS hook**: `.js-behavior-hook` (`js-` prefix, NOT to be used for styling).

### Application in markup

✅ Correct:
```html
<article class="card card--featured">
  <header class="card__header">
    <h2 class="card__title">Featured title</h2>
  </header>
  <div class="card__body">
    <p class="card__text">Content</p>
  </div>
</article>

<!-- Modal with state -->
<div class="dialog dialog--centered is-open js-modal-trigger">
  ...
</div>
```

### What NOT to do

❌ Generic classes:
```html
<div class="content">  <!-- too generic -->
<div class="blue-box"> <!-- describes appearance, not function -->
<div class="title">    <!-- conflict with .card .title, .menu .title, etc. -->
```

❌ Classes with `_` only (non-Drupal style):
```html
<div class="card_header"> <!-- use `__` for sub-objects -->
```

❌ Mixing JS hooks with styling:
```html
<div class="js-modal"> <!-- and then styling it in CSS -->
```

---

## 7. IDs in markup

### General rule

- **Do NOT use `id` for CSS hooks**. Causes specificity issues and is not reusable.
- **Use `id` for**:
  1. Performant JavaScript hooks.
  2. Anchors within the document (`<a href="#section">`).
  3. Associate labels with form elements (`<label for="email">`).
  4. ARIA relationships (`aria-describedby`, `aria-labelledby`, `aria-controls`).

### Unique IDs

- **Each `id` must be unique on the page**. If you need multiple similar elements, use classes.
- **Naming**: `kebab-case` (`user-form-email`, `main-content-region`).
- **Templates and theme functions can render more than once per request** (e.g. multiple instances of the same block, paragraphs, views rows). A hard-coded `id="my-thing"` in a render array or Twig template will produce duplicates and break HTML validity, `for`/`aria-*` references, and JS query selectors.
- Use [`\Drupal\Component\Utility\Html::getUniqueId($string)`](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Component%21Utility%21Html.php/function/Html%3A%3AgetUniqueId/11.x) to produce a per-request unique id (`my-thing`, `my-thing--2`, …). For situations where you only need a registered id without uniqueness enforcement, use `Html::getId()`.

```php
// In a controller / preprocess function:
$build['#attributes']['id'] = Html::getUniqueId('my-widget');

// In Twig (id passed in from preprocess):
<div id="{{ id }}" aria-describedby="{{ id }}-description">
```

---

## 8. Base templates and theme hooks

### Philosophy

Drupal core (and contrib modules) provide **base templates** that themes can override. The rule for modules:

- **Default markup must be minimalist and design-agnostic**.
- **Include classes with enough functional semantics** so the theme can override them.
- **Every template must be overridable**: what a module renders, a theme can replace.

### Template suggestions

Drupal generates **template suggestions** automatically based on context. E.g. for a node:

- `node--article--full.html.twig` (most specific)
- `node--article.html.twig`
- `node--full.html.twig`
- `node.html.twig` (most general)

The theme can create any of these to override.

### Theme hooks in modules

When a module defines custom markup, it registers a theme hook in `hook_theme()`:

```php
function my_module_theme($existing, $type, $theme, $path) {
  return [
    'my_widget' => [
      'variables' => [
        'title' => NULL,
        'content' => NULL,
        'attributes' => [],
      ],
      'template' => 'my-widget',
    ],
  ];
}
```

And provides a default template in `my_module/templates/my-widget.html.twig`:

```twig
{#
/**
 * @file
 * Default template for my_widget.
 *
 * Available variables:
 * - title: The widget title.
 * - content: The widget content.
 * - attributes: HTML attributes for the wrapper.
 */
#}
<div{{ attributes.addClass('my-widget') }}>
  {% if title %}
    <h3 class="my-widget__title">{{ title }}</h3>
  {% endif %}
  <div class="my-widget__content">
    {{ content }}
  </div>
</div>
```

---

## 9. Render arrays as markup output

In Drupal, markup is rarely hardcoded: it's generated via **render arrays** that pass through theme functions / templates.

### Advantages

- Cacheable (cache tags, cache contexts).
- Alterable by other modules (hook_preprocess, hook_alter).
- Accessible (correct attributes by default).
- Localizable.

### Example

```php
$build['greeting'] = [
  '#markup' => $this->t('Hello, @name', ['@name' => $username]),
];

$build['list'] = [
  '#theme' => 'item_list',
  '#items' => $items,
  '#title' => $this->t('My items'),
  '#attributes' => ['class' => ['my-list']],
];
```

### What NOT to do

❌ Concatenating HTML strings directly:
```php
$output = '<div class="my-block"><h2>' . $title . '</h2><p>' . $content . '</p></div>';
return ['#markup' => $output];
```

✅ Use structured render arrays:
```php
$build = [
  '#type' => 'container',
  '#attributes' => ['class' => ['my-block']],
  'title' => [
    '#type' => 'html_tag',
    '#tag' => 'h2',
    '#value' => $title,
  ],
  'content' => [
    '#type' => 'html_tag',
    '#tag' => 'p',
    '#value' => $content,
  ],
];
```

---

## 10. Sanitization and output

### Dynamic strings in markup

- **Translatable strings**: `t()` (procedural) or `$this->t()` (in classes). In Twig, use the `|t` filter (`|trans` is an **alias of the same filter** — same behavior).
- **Plural / multi-variable translatable strings**: use the **`{% trans %}` block** in Twig (it's a block, not a filter — see `references/TWIG.md`). Inside the block, the `placeholder` filter is the equivalent of the `%` prefix used with `t()`.
- **Strings with HTML inserted by user**: pass through `Xss::filter()` or `Xss::filterAdmin()`.
- **Strings escaped as plain text**: Twig escapes by default with `{{ var }}`.

### Twig auto-escape

Twig automatically escapes variable output:

```twig
{# Drupal does HTML escape automatically. #}
<p>{{ user_input }}</p>

{# If you want unescaped output (ONLY if you trust the source). #}
<p>{{ trusted_html|raw }}</p>
```

### Markup objects in Drupal

- **`Markup::create($string)`**: marks a string as "safe markup" — not escaped on render.
- **`FormattableMarkup`**: for strings with placeholders (similar to `t()` but without translation).

---

## 11. Markup that must NOT be hardcoded

Things that **NEVER** go directly in markup in Drupal:

- **URLs**: use `\Drupal\Core\Url::fromRoute()` or Twig filters.
  ```twig
  {# ✅ Good #}
  <a href="{{ path('user.page') }}">My account</a>

  {# ❌ Bad #}
  <a href="/user">My account</a>
  ```

- **Asset paths** (CSS, JS, images in modules/themes): use libraries, asset paths, or Twig filters.

- **Text strings**: use `{{ 'My text'|t }}` (`|trans` is an alias of the same filter — same behavior).

- **CSRF tokens**: Form API handles them automatically.

- **Inline styles**: put them in CSS files with classes. Exception: runtime-computed values (e.g. `width: 63%` on a progress bar) — add via JS over JS-hook classes.

---

## Markup review checklist

### Structure
- [ ] **Document-level templates only** (`html.html.twig`, `maintenance-page.html.twig`, `install-page.html.twig`, custom full-page templates): start with `<!DOCTYPE html>`
- [ ] **Fragment templates** (`page`, `region`, `block`, `node`, `field`, `paragraph`, `views-view*`, menu, form/element, custom `hook_theme()` templates): **no doctype**, no `<html>`/`<body>` wrapper
- [ ] `<html lang>` and `<html dir>` present in the document-level template (via Drupal vars)
- [ ] `<meta charset="utf-8">` in `<head>`
- [ ] `<meta name="viewport">` (best practice for responsive themes — not part of the official Markup Style Guide, but Drupal core's `html.html.twig` includes it)
- [ ] A single `<h1>` per page
- [ ] Logical heading hierarchy

### Semantics
- [ ] Correct landmarks (`<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`)
- [ ] `<article>` for independent content
- [ ] `<section>` with its own `<h*>`
- [ ] Lists with `<ul>`/`<ol>`/`<dl>`
- [ ] `<time datetime>` for dates
- [ ] `<button type="...">` always explicit
- [ ] `<table>` only for data, with `<caption>`/`<thead>`/`<th scope>`

### Attributes
- [ ] Double quotes on values
- [ ] Boolean attributes without value (`required`, not `required="required"`)
- [ ] `id` unique per page
- [ ] `for` in `<label>` points to existing input
- [ ] `aria-*` correct when used

### Naming and CSS hooks
- [ ] Classes in `.component`, `.component__sub`, `.component--variant`
- [ ] States with `.is-*`
- [ ] JS hooks with `.js-` prefix and NOT styled
- [ ] No generic classes (`.content`, `.title`, `.blue-box`)
- [ ] No `id` as CSS hook

### Indentation
- [ ] 2 spaces, no tabs
- [ ] No trailing whitespace
- [ ] Final newline
- [ ] Consistent nesting

### Drupal patterns
- [ ] URLs via `path()` or `Url::fromRoute()`, not hardcoded
- [ ] Strings via `t()` / `$this->t()` / `|t`
- [ ] Structured render arrays, no HTML concatenation
- [ ] Template hooks registered with `hook_theme()`
- [ ] Templates with docblock of available variables
- [ ] `{{ attributes }}` printed to allow extension from modules
- [ ] Drillable attributes: explicit `class` + `{{ attributes }}` at the end

### Sanitization
- [ ] Twig auto-escape NOT bypassed with `|raw` without justification
- [ ] User HTML filtered with `Xss::filter*`
- [ ] No inline styles (except runtime values)
- [ ] No inline JavaScript (`onclick`, etc.)

### Accessibility (cross-reference with ACCESSIBILITY.md)
- [ ] `<img alt>` present
- [ ] `<label>` associated with each input
- [ ] Visible focus
- [ ] Skip links
- [ ] Sufficient contrast
- [ ] No info only by color

---

## Official resources

- [Drupal Markup Style Guide (groups.drupal.org)](https://groups.drupal.org/node/6355)
- [HTML Living Standard](https://html.spec.whatwg.org/)
- [Drupal Theming guide](https://www.drupal.org/docs/develop/theming-drupal)
- [Drupal Render API](https://www.drupal.org/docs/drupal-apis/render-api)
