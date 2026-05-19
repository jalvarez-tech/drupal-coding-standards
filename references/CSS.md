# CSS — Drupal Coding Standards Reference

Official consolidation of:
- [CSS coding standards](https://project.pages.drupalcode.org/coding_standards/css/coding/)
- [CSS formatting guidelines](https://project.pages.drupalcode.org/coding_standards/css/format/)
- [CSS architecture (for Drupal 9)](https://project.pages.drupalcode.org/coding_standards/css/architecture/)
- [CSS file organization](https://project.pages.drupalcode.org/coding_standards/css/file-organization/)
- [What to look for when reviewing CSS](https://project.pages.drupalcode.org/coding_standards/css/review/)
- [CSScomb settings for Drupal](https://project.pages.drupalcode.org/coding_standards/css/csscomb/)

Based on: [Principles of writing consistent, idiomatic CSS](https://github.com/necolas/idiomatic-css) (Nicolas Gallagher), [SMACSS](http://smacss.com) (Jonathan Snook), [OOCSS](http://www.stubbornella.org/content/2009/02/12/css-doesn’t-suck-you’re-just-doing-it-wrong/) (Nicole Sullivan).

> "Part of being a good steward to a successful project is realizing that writing code for yourself is a Bad Idea™. If thousands of people are using your code, then **write your code for maximum clarity**." — Idan Gazit

---

## Table of contents

1. [Whitespace](#1-whitespace)
2. [Comments](#2-comments)
3. [Direction-specific rules and RTL](#3-direction-specific-rules-and-rtl)
4. [Rulesets](#4-rulesets)
5. [Properties](#5-properties)
6. [Declaration order](#6-declaration-order)
7. [Architecture — Goals](#7-architecture--goals)
8. [The Component](#8-the-component)
9. [Common pitfalls](#9-common-pitfalls)
10. [Best practices](#10-best-practices)
11. [SMACSS categories](#11-smacss-categories)
12. [Class naming conventions](#12-class-naming-conventions)
13. [Specificity, ids, !important](#13-specificity-ids-important)
14. [File organization](#14-file-organization)
15. [Aggregation](#15-aggregation)
16. [Review checklist](#16-review-checklist)
17. [CSScomb (automated formatter and sorter)](#17-csscomb-automated-formatter-and-sorter)

---

## 1. Whitespace

### Indentation

- **2 spaces** per level (same standard as Drupal PHP and JS).
- **Declarations** indented one level relative to the selector.
- **Rulesets within a media query** indented one level relative to the media statement.
- **Comments** maintain the indentation of their declaration/ruleset.

```css
.tabs__tab {
  display: none;
  margin: 0;
  margin-block-end: calc(-1 * var(--tabs-border-width));

  &.is-active {
    display: flex;
  }
}
```

### Blank lines

- In general, **separate each ruleset with a blank line** when using PostCSS.
- If a ruleset has a **preceding comment (Doxygen or single-line)**, leave a **blank line before the comment**.
- If two rulesets **have no blank line between them**, **they must be logically related**. If not, add a blank line + comment describing the second one.

```css
.tabs {
  --tabs-height: var(--sp3);
  --tabs-padding-inline: var(--sp1-5);
  --tabs-active-border-size: 6px;
  --tabs-highlight-color: var(--color--primary-50); /* Minimum 3:1 contrast ratio against --tabs-background-color and --tabs-background-color-hover. */

  display: flex;
  flex-direction: column;
}

/**
 * I'm a Doxygen-style comment. Hooray!
 */
.tabs__tab {
  display: none;
  margin: 0;
  margin-block-end: calc(-1 * var(--tabs-border-width));

  /* Show tabs when JavaScript disabled. */
  @nest html:not(.js) & {
    display: flex;
  }
}
```

### Line endings

- **NO** whitespace at end of line.
- Files end with **a single blank line**.
- **Unix line endings (`\n` or `LF`)** — also macOS default.
- Configure editor to "show invisibles".
- Drupal 8+ includes **`.editorconfig`** at root to maintain this.

---

## 2. Comments

Drupal borrows comment styles from the [PHP Doxygen standards](https://project.pages.drupalcode.org/coding_standards/php/documentation/).

### File comments

Each file starts with a **comment describing what it does**. **Blank line** after the file comment. **80 columns** max where possible. Use the Doxygen `/**` opener (two asterisks) — the `@file` tag only renders correctly with a docblock.

```css
/**
 * @file
 * Example CSS file to demonstrate all variations of CSS comments.
 */
```

### Single line comments describing a ruleset

Short comments describing a ruleset = **one line**.

```css
/* Short description of the ruleset. */
.my-component {
  display: block;
}
```

### Multi-line comments describing a ruleset

Comments of 2+ lines describing ruleset(s) use **Doxygen comment style** (docblock):

```css
/**
 * This is a multiline comment that describes an entire ruleset (even if
 * nested). We try to make it not go very far over 80 characters. We want to
 * make it very verbose to help out future developers.
 */
&[aria-expanded="true"] {
  visibility: visible;
}
```

### Multi-line comments inside a ruleset

Within a ruleset, multi-line comments start with `/*` and end with `*/`. **Text indented to maintain alignment**:

```css
.component {
  /* This is a multiline comment WITHIN a ruleset. It describes the next rule
     when there is not enough space on the same line as the rule. We try to
     make it not go very far over 80 characters. We want to make it very verbose
     to help out future developers. */
  visibility: hidden;
}
```

### Single-line comments

Short inline comments: simple CSS comment.

```css
.component {
  position: relative; /* Anchor pseudo-element. */
  z-index: 502; /* Appear above toolbar. */
}
```

### Complete example of all styles

```css
/**
 * @file
 * Example CSS file to demonstrate all variations of CSS comments. This file uses
 * modern syntax including nesting, and `:dir()` pseudo-class that PostCSS
 * transpiles into browser-readable CSS.
 */

.component {
  position: relative; /* Anchor pseudo-element. */
  z-index: 502; /* Appear above toolbar. */
  /* This is a multiline comment WITHIN a ruleset. It describes the next rule
     when there is not enough space on the same line as the rule. */
  visibility: hidden;
  transform: translateX(50%); /* LTR */
  border-radius: var(--border-radius);
  will-change: transform; /* Needed for Safari 16. */

  /*
   * This is a multiline comment that describes an entire ruleset (even if
   * nested). We try to make it not go very far over 80 characters.
   */
  &[aria-expanded="true"] {
    visibility: visible;
  }

  /* Short comment outside ruleset. */
  &::marker {
    display: none;
  }

  &:dir(rtl) {
    transform: translateX(-50%);
  }
}
```

---

## 3. Direction-specific rules and RTL

For properties that **have no logical properties equivalent** (e.g. `transform`), add a **`/* LTR */` comment** on the same line preceded by a single space. Follow with an additional ruleset using **either** the `[dir="rtl"]` attribute selector **or** the `:dir(rtl)` pseudo-class:

```css
/* Option A — [dir="rtl"] selector. Works in every browser without transpilation. */
.my-component {
  transform: translateX(20px); /* LTR */
}

[dir="rtl"] .my-component {
  transform: translateX(-20px);
}

/* Option B — :dir(rtl) pseudo-class with PostCSS nesting. */
.my-component {
  transform: translateX(20px); /* LTR */

  &:dir(rtl) {
    transform: translateX(-20px);
  }
}
```

Both are acceptable. Drupal core themes (Olivero, Claro) use `:dir()` with PostCSS to transpile down to `[dir="rtl"]` for older browser support; if your build pipeline doesn't include PostCSS, use the `[dir="rtl"]` form directly.

**Prefer logical properties when available** (`margin-inline-start`, `padding-block-end`, `inset-inline`, etc.) — they remove the need for an RTL override entirely.

---

## 4. Rulesets

- **One selector per line** when there is a group separated by commas.
- When possible, use **functional pseudo-classes** like `:is()`, `:not()`, `:where()`.
- **One declaration per line** in the block.

```css
/**
 * Example of one selector per line.
 */
.my-component,
.my-other-component,
.yet-another-component {
  background-color: #cccccc;
}

/**
 * Example of the :is() pseudo-class.
 */
:is(.my-component, .my-other-component, .yet-another-component) {
  background-color: #cccccc;
}
```

---

## 5. Properties

- **No space before the colon, exactly one space after**: `color: red;` (not `color :red;`, not `color:red;`, not `color  : red;`).
- **Semicolon at end of EVERY declaration**, even the last one.
- **Double quotes for values requiring quotes**: `font-family: "Arial Black", Arial, sans-serif;`, `content: " ";`.
- **rem units by default**, unless it creates an undesired effect.
  - With PostCSS: automatic via the `PxToRem` plugin.
- **Quote attribute values in selectors**: `input[type="checkbox"]`.
- **No units on zero-values**: `margin: 0;` NOT `margin: 0px;` (where allowed).
- **Space after each comma** in comma-separated values: `color: rgba(0, 0, 0, 0.8);`.
- **No spaces around parentheses** in functions.
- **Lowercase function names**: `rgba(...)` not `RGBA(...)`.

---

## 6. Declaration order

The order must make the **purpose obvious** of the declaration block. **Clarity is the guiding principle**.

1. **Positioning properties**: `position`, `float`, `clear`, `inset`, `top`, `right`, `bottom`, `left`, `direction`, `z-index`, and logical properties (`block-start`, `block-end`, `inline-start`, `inline-end`).
2. **Box model properties**:
   - `display`
   - **Sizing**: `block-size`, `inline-size` (logical), `width`, `height`, `(max|min)-` variants
   - **Margins** and logical equivalents, longhand forms (`margin-block`, `margin-top`, `margin-inline-end`…)
   - **Paddings** and logical equivalents, longhand forms
   - **Borders** and logical equivalents, longhand forms
   - `box-sizing`
3. **Other declarations**.

Within each group: alphabetical or group like-properties (font + text, for example). Drupal is deliberately vague here.

**Vendor-prefixed** properties go **directly before** the non-prefixed version (if autoprefixer doesn't do it automatically).

> The order reinforces the purpose of the ruleset. It's **more important to add comments** to the ruleset than to worry about perfect ordering.

---

## 7. Architecture — Goals

Well-architected CSS must be:

### 1. Predictable

CSS throughout Drupal core and contribs must be consistent and understandable. Changes must do what's expected without side-effects.

### 2. Reusable

> CSS rules should be abstract and decoupled enough that you can build new components quickly from existing parts without having to recode patterns and problems you've already solved. — Philip Walton

### 3. Maintainable

Add/modify/extend CSS without breaking or refactoring existing styles.

### 4. Scalable

Manageable for a single dev or for large distributed teams (like Drupal).

---

## 8. The Component

**Components** = discrete, purpose-built visual elements that make up UI. They consist of HTML + CSS + sometimes JS.

Examples: navbars, dialogs, buttons, carousels. They can be simple (icon, button) or composed of other components.

---

## 9. Common pitfalls

### ❌ Modifying components based on context

```css
/* WRONG: Modifying a component when it's in a sidebar */
.sidebar .component {}
```

Makes CSS **less predictable and maintainable**. Sooner or later you need the component outside the sidebar. Or a new dev puts it in the sidebar without expecting the appearance change.

### ❌ Relying on HTML structure

```css
/* Overly complex selectors */
nav > ul > li > a
article p:first-child

/* Qualified selectors */
a.button
ul.nav
```

Makes CSS **fragile** to markup changes and **hard to reuse**.

### ❌ Overly generic class names

```css
.widget {}
.widget .title {}
.widget .content {}
```

`.title` and `.content` are too generic. A standalone `.title` later will affect widgets unintentionally.

### ❌ Making a rule do too much

Positioning + margins + padding + colors + borders + text styles **all in a single rule** → impossible to reuse parts.

### ❌ Needing to undo styles

Rules like `.component-no-padding` indicate that **some rule is doing too much**. CSS becomes over-complex, hard to understand, hard to maintain, and bloated.

---

## 10. Best practices

### 1. Avoid reliance on HTML structure

- CSS must define the appearance of an element **anywhere and everywhere** it appears.
- **Use classes** to assign appearance to markup. **NEVER id selectors in CSS**.
- **Short selectors**. **The best selector is a single class or an element**.

Sometimes multi-part selectors are pragmatic:

```css
/**
 * Add a horizontal rule between adjacent list rows.
 */
.slat + .slat {
  border-top: 1px solid #cccccc;
}
```

Care with multi-part selectors:

1. **Avoid elements without native semantics** (`div`, `span`) in multi-part.
2. **Avoid descendant selector** (`.my-list li`) — affects unintended nested elements. Prefer child selector: `.my-list > li`.
3. **Maximum 2 combinators**: `.my-list > li > a` is the limit.
4. When in doubt, add a class.

### 2. Use component elements with their own classes

To avoid relying on markup structure and generic class names, **define component elements explicitly** prefixing with the component name + **double underscore** (`__`):

```css
.component {}

/* Component elements */
.component__header {}
.component__body {}
```

> **Don't reflect DOM structure in the class name**. Not `.menu__item__link`. The name `.menu__link` is specific enough.

### 3. Extend components using modifier classes

Variants with suffix + **double dash** (`--`). The modifier **only contains** the styles to extend the original. **Both classes (base and modifier) MUST appear together** in the markup:

```css
/* Button component */
.button {
  /* styles */
}

/* Button modifier class */
.button--primary {
  /* modifications and additions */
}
```

```html
<!-- Button variant created by applying both component and modifier classes -->
<button class="button button--primary">Save</button>
```

### 4. Separate concerns

- **Components are NOT responsible for their positioning or layout** within the site.
- **NEVER apply `width` or `height`** except to elements that natively have these properties (e.g. `<img>`).
- **Separate structural rules from stylistic rules** within components.
- **JavaScript hooks use dedicated classes**, NOT classes already used for CSS. Prefix `js-`. **These `js-` classes NEVER appear in stylesheets**.
- **Avoid inline styles via JS**. If the behavior describes state, apply a class with state semantics (`is-active`). Only use inline styles via JS when the style attribute value must be **computed at runtime**.

### 5. Name components using design semantics

Class names must reflect **design semantics over content semantics**. They reflect the **intent and purpose** of the design element.

✅ OK presentational class names:
- `.grid-3` (grid system)
- `.leader`, `.trailer` (baseline grid spacing)
- `.text-center`

❌ Not good:
- `.blue-box` (only reflects a superficial attribute)

✅ Reframe:
```css
.notification { /* general styles for all notifications */ }
.notification--info { /* blue color adjustments */ }
```

**For core developers**: core is design-agnostic, be cautious about what classes to include.

**For module developers**: use existing theme hooks and design-derived class names. If the module has no default design, use the module name (e.g. `class="views"`).

### 6. Formatting class names

- **Full words**, no abbreviations: `class="button"`, NOT `class="btn"`.
- **Dash between words** in components: `class="button-group"`, NOT `class="buttongroup"`.

Drupal naming convention:

```
/* Component Rules */
.component-name
.component-name--variant
.component-name__sub-object
.component-name__sub-object--variant  /* this configuration should be uncommon */

/* Layout Rules */
.layout-layout-method  /* e.g. '.layout-container' */
.grid-*

/* State Classes — always applied via JS, describe non-default state */
.is-state  /* e.g. '.is-active' */

/* Functional JS Hooks — dedicated classes not used for styling */
.js-behavior-hook  /* e.g. .js-slider, .js-dropdown */
```

> `js-` classes **don't appear in stylesheets**. Only used in JS.

---

## 11. SMACSS categories

Drupal 8/9+ uses SMACSS to categorize rules. **State and Theme are less used with modern CSS**.

### Base

- **Styling ONLY for HTML elements** (CSS reset, [normalize.css](https://github.com/necolas/normalize.css/)).
- **NEVER** includes class selectors.
- To avoid undoing in components, base styles reflect the **simplest possible appearance** of the element.
- E.g. simplest usage of `<ul>` = no list markers or indents, relying on component class for other usages.

### Layout

Arrangement of elements on the page: grid systems, region layout.

> Grid systems should be thought of as shelves. They contain content but are not content in themselves. — Harry Roberts

### Component (SMACSS "module")

Reusable and discrete UI elements. **Form the bulk** of Drupal CSS.

### State

Styles for transient appearance changes. Often client-side (hover, modal open). Sometimes server-set (active main nav). Ways:

- **Custom classes**, often via JS. Prefix `.is-`: `.is-transitioning`, `.is-open`.
- **Pseudo-classes**: `:hover`, `:checked`.
- **HTML attributes with state semantics**: `details[open]`.
- **Media queries**.

### Theme

Visual styling: borders, box-shadows, colors, backgrounds, font properties.

Ideally **separable enough from component structure** to be "swappable". Omitting them must not break functionality or basic usability.

---

## 12. Class naming conventions

### Component

```
.component-name
.component-name--variant
.component-name__sub-object
.component-name__sub-object--variant  /* uncommon */
```

### Layout

```
.layout-layout-method   /* .layout-container */
.grid-*
```

### State

```
.is-state   /* .is-active, .is-open, .is-disabled */
```

### JS hook

```
.js-behavior-hook   /* .js-slider, .js-dropdown */
```

> JS hooks NEVER appear in CSS. Prefer dedicated `.js-*` classes for query/manipulation. `id` is acceptable only when a single uniquely-targeted hook is needed for performance reasons (e.g. one-element-on-page lookups via `document.getElementById`).

---

## 13. Specificity, ids, !important

### `id` selector

**AVOID `id` in CSS**. No general benefit and seriously increases specificity.

The `id` attribute IS useful for:
- **Performant JavaScript hook**
- **Anchor** within the document (`<a href="#anchor">`)
- **Associating labels** with form elements
- **Associating DOM elements** via ARIA attributes

### `!important`

**Sparingly and appropriately**, generally **restricted to themes**.

Use for states that **must supersede all others**, regardless of component variant. Valid examples:
- **Error states**
- **Disabled states**

**NEVER** use `!important` to **resolve specificity problems** in general rules.

**Drupal core and contrib modules must avoid `!important`** because their CSS must be easy to override.

---

## 14. File organization

> Historical context: issue [#1921610](https://www.drupal.org/project/drupal/issues/1921610) framed file organization as "not yet implemented" in 2013. Current Drupal core (10/11) **does** implement SMACSS groupings via `.libraries.yml` categories (`base`, `layout`, `component`, `state`, `theme`), and the Olivero and Claro themes follow the structure documented below.

### SMACSS structure applied to files

1. **Base** — Element styling (a, ul, etc.), typography, root-scoped CSS custom properties, resets, utility classes.
2. **Layout** — Page template layout, region layout, etc.
3. **Component (SMACSS "module")** — Majority of a theme's CSS. Scoped to their own files. E.g. pager, navigation, footer.
4. **State** — Originally for modifying states, now best practice is to include them within the component stylesheet.
5. **Theme** — Originally for overall look-and-feel (colors), now done better by modifying CSS custom properties.

### CSS files for Drupal modules

Everything in `css/` sub-directory:

| File | Content |
|---|---|
| `module_name.module.css` | **Minimal** styles for the module's functionality to work. Layout + component + state. |
| `module_name.theme.css` | **Extra** styles to make functionality aesthetically pleasing. Usually theme styles. |
| `module_name.admin.css` | Minimal styles for admin screens. |
| `module_name.admin.theme.css` | Extra styles for admin screens. |

> **Modules NEVER have base styles**. Drupal core doesn't have them — uses normalize.css + drupal.base.css.

If the module attaches a CSS to a template, the CSS file is **named the same** as the template: `system-plugin-ui-form.html.twig` → `system-plugin-ui-form.css`.

### CSS files for Drupal themes

1. **Always separate Base, Layout, Component into their own files**. Drupal aggregates, no performance issue.
2. **Complex themes**: each component or family of components in its own file.
3. **State rules + media queries**: include with the component to which they apply.
4. **Theme rules**: may or may not have their own files. **Starter themes encouraged** to separate.

Minimal structure:

```yaml
css:
  base: base.css
  layout: layout.css
  component: components.css
```

More complex structure (mobile-first):

```yaml
css:
  base:
    css/base/normalize.css
    css/base/elements.css
  layout:
    css/layout/layout.css
    css/layout/layout--medium.css
    css/layout/layout--wide.css
  component:
    css/components/button.css
    css/components/dropdown.css
    css/components/pagination.css
    css/components/tabs.css
  theme:
    css/theme/theme--light.css
    css/theme/theme--dark.css
```

The `state` SMACSS group (`state: { css/state/something.css: {} }`) is also a valid top-level library key. Current best practice in Olivero/Claro is to **colocate state with the component** (e.g. `.is-open` rules live in `button.css` next to `.button`), so you'll rarely need a separate `state:` block — but it remains a documented option for migrating legacy themes.

> **Module styles always load BEFORE theme styles**. SMACSS groups **don't interlace** between modules and themes. A `normalize.css` loaded in theme's "base" group will appear **after** stylesheets defined in core/contrib modules, even if they're in "component" or "theme" group. See [#3046636](https://www.drupal.org/project/drupal/issues/3046636).

---

## 15. Aggregation

### Drupal 8+ (modern)

Themes can override stylesheets without affecting the "every page" option of the original. This simplifies the default grouping to **2 aggregated files** (vs 6 in D7):

1. **Every page aggregate**: base + layout + component + state + theme (modules then themes).
2. **Conditionally-loaded aggregate**: layout + component + state + theme.

> There are no conditionally-loaded base styles.

### Default weights

```php
const CSS_BASE = -200;
const CSS_LAYOUT = -100;
const CSS_COMPONENT = 0;
const CSS_STATE = 100;
const CSS_THEME = 200;
```

### SMACSS + Sass

Sass globbing allows multiple `.scss` files with a single `.css` output via partials.

---

## 16. Review checklist

When reviewing CSS, validate (based on [What to look for when reviewing CSS](https://project.pages.drupalcode.org/coding_standards/css/review/)):

### Architecture

- [ ] **Is all code still in use?** Verify there is no CSS pointing to classes/IDs that no longer exist in the markup.
- [ ] **Is there redundant code?** Look for overrides that are no longer necessary (e.g. `padding: 0; clear: left;` when default is `padding: 0`).
- [ ] **Are components well named?** Does the name reflect correct design semantics?
- [ ] **Could it be abstracted into a reusable class?** Look for opportunities to unify custom CSS into shared components.
- [ ] **Are selectors correct?** Short, component-based selectors. Replace nested selectors with component/sub-component classes.
- [ ] **Is code in the correct file?** Each component (with its sub-components) in a single file. No styles of the same component spread across multiple files.

### Formatting

- [ ] **File comment at the top** of the stylesheet (`/* @file ... */`).
- [ ] **Comments correctly formatted** (single line vs Doxygen vs inline).
- [ ] **Correct whitespace**: 2-space indent, blank lines between rulesets.
- [ ] **Ruleset format**: one selector per line, one declaration per line, final semicolon.
- [ ] **Properties**: space after `:`, double quotes for values, no units on zeros, rem by default.
- [ ] **Declaration order**: positioning → box model → other.
- [ ] **RTL styles**: `/* LTR */` with initial space, additional ruleset for `[dir="rtl"]` or use of logical properties.

### Naming

- [ ] **Component naming**: `.component-name`, dash between words, no abbreviations.
- [ ] **Sub-objects**: `.component-name__sub-object` (double underscore).
- [ ] **Modifiers**: `.component-name--variant` (double dash).
- [ ] **State classes**: `is-` prefix (`.is-active`, `.is-open`).
- [ ] **JS hooks**: `js-` prefix **NEVER in stylesheets**.
- [ ] **Layout**: `layout-` prefix (`.layout-container`).

### Specificity

- [ ] **No `id` selectors in CSS**.
- [ ] **No deep selectors**: maximum 2 combinators.
- [ ] **No descendant selectors** when child or specific classes can be used.
- [ ] **No qualified selectors**: `.button` not `a.button` or `ul.nav`.
- [ ] **`!important` only** for error/disabled states or mandatory theme overrides. **Not in core/contrib modules**.

### Architecture pitfalls

- [ ] **No component modifications based on context** (`.sidebar .component`).
- [ ] **No reliance on HTML structure** (no `nav > ul > li > a`).
- [ ] **No overly generic class names** (no `.title`, `.content` alone).
- [ ] **No rules doing too much** (positioning + sizing + colors + text in one rule).
- [ ] **No rules that "undo" styles** (no `.no-padding`, `.no-margin`).

### Separation of concerns

- [ ] Components **DO NOT** define their positioning/layout.
- [ ] `width`/`height` **only** on elements that natively have them.
- [ ] JS classes **separated** from styling classes.
- [ ] **NO inline styles via JS** except runtime-computed values.

### File organization

- [ ] **Modules**: structure in `css/`, files `<module>.module.css` and `<module>.theme.css`.
- [ ] **Modules without base styles**.
- [ ] **Themes**: separate base, layout, component into files.
- [ ] State + media queries with their component.

---

## 17. CSScomb (automated formatter and sorter)

**Source:** [CSScomb settings for Drupal](https://project.pages.drupalcode.org/coding_standards/css/csscomb/).

CSScomb formats and sorts CSS properties in `.css`, `.scss`, `.sass`, and `.less` files. The Drupal project ships a canonical `.csscomb.json` configuration aligned with the [CSS formatting guidelines](https://project.pages.drupalcode.org/coding_standards/css/format/) and a property sort order derived from the Yandex configuration (with SASS/LESS variables placed first so they are declared before use).

### Available integrations

- Command-line tool: `csscomb` (npm).
- Sublime Text plugin.
- Atom plugin.
- Grunt post-processing plugin.
- Gulp post-processing plugin.
- PhpStorm external tool.
- Vim plugin.

### Key settings (extract)

The official `.csscomb.json` enforces, among others:

| Setting | Value | What it means |
|---|---|---|
| `always-semicolon` | `true` | Every declaration ends with `;`. |
| `remove-empty-rulesets` | `true` | Drop `.foo {}` blocks with no declarations. |
| `color-case` | `"lower"` | `#fff` not `#FFF`. |
| `color-shorthand` | `true` | `#fff` not `#ffffff` when equivalent. |
| `element-case` | `"lower"` | `a` not `A` in selectors. |
| `eof-newline` | `true` | File ends with `\n`. |
| `leading-zero` | `true` | `0.5em`, not `.5em`. |
| `quotes` | `"double"` | `content: "x"`, not `'x'`. |
| `block-indent` | `2` | 2-space indent for declarations inside a ruleset. |
| `tab-size` | `2` | 2-space tabs (consistent with `block-indent`). |
| `unitless-zero` | `true` | `margin: 0;`, not `margin: 0px;`. |
| `lines-between-rulesets` | `false` | One blank line between rulesets (configured elsewhere). |
| `sort-order-fallback` | `"abc"` | Properties not in the explicit sort order fall back to alphabetical. |
| `sort-order` | (Yandex-based, see source) | Defines the canonical order: `position`, `z-index`, `top/right/bottom/left`, `display`, `visibility`, then box model, typography, decoration, etc. SASS/LESS variables / `$charset` / `$import` come first. |
| `exclude` | core/contrib/build dirs | Skips `core/**`, `modules/**`, `themes/**`, `vendor/**`, `node_modules/**`, etc. |

CSScomb groups declarations into ordered blocks. The Drupal sort-order (summarized) is: **positioning** (`position`, `z-index`, `top/right/bottom/left`, `display`, `visibility`) → **box model** (`float`, `clear`, `width`, `height`, `margin`, `padding`, `border`, `box-sizing`) → **typography** (`font`, `line-height`, `letter-spacing`, `color`, `text-*`) → **decoration** (`background`, `box-shadow`, `opacity`) → **at-rules** (`$media`, `$supports`, `$document`, `$page`, `$viewport`) appended at the end of each ruleset. See the [CSScomb settings page](https://project.pages.drupalcode.org/coding_standards/css/csscomb/) for the full sort-order list.

For the **full configuration** (including the complete `sort-order` array, which is ~150 properties long), copy/paste the JSON from the source page directly — the skill does not embed it here because it changes upstream and would drift.

### Adopting CSScomb in a Drupal project

CSScomb is **not** part of Drupal core's required toolchain — many Drupal projects build CSS via SASS / PostCSS / esbuild and review formatting against the rules in sections 1-16 manually. If you want to wire CSScomb into a theme:

1. From the theme directory, run `npm install --save-dev csscomb` (or `ddev npm install --save-dev csscomb` inside DDEV).
2. Drop the JSON from the official source page into `<theme>/.csscomb.json`.
3. Run `npx csscomb scss/` (or the equivalent path) from the theme directory.

Treat CSScomb output as 🟡 standards-violation territory — it matches the official formatting guidelines but is not itself a runtime requirement, and many project CI pipelines do not run it.

---

## Naming patterns summary

```css
/* Component base */
.component {}

/* Component sub-element (double underscore) */
.component__header {}
.component__body {}

/* Component modifier (double dash) — use with the base */
.component--large {}
.component--primary {}

/* State (is- prefix) — generally via JS */
.is-active {}
.is-disabled {}
.is-loading {}

/* JS hook (js- prefix) — NOT in CSS */
.js-slider {}      /* HTML+JS only */
.js-dropdown {}    /* HTML+JS only */

/* Layout */
.layout-container {}
.grid-3 {}
```

```html
<!-- Correct application: base + modifier + state + js hook -->
<button class="button button--primary is-active js-submit">
  Save
</button>
```
