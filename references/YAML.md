# YAML — Drupal Coding Standards Reference

Based on:
- [YAML Configuration files (official)](https://project.pages.drupalcode.org/coding_standards/yaml/configuration-files/)
- [YAML 1.2 Specification](https://yaml.org/spec/)
- [Drupal Configuration management documentation](https://www.drupal.org/docs/drupal-apis/configuration-api)

---

## Table of contents

1. [Format: YAML syntax](#1-format-yaml-syntax)
2. [Filename: convention and character limits](#2-filename-convention-and-character-limits)
3. [Simple configuration](#3-simple-configuration)
4. [Configuration entities](#4-configuration-entities)
5. [Bundles and nested configuration](#5-bundles-and-nested-configuration)
6. [Comments](#6-comments)
7. [Whitespace and indentation](#7-whitespace-and-indentation)
8. [Drupal YAML file types](#8-drupal-yaml-file-types)
9. [Common errors](#9-common-errors)
10. [YAML review checklist](#yaml-review-checklist)

---

## 1. Format: YAML syntax

Drupal uses **YAML 1.2** ([yaml.org/spec/](https://yaml.org/spec/)). YAML is a whitespace-sensitive language where indentation has semantic meaning (represents nested structures).

### Basic rules

- **Indent with 2 spaces**. **Never tabs**. *(Drupal coding standard.)*
- Keys and values separated by `:` + space.
- Lists with `- ` (dash + space).
- Strings without quotes in general; use quotes only when necessary (special characters, values that look like booleans/numbers, etc.). *(YAML 1.2 spec hygiene — the official Drupal YAML page does not codify quoting rules; the guidance below in §9 is YAML-spec-derived to prevent silent type-coercion bugs.)*

### Basic example

```yaml
name: 'My Module'
type: module
description: 'Module description in US English.'
core_version_requirement: ^10.3 || ^11
package: Custom
dependencies:
  - drupal:node
  - drupal:user
configure: my_module.settings
```

---

## 2. Filename: convention and character limits

### Master rule

> The configuration filename is **equal to the unique configuration name** with extension **`.yml`**.

### Global limit

> **The unique configuration name CANNOT exceed 250 characters.**

This is due to limits in databases where Drupal stores configuration.

### Summary of limits

| Element | Maximum |
|---|---|
| Unique configuration name (total) | **250 characters** |
| Extension name (machine name of module/theme/profile) | **50 characters** |
| Config entity `config_prefix` annotation | **32 characters** |
| Suffix (rest of the name) | **150 characters** |
| Individual entity/bundle IDs and config prefixes | **32 characters** |

---

## 3. Simple configuration

For **simple configuration** (flat configuration objects, not entities), the name **MUST start with the extension name** (machine name of the module, theme, or install profile that owns the configuration).

### Convention

```
<extension>.<descriptive_suffix>
```

### Examples

For a module with machine name `my_module`:

- ✅ `my_module.settings` (typical for settings)
- ✅ `my_module.features`
- ✅ `my_module.maintenance_mode`
- ❌ `settings.my_module` (wrong order)
- ❌ `my_settings` (no module prefix)

> Using the name `settings` is **common practice** but **not mandatory**. A module may have multiple configuration files if separating them makes logical sense (e.g. `my_module.settings`, `my_module.features`, `my_module.notifications`).

### File

```yaml
# my_module.settings.yml
default_theme: bartik
admin_theme: claro
items_per_page: 25
maintenance_mode: false
welcome_message: 'Welcome to our site!'
```

---

## 4. Configuration entities

For **configuration entities** (objects configurable with CRUD via UI, e.g. content types, view modes, image styles), the convention is more structured:

### Convention

```
<extension>.<config_prefix>.<machine_name>
```

Where:
- **`<extension>`**: machine name of the module that defines the config entity, or `"core"` for core entities.
- **`<config_prefix>`**: defined in the entity annotation (`config_prefix` in `@ConfigEntityType`). If not defined, default = entity machine ID.
- **`<machine_name>`**: machine name of the specific item.

### Examples

- **Image style** named `large`: `image.style.large`
- **View** named `frontpage`: `views.view.frontpage`
- **Date format** named `medium`: `core.date_format.medium`
- **Filter format** named `basic_html`: `filter.format.basic_html`

### Specific limits

- Individual machine name (the part after the prefix): **≤ 150 characters**.
- Config prefix defined in the annotation: **≤ 32 characters**.
- Extension name: **≤ 50 characters**.

### Defining config_prefix in the entity

```php
/**
 * @ConfigEntityType(
 *   id = "image_style",
 *   label = @Translation("Image style"),
 *   config_prefix = "style",
 *   ...
 * )
 */
class ImageStyle extends ConfigEntityBase {
  // ...
}
```

With that, an image style named `thumbnail` is saved as `image.style.thumbnail.yml`.

---

## 5. Bundles and nested configuration

### Entity bundles

Entity bundles have an even more nested structure:

```
<extension>.<entity_id>.<bundle_config_prefix>.<bundle_machine_name>
```

Where:
- **`<extension>`**: module that defines the entity (or `"core"`).
- **`<entity_id>`**: machine name of the entity of which this is a bundle.
- **`<bundle_config_prefix>`**: bundle's config prefix (default = config entity ID if not defined).
- **`<bundle_machine_name>`**: machine name of the bundle.

### Example: Content types

NodeType is a bundle of Node. NodeType's config prefix is `type`. For a content type named `article`:

```
node.type.article
```

File: `node.type.article.yml`.

### Example: View modes

```
core.entity_view_mode.<target_entity_type>.<view_mode_machine_name>
```

- `core.entity_view_mode.node.teaser`
- `core.entity_view_mode.user.compact`
- `core.entity_view_mode.taxonomy_term.full`

### Example: Field instances

```
field.field.<target_entity_type>.<target_bundle>.<field_machine_name>
```

- `field.field.node.article.body`
- `field.field.user.user.field_profile_picture`

### Field storage configs

```
field.storage.<target_entity_type>.<field_machine_name>
```

- `field.storage.node.body`
- `field.storage.user.field_profile_picture`

### Important: respect the 250 char limit

For these cases with many components, **each component must have a reasonable maximum** so the total doesn't exceed 250 chars. In practice, this means:

- `entity_id` and `bundle_id`: short (≤ 32 chars each).
- `field_machine_name`: descriptive but concise (≤ 32 chars ideally).

---

## 6. Comments

YAML supports comments with `#`.

### Drupal convention

- **Comments are rare in Drupal YAML files**. Exported config formats don't have them.
- If you use them (in `.libraries.yml`, `.info.yml`, doc files), they go in **US English** and follow general comment conventions (proper sentences).

### Examples

```yaml
# This file defines the libraries provided by the My Module module.

# Main library, loaded on every page where the block is rendered.
main:
  version: 1.x
  css:
    component:
      css/my-module.css: {}
  js:
    js/my-module.js: {}
  dependencies:
    - core/drupal

# Library only loaded in admin pages.
admin:
  version: 1.x
  css:
    theme:
      css/my-module-admin.css: {}
```

### When NOT to use comments

- In exported config files (`config/install/*.yml`, `config/optional/*.yml`, `config/schema/*.yml`): Drupal regenerates these files on config export, removing comments. Any comment is lost.

---

## 7. Whitespace and indentation

### Mandatory indent

> **Use two spaces to indent in config files.** In YAML, whitespace has semantic meaning to represent nested structures.

### Rules

- **2 spaces per level**. **Never tabs** (breaks YAML parsing).
- No trailing whitespace at end of lines.
- Final newline in the file.
- Line endings `\n` (Unix).
- UTF-8 without BOM.

### Example with correct indentation

```yaml
name: 'My Module'
type: module
description: 'Description.'
core_version_requirement: ^10.3 || ^11
dependencies:
  - drupal:node
  - drupal:user
configure: my_module.settings
package: Custom

# Permissions example (in my_module.permissions.yml):
administer my module:
  title: 'Administer My Module'
  description: 'Allows users to configure My Module settings.'
  restrict access: true
```

### Anti-patterns

❌ Tab indentation (shown with `<TAB>` placeholder so this file itself stays tab-free):

```yaml
name: 'My Module'
dependencies:
<TAB>- drupal:node    # ← literal tab before the dash, breaks YAML
```

❌ Inconsistent indentation:
```yaml
dependencies:
  - drupal:node
   - drupal:user   # ← 3 spaces instead of 2, breaks parsing
```

❌ Trailing whitespace:
```yaml
name: 'My Module'   ← spaces at the end
```

---

## 8. Drupal YAML file types

Drupal uses YAML for many file types. Each has specific conventions:

### `.info.yml`

Module/theme/profile metadata.

**Required keys** (per [Let Drupal know about your module with an .info.yml file](https://www.drupal.org/docs/develop/creating-modules/let-drupal-know-about-your-module-with-an-infoyml-file)):

- `name` — human-readable name.
- `type` — `module`, `theme`, or `profile`.
- `core_version_requirement` — Composer-style constraint (use `^10.3 || ^11` for cross-version compatibility, `^11` for D11-only).

**Optional keys**: `description`, `package`, `configure`, `dependencies`, `test_dependencies`, `lifecycle`, `hidden`, `required`, `php`.

```yaml
# Required:
name: 'My Module'
type: module
core_version_requirement: ^10.3 || ^11

# Optional (but conventional):
description: 'Provides custom functionality for My Site.'
package: Custom
configure: my_module.settings
dependencies:
  - drupal:node
  - drupal:user
```

For a D11-only module, use `core_version_requirement: ^11` (and bump your composer.json PHP floor to `>=8.3`).

**Key ordering** (convention, not codified): the order used in Drupal core and recommended for new modules is `name → type → description → package → core_version_requirement → dependencies → configure`. Following this order makes `.info.yml` files grep-friendly across the codebase — but no tool enforces it.

### `.routing.yml`

Module routes.

```yaml
my_module.settings:
  path: '/admin/config/system/my-module'
  defaults:
    _form: '\Drupal\my_module\Form\SettingsForm'
    _title: 'My Module Settings'
  requirements:
    _permission: 'administer my module'

my_module.view:
  path: '/my-module/view/{node}'
  defaults:
    _controller: '\Drupal\my_module\Controller\ViewController::display'
  requirements:
    _entity_access: 'node.view'
    node: \d+
```

### `.services.yml`

Service definitions.

```yaml
services:
  my_module.helper:
    class: Drupal\my_module\Service\Helper
    arguments: ['@entity_type.manager', '@config.factory']

  my_module.event_subscriber:
    class: Drupal\my_module\EventSubscriber\MySubscriber
    arguments: ['@my_module.helper']
    tags:
      - { name: event_subscriber }
```

### `.libraries.yml`

CSS/JS library definitions.

```yaml
main:
  version: 1.x
  css:
    component:
      css/my-module.css: {}
  js:
    js/my-module.js: {}
  dependencies:
    - core/drupal
    - core/once

admin:
  version: 1.x
  css:
    theme:
      css/my-module-admin.css: {}

# Use header: true ONLY when the script must load in <head> (rare —
# typically only for things that must run before <body> paints, such as
# CSP/feature-detection bootstrap). Default is footer loading.
critical_bootstrap:
  version: 1.x
  header: true
  js:
    js/bootstrap.js: { minified: true }
  dependencies:
    - core/drupalSettings
```

### `.permissions.yml`

Module permissions.

```yaml
administer my module:
  title: 'Administer My Module'
  description: 'Allows users to configure My Module settings.'
  restrict access: true

view my module content:
  title: 'View My Module content'
```

### `.links.menu.yml`, `.links.task.yml`, `.links.action.yml`, `.links.contextual.yml`

Menu links, local tasks, actions, contextual links.

```yaml
# my_module.links.menu.yml
my_module.settings:
  title: 'My Module'
  description: 'Configure My Module settings.'
  route_name: my_module.settings
  parent: system.admin_config_system
```

### `.schema.yml` (in `config/schema/`)

Schema definitions for configuration entities.

```yaml
my_module.settings:
  type: config_object
  label: 'My Module settings'
  mapping:
    enabled:
      type: boolean
      label: 'Enabled'
    api_key:
      type: string
      label: 'API key'
    items_per_page:
      type: integer
      label: 'Items per page'
```

### Config exports (`config/install/*.yml`, `config/optional/*.yml`)

Default module configuration. Drupal regenerates them on config export; don't add comments here (they will be lost).

---

## 9. Common errors

### Mixing tabs and spaces

```yaml
# ❌ Breaks the parser (the second list item is preceded by a literal tab,
# shown here as <TAB> so this reference file itself stays tab-free).
dependencies:
  - drupal:node
<TAB>- drupal:user
```

### Forgetting the space after `:`

```yaml
# ❌
name:'My Module'

# ✅
name: 'My Module'
```

### Forgetting the space after `-`

```yaml
# ❌
dependencies:
  -drupal:node

# ✅
dependencies:
  - drupal:node
```

### Strings that look like booleans/numbers without quotes

YAML interprets `yes`, `no`, `on`, `off`, `true`, `false` as booleans, and numbers as integers/floats. If you want them to be strings, **use quotes**:

```yaml
# ❌ This parses as boolean `false`, not string
flag: no

# ✅
flag: 'no'

# ❌ This parses as integer 1.20
version: 1.20

# ✅
version: '1.20'
```

### Special characters in strings without quotes

Strings with `:`, `#`, `&`, `*`, `!`, `|`, `>`, `'`, `"`, `%`, `@`, `` ` ``: use quotes.

```yaml
# ❌
description: A description with : a colon

# ✅
description: 'A description with : a colon'
```

### UK English in YAML

```yaml
# ❌
description: 'Customise the colour palette.'

# ✅
description: 'Customize the color palette.'
```

### Machine names with invalid characters

Machine names must be **lowercase, snake_case, only letters/digits/underscores**. No hyphens, no uppercase, no spaces.

```yaml
# ❌
my-module.settings    # hyphen instead of underscore
MyModule.settings     # CamelCase

# ✅
my_module.settings
```

---

## YAML review checklist

### Syntax
- [ ] 2-space indent, no tabs
- [ ] No trailing whitespace
- [ ] Final newline
- [ ] Line endings `\n`
- [ ] UTF-8 without BOM
- [ ] Space after `:` in key-values
- [ ] Space after `-` in lists
- [ ] Ambiguous strings in quotes (numbers with leading zero, "no"/"yes", etc.)
- [ ] Special chars in strings escaped with quotes

### Naming and filename
- [ ] Unique configuration name ≤ 250 chars
- [ ] Extension name (module machine name) ≤ 50 chars
- [ ] Config prefix ≤ 32 chars
- [ ] Suffix ≤ 150 chars
- [ ] Filename = unique config name + `.yml`
- [ ] Simple config starts with extension name
- [ ] Config entities with structure `<extension>.<config_prefix>.<machine_name>`
- [ ] Machine names in lowercase snake_case (no hyphens, no CamelCase)
- [ ] Machine names in US English

### Comments
- [ ] Comments only where they add value (not in config exports)
- [ ] Comments in US English
- [ ] Proper sentences (initial capital, final period)

### Per file type

#### `.info.yml`
- [ ] `name` present and in quotes if it has spaces
- [ ] `type` correct (module/theme/profile)
- [ ] `description` in US English
- [ ] `core_version_requirement` matches target core (`^11` for Drupal 11-only projects)
- [ ] `package` defined (ideally `Custom` for custom modules)
- [ ] `dependencies` listed with format `extension:module`
- [ ] `core_version_requirement` set correctly (`^10.3 || ^11` or `^11` only)

#### `.routing.yml`
- [ ] Route names follow `<module>.<route>` convention
- [ ] `path` starts with `/`
- [ ] `defaults` with appropriate `_form`/`_controller`/`_entity_view`
- [ ] `requirements` with appropriate `_permission`/`_role`/`_entity_access`
- [ ] Title present (when applicable)

##### Routing for REST endpoints and state-changing routes (extra checklist)

> **Source / scope**: this sub-checklist comes from the [Drupal Routing API](https://www.drupal.org/docs/drupal-apis/routing-system) and security advisories, **not** from `coding_standards/yaml/configuration-files/` (which only documents filename + character limits + indentation). Severity follows the rules in `SKILL.md` — a real runtime/security gap is 🔴 (missing CSRF on a state-changing route, `_access: 'TRUE'` on a non-public path), a documented-but-non-runtime gap is 🟡, a style preference is 🟢.

When the route is **not** rendered through `FormBuilder` (REST controllers, custom callbacks, AJAX endpoints, webhook receivers), CSRF is not automatic and additional keys matter:

- [ ] **State-changing methods** (`POST`, `PUT`, `PATCH`, `DELETE`) declare
  `methods: [POST, …]` at the route root, at the same level as `path`,
  `defaults`, and `requirements`.
- [ ] **CSRF for state-changing routes**: `requirements: { _csrf_token: 'TRUE' }` — required for any state change initiated from a Drupal-rendered page. Omitted **only** for endpoints that authenticate via a token / OAuth and explicitly opt out (then document why).
- [ ] **Access control**: `_permission`, `_role`, `_entity_access`, or `_custom_access` declared — `_access: 'TRUE'` is a 🔴 unless the path is genuinely public.
- [ ] **`_format`** declared for REST: `requirements: { _format: 'json' }` (or `hal_json`, `xml`, etc.) so Drupal negotiates content type.
- [ ] **Authentication providers** explicitly listed when not using session: `options: { _auth: [basic_auth, oauth2] }`.
- [ ] Controller returns `\Drupal\Core\Cache\CacheableJsonResponse` (with proper cache metadata) when the response can be cached, or `JsonResponse` only for truly per-request data.
- [ ] **Path parameters typed**: `requirements: { node: \d+ }` for numeric IDs to prevent route matching against unintended paths.
- [ ] **No reserved request-attribute names** in `defaults` (see `php/PHP-OO-PSR4.md` § 23 — `_route`, `_controller`, `_content` etc. are reserved by Symfony/Drupal).

Example REST endpoint with CSRF + permission + format:

```yaml
my_module.api_save:
  path: '/api/v1/my-module/save'
  defaults:
    _controller: '\Drupal\my_module\Controller\ApiController::save'
  methods: [POST]
  requirements:
    _permission: 'access my module api'
    _csrf_token: 'TRUE'
    _format: 'json'
  options:
    no_cache: 'TRUE'
```

#### `.services.yml`

> **Source / scope**: the only rule the official `coding_standards/php/naming-services/` page documents is the **Request-attribute** `_` prefix (see `php/PHP-OO-PSR4.md` § 23). The bullets below are **Drupal Service Container conventions** from the [Services and Dependency Injection docs](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection), not formal coding-standards rules. Severity by impact: a wrong `class` FQN is a 🔴 runtime break, a non-conventional ID is 🟢.

- [ ] Service IDs follow `<module>.<service_name>` convention (Drupal core convention, not codified at `coding_standards/`)
- [ ] `class` with full FQN
- [ ] `arguments` reference other services with `@`
- [ ] Correct tags for event subscribers, etc.

#### `.libraries.yml`
- [ ] Descriptive library names
- [ ] `version` defined
- [ ] CSS grouped by SMACSS layer (base/layout/component/state/theme)
- [ ] Dependencies declared

#### `.permissions.yml`
- [ ] Permission IDs in lowercase with spaces (`administer my module`)
- [ ] `title` in quotes if it has spaces, capitalized
- [ ] Clear `description`
- [ ] `restrict access: true` for any permission that grants trust to bypass typical access controls (admin-like permissions); Drupal renders a warning when this permission is granted

#### `.links.menu.yml`, `.links.task.yml`
- [ ] Link IDs follow `<module>.<link_name>` convention
- [ ] `title` and `description` in US English
- [ ] `parent` points to existing link/menu
- [ ] Valid `route_name`

#### Config schema (`.schema.yml`)
- [ ] Schema defined for each config object
- [ ] Correct `type` (config_object, mapping, sequence, string, integer, boolean, etc.)
- [ ] Labels in US English
- [ ] Validation constraints when applicable

---

## Official resources

- [YAML Configuration files (Drupal)](https://project.pages.drupalcode.org/coding_standards/yaml/configuration-files/)
- [YAML 1.2 Specification](https://yaml.org/spec/)
- [Drupal Configuration API](https://www.drupal.org/docs/drupal-apis/configuration-api)
- [Drupal Service Container documentation](https://www.drupal.org/docs/drupal-apis/services-and-dependency-injection)
- [Drupal Routing API](https://www.drupal.org/docs/drupal-apis/routing-system)
