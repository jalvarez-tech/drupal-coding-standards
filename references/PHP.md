# PHP — Drupal Coding Standards Reference (index)

This file is the **router** for the PHP sub-skills. Load only the file matching the area of the code you are reviewing.

## Sub-files

| Sub-file | Load when reviewing | Topics |
|---|---|---|
| `php/PHP-CODING.md` | any `.php` / `.module` / `.theme` / `.install` / `.inc` / `.engine` / `.profile` / `.test` / `.api.php` body | whitespace, file header, quotes, arrays, operators, control structures, function calls, line length, string concatenation, constructor calls, placeholders, E_ALL, example URLs |
| `php/PHP-DOCS.md` | docblocks, comments, `@param` / `@return` / `@throws` | docblock format, hook docs, form/theme/preprocess function docs, tag reference, inline comments, `@var` |
| `php/PHP-OO-PSR4.md` | classes, interfaces, traits, enums, services, namespaces | naming conventions, visibility, type hinting, namespaces, PSR-4, service & request-attribute naming |
| `php/PHP-EXCEPTIONS.md` | exception classes, `try` / `catch`, throwing | exception class naming, message conventions, subclass strategy, try/catch formatting |
| `php/PHP-LEGACY.md` | files with D6/D7 signals (`variable_get`, `db_query`, `drupal_set_message`, `.tpl.php`, `hook_menu`, etc.) | legacy patterns + their modern replacements; **read-only** for D8+/D11 projects |

> **Default load order** for a typical `.module` file: `php/PHP-CODING.md` + `php/PHP-DOCS.md`. Add `php/PHP-OO-PSR4.md` if there are classes in `src/`. Add `php/PHP-EXCEPTIONS.md` if exceptions are thrown/caught. Add `php/PHP-LEGACY.md` only if legacy signals appear.

## Official sources (consolidated)

