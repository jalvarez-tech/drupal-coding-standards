# PHP — OO, naming, namespaces, PSR-4, services

Scope: naming conventions (procedural + OO), classes/interfaces/traits/enums, visibility, type hinting, namespaces, PSR-4 autoloading, service & request-attribute naming.

Sources:
- [PHP coding standards (naming sections)](https://project.pages.drupalcode.org/coding_standards/php/coding/)
- [Namespaces](https://project.pages.drupalcode.org/coding_standards/php/namespaces/)
- [PSR-4 and autoload](https://project.pages.drupalcode.org/coding_standards/php/psr4/)
- [Naming standards for services and extending Symfony](https://project.pages.drupalcode.org/coding_standards/php/naming-services/)

For formatting/syntax: see `PHP-CODING.md`.
For docblocks: see `PHP-DOCS.md`.

---

## 11. Naming conventions

### Functions and variables (procedural)

- **Functions**: lowercase, words separated by `_`. **Prefix with module name** to avoid collisions.
  - ✅ `my_module_helper_function()`
  - ❌ `myModuleHelperFunction()`
- **Variables**: lowercase. Can use `$lowerCamelCase` or `$snake_case` — **but be consistent within the same file**, don't mix.

### Persistent variables

> Drupal 7 used `variable_get()` / `variable_set()` with `lowercase_with_underscores` + module prefix. Drupal 8+ uses **Configuration API** (`\Drupal::config()`, `*.settings.yml`). See `PHP-LEGACY.md`.

### Constants

- **ALL_UPPERCASE_UNDERSCORES**.
- **Module prefix in uppercase**.
- Also applies to `TRUE`, `FALSE`, `NULL`.
- **Drupal 8+**: use `const` (not `define()`) for performance:
  ```php
  /**
   * Indicates that the item should be removed at the next general cache wipe.
   */
  const CACHE_TEMPORARY = -1;
  ```
- **`define()`** only when defined conditionally or with non-literal value:
  ```php
  if (!defined('MAINTENANCE_MODE')) {
    define('MAINTENANCE_MODE', 'error');
  }
  ```

### Global variables

If you need to define globals, their name starts with `_` + module name + `_`.

### Classes, methods and properties

1. **Classes / interfaces / traits / enums**: `UpperCamelCase`.
2. **Methods and class properties**: `lowerCamelCase`. *Exception*: properties of **configuration entities** in Drupal 8+ may use underscores.
3. **Acronyms in CamelCase**: `SampleXmlClass`, NOT `SampleXMLClass`. (Standard adopted in March 2013, reversing the previous one.)
4. **NO** underscores in class names unless absolutely necessary.
5. **NO** including "Drupal" in the name.
6. **NO** "Class" as suffix.
7. **Interfaces** always suffix `Interface`. **Non-interfaces** never with suffix `Interface`.
8. **Traits** always suffix `Trait`. **Non-traits** never with suffix `Trait`.
9. **Test classes** always suffix `Test`. **Non-tests** never with suffix `Test`.
10. **Protected/private properties and methods** **do NOT use `_` prefix**. The underscore prefix **has no meaning**.
11. **Standalone names**: must describe what they do without relying on the namespace, be short and unambiguous.

| Namespace | ✅ Good name | ❌ Bad names |
|---|---|---|
| `Drupal\Core\Database\Query\` | `QueryCondition` | `Condition` (ambiguous), `DatabaseQueryCondition` (redundant) |
| `Drupal\Core\FileTransfer\` | `LocalFileTransfer` | `Local` (ambiguous) |
| `Drupal\Core\Cache\` | `CacheDatabaseDriver` | `Database` (ambiguous), `DatabaseDriver` (ambiguous) |
| `Drupal\entity\` | `Entity`, `EntityInterface` | `DrupalEntity`, `EntityClass` |
| `Drupal\comment\Tests\` | `ThreadingTest` | `CommentThreadingTest`, `Threading` |

Complete class + interface example:

```php
interface FelineInterface {

  public function meow();

  public function eatLasagna($amount);

}

class GarfieldTheCat implements FelineInterface {

  protected $lasagnaEaten = 0;

  public function meow() {
    return t('Meow!');
  }

  public function eatLasagna($amount) {
    $this->lasagnaEaten += $amount;
  }

}
```

### Enums

Same rules as classes, but **cases use `UpperCamelCase`**:

```php
enum Exists {

  case ErrorIfExists;
  case ErrorIfNotExists;
  case ReturnEarlyIfExists;
  case ReturnEarlyIfNotExists;

}
```

### File names

- Documentation: `.txt` (plain text) or `.md` (Markdown).
- **Base file name in uppercase** (`README`, NOT `readme`).
- **Extension in lowercase** (`.md`, NOT `.MD`).
- Examples: `README.md`, `INSTALL.txt`, `TODO.txt`, `CHANGELOG.txt`.

---

## 12. Classes, interfaces, traits, enums

### Where to define

- **One class / interface / trait per file**.
- The file is named the same as the class: `FooInterface` → `FooInterface.php`.
- Autoloading via **PSR-4** (see section 21).
- In core: PSR-4 tree from `core/lib/`.
- In modules: PSR-4 tree from `modulename/src/`.
- **Don't define classes in `.module`** unless the class has no superclass that might not be available when loading the `.module`.

### Use of interfaces

- **Strongly recommended** to separate interface from implementation.
- If there is a remote possibility of swapping a class for another implementation, **separate method definitions into a formal Interface**.
- A class intended to be extended must **always provide an Interface** instead of forcing other classes to extend the base.

### Instantiation

- **Use a factory function when you need an indirection layer** — the function can return different object implementations (same interface) based on context. This pattern improves testability and flexibility, regardless of chaining syntax.

---

## 13. Visibility

### Methods and functions

- **Visibility MANDATORY** on all methods.
- **NO** `_` prefix to indicate protected/private.
- **NO** space after the method name.
- **Closing brace** on its own line following the body.
- **No space** after `(` or before `)`.

### Properties

- **Visibility MANDATORY** on all properties.
- **Public properties STRONGLY DISCOURAGED** — they generate unwanted side effects and expose implementation details.

### abstract, final, static

- When present, **`abstract` and `final`** go **before** the visibility declaration.
- When present, **`static`** goes **after** the visibility declaration.

```php
abstract public function meow();
final protected function eat();
public static function newKitten();
```

*Extract from [PSR-12 section 4.6](https://www.php-fig.org/psr/psr-12/#46-abstract-final-and-static).*

---

## 14. Type hinting (parameters and return)

From **Drupal 9** onwards:

### New functions / methods

- **Required for all new functions and methods, including new child implementations of methods for existing classes and interfaces** — for both parameters and return values. Official wording from [PHP Coding standards](https://project.pages.drupalcode.org/coding_standards/php/coding/): *"Parameter and return type hints should be included for all new functions and methods, including new child implementations of methods for existing classes and interfaces."*

```php
public function myMethod(MyInterface $myClass, string $id): array {
  // Method code here.
}
```

When a type hint genuinely cannot be expressed (callback-style return values, mixed/union edge cases — see "Notes per type" below), omit it but document the rationale in the docblock.

### Existing functions / methods

- Adding type hints to a public API is a **backwards compatibility break** — it requires a **BC / deprecation review**.
- Can be added in a major version if a **deprecation warning** is emitted in a previous minor.

### Notes per type

- **`mixed`**: only in rare cases (callback return values, markup strings that can be objects or scalar). If you need `mixed` or a **union type**, consider refactoring.
- **Nullable types**: use `?Type` when the data may be a specific type or `null`.
- **Objects**: use **interface type hint**, NOT concrete classes. The most specific interface that covers all possible values. **NEVER `stdClass`**.
- **Void**: if the function/method returns nothing, use `void`.

---

## 20. Namespaces and use

### "use"-ing classes

- **Classes/interfaces with `\` in the FQN** (e.g. `Drupal\Tests\BrowserTestBase`): **don't use FQN inline**. If the namespace differs, **`use`** at the top:
  ```php
  namespace Drupal\my_module\Tests\Foo;

  use Drupal\Tests\BrowserTestBase;

  /**
   * Tests that the foo bars.
   */
  class BarTest extends BrowserTestBase {
    // ...
  }
  ```
- **Classes/interfaces without `\` in the FQN** (e.g. PHP's built-in `Exception`): **must be fully qualified** in namespaced files — reference them with a leading `\` inline (e.g. `new \Exception();`, `\InvalidArgumentException::class`). The convention in Drupal core is to write the leading `\` at the use site rather than importing global classes via `use \Exception;`.
- In a file WITHOUT declared namespace (global namespace), classes in other namespaces are specified with `use` at the top.
- **No leading `\`** in use statements ([official PHP recommendation](http://www.php.net/manual/en/language.namespaces.importing.php)).
- When specifying class name in a string, **FQN without leading `\`**.
- In **double-quoted strings escape the separator**: `"Drupal\\Context\\ContextInterface"`.
- In **single-quoted DO NOT escape**: `'Drupal\Context\ContextInterface'`. Single quotes preferred.
- **One class per `use` statement**. Don't mix several in a single `use`.
- There is no strict standard for the order of `use`; use readability criteria.
- API docs (in `.api.php`): **full FQNs**.

### Class aliasing

Only to avoid **collisions**. If two classes collide, **alias BOTH** prefixing the next portion of the namespace:

```php
use Foo\Bar\Baz as BarBaz;
use Stuff\Thing\Baz as ThingBaz;
```

### Module namespaces

Module convention: `Drupal\module_name\…`

Drupal follows **PSR-4**. A class in `'module folder'/src/SubFolder1/SubFolder2` declares namespace `Drupal\module_name\SubFolder1\SubFolder2`. The `/src/` is **omitted** from the namespace.

Examples:
- Class `Drupal\example_module\Foo` → file `example_module/src/Foo.php`.
- Class `Drupal\example_module\Foo\Bar` → file `example_module/src/Foo/Bar.php`.

---

## 21. PSR-4 autoloading

Drupal follows [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md). PSR-4 autoloading applies to **both modules and themes** (and profiles) — the directory layout, namespace prefix, and `src/` mapping are identical. The examples below use a module, but `themes/<theme_name>/src/` resolves to `Drupal\<theme_name>\…` the same way; e.g. `themes/olivero/src/Plugin/Block/FooBlock.php` → `Drupal\olivero\Plugin\Block\FooBlock`.

### Typical module structure

```
modules/vegetable/
├── css/
├── js/
├── src/
│   ├── Controller/
│   │   └── VegetableController.php  → Drupal\vegetable\Controller\VegetableController
│   ├── Form/
│   │   └── VegetableForm.php        → Drupal\vegetable\Form\VegetableForm
│   ├── Plugin/
│   │   └── Block/
│   │       └── VegetableBlock.php   → Drupal\vegetable\Plugin\Block\VegetableBlock
│   └── Entity/
│       ├── Tomato.php               → Drupal\vegetable\Entity\Tomato
│       └── Cucumber.php             → Drupal\vegetable\Entity\Cucumber
├── templates/
├── tests/
│   └── src/
│       ├── Functional/
│       ├── Kernel/
│       ├── Unit/
│       │   └── TomatoTest.php       → Drupal\Tests\vegetable\Unit\TomatoTest
│       └── Traits/
│           └── VegetableTestTrait.php  → Drupal\Tests\vegetable\Traits\VegetableTestTrait
├── vegetable.info.yml
├── vegetable.routing.yml
└── vegetable.module
```

### Logic

1. Each module has namespace `Drupal\<module>\`.
2. The namespace maps to `./src/`.
3. What comes after the namespace maps directly to the directory structure inside `src/`.

### Namespace resolution

| Base namespace | Base directory | Contains |
|---|---|---|
| `Drupal\Component\` | `core/lib/Drupal/Component/` | Components reusable outside of Drupal |
| `Drupal\Core\` | `core/lib/Drupal/Core/` | Drupal-specific components |
| `Drupal\Tests\` | `core/tests/Drupal/Tests/` | PHPUnit tests for core |
| `Drupal\$modulename\` | `modules/$modulename/src/` | Module integration |
| `Drupal\$themename\`¹ | `themes/$themename/src/` | Theme integration (blocks, plugins, services declared by a theme) |
| `Drupal\Tests\$modulename\` | `modules/$modulename/tests/src/` | PHPUnit tests for the module |
| `Drupal\Tests\$themename\` | `themes/$themename/tests/src/` | PHPUnit tests for the theme |

For modules and themes, `$modulename` / `$themename` = machine name in lowercase with underscores.

> ¹ **Theme PSR-4** is a Drupal core convention — themes can ship PSR-4-loadable PHP under `src/` (plugins, hooks, services). The [official PSR-4 page](https://project.pages.drupalcode.org/coding_standards/php/psr4/) only documents the module mapping; theme support is implemented by core but is not codified on that page.

**Each class/interface/trait lives in a separate file**. E.g. `Drupal\Component\Diff\Engine\DiffEngine` → `core/lib/Drupal/Component/Diff/Engine/DiffEngine.php`.

> Drupal 7 autoloading (`.inc` files + `files[]` in `.info`) is **legacy** — see `PHP-LEGACY.md`.

---

## 22a. Modern PHP 8.x / Drupal 11 patterns

> **Source note**: the official Drupal coding standards page does **not** prescribe any of the items below. They are PHP language features (PHP 8.0–8.3) and Drupal 11.x conventions that have emerged in core itself but are not codified at `project.pages.drupalcode.org`. Treat findings here as **🟢 Suggestion** by default; promote to 🟡 only when the codebase has already adopted the pattern consistently and a single file deviates from the local convention.

### Constructor property promotion (PHP 8.0+)

Drupal core uses constructor property promotion in services and event subscribers introduced from D10.2 onward. The pattern collapses property declaration, constructor parameter, and assignment into a single signature:

```php
// ✅ Modern (PHP 8.0+).
public function __construct(
  private readonly EntityTypeManagerInterface $entityTypeManager,
  private readonly LoggerChannelInterface $logger,
  protected ConfigFactoryInterface $configFactory,
) {}

// ❌ Pre-PHP 8 style — still valid, but verbose. Don't migrate existing code
// just for style; new services and refactors should prefer promotion.
protected EntityTypeManagerInterface $entityTypeManager;
protected LoggerChannelInterface $logger;
protected ConfigFactoryInterface $configFactory;

public function __construct(
  EntityTypeManagerInterface $entity_type_manager,
  LoggerChannelInterface $logger,
  ConfigFactoryInterface $config_factory,
) {
  $this->entityTypeManager = $entity_type_manager;
  $this->logger = $logger;
  $this->configFactory = $config_factory;
}
```

**Notes**:
- Promoted parameters **may** carry `readonly`, visibility, attributes, and default values.
- Promotion is **only** allowed in non-abstract constructors. Abstract bases must still declare properties explicitly.
- Camel-case the promoted parameter name (it becomes a property): `$entityTypeManager`, not `$entity_type_manager`.

### `readonly` properties (PHP 8.1+) / readonly classes (PHP 8.2+)

Use `readonly` for dependencies that are injected once and never mutated:

```php
final class MyService {
  public function __construct(
    private readonly EntityTypeManagerInterface $entityTypeManager,
  ) {}
}
```

PHP 8.2 added `readonly class MyValueObject { … }` — every property becomes implicitly readonly. Drupal core uses readonly value objects in newer subsystems (e.g. `Drupal\Core\Render\AttachmentsInterface` adjacent types). Apply only when the type is truly immutable.

### Backed enums (PHP 8.1+)

Drupal core has migrated several constant groups to backed enums (e.g. `Drupal\Core\Cache\Cache::PERMANENT` → enum cases in newer modules). When you introduce one:

```php
enum CacheLifetime: int {
  case Permanent = -1;
  case Default = 0;
  case OneHour = 3600;

  public function inSeconds(): int {
    return $this->value;
  }
}
```

- Use **backed enums** (`enum Foo: int { … }` or `enum Foo: string { … }`) when the cases must serialize to a primitive (config, database, query params).
- Use **pure enums** (no backing type) only when the cases never leave PHP memory.
- Case names: `UpperCamelCase` (see § 11).
- Type-hint enums in signatures: `function setLifetime(CacheLifetime $lifetime): void`.

### First-class callable syntax (PHP 8.1+)

Replace `Closure::fromCallable($obj->method(...))` boilerplate with the first-class form:

```php
// ✅ Modern.
$callback = $this->renderer->render(...);

// ❌ Pre-PHP 8.1.
$callback = [$this->renderer, 'render'];
$callback = \Closure::fromCallable([$this->renderer, 'render']);
```

Useful when registering event listeners, queue worker callbacks, or `array_map` arguments.

### `#[\Override]` attribute (PHP 8.3+)

PHP 8.3 introduces `#[\Override]` to assert at compile time that a method overrides a parent (instead of accidentally introducing a new method that no longer overrides anything). Drupal 11 runs on PHP 8.3 — annotate overrides in new code:

```php
final class MyBlock extends BlockBase {

  #[\Override]
  public function build(): array {
    return [
      '#markup' => $this->t('Hello'),
    ];
  }

  #[\Override]
  public function defaultConfiguration(): array {
    return parent::defaultConfiguration() + [
      'foo' => 'bar',
    ];
  }
}
```

If you rename or remove the parent method, the runtime fatally errors instead of silently dropping the override. Pair with the standard `{@inheritdoc}` docblock.

### Hooks as OOP methods with `#[Hook]` (Drupal 11.1+)

From Drupal 11.1, hook implementations may be declared as methods on a class with the `#[\Drupal\Core\Hook\Attribute\Hook]` attribute, removing the need for procedural `mymodule_hook_name()` functions in the `.module` file:

```php
// src/Hook/MyModuleHooks.php
namespace Drupal\my_module\Hook;

use Drupal\Core\Hook\Attribute\Hook;

final class MyModuleHooks {

  #[Hook('node_presave')]
  public function nodePresave(NodeInterface $node): void {
    // Logic that used to live in my_module_node_presave().
  }

  #[Hook('form_node_form_alter')]
  public function formNodeFormAlter(array &$form, FormStateInterface $form_state, string $form_id): void {
    // ...
  }

}
```

**Notes**:
- Method name: `lowerCamelCase` (not `snake_case`).
- The class must be discoverable — typically under `src/Hook/`.
- Class itself does not need to extend anything; the attribute is the discovery mechanism.
- A module **may mix** procedural hooks and OOP hooks during a transition; both run.
- Do **not** retroactively migrate hooks in a stable contrib module without coordinating — keep procedural where the surrounding code is procedural.

### Migration tooling

For automated upgrades to the patterns above, use [`palantirnet/drupal-rector`](https://github.com/palantirnet/drupal-rector). It ships sets for `D9_LEGACY`, `D10_ALL_DEPRECATIONS`, and `D11_ALL_DEPRECATIONS` and can apply many of the conversions (procedural hooks → OOP, annotations → attributes, deprecated method renames). Run it as `vendor/bin/rector process modules/custom --dry-run` to preview.

### Severity guidance for this section

| Finding | Default severity |
|---|---|
| New code missing `#[\Override]` on a clear override | 🟢 Suggestion |
| Mixing styles in the same class (some properties promoted, some not, no reason given) | 🟡 Standards violation (local convention) |
| Using `!` placeholder in `t()`, missing `readonly` on injected deps, etc. | Stay 🟢 unless the codebase has documented the rule |
| Migrating a stable hook to `#[Hook]` and leaving the old function in place | 🟡 (duplicate registration is a smell) |

---

## 23. Naming standards for services and Symfony

### Request attributes

Elements added to the attributes of the Request object by modules or services must have **`_` prepended** unless they come from the path.

```php
\Drupal::request()->attributes->set('_context_value', $myValue);
```

Only values that come from the path (e.g. pattern `/node/{node}`) omit the `_`.

### Reserved attributes (DO NOT override)

Drupal core and Symfony add these prefixed ones:

- `_system_path`
- `_title`
- `_account`
- `_route` (from `Symfony\Cmf\Component\Routing\RouteObjectInterface`)
- `_route_object`
- `_controller`
- `_content`
