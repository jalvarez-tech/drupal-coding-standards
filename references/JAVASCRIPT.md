# JavaScript — Drupal Coding Standards Reference

Official consolidation of:
- [Coding standards](https://project.pages.drupalcode.org/coding_standards/javascript/coding/)
- [Best practices](https://project.pages.drupalcode.org/coding_standards/javascript/best-practice/)
- [ESLint settings](https://project.pages.drupalcode.org/coding_standards/javascript/eslint/)
- [API documentation and comment standard](https://project.pages.drupalcode.org/coding_standards/javascript/documentation/)
- [JQuery coding standard](https://project.pages.drupalcode.org/coding_standards/javascript/jquery/)

> Drupal uses **ESLint** with `eslint-config-airbnb` as base. By extension, the [Airbnb JavaScript Style Guide](https://github.com/airbnb/javascript/) is the underlying reference.

---

## Table of contents

1. [Indenting](#1-indenting)
2. [Semicolons](#2-semicolons)
3. [File-closure](#3-file-closure)
4. [CamelCasing](#4-camelcasing)
5. [Variables and arrays](#5-variables-and-arrays)
6. [Functions](#6-functions)
7. [Constructors](#7-constructors)
8. [Comments](#8-comments)
9. [String concatenation](#9-string-concatenation)
10. [Control structures](#10-control-structures)
11. [Operators and comparisons](#11-operators-and-comparisons)
12. [Best practices](#12-best-practices)
13. [Drupal-specific (behaviors, t, theming)](#13-drupal-specific)
14. [jQuery standards](#14-jquery-standards)
15. [Documentation (JSDoc)](#15-documentation-jsdoc)
16. [ESLint configuration](#16-eslint-configuration)
17. [Review checklist](#17-review-checklist)

---

## 1. Indenting

- **2 spaces**.
- **NO tabs**.
- **No trailing whitespace**.

---

## 2. Semicolons

JavaScript allows "semicolon insertion" — **Drupal standards do NOT**.

- **ALL statements** (except `for`, `function`, `if`, `switch`, `try`, `while`) end with `;`.
- **Return values** start on the **same line** as `return`.

### Exceptions

- **Anonymous functions assigned to a variable** **MUST** end with `;`:
  ```js
  Drupal.behaviors.tableSelect = function (context) {
    // Statements...
  };
  ```
- **do/while** **MUST** end with `;`:
  ```js
  do {
    // Statements...
  } while (condition);
  ```

---

## 3. File-closure

**ALL** JavaScript code goes **inside a closure** that wraps the entire file.

```js
/**
 * @file
 */
(() => {

  // All the JavaScript for this file.

})();
```

This prevents variable leaks to global scope.

---

## 4. CamelCasing

For variables that are **NOT** constants or constructors, **multi-word variables and functions** must be `lowerCamelCase`.

- **First letter lowercase**.
- **Subsequent words capitalized**.
- **NO underscores** between words.

### jQuery variables

If a variable contains a **jQuery object**, **it MUST start with `$`**:

```js
let $form = $('#search-block-form');
let $inputs = $form.find('input');
```

---

## 5. Variables and arrays

Strict mode active → undeclared variable **halts script**. In older browsers, those variables are implicitly exported to global scope.

- **ALL variables are declared with `let` or `const`** before being used, and only once.
- **Declare all at the start of the function**.
- **One assignment per line** — even if you only declare without assigning.

```js
let anArray = [];
let eventCallback = function () {};
let curTableDragSetting;
let curTableDragIndex;
```

### Global variables

**Drupal JS NEVER defines global variables**.

### Constants

- **Pre-defined constants** = **UPPER_SNAKE_CASE**: `UPPER_UNDERSCORED`.
- **Variables added via PHP** = **lowerCamelCase** (consistency with other JS variables):

> 🟥 **The snippet below is Drupal 7 only.** `Drupal.settings` and `drupal_add_js(..., 'setting')` are removed in Drupal 8+. New code must use `drupalSettings` and `#attached['drupalSettings']` — see `php/PHP-LEGACY.md` § "Drupal.settings" and the modern example further down.

```php
// ❌ Legacy (Drupal 7). Do NOT use in new code.
$element['#attached']['js'][] = array(
  'data' => array('myModule' => array('basePath' => base_path())),
  'type' => 'setting',
);
```

```js
// ❌ Legacy (Drupal 7).
Drupal.settings.myModule.basePath;
```

Modern equivalent (Drupal 8+):

```php
// ✅ PHP side — attach via render array.
$element['#attached']['drupalSettings']['myModule']['basePath'] = base_path();
$element['#attached']['library'][] = 'my_module/main';
```

```js
// ✅ JS side — read drupalSettings injected from the library.
(($, Drupal, drupalSettings) => {
  Drupal.behaviors.myModule = {
    attach(context) {
      const basePath = drupalSettings.myModule.basePath;
    },
  };
})(jQuery, Drupal, drupalSettings);
```

### Arrays

- **Space after commas**, space around `=`:
  ```js
  let someArray = ['hello', 'world'];
  ```
- If exceeding 80 chars, **one element per line**, **indent one level**.
- **Trailing comma MANDATORY** in multi-line arrays (simplifies add/remove, improves git diffs):
  ```js
  // ❌ bad
  let fruits = [
    'apples',
    'banana',
    'pineapple'
  ];

  // ✅ good
  let fruits = [
    'apples',
    'banana',
    'pineapple',
  ];
  ```

### `typeof`

The tested value is **NOT wrapped in parentheses**:

```js
if (typeof myVariable === 'string') {
  // ...
}
```

---

## 6. Functions

### Function and method names

Start with the **module or theme name** to avoid collisions:

```js
// 🟥 Legacy (Drupal 7) — kept here as an example of "named function with no space".
// The shape `Drupal.behaviors.foo = function(context) { … }` and `Drupal.settings`
// no longer exist in Drupal 8+. Modern equivalent shown above under § 5.
Drupal.behaviors.tableDrag = function (context) {
  Object.keys(Drupal.settings.tableDrag).forEach(function (base) {
    $('#' + base).once('tabledrag', addBehavior);
  });
};
```

### Function declarations

- **`function` keyword + one space** before `(`.
- **Named functions**: **no space** between name and `(`.
- **Optional arguments** (with default values) go **at the end**.
- **Every function tries to return a meaningful value**.

```js
// 🟥 Legacy (Drupal 7) — `Drupal.behaviors.foo = function(context) { … }`
// and `Drupal.settings` no longer exist in Drupal 8+.
// Modern form: `Drupal.behaviors.foo = { attach(context, settings) { … } }`
// with `drupalSettings`. See modern attach example earlier in this file.
Drupal.behaviors.tableDrag = function (context) {
  this.clickCallback = function (e) {
    return false;
  };
};

function funStuff(field, settings) {
  const effectiveSettings = settings || Drupal.settings;  // 🟥 Legacy — use drupalSettings.
  alert("This JS file does fun message popups.");
  return field;
}

// Anonymous:
() => {}

// Named:
function closeDialog() {}
```

### Function calls

- **No spaces** between name, `(`, first parameter.
- **One space** between each comma and the next parameter.
- **No space** between last parameter, `)`, and `;`.

```js
let foobar = foo(bar, baz, quux);
```

- **One space** on each side of `=` when assigning return value.

---

## 7. Constructors

Constructors are functions designed to be used with **`new`**. The prefix:
1. Creates a new object based on the prototype.
2. Binds that object to the implicit `this` param.

JS **does not emit warning** at compile or runtime if **you omit the required `new`**. Without `new`, no new object is created and operations bind to the global object.

### Rules

- **Constructor functions must start with UPPERCASE**.
- **A function with initial UPPERCASE NEVER gets called without `new`**.

```js
function CollapsibleDetails(node) {}

let collapsibleDetail = new CollapsibleDetails(element);
```

---

## 8. Comments

### Inline documentation

Source files **follow JSDoc** (see [Documentation](#15-documentation-jsdoc) section).

### Non-JSDoc comments

**Strongly recommended**. If you look at a section and think "Wow, I don't want to describe this", **comment it** before you forget how it works. Comments **MAY** be removed by JS compressors afterwards.

- **Capitalized sentences with punctuation**.
- **Separate line immediately before** the code line/block.

```js
// Unselect all other checkboxes.
doSomething();
```

If each line of a list needs a separate comment, **MAY** go on the same line **uniformly indented**.

- `/* */` and `//` **both allowed**.

---

## 9. String concatenation

- **One space before and after** `+`:
  ```js
  let string = 'Foo' + bar;
  string = bar + 'foo';
  string = bar() + 'foo';
  string = 'foo' + 'bar';
  ```
- **`+=`**: **one space on each side**. Declare the variable first:
  ```js
  let string = '';
  string += 'Foo';
  string += bar;
  string += baz();
  ```

---

## 10. Control structures

- **One space** between control keyword and `(` (distinguishes from function calls).
- **Curly braces ALWAYS**, even when optional.

### if/else if/else

```js
if (condition1 || condition2) {
  action1();
}
else if (condition3 && condition4) {
  action2();
}
else {
  defaultAction();
}
```

> Note: in JS Drupal uses `else if` (separated), unlike PHP which uses `elseif`.

### switch

```js
switch (condition) {
  case 1:
    action1();
    break;

  case 2:
    action2();
    break;

  default:
    defaultAction();
}
```

### try/catch/finally

```js
try {
  // Statements...
}
catch (error) {
  // Error handling...
}
finally {
  // Statements...
}
```

### for...in

The body **MUST be wrapped in an `if`** that filters. Can select by type or range, or exclude functions, or exclude prototype properties:

```js
for (let variable in object) {
  if (filter) {
    // Statements...
  }
}
```

**Use `hasOwnProperty`** to distinguish true members. **Inside the loop, NOT on the same line**:

```js
for (let variable in object) {
  if (object.hasOwnProperty(variable)) {
    // Statements...
  }
}
```

---

## 11. Operators and comparisons

### Strict equality

`==` and `!=` do **type coercion** → unexpected errors. **Always use `===` or `!==`**.

### Comma operator

**Do NOT use the comma operator**, except in the control part of `for` statements.

```js
// ❌ Confusing
let x = (y = 3, z = 9);  // x === 9
```

---

## 12. Best practices

### JavaScript code placement

JS **must NOT be embedded in HTML** when avoidable. Adds page weight without caching/compression opportunity.

### Literal expressions

**Use literals** instead of `new` operator:

- `[]` instead of `new Array()`
- `{}` instead of `new Object()`

**Recommended** literals instead of wrappers `new Number`, `new String`, `new Boolean` when they yield the same result. BUT object instances **MAY** be used when it matters:

```js
const literalNum = 0;
const objectNum = new Number(0);
if (literalNum) { }              // false (0 is falsy)
if (objectNum) { }               // true (objectNum exists as object)
if (objectNum.valueOf()) { }     // false (value is 0)
```

### `with` statement

**FORBIDDEN** — strict mode doesn't allow it. Use explicit version or references:

```js
foo.bar.foobar.abc = true;
foo.bar.foobar.xyz = true;

// Or:
const o = foo.bar.foobar;
o.abc = true;
o.xyz = true;
```

### Avoiding unreachable code

`return`, `break`, `continue`, or `throw` **should be followed by `}`, `case`, or `default`**.

### `eval()` is evil

The official standard says `eval()` **SHOULD NOT** be used. It creates a new
environment, imports vars, executes, collects result, re-exports, and cannot be
cached. Treat it as a standards violation; promote to 🔴 only when untrusted
data can execute.

JS implicitly uses `eval()` for other constructs:
- Avoid the `Function` constructor.
- Avoid passing strings to `setTimeout()` or `setInterval()`.

### Preventing XSS

User-provided content output to the browser **SHOULD be escaped before HTML
insertion**. `Drupal.checkPlain()` is Drupal's official helper for plain-text
HTML contexts; equivalent safe APIs (`textContent`, jQuery `.text()`, or
sanitized Drupal render output) are acceptable. Promote to 🔴 if
user-controlled content reaches HTML or script execution unsanitized.

### Modifying the DOM

The official docs say code **SHOULD NOT** use `document.createElement()` to add
HTML. For Drupal/browser consistency, prefer the jQuery equivalent. Report
`document.createElement()` as a 🟢 suggestion unless it also bypasses
sanitization, behavior attachment, or project lint rules:

```js
// ✅ Correct
this.popup = $('<div id="autocomplete"></div>')[0];

// ❌ Avoid
this.popup = document.createElement('div');
this.popup.id = 'autocomplete';
```

---

## 13. Drupal-specific

### Theming

Any module with JS that **produces HTML content** **MUST provide default theme functions** in the `Drupal.theme.prototype` namespace.

### String Translation

User-visible strings in JS files **MUST** be wrapped in **`Drupal.t()`** (JS equivalent of `t()`). For plurals use **`Drupal.formatPlural()`** (equivalent of `format_plural()`). Param order is identical to the server-side functions.

**Placeholder prefixes** — same semantics as PHP `t()` / `FormattableMarkup`. Match the prefix to the data:

| Prefix | Behavior | Use when |
|---|---|---|
| `@variable` | HTML-escaped (default, safest) | Plain text from any source |
| `%variable` | HTML-escaped **and** wrapped in `<em class="placeholder">…</em>` | Plain text you want visually emphasized |
| `:variable` | URL-encoded for HTML attribute use (`href` etc.) | The value is a URL |
| `!variable` | **Not escaped** — **removed** in modern Drupal | Legacy only; treat any remaining occurrence as 🔴 |

Picking the wrong prefix is an XSS bug. For UGC always default to `@`.

```js
const message = Drupal.t('Hello @name', {'@name': userName});

const count_msg = Drupal.formatPlural(
  count,
  '1 item',
  '@count items'
);
```

### Behaviors

`Drupal.behaviors` is the **main mechanism** for attaching JS to pages in Drupal. Each behavior initializes with `attach` and optionally `detach`. The modern Drupal-11 form uses ES6 method shorthand and wraps the body in `once()` from the [`core/once`](https://www.drupal.org/node/3158256) library so the behavior is idempotent across AJAX reloads:

```js
// my_module/js/myfeature.js
((Drupal, once) => {
  Drupal.behaviors.myFeature = {
    attach(context, settings) {
      once('my-feature', '[data-my-feature]', context).forEach((el) => {
        // Initialise each element exactly once.
        el.addEventListener('click', () => { /* … */ });
      });
    },
    detach(context, settings, trigger) {
      // Optional: tear down anything attach() set up.
    },
  };
})(Drupal, once);
```

Declare the dependency on `core/once` (and `core/drupal` for `Drupal.behaviors`) in `*.libraries.yml`:

```yaml
myfeature:
  version: 1.x
  js:
    js/myfeature.js: {}
  dependencies:
    - core/drupal
    - core/once
```

The `once()` API was introduced in Drupal 9.2 specifically to remove the jQuery dependency from `Drupal.behaviors` patterns. Signature: `once(id, elements, context)` returns an array of newly-marked elements. Helpers (per the [official once API](https://www.drupal.org/node/3158256)):

- `once.filter(id, els)` — returns elements from `els` that already have the marker `id` applied.
- `once.find([id], [context])` — finds elements previously processed by `once()`; both arguments are optional. Call with no args to find every onced element on the page, with just an `id` to filter to one marker, or with both to scope inside `context`.
- `once.remove(id, els)` — removes the marker `id` from `els` and returns the matching elements (useful inside `detach`).

Use these helpers instead of the legacy `$().once('id', …)` jQuery plugin.

### `Drupal.announce()` — accessibility announcements

For dynamic UI changes that screen-reader users need to hear (filter applied, item added to cart, error revealed), use `Drupal.announce(text, priority)`. It writes to a polite (default) or assertive ARIA live region maintained by core:

```js
Drupal.announce(Drupal.t('Filter applied: @count results.', {'@count': results.length}));
Drupal.announce(Drupal.t('Save failed.'), 'assertive');
```

`'polite'` (default) waits for the user to be idle; `'assertive'` interrupts. Use `'assertive'` sparingly and only for errors or time-sensitive feedback.

---

## 14. jQuery standards

> **⚠️ Obsolete page.** The Drupal jQuery coding-standards page is formally marked **`[Obsolete]`** on drupal.org. jQuery is **no longer a hard dependency** of Drupal core JavaScript — `core/once` (D9.2+) provides the vanilla equivalent of `$.once()`, and most behaviors in Drupal 10/11 core have been rewritten without jQuery. **Do not introduce jQuery into new code.** The guidance below applies only when maintaining legacy modules that still attach `core/jquery`; even then, the recommended migration path is to remove the jQuery dependency.

### `$` prefix for jQuery objects

**Any variable that is a jQuery object MUST start with `$`**:

```js
// ❌ Wrong
var foo = $('.foo');
var object = { bar: $('.bar') };

// ✅ Correct
var $foo = $('.foo');
var object = { $bar: $('.bar') };
```

### Avoiding compatibility issues

Historical: D6/D7 sites used the `jquery_update` contrib module to bump jQuery. That module is end-of-life and not maintained for D9+. Drupal core ships its own jQuery version when needed; the path forward is to remove jQuery dependencies via `core/once` (see §13 "Behaviors").

### Chaining

For chaining: you can use CSS selector `$('a b > c')` or JS chaining `$('a').find('b').children('c')`. **The second is slightly faster**.

If `.children()` and `.find()` give the same result, **use `.find()`**.

```js
// ❌ Incorrect
this.$el
    .find('.contextual-links')
    .prop('hidden', !isOpen);

// ✅ Correct
this.$el.find('.contextual-links')
    .prop('hidden', !isOpen);
```

### Event delegation

Each event ("click", "mouseover") **bubbles up** the DOM tree. Useful for many elements that call the same function: instead of binding to all, bind to the parent and figure out which node fired it.

`return false;` to prevent default **is frequently misused** — it also **prevents propagation** (bubbling). You don't always want that. If you want to prevent default OR stop propagation, **be explicit**:

```js
// ❌ Incorrect (calls both preventDefault and stopPropagation)
$('.item').click(function(event) {
  return false;
});

// ✅ Correct (Drupal 7)
$menus.delegate('.item', 'click', function(event) {
  event.preventDefault();   // Prevents default behavior (i.e., click).
  event.stopPropagation();  // Prevents bubbling.
});

// ✅ Correct (Drupal 8+)
$menus.on('click', '.item', function(event) {
  event.preventDefault();
  event.stopPropagation();
});
```

### Functions

**Separate** individual functions and call them when needed:

```js
// ❌ Incorrect (only callable on click)
$('.btn').click(function() { /* ... */ });

// ✅ Correct (callable anywhere)
function clickFunction() { /* ... */ }
$('div').click(function() { clickFunction(); });
```

### Context

**Always give context** to selectors. Default context = entire DOM. If context is a cached selection, **use `.find()`**:

```js
// ❌ No context, very slow
var $element = $('.element');

// ⚠️ Improved
var $element = $('.element', '#sidebar');

// ✅ Correct
var $sidebar = $('#sidebar');
var $element = $sidebar.find('.element');
```

### `#id` vs `.class`

**`#id` is much faster** than `.class`. If the target appears **only once** on the page, **select by `#id`**. If there are multiple, use `.class` **descending from an `#id`**.

### `jQuery.attr()`

**Only applies to D6 and D7**. D8+ uses **`.prop()`** for properties.

Use booleans, NOT empty strings, to set properties. Don't assume return values are always booleans (`.attr('checked')` may return `true` or `'checked'` depending on jQuery version):

```js
// ❌ Incorrect
$element.attr('disabled', '');
if ($element.attr('checked') === true) { /* ... */ }

// ✅ Correct
$element.attr('disabled', false);
$element.attr('disabled', true);
if ($element.attr('checked')) { /* ... */ }
```

### `jQuery.each()`

Powerful for iterating jQuery objects, **misused** for iterating native arrays/objects.

**Native `for` loops are 300-1000x faster** than `$.each()`:

```js
// ❌ Incorrect
var array = [ /* ... */ ];
$.each(array, function(i, item) { /* ... */ });

// ✅ Correct
var array = [ /* ... */ ];
var i;
for (i = 0, len = array.length; i < len; i += 1) {
  var element = array[i];
}
```

---

## 15. Documentation (JSDoc)

JS docs **very similar** to PHP, with modifications by the [JSDoc3](https://jsdoc.app/) parser.

### Differences with PHP docs

- **ALL items** (methods, constructors, properties, functions, vars) **need doc headers** or they are not recognized. **Only behaviors are documented specially** (see example).
- **Not all `@tags` are supported** (see tag order list).
- **Data types between `{}`** after the tag:
  ```js
  @param {TheType} paramName
  @return {TheType}
  ```
  Types: `number`, `string`, `bool`, `null`, `undefined`, `object`, `function`, `Array`. For objects, **the constructor name**: `Date`, `RegExp`, `HTMLElement`, `HTMLInputElement`, `Drupal.Ajax`.
- **Additional tag `@fires`** for events fired by a function (equivalent of PHP's `@throws`).
- Custom events: add docblock with `@event` immediately before the first code line that fires the event. Only **one `@event` block** per custom event, but `@fires` in **every function** that fires it.
- **Additional tag `@prop {type} name`** to document object properties (works like `@param`).
- **Additional notation** so JSDoc identifies what is being documented:
  - **`@name`**: the actual name if it doesn't match the code (typically `DropButton` instead of `Drupal.DropButton`).
  - **`@constructor`**: function = class constructor.
  - **`@namespace`**: object = namespace.
  - **`@function`**: usually NOT needed; JSDoc assumes function by default unless overridden by the above.

### Tag order

```
@global

@typedef
@var
@name
@namespace
@constructor
@callback
@event
@function

@augments
@lends

@type
@prop

@param
@return

@throws
@fires
@listens

@ingroup
@deprecated
@see
@todo
@ignore
```

### Documenting a JS file

```js
/**
 * @file
 * Provides some feature
 *
 * The extra line between the end of the @file docblock
 * and the file-closure is important.
 */

(() => {
  // File body — modern Drupal 10/11 arrow IIFE form, no jQuery injection.
})();
```

> **Style note**: the example above is the modern Drupal 10/11 form — an arrow IIFE with no parameters and no `"use strict"` (modules and class bodies are already strict by default). The legacy `(function ($) { … "use strict"; })();` shape is still found in older modules that depend on jQuery; if you're maintaining one, keep the legacy form for consistency within that file, but new files should use the arrow form above.

### Documenting behaviors

```js
/**
 * Attaches the table drag behavior to tables.
 *
 * @type {Drupal~behavior}
 *
 * @prop {Drupal~behaviorAttach} attach
 *   Specific description of this attach function goes here.
 * @prop {Drupal~behaviorDetach} detach
 *   Specific description of this detach function goes here.
 */
Drupal.behaviors.tableDrag = {
  attach: function (context, settings) {
    // ...
  },
  detach: function (context, settings, trigger) {
    // ...
  }
};
```

### Documenting common constructs

```js
/**
 * Holds JavaScript settings and other information for Drupal.
 *
 * @namespace
 */
var Drupal = {
  /**
   * Holds behaviors for Drupal.
   *
   * @namespace
   */
  'behaviors': {},
};

/**
 * Returns the value of foo for the current widget.
 *
 * @return
 *   The value of foo in the current widget.
 */
Drupal.getCurrentFoo = function () {
  // ...
};

/**
 * Constructs a table drag object.
 *
 * @constructor
 *
 * @param {HTMLTableElement} table
 *   DOM object for the table to be made draggable.
 * @param {object} tableSettings
 *   Settings for the table.
 */
Drupal.tableDrag = function (table, tableSettings) {
  // ...
};

/**
 * Hides the columns containing weight and parent form elements.
 *
 * @fires event:columnschange
 *
 * @see Drupal.tableDrag.showColumns
 */
Drupal.tableDrag.prototype.hideColumns = function () {

  /**
   * Indicates that columns have changed in a table.
   *
   * @param {string} type
   *   Type of change: 'show' or 'hide'.
   *
   * @event columnschange
   */
  $('table.tableDrag-processed').trigger('columnschange', 'hide');
};

/**
 * Shows the columns containing weight and parent form elements.
 *
 * @fires columnschange
 *
 * @see Drupal.tableDrag.hideColumns
 */
Drupal.tableDrag.prototype.showColumns = function () {
  // This event is documented in Drupal.tableDrag.hideColumns.
  $('table.tabledrag-processed').trigger('columnschange', 'show');
};
```

---

## 16. ESLint configuration

Drupal 8+ uses **ESLint** to validate JS: free of syntax errors, no variable leaks, minifiable.

### Setup

Drupal core ships its canonical ESLint config at the repo root:
- [`.eslintrc.json`](https://git.drupalcode.org/project/drupal/-/blob/11.x/.eslintrc.json) — extends [`airbnb-base`](https://www.npmjs.com/package/eslint-config-airbnb-base) and integrates **Prettier** for formatting.
- [`.eslintignore`](https://git.drupalcode.org/project/drupal/-/blob/11.x/.eslintignore)

When ESLint is invoked from the repo root, it **auto-detects** these files. For per-module overrides, drop a `.eslintrc.json` in the module directory.

> **Flat-config note (forward-looking).** ESLint 9 (released 2024) replaces `.eslintrc.json` with `eslint.config.js` (flat config). Drupal 11 core still ships the legacy `.eslintrc.json` format; until core migrates, stay on the legacy format for compatibility. Track [#3344609](https://www.drupal.org/project/drupal/issues/3344609) for the core migration.

### Quick installation

```bash
# Prerequisites: node.js + npm + npx
# Match Drupal core's stack — airbnb-base + prettier + drupal-specific rules.
npm install --save-dev eslint eslint-config-airbnb-base eslint-config-prettier eslint-plugin-prettier eslint-plugin-yml
# The contrib drupal config package is optional and ships an extends-able preset:
npm install --save-dev eslint-config-drupal

npx eslint modules/custom/
npx eslint themes/custom/
```

### Module-specific overrides

If a module uses third-party code that defines globals (e.g. Google Analytics `ga`), create `.eslintrc.json` in the module dir:

```json
{
  "extends": [
    "drupal"
  ],
  "globals": {
    "ga": true
  }
}
```

ESLint **merges configs** down the tree — see [Configuration Cascading and Hierarchy](https://eslint.org/docs/latest/use/configure/configuration-files#cascading-and-hierarchy).

---

## 17. Review checklist

### File structure

- [ ] File-closure present: `(() => { ... })();` or `(function ($) { ... })();`
- [ ] `@file` docblock at the top
- [ ] No trailing whitespace
- [ ] Unix line endings, final newline

### Format

- [ ] 2-space indent, no tabs
- [ ] Semicolons on ALL statements
- [ ] Assigned anonymous functions end with `;`
- [ ] `do/while` ends with `;`
- [ ] Trailing comma in multi-line arrays

### Naming

- [ ] Variables/functions: `lowerCamelCase`
- [ ] Constructors: `UpperCamelCase` (and always with `new`)
- [ ] Constants: `UPPER_SNAKE_CASE`
- [ ] PHP-passed vars: `lowerCamelCase`
- [ ] jQuery objects: `$` prefix
- [ ] Functions with module prefix

### Variables

- [ ] Only `let` or `const`, NO `var` (modern)
- [ ] Declared at start of function
- [ ] One assignment per line
- [ ] No globals

### Functions

- [ ] Space after `function` (anonymous)
- [ ] No space between name and `(` (named)
- [ ] Optional args at the end
- [ ] Meaningful return value

### Control structures

- [ ] Curly braces always
- [ ] Space between keyword and `(`
- [ ] `else if` (separated)
- [ ] `for...in` with `hasOwnProperty()` or filter
- [ ] `try/catch/finally` with catch on its own line

### Operators

- [ ] `===` and `!==` (strict equality)
- [ ] No comma operator (except `for`)
- [ ] Space around `+`, `+=`, `=`

### Best practices

- [ ] Literals (`[]`, `{}`) instead of `new Array()`, `new Object()`
- [ ] No `with`
- [ ] Avoid `eval()` and the `Function` constructor; 🔴 only for untrusted executable
  strings
- [ ] Avoid strings in `setTimeout`/`setInterval`
- [ ] Escape user-provided HTML insertion (`Drupal.checkPlain()`, `textContent`,
  jQuery `.text()`, or sanitized render output)
- [ ] Prefer jQuery over `document.createElement()` when adding Drupal-managed HTML
- [ ] `Drupal.t()` for user-visible UI strings
- [ ] `Drupal.formatPlural()` for plurals

### jQuery

- [ ] `$` prefix for jQuery objects
- [ ] `.prop()` in D8+ (not `.attr()`)
- [ ] Context given to selectors
- [ ] `#id` when there's a single element
- [ ] `event.preventDefault()` / `event.stopPropagation()` explicit (not `return false`)
- [ ] Native `for` loop instead of `$.each()` for arrays/objects
- [ ] `.find()` when context is cached

### Documentation (JSDoc)

- [ ] All functions/methods/properties/vars documented
- [ ] Types between `{}`: `{string}`, `{HTMLElement}`, `{Drupal.Ajax}`
- [ ] `@fires` for custom events fired
- [ ] `@event` block before the custom event trigger (only once)
- [ ] `@constructor` for constructors
- [ ] `@namespace` for namespaces
- [ ] Correct tag order
- [ ] Behaviors documented with `@type {Drupal~behavior}` + `@prop` for attach/detach

### ESLint

- [ ] Passes `npx eslint modules/custom/` or `themes/custom/`
- [ ] If custom global needed, override in local `.eslintrc.json`

---

## Recommended setup

```bash
# Install dependencies for custom JS checking
npm install eslint eslint-config-airbnb eslint-plugin-yml --save-dev
npm i eslint-config-drupal

# Validate custom code
npx eslint modules/custom/
npx eslint themes/custom/

# Autofix what's autofixable
npx eslint modules/custom/ --fix
```
