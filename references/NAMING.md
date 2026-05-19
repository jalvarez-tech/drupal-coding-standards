# Naming — Drupal Conventions Consolidated

Scope: every Drupal symbol — variables, functions, hooks, services, plugins, fields, bundles, routes, blocks, paragraphs, content types, views, libraries, permissions, config keys — in one place. Cross-references the dispersed sub-skills so reviewers don't have to chase rules across files.

Sources (canonical):
- [PHP coding standards § Naming](https://project.pages.drupalcode.org/coding_standards/php/coding/) — covers procedural functions, variables, classes, methods, properties, constants, enums.
- [Naming standards for services and extending Symfony](https://project.pages.drupalcode.org/coding_standards/php/naming-services/) — service IDs, request attributes.
- [PSR-4 and autoload](https://project.pages.drupalcode.org/coding_standards/php/psr4/) — namespace ↔ directory mapping.
- [SQL coding conventions](https://project.pages.drupalcode.org/coding_standards/sql/conventions/) — table and column names.
- [Drupal.org module naming](https://www.drupal.org/docs/develop/coding-standards/module-naming-conventions) — module machine names.
- [Drupal.org permission naming](https://www.drupal.org/docs/develop/coding-standards/permission-naming-conventions) — `verb noun` form.

For the formal rule text and full examples, see `php/PHP-OO-PSR4.md` § 11–14, § 20–21, § 23. This file is the **quick-reference**; the sub-files are the source of truth.

---

## 1. Modules and themes (machine names)

- **lowercase**, words separated by `_`. Match `[a-z_][a-z0-9_]*`.
- Start with an **organization prefix** when the module is project-specific (e.g. `acme_<purpose>`, `mycompany_<purpose>`). Stay consistent inside a single codebase — pick one prefix and use it everywhere new custom modules live. Legacy prefixes from earlier eras of the project are fine to keep on existing modules but should not be introduced for new ones.
- Avoid generic words alone (`block`, `node`, `views`) — they collide with core.

| ✅ | ❌ |
|---|---|
| `acme_blocks` | `AcmeBlocks` |
| `acme_cross_reference` | `acme-cross-reference` |
| `mycompany_eloqua` (legacy prefix kept for an existing module) | `eloqua` |

Themes follow the same rules. Custom theme machine names use the same lowercase-underscore form (e.g. `mytheme`, `acme_theme`).

## 2. Procedural functions and variables

- **Functions**: `lowercase_with_underscores`, **prefixed with the module/theme machine name** to avoid collisions across the autoloaded global namespace.
- **Variables**: `$lowercase_with_underscores` OR `$lowerCamelCase`. Pick one per file and be consistent within that file. New code defaults to `$lowercase_with_underscores` for parity with procedural Drupal code.

| ✅ | ❌ |
|---|---|
| `acme_blocks_preprocess_block(&$variables)` | `preprocess_block($variables)` (no prefix) |
| `acme_blocks_preprocess_block(&$variables)` | `acmeBlocksPreprocessBlock(&$variables)` (camelCase) |
| `$node_revision` or `$nodeRevision` | mixing `$node_revision` and `$nodeRevision` in the same file |

Hooks are procedural: `<module>_<hook_name>(...)`. See § 5.

## 3. Classes, interfaces, traits, enums (`src/`)

- **Class / interface / trait / enum names**: `UpperCamelCase`, no `_`.
- **Acronyms** in CamelCase form: `Xml`, `Json`, `Api`, `Sso`, `Pdf`. **Not** `XML`, `JSON`, `API`.
- **Do NOT** include `Drupal` in the class name.
- **Do NOT** suffix `Class`.
- **Interfaces** end with `Interface`. Non-interfaces never carry that suffix.
- **Traits** end with `Trait`. Non-traits never carry that suffix.
- **Test classes** end with `Test`. Non-tests never carry that suffix.
- **Enum cases**: `UpperCamelCase` (`ErrorIfExists`, not `ERROR_IF_EXISTS`).

| Namespace | ✅ | ❌ |
|---|---|---|
| `Drupal\acme_blocks\Plugin\Block` | `AcmeCrossReferenceBlock` | `AcmeCrossReferenceBlockClass` |
| `Drupal\acme_seo\Service` | `MetaTagBuilder`, `MetaTagBuilderInterface` | `DrupalMetaTagBuilder`, `IMetaTagBuilder` |
| `Drupal\acme_algolia\Plugin\AlgoliaIndexer` | `SoftwareToolIndexer` | `Software_Tool_Indexer` |

## 4. Methods and properties

- **`lowerCamelCase`**, no leading `_` for protected/private (Drupal explicitly states the underscore prefix has no meaning).
- **Public properties** are strongly discouraged — prefer getters/setters.
- **Configuration entity** properties may use `lowercase_with_underscores` (the entity's stored data uses underscores; matching the schema simplifies reading).

| ✅ | ❌ |
|---|---|
| `public function buildForm(...)` | `public function build_form(...)` |
| `protected MessengerInterface $messenger` | `protected MessengerInterface $_messenger` |

## 5. Hook implementations

- Procedural: `<module>_<hook_name>(...)`.
- Variants: `<module>_form_<form_id>_alter(...)`, `<module>_<entity_type>_presave(...)`, etc.
- Drupal 11 (10.2+) also supports **attribute-based hooks** via `src/Hook/`: a class method tagged with `#[\Drupal\Core\Hook\Attribute\Hook('node_presave')]`. The method name follows § 4 (`lowerCamelCase`); the hook name (the attribute argument) is the canonical procedural form. Pick one form per module — do **not** duplicate.

| ✅ | ❌ |
|---|---|
| `acme_blocks_block_presave(BlockInterface $block)` | `blockPreSave(BlockInterface $block)` (no module prefix, camelCase) |
| `#[Hook('node_presave')] public function nodePresave(NodeInterface $node)` | `#[Hook('nodePresave')]` (hook name must be procedural form) |

## 6. Services (DI container)

- **Service ID**: `<module>.<service_role>` — lowercase, dot-separated. Avoid `_` inside a segment (use dots to separate, `_` for word-joining within a segment).
- **Class**: `Drupal\<module>\<Subnamespace>\<Name>` (PSR-4 — see `php/PHP-OO-PSR4.md` § 21).
- **Interface for the service**: `Drupal\<module>\<Subnamespace>\<Name>Interface`. Use the interface as the type-hint in consumers, not the concrete class.

| Service ID | Class | Interface |
|---|---|---|
| `acme_blocks.cross_reference_builder` | `Drupal\acme_blocks\Service\CrossReferenceBuilder` | `Drupal\acme_blocks\Service\CrossReferenceBuilderInterface` |
| `acme_sso.jwt_manager` | `Drupal\acme_sso\JwtManager` (✅ current rule: `Jwt`, not `JWT`) | `Drupal\acme_sso\JwtManagerInterface` |

## 7. Request attributes (Symfony Request)

- Project-added attributes: prefix with `_` (e.g. `_context_value`). The `_` distinguishes them from path-pattern attributes (which omit the `_`).
- **Reserved** — do not override: `_system_path`, `_title`, `_account`, `_route`, `_route_object`, `_controller`, `_content`.

## 8. Hooks vs events

- **Hooks**: `<module>_<hook_name>` (see § 5). Drupal-specific.
- **Event subscribers**: class `Drupal\<module>\EventSubscriber\<Name>EventSubscriber` implementing `EventSubscriberInterface`. Method names are `lowerCamelCase` matching the event constant (`onKernelRequest`, `onPaymentComplete`).

## 9. Plugins (blocks, fields, formatters, etc.)

- **Plugin ID**: `<module>_<plugin_role>` — lowercase, `_` separated. Matches the directory name structure and is what the plugin manager registers under.
- **Class**: `Drupal\<module>\Plugin\<PluginType>\<UpperCamelCaseName>` per PSR-4.
- Annotation/attribute keys (`id`, `label`, `category`, etc.): keys are bare identifiers; values are quoted strings.

| Plugin type | Plugin ID | Class |
|---|---|---|
| Block | `acme_blocks_cross_reference` | `Drupal\acme_blocks\Plugin\Block\CrossReferenceBlock` |
| Field formatter | `acme_products_orderable_part_status` | `Drupal\acme_products\Plugin\Field\FieldFormatter\OrderablePartStatusFormatter` |

## 10. Routes

- **Route name**: `<module>.<verb>_<noun>` or `<module>.<short_name>` — lowercase, `_` inside a segment, `.` between module and route. Match `[a-z_][a-z0-9_.]*`.

| ✅ | ❌ |
|---|---|
| `acme_secure_access.request_form` | `acme_secure_access.RequestForm` |
| `acme_seo.canonical_tag_settings` | `acme-seo/canonical-tag-settings` |

## 11. Content types, paragraphs, media types, view modes

- **Machine name**: `lowercase_with_underscores`. Match `[a-z_][a-z0-9_]*`.
- Names are project-scoped (no module prefix required) but **must be unique across the site**.

| ✅ | ❌ |
|---|---|
| `generic_product` | `GenericProduct` |
| `application_category` | `application-category` |
| `case_study` | `caseStudy` |

## 12. Fields

- **Machine name**: `field_<short_name>`, lowercase, `_` separated. The `field_` prefix is mandatory for site fields (base entity fields use bare names).
- Keep field names **short and descriptive** — they show up in queries, exports, and YAML config.

| ✅ | ❌ |
|---|---|
| `field_part_number` | `field_PartNumber` |
| `field_orderable_parts` | `orderable_parts_field` |

Base entity fields (defined in `baseFieldDefinitions`) follow § 4 (`lowerCamelCase` property names but the **machine name in the entity definition** is `lowercase_with_underscores` — e.g. the property `$this->changed` maps to the field key `changed`).

## 13. Configuration keys and config-entity IDs

- **Config object name**: `<module>.<thing>` (e.g. `acme_seo.settings`) — file at `config/install/<name>.yml`.
- **Keys inside a config object**: `lowercase_with_underscores`. Nested keys allowed.
- **Config-entity IDs**: lowercase, `_` separated, **no leading digit**.

```yaml
# acme_seo.settings.yml
canonical:
  enabled: true
  base_url: 'https://www.example.com'
  fallback_strategy: 'language_neutral'
```

## 14. Permissions

- Form: `<verb> <noun>` — the permission key in `<module>.permissions.yml` is a space-separated, lowercase string (e.g. `approve secure access requests`). The same string is the machine name used by `user_has_permission()`; the human-readable label goes under the `title` sub-key:

```yaml
# acme_secure_access.permissions.yml
approve secure access requests:
  title: 'Approve secure access requests'
  description: 'Approve or deny user requests for restricted documents.'
  restrict access: true
```

The `restrict access: true` flag is required for any permission that grants trust to bypass typical access controls — see [Drupal.org permission naming](https://www.drupal.org/docs/develop/coding-standards/permission-naming-conventions).

## 15. Libraries (asset bundles)

- **Library ID**: `<module>/<bundle_name>`, lowercase, `_` separated.
- Declared in `<module>.libraries.yml`. Referenced as `module/bundle` in `#attached['library']`.

```yaml
# acme_frontend_widgets.libraries.yml
cross_reference:
  version: 1.0
  js:
    js/cross-reference.js: {}
  css:
    component:
      css/cross-reference.css: {}
  dependencies:
    - core/drupal
    - core/once
```

## 16. Views

- **Machine name**: `lowercase_with_underscores`. Match `[a-z_][a-z0-9_]*`.
- Display names: human-readable (any case); Views auto-derives the machine name from the title on first save — review before exporting config.

## 17. Database tables and columns (Schema API)

- **Table name**: `lowercase_with_underscores`. Drupal-managed tables get a curly-brace wrapper in queries (`{node}`) so the DB prefix can be applied.
- **Column name**: `lowercase_with_underscores`. Reserved SQL keywords (`UID`, `ORDER`, `TYPE`, `KEY`, etc. — see `SQL.md`) must be quoted via Schema API, but **avoid them** for cross-DB portability.
- **Index name**: `<table>__<columns>__idx` or follow Drupal core's convention seen in `*.install` schema hooks.

## 18. Migrations

- **Migration ID**: `<source>_to_<destination>` or `<module>_<noun>` — lowercase, `_` separated. Match the file name `migrations/<id>.yml`.

## 19. Cache tags, contexts, max-age

- **Cache tags**: `<entity_type>:<id>`, `<entity_type>_list`, or `config:<config_name>` — lowercase, `_` and `:` separators. Defined under `cache.tags` in render arrays.
- **Cache contexts**: dot-separated paths (`user.permissions`, `url.query_args:foo`, `route.name`). Match Drupal core's set; don't invent new contexts in custom code without a core review.

## 20. JavaScript symbols

- **Variables**: `lowerCamelCase` (matches the JS-side standard, not the PHP one).
- **`Drupal.behaviors.<name>`**: the `<name>` is `lowerCamelCase`, matching the module's procedural prefix (e.g. `acmeBlocksCrossReference`). The internal pattern is `Drupal.behaviors.<lowerCamelCaseModuleName><BehaviorName>`.
- **`drupalSettings.<module>.<key>`**: module key is the **module machine name** (with `_`); the inner keys are `lowerCamelCase`.

```js
// ✅ Correct.
Drupal.behaviors.acmeBlocksCrossReference = {
  attach(context, settings) {
    const config = drupalSettings.acme_blocks.crossReference;
    once('acme-blocks-cross-reference', '.cross-reference', context)
      .forEach((el) => {
        // ...
      });
  },
};
```

## 21. CSS class names

- **BEM-ish**: `.<block>__<element>--<modifier>` where each segment is `lowercase` with `-` (not `_`) between words.
- **State classes**: `.is-<state>` (`.is-active`, `.is-loading`). Applied/removed by JS.
- **JS hooks**: `.js-<hook>`. **Never style `.js-*` classes** — they're behavior anchors only.

| ✅ | ❌ |
|---|---|
| `.cross-reference__row--highlighted` | `.crossReference-row.highlighted` |
| `.is-loading` | `.loading` |
| `.js-toggle-button` | `.toggle-button-js` |

## 22. Twig template files

- **File name**: `lowercase-with-dashes.html.twig` — match the suggestion (e.g. `node--article.html.twig`, `block--system-branding-block.html.twig`).
- The kebab-case form maps to Drupal's "theme hook suggestion" → file name resolution.

| Suggestion | File name |
|---|---|
| `node__article` | `node--article.html.twig` |
| `block__system_branding_block` | `block--system-branding-block.html.twig` |
| `field__node__field_part_number` | `field--node--field-part-number.html.twig` |

## 23. Test classes

- Class name: `<NounUnderTest>Test` — `UpperCamelCase`, ending with `Test`.
- Location depends on test type:
  - Unit: `tests/src/Unit/<Name>Test.php` → `Drupal\Tests\<module>\Unit\<Name>Test`.
  - Kernel: `tests/src/Kernel/<Name>Test.php` → `Drupal\Tests\<module>\Kernel\<Name>Test`.
  - Functional: `tests/src/Functional/<Name>Test.php` → `Drupal\Tests\<module>\Functional\<Name>Test`.
  - Functional JavaScript: `tests/src/FunctionalJavascript/<Name>Test.php`.
- Tag with `@group <module_name>` so Drupal's test runner picks it up.

---

## Quick lookup index

| You're naming … | Section | Form |
|---|---|---|
| A custom module | § 1 | `<org>_<purpose>` (e.g. `acme_blocks`) |
| A procedural function | § 2 | `<module>_<noun>` |
| A class | § 3 | `UpperCamelCase`, no `Drupal` prefix |
| A method | § 4 | `lowerCamelCase` |
| A hook implementation | § 5 | `<module>_<hook>` |
| A service ID | § 6 | `<module>.<role>` |
| A request attribute | § 7 | `_<key>` (unless from path) |
| A plugin ID | § 9 | `<module>_<role>` |
| A route name | § 10 | `<module>.<verb>_<noun>` |
| A content type / paragraph / media type / view mode | § 11 | `lowercase_with_underscores` |
| A field | § 12 | `field_<noun>` |
| A config object | § 13 | `<module>.<thing>` |
| A permission | § 14 | `verb noun` (space-separated permission key) |
| A library | § 15 | `<module>/<bundle>` |
| A DB table / column | § 17 | `lowercase_with_underscores` |
| A migration | § 18 | `<source>_to_<destination>` |
| A cache tag / context | § 19 | `<entity_type>:<id>` / `user.permissions` |
| A JS behavior | § 20 | `Drupal.behaviors.<lowerCamelCase>` |
| A CSS class | § 21 | `.block__element--modifier` |
| A Twig template | § 22 | `<suggestion>.html.twig` (kebab-case) |
| A test class | § 23 | `<Name>Test` |