- [PHP coding standards](https://project.pages.drupalcode.org/coding_standards/php/coding/)
- [API documentation and comment standards](https://project.pages.drupalcode.org/coding_standards/php/documentation/)
- [API documentation samples (concrete examples)](https://project.pages.drupalcode.org/coding_standards/php/documentation-examples/)
- [Namespaces](https://project.pages.drupalcode.org/coding_standards/php/namespaces/)
- [PSR-4 and autoload](https://project.pages.drupalcode.org/coding_standards/php/psr4/)
- [Naming standards for services and extending Symfony](https://project.pages.drupalcode.org/coding_standards/php/naming-services/)
- [Exceptions](https://project.pages.drupalcode.org/coding_standards/php/exceptions/)
- [Placeholders and delimiters](https://project.pages.drupalcode.org/coding_standards/php/placeholders-delimiters/)
- [E_ALL compliance](https://project.pages.drupalcode.org/coding_standards/php/e_all/)

---

## PHP review checklist (cross-cutting)

When reviewing a Drupal PHP file, validate:

### File structure (`php/PHP-CODING.md`)

- [ ] `<?php` at the start, **no** `?>` at the end
- [ ] File DocBlock `/** @file ... */` present (except PSR-4 files with namespace)
- [ ] `declare(strict_types=1)` (if used) well positioned
- [ ] Blank line between `<?php`, namespace, use, code
- [ ] Ends with ONE newline (`\n`)
- [ ] No BOM, no `\r\n`

### Syntax and format (`php/PHP-CODING.md`)

- [ ] 2-space indent, no tabs
- [ ] No trailing whitespace
- [ ] Lines ≤ 80 chars (with documented exceptions)
- [ ] Short array syntax `[]`
- [ ] Trailing comma in multi-line arrays
- [ ] Preferred quotes: single, except for interpolation
- [ ] Spaces around binary operators
- [ ] No spaces around unary
- [ ] Cast with space: `(int) $foo`
- [ ] Every statement ends with `;` (including one-line PHP blocks)

### Control structures (`php/PHP-CODING.md`)

- [ ] `elseif`, NOT `else if`
- [ ] Curly braces always
- [ ] Opening curly on same line
- [ ] Space between keyword and `(`

### Naming (`php/PHP-OO-PSR4.md`)

- [ ] Procedural functions: `snake_case` with module prefix
- [ ] Classes: `UpperCamelCase`, without "Drupal" or "Class"
- [ ] Methods/properties: `lowerCamelCase`, without `_` prefix
- [ ] Interfaces: suffix `Interface`
- [ ] Traits: suffix `Trait`
- [ ] Test classes: suffix `Test`
- [ ] Constants: `UPPER_SNAKE_CASE` with module prefix
- [ ] Enums: cases in `UpperCamelCase`
- [ ] Acronyms in CamelCase: `Xml`, NOT `XML`

### Type hinting (`php/PHP-OO-PSR4.md`)

- [ ] New functions/methods: type hints **required on params and return** (per [PHP coding](https://project.pages.drupalcode.org/coding_standards/php/coding/): "should be included for all new functions and methods, including new child implementations"). BC/deprecation review required only when adding to **existing** public APIs.
- [ ] No `mixed` unless justified
- [ ] No `stdClass` — use interface
- [ ] Void declared if no return
- [ ] Nullable types with `?`

### Modern PHP 8.x / Drupal 11 patterns (`php/PHP-OO-PSR4.md` § 22a)

These are **not in the official Drupal coding standards** — they are language features and core conventions. Findings default to 🟢 Suggestion unless the codebase has adopted the pattern consistently.

- [ ] Constructor property promotion for new services
- [ ] `readonly` on injected dependencies (PHP 8.1+)
- [ ] Backed enums for serialized cases (PHP 8.1+); pure enums for in-memory only
- [ ] First-class callable syntax `$obj->method(...)` over `Closure::fromCallable`
- [ ] `#[\Override]` attribute on overrides (PHP 8.3 / Drupal 11)
- [ ] Hook implementations via `#[\Drupal\Core\Hook\Attribute\Hook]` in `src/Hook/` (Drupal 11.1+) — do not duplicate the procedural form

### Visibility (`php/PHP-OO-PSR4.md`)

- [ ] Visibility declared on all methods
- [ ] Visibility declared on all properties
- [ ] `abstract`/`final` before visibility
- [ ] `static` after visibility

### Docblocks (`php/PHP-DOCS.md`)

- [ ] All functions/methods/classes/properties documented
- [ ] Summary < 80 chars, initial capital, ends in `.`
- [ ] Appropriate verb based on type (imperative for hooks, third person for classes)
- [ ] `@param` with type + description + 2-space indent (may be omitted for one-liner functions whose summary fully describes the parameter)
- [ ] `@return` with type + description (omitted if no return, OR if the function is easily described in one line)
- [ ] `@throws` listed with FQN
- [ ] `{@inheritdoc}` for identical overrides
- [ ] Hooks: `Implements hook_XYZ().`
- [ ] Correct data types: `bool` not `boolean`, `int` not `integer`, etc.
- [ ] FQN with leading `\` in `@param`/`@return`/`@var`

### Namespaces (`php/PHP-OO-PSR4.md`)

- [ ] PSR-4 compliant: `src/` maps to `Drupal\<module>\`
- [ ] Correct `use` statements (no leading `\`)
- [ ] One class per `use`
- [ ] Aliasing only for real collisions
- [ ] FQN with leading `\` for built-in classes in namespaced files

### Exceptions (`php/PHP-EXCEPTIONS.md`)

- [ ] `Exception` suffix
- [ ] Not translated
- [ ] Messages with hint about values
- [ ] Specific subclasses vs generic
- [ ] Try-catch with `catch` on separate line

### Security and E_ALL (`php/PHP-CODING.md` + `php/PHP-OO-PSR4.md`)

These items come from the official PHP coding standards pages:

- [ ] No use of reserved Symfony/Drupal request attributes (`_system_path`, `_title`, `_route`, `_controller`, `_content`, …) — see `php/PHP-OO-PSR4.md` § 23
- [ ] `isset()` / `!empty()` correct based on context — see `php/PHP-CODING.md` § 25
- [ ] No SQL injection: placeholders in queries (see also `SQL.md`)
- [ ] Alphanumeric placeholders with module prefix — see `php/PHP-CODING.md` § 24

### Drupal API / project conventions

These items are **Drupal API conventions**, not PHP coding standards in the strict sense (the `/coding_standards/php/*` pages do not cover them). Sources are the Drupal.org developer documentation and project-specific rules. When reporting deviations, use the main severity criteria in `SKILL.md` — a real runtime break is 🔴, a failed static-analysis baseline is 🟡, a stylistic divergence is 🟢. Do not auto-classify everything under this heading as 🟡; classify by impact, not by section.

- [ ] UI strings wrapped in `t()` / `$this->t()` — sources: [Drupal.org Translation API](https://www.drupal.org/docs/develop/coding-standards/translation-api), enforced by PHPStan with `mglaman/phpstan-drupal`

### Drupal-specific patterns (`php/PHP-DOCS.md` + `php/PHP-OO-PSR4.md`)

- [ ] Correct hook implementations
- [ ] Form constructors with `@ingroup forms`
- [ ] Themeable functions with `@ingroup themeable`
- [ ] Preprocess functions document `$variables`
- [ ] Render API callbacks documented
- [ ] Plugin annotations: keys one per line, trailing comma

### Legacy regression check (`php/PHP-LEGACY.md`)

- [ ] No `variable_get()` / `variable_set()` (use Config / State API)
- [ ] No `db_query()` / `db_select()` / `db_insert()` / `db_update()` / `db_delete()` (use `\Drupal::database()`)
- [ ] No `hook_menu()` (use `*.routing.yml`)
- [ ] No `drupal_set_message()` (use `\Drupal::messenger()`)
- [ ] No `Drupal.settings` in JS (use `drupalSettings` + `#attached`)
- [ ] No `module_load_include()` in new code
- [ ] No `\stdClass` typed entities — use `EntityInterface` and storage

---

## PHPStan baseline awareness

PHPStan supports a **baseline file** (`phpstan-baseline.neon` or similar) that records pre-existing findings the team has consciously deferred. Many Drupal projects ship one at the repo root or under `scripts/phpstan/`. Before flagging a PHPStan-shape finding as 🟡:

1. Run `vendor/bin/phpstan analyse --generate-baseline=/tmp/check.neon <path>` and diff against the project's baseline.
2. If the finding is already in the baseline, **it has been accepted** — do not re-report it. Mention it as supplementary context if relevant to the change.
3. If the change you are reviewing *removes* baseline'd code or modifies the surrounding lines, that finding may need to leave the baseline. Check whether `phpstan analyse` (without `--generate-baseline`) still passes.

This is the same principle the official phpstan docs describe under "Baseline" — Drupal does not codify it in `coding_standards/`, but every PHPStan-using project relies on it.

---

## Helper modules for validation

Default to the host binaries — these work on any Drupal project once `drupal/coder` and `phpstan/phpstan` are installed via Composer:

```bash
composer require --dev drupal/coder
vendor/bin/phpcs --standard=Drupal,DrupalPractice <path>
vendor/bin/phpcbf --standard=Drupal,DrupalPractice <path>

composer require --dev phpstan/phpstan mglaman/phpstan-drupal
vendor/bin/phpstan analyse <path>
```

If the project ships a DDEV (or Lando / Docksal / Makefile) wrapper, prefer the wrapped form so tool versions match CI:

```bash
# Direct path-scoped binaries inside the DDEV container.
ddev exec ./vendor/bin/phpcs --standard=Drupal,DrupalPractice <path>
ddev exec ./vendor/bin/phpcbf --standard=Drupal,DrupalPractice <path>
ddev exec ./vendor/bin/phpstan analyse <path>

# If the project exposes Composer scripts (e.g. check-code:phpcs, check-code:phpcbf),
# prefer those — but watch for two common defects:
#  (a) wrappers that derive their target list from `git status` or `git diff --cached`
#      will scan nothing on a clean post-commit tree and exit 0 without doing anything;
#  (b) a single EXTENSIONS list shared between PHPCS/PHPStan/cspell will break when
#      PHPStan is asked to parse .md or .yml as PHP.
# When in doubt, fall back to the direct path-scoped binaries above.
```
