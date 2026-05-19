# TWIG — Drupal Coding Standards Reference

Official consolidation of:
- [Twig coding standards (Drupal)](https://project.pages.drupalcode.org/coding_standards/twig/coding/)
- [Preprocess function documentation standards](https://www.drupal.org/node/1913208)
- Base: [Official Twig coding standards](https://twig.symfony.com/doc/3.x/coding_standards.html) + [Twig documentation](https://twig.symfony.com/documentation)

> **Most** Twig coding standards live on the Twig site. This file covers **what's specific to Drupal**.

---

## Table of contents

1. [DocBlock at the top of the template](#1-docblock-at-the-top-of-the-template)
2. [Variables in the DocBlock](#2-variables-in-the-docblock)
3. [Expressions](#3-expressions)
4. [HTML attributes (drillable)](#4-html-attributes-drillable)
5. [Whitespace control](#5-whitespace-control)
6. [Filters](#6-filters)
7. [Comments](#7-comments)
8. [Drupal-specific patterns](#8-drupal-specific-patterns)
9. [Autoescape and XSS prevention](#9-autoescape-and-xss-prevention)
10. [Review checklist](#10-review-checklist)

---

## 1. DocBlock at the top of the template

A docblock at the top of a Twig template must be **identical** to that of a PHPTemplate file ([Drupal API documentation standards for theme template files](https://project.pages.drupalcode.org/coding_standards/php/documentation/#theme-template-files)), but **the entire docblock wrapped in Twig comment markers `{#` and `#}`**.

```twig
{#
/**
 * @file
 * Default theme implementation for a region.
 *
 * Available variables:
 * - content: The content for this region, typically blocks.
 * - attributes: Remaining HTML attributes for the element, including:
 *   - class: HTML classes that can be used to style contextually through CSS.
 *
 * @see template_preprocess_region()
 *
 * @ingroup themeable
 */
#}
```

### `@ingroup themeable` — when to include it

- **ONLY** in templates that **provide the default themeable output** (base template of the module that defined the theme hook).
- **Do NOT include** in templates that override default output (in themes overriding a core/contrib template).

---

## 2. Variables in the DocBlock

### Reference by name

Variables in the docblock **are referenced by name**:
- **NOT** surrounded by `{{` and `}}` (they are not print indicators here).
- **NO** PHP `$`.
- **NO** separate "Other variables" section.

### Rule for converting PHP docs to Twig

1. Start from what was in Drupal 7.
2. If you see an "Other variables" section, **delete the title and the extra line above**.
3. If vars are deleted from preprocess, **delete them from the Twig docs too**.
4. If a var is useful and **isn't in Twig docs**, document it.
5. If you can **improve or reduce the verbosity** of D7 docs, do it.

### Variable types — do NOT document

Variable definitions **do NOT include type info** (array, object, string). In templates the type **shouldn't matter** — everything that gets printed ends up as a string.

### Variables referenced in-line in paragraphs

When a variable is referenced **inside a docblock paragraph**, the name goes **in single quotes**:

```twig
{#
 * Field variables: for each field instance attached to the node a corresponding
 * variable is defined; for example, 'node.body' becomes 'body'. When needing to
 * access a field's raw values, developers/themers are strongly encouraged to
 * use these variables. Otherwise they will have to explicitly specify the
 * desired field language; for example, 'node.body.en', thus overriding any
 * language negotiation rule that was previously applied.
#}
```

---

## 3. Expressions

Twig expressions are very similar to PHP expressions. In Drupal they are mainly used for:

### Checking variables available for printing

To print a markup wrapper **only if there is a value**:

```twig
{% if foo %}
  <div>{{ foo }}</div>
{% endif %}
```

**Prefer `{% if foo %}` over `{% if foo is defined %}`** for the common case of rendering a wrapper only when a value is present. `{% if foo %}` evaluates falsy values (empty string, `0`, `null`, empty array, `false`) consistently — which is what most templates actually want.

Use `is defined` (or `default(...)`) **only** when you need to distinguish "variable not provided at all" from "variable provided but falsy". This rare distinction matters mostly in:

- Templates that may be rendered with `strict_variables` enabled (array access on an unset variable throws).
- Preprocess-driven flags where `false` means "explicitly hidden" and undefined means "fall back to default".

See [#953034](https://www.drupal.org/project/drupal/issues/953034) for the historical discussion and workarounds.

### Looping

Twig uses **`for` loops** (in Drupal we are used to `foreach` in PHP).

**Without keys**:

```twig
{% for item in navigation %}
  <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
{% endfor %}
```

**With keys + values**:

```twig
{% for key, value in items %}
  <div class="{{ key }}">{{ value }}</div>
{% endfor %}
```

### Setting variables

```twig
{% set list = ['Alice', 'Bob'] %}
{% set text = ':person is a Twig fan'|t({':person': list[0]}) %}
```

---

## 4. HTML attributes (drillable)

In Drupal 8+, HTML attributes are **drillable**: you can print them all together (`{{ attributes }}`) or one by one.

### Default — print all

```twig
<div{{ attributes }}>
  {{ content }}
</div>
```

### Drilled (manual)

If you print individual attributes, **you MUST include the full `{{ attributes }}` at the end** so attributes added by modules still come through:

```twig
<div id="{{ attributes.id }}" class="{{ attributes.class }}"{{ attributes }}>
  {{ content }}
</div>
```

### Drupal core convention

**In Drupal core**, **only the `class` attribute** is printed explicitly. The rest go via `{{ attributes }}`:

```twig
<div class="{{ attributes.class }}"{{ attributes }}>
  {{ content }}
</div>
```

**Reason**: front-end devs must be able to add a class **anywhere**, easily, without having to understand the preprocess layer.

### Advanced theme devs

Can add classes in preprocess and remove the separate `class=""` to avoid the empty `class=""` output when there are no classes:

```twig
<div{{ attributes }}>
  {{ content }}
</div>
```

---

## 5. Whitespace control

> This section of the official docs is marked as **outdated** and needs updating. See [#3094850](https://www.drupal.org/project/drupal/issues/3094850) and [Better white-space control in Twig templates](https://symfony.com/blog/better-white-space-control-in-twig-templates).

Twig has two features for whitespace:

### Spaceless filter (with `apply` tag)

Removes whitespace **between** HTML and Twig tags. **Preferred in Drupal core** for code blocks.

Each use introduces **an extra indentation level**.

```twig
{% if block.show %}
<div class="admin-panel">
  {% apply spaceless %}
    {% if block.title %}
      {{ title_prefix }}
      <h3>{{ block.title }}</h3>
      {{ title_suffix }}
    {% endif %}
  {% endapply %}

  {% if block.content %}
    <div class="body">{{ block.content }}</div>
  {% elseif block.description %}
    <div class="description">{{ block.description }}</div>
  {% endif %}
</div>
{% endif %}
```

Expected output:

```html
<div class="admin-panel">
  <h3>Title</h3>
  <div class="body">Content</div>
</div>
```

### Dash modifier (`-`)

More precise but more confusing. When placed **inside** Twig tags, removes whitespace **outward** in the direction of the dash. Syntax: `{%-` / `-%}` on statements, `{{-` / `-}}` on expressions.

```twig
{# Input. #}
<ul>
  {%- for item in items -%}
    <li>{{- item -}}</li>
  {%- endfor -%}
</ul>

{# Output (no whitespace between <li> and its content, no blank lines between siblings). #}
<ul><li>a</li><li>b</li><li>c</li></ul>
```

**Not frequently used** in core — the spaceless filter is more readable.

### DOs and DON'Ts

- ✅ **If you can remove whitespace readably, do it**.
- ✅ Remove space before attributes: `<div{{ attributes }}>`
- ❌ **NEVER** remove spaces or add whitespace controllers around classes: `class="no {{ attributes.class }} no"`
- ✅ If you can't remove it readably, consider spaceless. Examples:
  - Apply spaceless around **commands** (`{% if foo %} ...`)
  - Apply spaceless around **comments** (`{# this is a comment #}`)

### Caveat: newlines at end of files

Git requires a newline at end of the Twig file. This may break tests or unwanted output. Solutions:

- **Change the test** if it needs to pass.
- Or **add a Twig template tag at the end**.

Outside Drupal core/contrib, **you can just remove the newline**.

---

## 6. Filters and operators

Common Drupal functions like `t` and `url` are available as **filters** in Twig templates. Filters are triggered with `|`.

```twig
<div class="preview-image-wrapper">
  {{ 'Original'|t }}
</div>
```

### Operator spacing (from the [official Twig coding standards](https://twig.symfony.com/doc/3.x/coding_standards.html))

- **No spaces around `|`** (filter pipe): `{{ name|upper }}`.
- **No spaces inside `()` for filter/function calls**: `{{ foo(bar) }}`, not `{{ foo( bar ) }}`.
- **One space around binary operators** including `+`, `-`, `*`, `/`, `%`, `==`, `<`, `>`, `~`, `and`, `or`, `in`, `is`.
- **One space after `,` and `:`** inside hashes/arrays: `{ key: value, other: other_value }`.

### String concatenation — `~`, never `+`

Twig has **no** numeric `+` operator on strings. `'a' + 'b'` performs numeric addition and returns `0`. Use `~` for concatenation:

```twig
{# ✅ Correct. #}
<a href="{{ '/node/' ~ node.id }}">…</a>

{# ❌ Wrong — yields the integer 0. #}
<a href="{{ '/node/' + node.id }}">…</a>
```

### Tests: `is defined`, `is empty`, `is null`

| Test | Returns `true` when… |
|---|---|
| `foo is defined` | The variable exists in scope (even if `false`/`null`/empty). |
| `foo is null` | The value is strictly `null`. |
| `foo is empty` | The value is `null`, `false`, `0`, `""`, `[]`, or `{}` (mirrors PHP `empty()`). |
| `if foo` | The value is truthy (NOT null, NOT false, NOT `0`, NOT empty string/array). |

Most templates want `{% if foo %}` (truthiness). Reach for `is empty` only when you need to treat `0` or `false` as "has a value". Reach for `is defined` only when distinguishing "unset" from "falsy" — see §3 "Expressions" above.

---

## 7. Comments

**All comments** are surrounded by `{#` and `#}`.

### Single-line

Markers on the **same line** as the comment:

```twig
<div class="image-widget-data">
  {# Render widget data without the image preview that was output already. #}
  {{ data|without('preview') }}
</div>
```

### Multi-line

Markers on **separate lines**. Comments wrap to **< 80 chars**:

```twig
{#
  This is a very long comment. It spans more than one line. This is a very
  long comment. It spans more than one line. This is a very long comment. It
  spans more than one line. This is a very long comment. It spans more than
  one line.
#}
<div class="{{ attributes.class }}"{{ attributes }}>
  {{ content }}
</div>
```

---

## 8. Drupal-specific patterns

### Template file naming

- Lowercase letters; **hyphens** as separators within and between suggestion parts.
- Always end in `.html.twig`.
- The base template hook (`node`, `field`, `paragraph`, etc.) is followed by `--` + specifier parts in order from most general to most specific.

Drupal **automatically converts underscores in the theme-hook suggestion to hyphens** in the filename. So a suggestion `node__article__full` resolves to:

```text
node--article--full.html.twig
```

Examples of the suggestion → filename mapping:

| Theme-hook suggestion | Template filename |
|---|---|
| `node__article` | `node--article.html.twig` |
| `node__article__teaser` | `node--article--teaser.html.twig` |
| `field__node__field_image` | `field--node--field-image.html.twig` |
| `block__system_branding_block` | `block--system-branding-block.html.twig` |
| `paragraph__hero` | `paragraph--hero.html.twig` |

Don't substitute underscores for hyphens by hand — let the theme registry generate the suggestion list and pick the filename that matches.

### Template overrides

When overriding a core/contrib template:

- **Do NOT include** `@ingroup themeable` in the override.
- **Include** a `@see` pointing to the original template or preprocess function.

```twig
{#
/**
 * @file
 * Theme override for a region.
 *
 * Available variables:
 * - content: The content for this region, typically blocks.
 * - attributes: Remaining HTML attributes for the element.
 *
 * @see template_preprocess_region()
 * @see region.html.twig
 */
#}
```

### Translation

Use the `|t` filter:

```twig
<h2>{{ 'Welcome back!'|t }}</h2>

{# With placeholders — @ escapes plain text. #}
<p>{{ 'Hello @name'|t({'@name': user.name}) }}</p>

{# With placeholders — % escapes AND wraps in <em class="placeholder">. #}
<p>{{ 'You are editing %title'|t({'%title': node.label}) }}</p>

{# With placeholders — : URL-encodes (valid only as an attribute value, e.g. href). #}
<a href="{{ ':link'|t({':link': url('user.page')}) }}">Profile</a>
```

> **Do not use `!`-prefixed placeholders** — they bypass escaping and are a 🔴 XSS risk. See section 9 (Autoescape and XSS prevention).

### Trans block for complex texts

For long strings with plurals or multiple variables:

```twig
{% trans %}
  Hello {{ name }}, you have {{ count }} new messages.
{% plural count %}
  Hello {{ name }}, you have {{ count }} new messages.
{% endtrans %}
```

### URL generation

Use the `url()` helper:

```twig
<a href="{{ url('entity.node.canonical', {'node': node.id}) }}">
  {{ node.label }}
</a>
```

### Render arrays as child elements

```twig
{# Render entire array #}
{{ content }}

{# Render single element #}
{{ content.field_name }}

{# Render element with options #}
{{ content|without('comments') }}
```

### `set` to prepare local markup

```twig
{% set classes = [
  'node',
  'node--type-' ~ node.bundle|clean_class,
  node.isPromoted() ? 'node--promoted' : '',
  node.isSticky() ? 'node--sticky' : '',
]|filter(v => v) %}

<article{{ attributes.addClass(classes) }}>
  {# ... #}
</article>
```

> Twig requires the `else` branch of a ternary (the short ternary `?:` works when both branches share the truthy value). Trailing empty strings are dropped with `|filter(v => v)` so `addClass()` never receives blanks.

### Conditional classes

```twig
{% set classes = ['button'] %}
{% if disabled %}
  {% set classes = classes|merge(['button--disabled', 'is-disabled']) %}
{% endif %}
{% if loading %}
  {% set classes = classes|merge(['is-loading']) %}
{% endif %}

<button{{ attributes.addClass(classes) }}>{{ label }}</button>
```

### `attach_library` for CSS/JS

```twig
{{ attach_library('my_theme/component-name') }}
```

---

## 9. Autoescape and XSS prevention

> **Severity note**: bypassing autoescape with unsafe input is a **🔴 Blocker** (XSS), not a 🟡 standards violation. Source: Drupal security advisories on Twig output (e.g. SA-CORE-2018-002).

### Autoescape is ON by default in Drupal

Drupal core configures Twig with **autoescape enabled** in HTML context. Every `{{ variable }}` is HTML-escaped automatically before rendering:

```twig
{# Safe by default — autoescape will encode <, >, &, ', ". #}
<p>{{ user_input }}</p>
```

This is a deliberate Drupal-specific choice (vanilla Twig leaves autoescape off by default). **Do not disable it** at the template or environment level in custom code — Drupal's render pipeline assumes it.

### What disables / bypasses autoescape

The following turn an escaped string into raw HTML. Each is an XSS risk if the source is **not** known-safe:

| Construct | Effect | Severity if source is user-controlled |
|---|---|---|
| `{{ var\|raw }}` | Print without escape | 🔴 Blocker |
| `{% autoescape false %} … {% endautoescape %}` | Disables autoescape inside the block | 🔴 Blocker |
| `t('msg', {'!unsafe': $value})` | `!`-prefixed placeholder is **not** escaped | 🔴 Blocker |
| `Markup::create($string)` (PHP side) | Marks the string as safe — Twig prints it raw | 🔴 if `$string` contains user input |
| `FormattableMarkup` with raw value | Same as above when passed to render | 🔴 if value is user-controlled |
| `{{ var\|render }}` on a render array with `#markup` | If `#markup` contains unfiltered user HTML, it leaks | 🔴 |

### When `|raw` is acceptable

Only when **the source is trusted markup** that has already been sanitized or is fully controlled by the codebase:

- Markup built from a `Markup::create()` call after `Xss::filter()` / `Xss::filterAdmin()`.
- Static markup hard-coded in the module/theme (no user input path).
- Output of a Drupal renderer that already produced safe `MarkupInterface`.

When you use `|raw`, **leave a comment explaining why it is safe**:

```twig
{# Safe: $admin_html is the output of Xss::filterAdmin(), only admin roles can edit it. #}
{{ admin_html|raw }}
```

### Translation placeholders

`t()` / `|t` accept three placeholder styles. **Match the placeholder to the data**:

| Prefix | Behaviour | Use when |
|---|---|---|
| `@variable` | HTML-escaped | The value is plain text from any source (default, safest). |
| `%variable` | HTML-escaped **and** wrapped in `<em class="placeholder">…</em>` | The value is plain text that you want visually emphasised. |
| `:variable` | URL-encoded; only valid as an HTML attribute value (e.g. `href`) | The value is a URL. Twig + `t()` will fail loudly if you misuse it. |
| `!variable` | **Not escaped** at all | **Removed.** Drupal 11's [`FormattableMarkup`](https://api.drupal.org/api/drupal/core%21lib%21Drupal%21Component%21Render%21FormattableMarkup.php/class/FormattableMarkup/11.x) only documents `@`, `%`, `:`. Any remaining `!placeholder` in legacy code is 🔴 — replace it. |

```twig
{# ✅ Good — @ placeholder escapes the username. #}
<p>{{ 'Welcome, @user'|t({'@user': account.name}) }}</p>

{# ✅ Good — : placeholder for an href. The source string is a real translatable phrase. #}
<a href="{{ profile_url }}">{{ 'View profile'|t }}</a>
{# Or with a single t() call that embeds the URL: #}
<p>{{ 'See <a href=":url">your profile</a>'|t({':url': profile_url}) }}</p>

{# 🔴 Bad — ! is no longer supported by t() / FormattableMarkup. #}
<p>{{ 'Hello !name'|t({'!name': user.name}) }}</p>
```

### `{% autoescape %}` blocks

Drupal templates **should not** open `{% autoescape false %}`. If you find yourself wanting one, the underlying problem is almost always a render-array shape issue (the value should be a render array or `MarkupInterface`, not raw HTML at the template layer).

The escape strategy can be tightened with `{% autoescape 'html' %}` (the default) or scoped to a different context (`'html_attr'`, `'js'`, `'css'`, `'url'`). Use the context-specific form when emitting a string into a non-HTML position — for example, a JSON literal inside a `<script>` tag.

### Sanitization helpers (PHP side, for reference)

When you must build markup in PHP and hand it to Twig:

| Helper | Use for |
|---|---|
| `\Drupal\Component\Utility\Html::escape($string)` | Plain-text escape (`htmlspecialchars`-equivalent). |
| `\Drupal\Component\Utility\Xss::filter($string)` | Strip dangerous tags, keep a safe whitelist (links, basic formatting). |
| `\Drupal\Component\Utility\Xss::filterAdmin($string)` | More permissive whitelist for admin-only input. |
| `\Drupal\Core\Render\Markup::create($safeString)` | Wrap a string you have already sanitized so Twig prints it unescaped. **Do not feed user input directly.** |
| `\Drupal\Component\Render\FormattableMarkup` | Format with `t()`-style placeholders (`@`, `%`, `:`) **without** translation. |

---

## 10. Review checklist

### Structure

- [ ] **DocBlock `@file`** wrapped in `{# ... #}` at the start
- [ ] Summary, description, `Available variables:`, `@see`, `@ingroup themeable` (if applies)
- [ ] Variables documented by name, **no types**, **no `$`**, **no `{{ }}`**
- [ ] `@ingroup themeable` **only** in base template, not in overrides

### Variables

- [ ] In-line variables in paragraphs: **single quotes** (`'node.body'`)
- [ ] **No separate "Other variables"** section
- [ ] Docblock variables match what the preprocess function provides

### Expressions

- [ ] `{% if foo %}` (without `is defined`)
- [ ] `{% for item in items %}` (without keys) or `{% for key, value in items %}` (with keys)
- [ ] `{% set var = value %}` for local variables

### HTML attributes

- [ ] If printing individual attributes, `{{ attributes }}` also at the end
- [ ] **Core convention**: print `class` separately and the rest via `{{ attributes }}`
- [ ] `attributes.addClass()` to add classes from Twig
- [ ] `attributes.removeClass()` and `attributes.setAttribute()` when needed

### Whitespace

- [ ] **Prefer `{% apply spaceless %}`** over dash modifier
- [ ] **Do NOT** remove whitespace around `class=""`
- [ ] Final newline if the system requires (git/core), or final template tag

### Filters

- [ ] **No spaces** around the pipe `|`
- [ ] `|t` (or its alias `|trans` — same filter) for UI strings
- [ ] `{% trans %}…{% plural %}…{% endtrans %}` **block** for plurals or natural-text strings with multiple variables (this is a block, not a filter)
- [ ] `|clean_class` to sanitize dynamic class names
- [ ] `|without('key')` to exclude keys from render arrays

### Comments

- [ ] **Single line**: `{# text #}` on the same line
- [ ] **Multi-line**: `{#` and `#}` on separate lines
- [ ] Multi-line comments wrapped to <80 chars

### Drupal-specific

- [ ] UI strings with `|t` filter
- [ ] URLs with `url()` helper, NO hardcoded
- [ ] Libraries attached with `{{ attach_library('...') }}`
- [ ] Override templates with `@see` to the original
- [ ] Render arrays rendered correctly
- [ ] Dynamic classes via `attributes.addClass()` (not concatenating strings)

### Autoescape and XSS (🔴 if violated with user input)

- [ ] **No `{% autoescape false %}` blocks** in custom templates
- [ ] **`|raw` only on values that are demonstrably safe** (output of `Xss::filter*`, hard-coded markup, or `MarkupInterface`); accompanied by a comment justifying why
- [ ] **No `!placeholder` in `t()` / `|t`** — use `@` (text), `%` (emphasised text), or `:` (URL)
- [ ] No raw HTML built via string concatenation in PHP and printed unescaped
- [ ] `Markup::create()` / `FormattableMarkup` calls trace back to sanitized input
- [ ] Render arrays preferred over `#markup` with HTML strings (let Drupal handle escaping)

### Accessibility and semantics

- [ ] Use HTML5 landmarks (`<header>`, `<nav>`, `<main>`, `<footer>`)
- [ ] Hierarchical headings (`<h1>` → `<h2>` → `<h3>`)
- [ ] Form elements associated with `<label for="...">` or `aria-label`
- [ ] Images with `alt` (empty `alt=""` if decorative)
- [ ] Lang attribute for content in another language
- [ ] See `references/ACCESSIBILITY.md` for full WCAG checklist
- [ ] See `references/MARKUP.md` for HTML structure

---

## Useful Drupal functions and filters in Twig

| Filter / Function | Use |
|---|---|
| `\|t` | Translate string |
| `\|t({'@name': value})` | Translate with safe-escape placeholder (`@`) |
| `\|t({'%name': value})` | Translate with placeholder + wrap in `<em>` (`%`) |
| `\|t({'!name': value})` | Translate with unescaped placeholder (`!`) — **Removed** in modern Drupal. Will not be processed by `FormattableMarkup`. |
| `\|safe_join(', ')` | Join array as safe markup |
| `\|without('key', 'key2')` | Exclude keys from render array |
| `\|clean_class` | Sanitize string to valid class name |
| `\|clean_id` | Sanitize string to valid id |
| `\|raw` | Output without escape (careful with XSS) |
| `\|render` | Force render of render array to string |
| `\|format_date('format')` | Format timestamp |
| `\|striptags` | Remove HTML tags |
| `url('route.name', {param: value})` | Generate URL from route |
| `path('route.name', {param: value})` | Path-only from route |
| `link(text, url)` | Generate `<a>` markup |
| `attach_library('module/library')` | Attach library |
| `active_theme()` | Get active theme name |
| `active_theme_path()` | Get active theme path |
| `file_url('path/to/file')` | Generate URL for public file |

---

## External resources

- [Official Twig coding standards](https://twig.symfony.com/doc/3.x/coding_standards.html)
- [Full Twig documentation](https://twig.symfony.com/documentation)
- [Theme system overview in Drupal](https://www.drupal.org/docs/theming-drupal)
- [Filter and function reference](https://www.drupal.org/docs/theming-drupal/twig-in-drupal/functions-in-twig-templates)
