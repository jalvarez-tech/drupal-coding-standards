# PHP — Coding (formatting and syntax)

Scope: whitespace, file headers, quotes, arrays, operators, control structures, function calls, line length, string concatenation, constructor calls, placeholders, E_ALL, example URLs.

Sources:
- [PHP coding standards](https://project.pages.drupalcode.org/coding_standards/php/coding/)
- [Placeholders and delimiters](https://project.pages.drupalcode.org/coding_standards/php/placeholders-delimiters/)
- [E_ALL compliance](https://project.pages.drupalcode.org/coding_standards/php/e_all/)

For naming, OO, namespaces, PSR-4, services: see `PHP-OO-PSR4.md`.
For docblocks: see `PHP-DOCS.md`.
For exceptions: see `PHP-EXCEPTIONS.md`.
For pre-Drupal-9 patterns: see `PHP-LEGACY.md`.

---

## 1. Whitespace and indentation

- **Indent**: 2 spaces. **NEVER** tabs.
- **No trailing whitespace** at end of lines.
- **Line endings `\n`** (Unix). NOT `\r\n` (Windows).
- **Files end with ONE final newline** (`\n`). This avoids the `\ No newline at end of file` warning and simplifies diffs.
- **Blank line** between each block at the start of a PHP file: between `<?php`, namespace, use statements and the following code.
- **Blank line** between the start of a class and the first property/method.
- **Blank line** between the end of property definitions and the first method.
- **Blank line** between methods.

Correct example of a file header with a class:

```php
<?php

namespace This\Is\The\Namespace;

use Drupal\foo\Bar;

/**
 * Provides examples.
 */
class ExampleClassName {

  // Properties and methods here.

}
```

Correct example of a `.module` file header:

```php
<?php

/**
 * @file
 * Provides example functionality.
 */

use Drupal\foo\Bar;

/**
 * Implements hook_help().
 */
function example_help($route_name) {
  // ...
}
```

---

## 2. PHP tags and file closing

- **Use the full opening tag `<?php`**. Do **NOT** use the short open tag `<?` — the official Drupal PHP coding standards forbid it explicitly ("Always use `<?php ?>` to delimit PHP code, not the shorthand, `<? ?>`"). The short-echo tag `<?=` is **not banned** by the Drupal standard (the page doesn't address it) and does appear in Drupal core templates; treat it as acceptable unless a specific project has a written policy against it.
- Use the closing tag `?>` **only for inline PHP blocks inside non-PHP content** (e.g. legacy `.tpl.php` snippets). **Never** as the final delimiter of a PHP file. Reasons:
  1. Avoids unintended whitespace that causes "header already sent" errors.
  2. The closing delimiter at the end is optional per PHP.
  3. PHP.net itself omits it in their own files.

---

## 2a. Semicolons

- **Every statement ends with `;`** — including the last statement inside a single-line PHP block. PHP itself allows omission before `?>` in inline templates, but **Drupal mandates the semicolon** in all cases for consistency.

```php
// ✅ Correct (semicolon required even in one-liners).
<?php print $foo; ?>

// ❌ Incorrect — PHP tolerates it, Drupal forbids it.
<?php print $foo ?>
```

- Source: [PHP coding standards › Semicolons](https://project.pages.drupalcode.org/coding_standards/php/coding/).

---

## 3. Strict types

If you define `declare(strict_types=1)`:

- Goes on a **new line** after the opening `<?php`.
- **No spaces** around the `=`.
- Goes **after** the file DocBlock if present.
- **Empty newlines** around it.

```php
<?php

/**
 * @file
 * This is a file DocBlock.
 */

declare(strict_types=1);
```

---

## 4. Quotes

Drupal has **no hard standard** between single and double quotes. But the recommendation is:

- **Single quotes by default** (`'…'`).
- **Double quotes** only in these two cases:
  1. **Deliberate in-line interpolation**: `"<h2>$header</h2>"`
  2. **Translated strings** where avoiding escaping single quotes improves readability: `"He's a good person."` vs `'He\'s a good person.'`. Escapes can break `.pot` generators for translation.

---

## 5. Arrays

- **Short array syntax** `[…]`, NOT `array(…)`.
- **Space after commas**: `['hello', 'world', 'foo' => 'bar']`.
- **Spaces around `=>`**.
- If the line exceeds **80 chars** (common in form/menu declarations), **one element per line**, indented one level.
- **Mandatory trailing comma** in multi-line arrays — prevents parse errors when adding an element at the end.

```php
$some_array = ['hello', 'world', 'foo' => 'bar'];

$form['title'] = [
  '#type' => 'textfield',
  '#title' => t('Title'),
  '#size' => 60,
  '#maxlength' => 128,
  '#description' => t('The title of your node.'),
];
```

---

## 6. Operators

- **Binary operators** (`+`, `-`, `=`, `==`, `!=`, `>`, etc.): **one space before and after**.
- **Unary operators** (`++`, `--`): **no space** between the operator and the variable.
- **`!=`** for weak inequalities. **`<>`** is FORBIDDEN.
- **Short ternary `?:`** when the first operand matches the condition:
  ```php
  // ✅ Correct
  $result = $condition ?: 'default';

  // ❌ Incorrect
  $result = $condition ? $condition : 'default';
  ```
- **Null coalescing `??`** instead of `isset() ? : `:
  ```php
  // ✅ Correct
  $result = $values['entry'] ?? 'default';

  // ❌ Incorrect
  $result = isset($values['entry']) ? $values['entry'] : 'default';
  ```

---

## 7. Casting

- **One space** between `(type)` and the variable: `(int) $myNumber`, `(string) $foo`, `(array) $bar`.

---

## 8. Chaining / fluent interface

- A **method should return `$this`** and be chainable when there is no other logical return value (setters, methods that modify state).
- When the chain spans more than one line, **indent method calls 2 spaces**.

```php
$query = $connection->select('node')
  ->condition('type', 'article')
  ->condition('status', 1)
  ->execute();
```

> For the **Drupal 7 `db_select()` chain** style, see `PHP-LEGACY.md`.

---

## 9. Control structures

- **One space** between the keyword and the opening parenthesis (`if (`, `while (`, `for (`).
- **Curly braces ALWAYS**, even when technically optional.
- **Opening curly** on the same line, preceded by one space.
- **Closing curly** on its own line, indented to the same level as the opening statement.
- **`elseif`** as one word, NOT `else if`.

```php
if (condition1 || condition2) {
  action1;
}
elseif (condition3 && condition4) {
  action2;
}
else {
  default_action;
}
```

**switch**:

```php
switch (condition) {
  case 1:
    action1;
    break;

  case 2:
    action2;
    break;

  default:
    default_action;
}
```

**do-while**:

```php
do {
  actions;
} while ($condition);
```

> **Alternate syntax `if (…):` / `endif;`** is **legacy `.tpl.php` (Drupal 7) only**. Drupal 8+ uses Twig. See `PHP-LEGACY.md` (sibling) and `../TWIG.md`.

---

## 10. Function calls and declarations

### Function calls

- **No spaces** between the name, the `(`, and the first parameter.
- **Spaces after commas** between params.
- **No space** between the last param, the `)` and the `;`.

```php
$var = foo($bar, $baz, $quux);
```

### Multi-line function declarations

When the argument list spans multiple lines:

- **First item** on the next line.
- **One argument per line**.
- **Trailing comma** on the last argument.
- **`)` and `{` on the same line**, separated by one space.

```php
function funStuff_system(
  string $foo,
  string $bar,
  int $baz,
) {
  // body
}
```

### Anonymous functions

- **One space** between `function` and `(`:

```php
array_map(function ($item) use ($id) {
  return $item[$id];
}, $items);
```

---

## 15. Including code

- **`require_once()`** for unconditional include of class files.
- **`include_once()`** for conditional include (factory methods, etc.).
- They share the same file list, no risk of mixing.
- **`include_once` and `require_once` are statements, NOT functions** — you don't need parentheses around the file name.
- For includes in the same directory or sub-directory, prefix `./`: `include_once ./includes/my_module_formatting.inc`.
- In **Drupal 8+**, prefer `\Drupal\Core\Extension\ExtensionPathResolver` or container parameters (e.g. `app.root`) over `DRUPAL_ROOT`. Direct file includes are rare in modern code — most loading goes through PSR-4 autoload. When you do need a path:
  ```php
  // ✅ Drupal 8+ — config via the Config API.
  $cache_inc = \Drupal::config('my_module.settings')->get('cache_inc') ?? 'includes/cache.inc';
  $module_path = \Drupal::service('extension.path.resolver')->getPath('module', 'my_module');
  require_once $module_path . '/' . $cache_inc;
  ```

> The `module_load_include()` and `variable_get()/variable_set()` helpers are **legacy (D7) and removed in D9+** — `variable_get()` was replaced by the Config API and State API; see `PHP-LEGACY.md` for the migration table.

---

## 16. Line length and wrapping

- **Lines ≤ 80 characters** in general.
- **Allowed exceptions**: long function names, function/class definitions, variable declarations, simple conditions.
- **Control conditions may exceed 80** if they are easy to read.
- **Conditions are NOT wrapped across multiple lines** — extract them into intermediate variables with descriptive names.

❌ Bad (compacting complex conditions into a single line):
```php
if ((isset($key) && !empty($user->uid) && $key == $user->uid) || (isset($user->cache) ? $user->cache : '') == ip_address() || isset($value) && $value >= time())) {
  // ...
}
```

✅ Good (split into documented variables):
```php
// Key is only valid if it matches the current user's ID, as otherwise other
// users could access any user's things.
$is_valid_user = isset($key) && !empty($user->uid) && $key == $user->uid;

// IP must match the cache to prevent session spoofing.
$is_valid_cache = isset($user->cache) ? $user->cache == ip_address() : FALSE;

// Alternatively, if the request query parameter is in the future, then it
// is always valid.
$is_valid_query = $is_valid_cache || (isset($value) && $value >= time());

if ($is_valid_user || $is_valid_query) {
  // ...
}
```

---

## 17. String concatenation

- **Space before and after `.`**:
  ```php
  $string = 'Foo' . $bar;
  $string = $bar . 'foo';
  $string = bar() . 'foo';
  ```
- For simple variables, you can use double quotes with interpolation:
  ```php
  $string = "Foo $bar";
  ```
- **`.=` (concatenating assignment)**: **one space on each side**:
  ```php
  $string .= 'Foo';
  $string .= $bar;
  ```

---

## 18. Class constructor calls

- **Always include parentheses** even if there are no arguments. Maintains consistency with constructors with arguments:
  ```php
  $foo = new MyClassName();
  $foo = new MyClassName($arg1, $arg2);
  ```
- With variable class name, same syntax:
  ```php
  $bar = 'MyClassName';
  $foo = new $bar();
  $foo = new $bar($arg1, $arg2);
  ```

---

## 24. Placeholders and delimiters

When writing filters or code that processes content, **do NOT use obscure characters** as placeholders. Use **alphanumeric strings prefixed with the module + `-` or `_`**, wrapped in `[…]`.

If you need delimiters, the closing can incorporate `/` after the initial `[`.

### PCRE example

```php
'@\[modulename-tag\](.+?)\[/modulename-tag\]@'
```

Or with modulename suffix:

```php
'@\[modulename-tag\](.+?)\[/tag-modulename\]@'
```

---

## 25. E_ALL compliance

### Error reporting

- **Drupal 6 (legacy)**: ignored `E_NOTICE`, `E_STRICT`, `E_DEPRECATED` by default. Not applicable to Drupal 8+ projects.
- **Drupal 7+**: reports all `E_ALL`. PHP can be configured to report additional levels.

For development, in `.htaccess`: `php_value error_reporting -1`.

### `isset()` vs `!empty()`

When testing the value of a variable that might not be defined:

- **`isset()`**: TRUE even if the var is `''` or `0`. FALSE if it's `NULL`.
- **`!empty()`**: FALSE for `''`, `0`, `NULL`, empty arrays.

Decide based on whether `''` or `0` are valid expected values.

❌ Bad (causes E_NOTICE if `#input` doesn't exist):
```php
if ($form['#input']) {
  // some code
}
```

✅ Good:
```php
if (!empty($form['#input'])) {
  // some code
}
```

**Tip**: `isset()` returns TRUE if var is `0` but FALSE if it's `NULL`. In SQL queries cases, `is_null()` may be better.

---

## 26. Example URLs

Use **`example.com`** for all example URLs ([RFC 2606](https://www.rfc-editor.org/rfc/rfc2606.txt) reserved-domain registry).
