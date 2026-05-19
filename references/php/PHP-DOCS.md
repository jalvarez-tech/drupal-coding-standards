# PHP — Documentation (Doxygen / docblocks)

Scope: docblock format, summaries, hook docs, `@param` / `@return` / `@throws`, inline comments, `@var`, tag reference, PHPUnit tag conventions.

Sources:
- [API documentation and comment standards](https://project.pages.drupalcode.org/coding_standards/php/documentation/)
- [API documentation samples (concrete examples)](https://project.pages.drupalcode.org/coding_standards/php/documentation-examples/) — runnable docblock examples for files, functions, classes, hooks, forms, themeable functions, preprocess functions, render callbacks. Cross-referenced from each pattern section below.

For coding/formatting: see `PHP-CODING.md`.
For OO/naming/PSR-4: see `PHP-OO-PSR4.md`.

---

Drupal uses the **API module** to parse documentation. The syntax is inspired by PHPDoc/JavaDoc/Doxygen.

## General rules

- Documentation goes in **docblocks starting with `/**`**. Other styles (`//`, `/*`) are NOT docblocks.
- Each docblock line starts with `*`.
- **Blank line** within the docblock to create a new paragraph.
- Tags that trigger implicit blank line: `@param`, `@return`, `@see`, `@var`.
- **First paragraph = summary**. Must be **under 80 chars**, start with capital and end with `.`.
- **Every function, constant, class, interface, class member, file** **must be documented**. Even private members.
- Comments use **proper sentences**, US English, US spelling.
- **Single period between sentences** (no double space).
- **All caps only** for constants (e.g. `TRUE`, `FALSE`, `NULL`).
- For modules/themes, write it as a proper noun: "the Foo Bar module", NOT "the Foo Bar Module module".
- Lines with comments **wrap to 80 chars** without exceeding (exceptions per tag).
- Lines after `@param`, `@return`, etc. with associated docs: **indent 2 spaces**.

## Docblock for hooks

**Hook definition** (in `.api.php`): the summary starts with an **imperative verb** explaining why a module would want to implement the hook.

```php
/**
 * Respond to node deletion.
 *
 * @param \Drupal\node\NodeInterface $node
 *   The node entity being deleted.
 */
function hook_node_delete(NodeInterface $node) {
  // Sample implementation.
}
```

**Hook implementation** (in `.module`): short docblock:

```php
/**
 * Implements hook_help().
 */
function blog_help($section) {
  // ...
}
```

If the hook has a variable part:

```php
/**
 * Implements hook_form_FORM_ID_alter() for node_type_form().
 */
function my_module_form_node_type_form_alter(&$form, &$form_state) {
  // ...
}
```

## Update functions (`hook_update_N`)

Different syntax because the summary is shown to the user running update.php:

```php
/**
 * Add several fields to the {node_revision} table.
 *
 * Longer description can go here, if necessary.
 */
function module_name_update_7002() {
  // ...
}
```

First line starts with an **imperative verb**.

## Callbacks for PHP built-ins

```php
/**
 * Sorts theme list alphabetically.
 *
 * Callback for uasort() within system_themes_page().
 */
```

## Drupal callbacks (with definition in `.api.php`)

- Callback definition names start with `callback_`.
- Bodies = sample implementation.
- Summary describes what the callback does, starts with imperative verb.
- Next line: `Callback for ___:` with the name of the hook/function that uses it.
- Add `@ingroup callbacks`.
- Real implementations: summary `Implements callback_NAME().`.

## Form functions

```php
/**
 * Form constructor for the user login form.
 *
 * @param string $message
 *   The message to display.
 *
 * @see user_login_form_validate()
 * @see user_login_form_submit()
 *
 * @ingroup forms
 */
function user_login_form($form, &$form_state, $message = '') {
  // ...
}

/**
 * Form validation handler for user_login_form().
 *
 * @see user_login_form_submit()
 */
function user_login_form_validate($form, &$form_state) {
  // ...
}

/**
 * Form submission handler for user_login_form().
 *
 * @see user_login_form_validate()
 */
function user_login_form_submit($form, &$form_state) {
  // ...
}
```

- **Omit `@param` and `@return`** for the standard parameters (`$form`, `$form_state`).
- Document parameters specific to the constructor.

## Render API callbacks

Summary: `Render API callback:` + description. In the body explain where the callback is assigned.

```php
/**
 * Render API callback: Validates the maximum upload size field.
 *
 * Ensures that a size has been entered and that it can be parsed by
 * parse_size().
 *
 * This function is assigned as an #element_validate callback in
 * file_field_instance_settings_form().
 */
function _file_generic_settings_max_filesize($element, &$form_state, $form) {
  // ...
}
```

## Themeable functions

> See [API documentation samples — Classes / themeable functions](https://project.pages.drupalcode.org/coding_standards/php/documentation-examples/) for the full runnable example.

- First line: `Returns HTML for a ...`.
- Document all components of `$variables`.
- Add `@ingroup themeable`.

```php
/**
 * Returns HTML for a foo.
 *
 * @param array $variables
 *   An associative array containing:
 *   - foo: The foo object that is being formatted.
 *   - show_bar: TRUE to show the bar component, FALSE to omit it.
 *
 * @ingroup themeable
 */
function theme_foo($variables) {
  // ...
}
```

## Preprocess functions

> See [API documentation samples — Classes / preprocess functions](https://project.pages.drupalcode.org/coding_standards/php/documentation-examples/) for the full runnable example.

- First line: `Prepares variables for [description] templates.`
- Second line: `Default template: foo-bar.html.twig.`
- Document all components of `$variables`.

```php
/**
 * Prepares variables for container templates.
 *
 * Default template: container.html.twig.
 *
 * @param array $variables
 *   An associative array containing:
 *   - element: An associative array containing the properties of the element.
 *     Properties used: #id, #attributes, #children.
 */
function template_preprocess_container(&$variables) {
  // ...
}
```

## Classes and interfaces

- **All classes and their methods** (including private, excluding constructors) **must be documented**.
- Methods that override with identical doc to parent: `{@inheritdoc}`.
- Summary starts with **third person singular**: "Represents a...", "Provides...".
- Exceptions are documented with `@throws`.
- If you use a namespace in docs, **always fully-qualified** (with leading backslash).
- After `@tag` (`@param`, `@return`, `@var`), class/interface names must include the FQN.

## Data types in documentation

Compliant with [PHPDoc Type syntax](https://phpstan.org/writing-php-code/phpdoc-types) by PHPStan.

- Use **interface names** or the most general class possible.
- **Prefix with FQN** (with leading `\`).
- You can omit description if `@return` is `$this` or `static`.
- **Exact spelling** (these apply to **docblock type expressions only** — not to PHP code or prose comments):
  - `array` (NOT `Array`)
  - `bool` (NOT `boolean` nor `Boolean`)
  - `true` / `false` (lowercase, NOT `TRUE`/`FALSE`)
  - `int` (NOT `integer`)
  - `null` (NOT `NULL`)
  - `object` (NOT `stdClass`)

> **In code and in prose comments, the constants are uppercase: `TRUE`, `FALSE`, `NULL`.** The lowercase form is required *only* in the docblock type position (e.g. `@param bool|null $foo`, `@return true`). Don't lowercase `TRUE`/`FALSE` in `if ($foo === TRUE)` or in a comment like `// Returns TRUE when …`.

## Parameters with `@param`

```php
/**
 * @param string $mail
 *   The email address. The description can be longer if necessary, and if so,
 *   you can wrap it to another line.
 * @param string $from
 *   (optional) The email address to send the mail from, if different from
 *   the site-wide address.
 */
```

- **`@param` description can be omitted** ONLY if ALL of these are met (per the official docs):
  - The param is type-hinted in the signature.
  - The type is a single class/interface/scalar **without** nullable.
  - The parameter name is self-documenting.
  - The description would not add useful info beyond the name and type.
  - The `@param` type is identical to the PHP signature.
- **Optional**: mark as `(optional)`.

## Return with `@return`

```php
/**
 * @return string
 *   The email address.
 */
```

- **Required for any function with a return value**, with one documented exception: functions whose behavior is **easily described in a single line** of summary may omit `@param` and `@return` entirely. This is the same one-liner exception that the official docs apply to `@param`; do not strip the tags from anything more complex than a trivial getter or helper.
- **Forbidden if the function returns nothing** (no `@return void`, simply no tag).
- Description can be omitted if: return is type-hinted, the type is a single class/interface without nullable, and the description would not add useful info.

## Inline comments

```php
// Unselect all other contact categories.
$connection->update('contact')->fields(['selected' => 0])->execute();
```

- Strong encouragement for inline comments.
- Comment on a separate line **immediately before** the code.
- `/* */` and `//` both OK; `/* */` discouraged inside functions.
- **Perl/shell style `#` FORBIDDEN**.

## `@var` inline

Type hint for local variables when the type is not obvious:

```php
/** @var \Drupal\node\NodeInterface $node */
$node = $this->entityTypeManager->getStorage('node')->load(123);
```

- Delimiters `/** */` with **double asterisk**.
- FQN of the type, **including the variable name** at the end.

## Lists in documentation

```php
/**
 * Description with a list:
 * - Item in the list.
 * - Another item, with a sub-list:
 *   - key: Sub-list with keys, first item.
 *   - key2: (optional) Second item with a key.
 * - Last item.
 */
```

- Hyphen `-` as the first character of the item.
- Same indent for items at the same level.
- No blank lines between items of the same list.
- Keys with `:` between key and description.
- The line before the list **ends with `:`**.
- `(optional)` and `(default)` next to the `:` if applicable.
- Keys are **NOT in quotes** unless they are literal strings.

## Theme template files

For modern **Twig templates** see `../TWIG.md`. For legacy `.tpl.php` see `PHP-LEGACY.md` (sibling file).

## Order of sections in docblock

```
1. Summary line (.)
2. Additional paragraphs
3. @var
4. @param
5. @return
6. @throws
7. @ingroup
8. @deprecated
9. @see
10. @todo
11. @Plugin / @Annotation
```

## Tag reference

- **`@code … @endcode`**: code samples. Tags on their own lines, no blank line between text and code.
- **`@deprecated`**: format prescribed by the official PHP documentation standards page — `"deprecated in version-string and is removed from version-string"`, with a link to the change record or relevant issue and a hint about the replacement:
  ```
  @deprecated in %deprecation-version% and is removed from %removal-version%. %extra-info%.
  @see %cr-link%
  ```
- **`@Event`**: documents events from the event dispatcher. Goes with `@see` to the point where it's fired.
- **`@file`**: documents files. **MANDATORY** in PHP files except: PSR-4 files (should NOT have it) and `Drupal.php` (no namespace).
- **`{@inheritdoc}`**: for overrides with identical docs to parent. **Must be the only line in the docblock** (no summary, no other tags), with `{}` around it. Works for both methods and properties.
- **`@link … @endlink`**: inline HTML links.
- **`@param`**: with description indented 2 spaces.
- **`@return`**: same.
- **`@throws`**: name of the exception class on the same line, optional description on the next.
- **`@todo`**: TODO notes; ideally reference a drupal.org issue.
- **`@see`**: each reference on its own line, with no additional text.
- **`@ingroup`**: adds an item to a group/topic defined with `@defgroup`.
- **`@defgroup`**: defines a topic. In its own docblock (NOT in file/function/class).
- **`@addtogroup … @}`**: opens a group block; every item declared between `@addtogroup foo {` and `@}` is added to topic `foo`. Use sparingly — `@ingroup` per-item is usually clearer.
- **`@mainpage`**: defines the main landing page of the API documentation. Appears once per project, in a top-level documentation file (e.g. `core/MAINTAINERS.txt` or a dedicated `.api.php`).
- **`@section <id> <title>`** and **`@subsection <id> <title>`**: introduce a labelled section / sub-section inside a long docblock. The `<id>` is used as the anchor; the `<title>` is the human-readable heading. Used in conjunction with `@ref` and `@mainpage`.
- **`@ref <id>`**: cross-reference to a `@section` / `@subsection` / `@defgroup` / `@mainpage` defined elsewhere. Resolved by the API module into a link.
- **`@Annotation`, `@Plugin`, etc.**: for plugin discovery annotations. Each key on its own line. Do NOT wrap lines. **Trailing comma mandatory** after the last element.

### Plugin annotations vs PHP attributes (Drupal 10.2+)

Drupal 10.2 introduced **PHP 8 attributes** (`#[Block(...)]`) as the modern replacement for doc-block annotations (`@Block(...)`). Both styles are valid in Drupal 11. **Detect which the file uses and apply the appropriate rule set**.

#### Doc-block annotations (legacy)

Lives **inside** the class docblock, parsed by `doctrine/annotations`:

```php
/**
 * Provides a 'Hello' Block.
 *
 * @Block(
 *   id = "my_module_hello",
 *   admin_label = @Translation("Hello block"),
 *   category = @Translation("Custom"),
 * )
 */
class HelloBlock extends BlockBase {
  // ...
}
```

Rules:
- One key per line, indented one level past the opening `(`.
- **Trailing comma after the last key is mandatory** (lets `git diff` show a clean addition on the next change).
- Strings in double quotes; nested annotations like `@Translation()` are bare (not strings).
- No line continuation — long values stay on a single line.

#### PHP attributes (modern, Drupal 10.2+)

Lives **above** the class as a native PHP `#[…]` attribute:

```php
use Drupal\Core\Block\Attribute\Block;
use Drupal\Core\StringTranslation\TranslatableMarkup;

#[Block(
  id: 'my_module_hello',
  admin_label: new TranslatableMarkup('Hello block'),
  category: new TranslatableMarkup('Custom'),
)]
class HelloBlock extends BlockBase {
  // ...
}
```

Rules:
- Place the attribute **between the docblock and the class declaration**. The docblock still describes the class; the attribute carries the plugin metadata.
- Use **named arguments** (`id: '…'`) with PHP-8-style `:` (not `=`).
- Translatable labels use `new TranslatableMarkup(...)` (or the project's `t()` equivalent when available in context).
- Trailing comma after the last argument is allowed and **recommended** (same rationale as multi-line arrays).
- Each `use` statement for the attribute class goes at the top of the file like any other import.

#### Heuristics for a reviewer

| Situation | Recommended call |
|---|---|
| Module already on Drupal 10.2+ with a fresh plugin file | Prefer PHP attributes. Flag annotations as 🟢 ("consider migrating"). |
| File mixes annotations on one plugin and attributes on another in the same module | 🟡 — local inconsistency, pick one style per module. |
| Existing annotated plugin on Drupal 9 / early 10 | Leave it. Migration is a separate change, not a code-review fix. |
| New file in a codebase that has standardised on attributes | 🟡 if the new file uses annotations. |

#### Tooling

`drupal/coder` ships sniffers for **both** styles. `palantirnet/drupal-rector` has a set that converts annotations to attributes — useful when the team has decided to migrate.

### Typed properties and `@var`

PHP 7.4+ typed properties express their type at the language level. When a property is declared with a native type, the docblock **may omit `@var`** if the type is fully expressed by the declaration:

```php
class Foo {

  /**
   * The entity type manager.
   */
  protected EntityTypeManagerInterface $entityTypeManager;

}
```

Add `@var` back **only when the type cannot be expressed natively** — for example, generic-collection hints (`array<int, FooInterface>`), union types where the project still targets a PHP version that does not support them, or interface intersections in older toolchains. A non-trivial description **does not** justify a redundant `@var`: keep the prose in the docblock summary and leave the native type as the sole source of truth.

## PHPUnit tests

- Follow class standards.
- Specific tags: `@group`, `@covers`, `@coversDefaultClass`.
