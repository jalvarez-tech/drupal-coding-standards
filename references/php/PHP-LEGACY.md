# PHP — Legacy patterns (Drupal 6 / 7)

Scope: patterns the official Drupal coding standards still document for historical completeness, but that **must NOT be used in new code** for Drupal 8+ projects.

On any Drupal 8+ project (and especially on D10/D11), treat anything in this file as **read-only context** for understanding old code you may encounter in contrib modules, patches, or legacy custom modules — not as guidance for new code.

| Severity meaning in this file | |
|---|---|
| 🟥 Forbidden in new code | Modern equivalent exists; using the legacy form is a regression. |
| 🟧 Recognize only | Pattern shows up in old code; do not introduce it but you may need to read it. |

---

## 🟥 `.tpl.php` and PHP alternate-syntax templates (D6/D7)

The official standards still document alternate PHP control syntax for `.tpl.php` files:

```php
<?php if (!empty($item)): ?>
  <p><?php print $item; ?></p>
<?php endif; ?>
```

This applies **only** to Drupal 7 and earlier `.tpl.php` templates. Drupal 8+ uses **Twig** (`.html.twig`) exclusively; alternate-syntax control structures should not appear in any new D8+ PHP file (procedural module code or PSR-4 class), but are legitimate when maintaining a legacy `.tpl.php` template. In new code, see `../TWIG.md`.

---

## 🟥 `variable_get()` / `variable_set()` (D6/D7)

D7 persistent variables:

```php
// Legacy — D6/D7.
$value = variable_get('my_module_setting', 'default');
variable_set('my_module_setting', $new);
```

Modern equivalent (Drupal 8+):

```php
// Configuration API.
$value = \Drupal::config('my_module.settings')->get('setting');

// Editable.
\Drupal::configFactory()
  ->getEditable('my_module.settings')
  ->set('setting', $new)
  ->save();
```

State API (non-config runtime values):

```php
$value = \Drupal::state()->get('my_module.last_run', 0);
\Drupal::state()->set('my_module.last_run', \time());
```

---

## 🟥 `db_query()`, `db_select()`, `db_insert()`, `db_update()`, `db_delete()`

D7-style database functions are removed in Drupal 9+. The modern equivalent is the **database service**:

```php
// Legacy.
$result = db_query('SELECT nid FROM {node} WHERE type = :type', [':type' => 'article']);

// Modern.
$connection = \Drupal::database();
$result = $connection->query('SELECT nid FROM {node} WHERE type = :type', [':type' => 'article']);

// Or via dependency injection for OO code.
public function __construct(\Drupal\Core\Database\Connection $database) {
  $this->database = $database;
}
```

`Schema API` calls (`db_create_table()`, etc.) move to `$connection->schema()` similarly.

---

## 🟥 `hook_menu()` (D6/D7)

Drupal 8+ uses YAML routing (`*.routing.yml`) plus controllers/forms in `src/Controller/` and `src/Form/`. There is no `hook_menu()` implementation in modern code.

---

## 🟥 `drupal_set_message()` (D7)

Replaced by the messenger service:

```php
// Legacy.
drupal_set_message(t('Saved.'));
drupal_set_message(t('Error.'), 'error');

// Modern.
\Drupal::messenger()->addMessage($this->t('Saved.'));
\Drupal::messenger()->addError($this->t('Error.'));
```

Or via DI:

```php
public function __construct(\Drupal\Core\Messenger\MessengerInterface $messenger) {
  $this->messenger = $messenger;
}
```

---

## 🟥 `t()` inside classes

Procedural `t()` is **still valid in `.module`, `.theme`, `.install`, hook implementations, and other procedural code**. Inside classes, use the **`StringTranslationTrait`** or DI:

```php
use Drupal\Core\StringTranslation\StringTranslationTrait;

class FooBlock extends BlockBase {

  use StringTranslationTrait;

  public function build() {
    return [
      '#markup' => $this->t('Hello @name', ['@name' => $name]),
    ];
  }
}
```

---

## 🟥 `SafeMarkup::format()`

Removed. Use:

- `\Drupal\Component\Render\FormattableMarkup` for non-translated formatted markup.
- `$this->t()` (with `@`, `%`, `:` placeholders) for translated UI strings.

---

## 🟥 `Drupal.settings` (legacy JavaScript)

D7 exposed PHP-side data to JS via `drupal_add_js(['my_module' => $data], 'setting');` and read via `Drupal.settings.my_module`.

Drupal 8+ uses `drupalSettings`:

```php
// PHP — attach via #attached.
$build['#attached']['drupalSettings']['my_module']['key'] = $value;
$build['#attached']['library'][] = 'my_module/main';
```

```js
// JS — read via drupalSettings.
(($, Drupal, drupalSettings) => {
  Drupal.behaviors.myModule = {
    attach(context) {
      const value = drupalSettings.my_module.key;
    },
  };
})(jQuery, Drupal, drupalSettings);
```

See `../JAVASCRIPT.md` for behaviors and library declarations.

---

## 🟧 `module_load_include()` and `files[]` in `.info` (D6/D7)

D7 modules sometimes loaded `.inc` files explicitly:

```php
module_load_include('inc', 'node', 'node.admin');
```

And declared additional class files in `.info`:

```ini
files[] = includes/MyClass.inc
```

Drupal 8+ uses PSR-4 autoloading from `src/` and `*.routing.yml` to declare routes; you should not need `module_load_include()` in new code. Recognize this pattern if you maintain very old contrib code.

---

## 🟧 `\stdClass` node loads (D6/D7)

D7 loaded entities as anonymous objects:

```php
$node = node_load($nid);            // stdClass-like
$value = $node->field_foo[$node->language][0]['value'];
```

Drupal 8+ uses typed entity objects:

```php
$node = \Drupal::entityTypeManager()->getStorage('node')->load($nid);
$value = $node->get('field_foo')->value;
```

In injected code use `EntityTypeManagerInterface`. Never type-hint `\stdClass` for entities — see `PHP-OO-PSR4.md` § 14.

---

## 🟧 `hook_init()`, `hook_boot()`, `hook_exit()` (D6/D7)

These full-bootstrap hooks no longer exist in Drupal 8+. Equivalent behaviors are implemented via **event subscribers** (`KernelEvents::REQUEST`, `KernelEvents::RESPONSE`, `KernelEvents::TERMINATE`) declared in `*.services.yml`.

---

## 🟧 `drupal_add_js()`, `drupal_add_css()`

D7 procedural attachment. Replaced by **library declarations** in `*.libraries.yml` plus `#attached` in render arrays:

```yaml
# my_module.libraries.yml
main:
  js:
    js/main.js: {}
  css:
    component:
      css/main.css: {}
  dependencies:
    - core/drupal
```

```php
$build['#attached']['library'][] = 'my_module/main';
```

---

## When this file fires

Load `PHP-LEGACY.md` only when:

1. You see a D6/D7-flavored signal in the file you are reviewing (`variable_get()`, `db_query()`, `drupal_set_message()`, `.tpl.php`, `hook_menu()`, `module_load_include()`, etc.), **and**
2. You need to suggest the modern replacement, **or**
3. The user explicitly asks "is this still valid in Drupal 11?"

If the file you are reviewing contains only modern Drupal 8+/9+/10+/11 code, **do not load this file** — the modern patterns live in the other PHP-*.md sub-skills.
