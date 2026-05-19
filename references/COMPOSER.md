# Composer — Drupal Coding Standards Reference

Based on:
- [Composer package name convention (official)](https://project.pages.drupalcode.org/coding_standards/composer/package-name/)
- [Composer documentation: package names](https://getcomposer.org/doc/01-basic-usage.md#package-names)
- [Add a composer.json file to define your module as a PHP package (Drupal.org)](https://www.drupal.org/node/2514612)

---

## Table of contents

1. [General Composer rule](#1-general-composer-rule)
2. [Drupal Projects](#2-drupal-projects)
3. [Modules, Themes and Profiles](#3-modules-themes-and-profiles)
4. [Components (PHP libraries inside Drupal projects)](#4-components-php-libraries-inside-drupal-projects)
5. [Conflict resolution](#5-conflict-resolution)
6. [composer.json template for custom modules](#6-composerjson-template-for-custom-modules)
7. [Patching and installer paths](#7-patching-and-installer-paths)
8. [Composer review checklist](#composer-review-checklist)

---

## 1. General Composer rule

Composer **only allows** package names in the format `vendor/project`. **No more than two levels**.

```
<vendor>/<project>
```

Drupal adopted Composer as a dependency manager, which forced the community to establish a naming convention to avoid conflicts among the thousands of modules, themes, and components in the ecosystem.

---

## 2. Drupal Projects

> **Any project hosted on drupal.org is considered a Drupal Project.** This includes:
> - Drupal core
> - Contrib modules
> - Contrib themes
> - Install profiles
> - Distributions

Each project on drupal.org has a URL of the form:

```
https://www.drupal.org/project/PROJECT
```

### Convention

Projects **must use** the package name:

```
drupal/PROJECT
```

where `PROJECT` is the portion after `/project/` in the URL.

### Examples

| Project | URL | Package name |
|---|---|---|
| Drupal | `https://www.drupal.org/project/drupal` | `drupal/drupal` |
| ctools | `https://www.drupal.org/project/ctools` | `drupal/ctools` |
| Views (D7) | `https://www.drupal.org/project/views` | `drupal/views` |
| core (Drupal subtree) | `https://www.drupal.org/project/core` | `drupal/core` |
| datetime (Drupal Core module) | `https://www.drupal.org/project/datetime` | `drupal/datetime` |
| pathauto | `https://www.drupal.org/project/pathauto` | `drupal/pathauto` |
| token | `https://www.drupal.org/project/token` | `drupal/token` |
| paragraphs | `https://www.drupal.org/project/paragraphs` | `drupal/paragraphs` |
| webform | `https://www.drupal.org/project/webform` | `drupal/webform` |

### Special cases

Some project URLs are not accessible or point to other projects because they are **reserved names** (sub-modules or sub-themes of existing projects, like Drupal core).

---

## 3. Modules, Themes and Profiles

A single Drupal project may contain **multiple modules, themes, and profiles**. Examples:

- **Drupal core** contains: `system`, `node`, `user`, `standard`, etc.
- **ctools** contains: `page_manager`, `views_content`, etc.

Since Drupal **does not allow running two modules with the same machine name**, **modules, themes, profiles, and Drupal projects share the same Composer namespace**.

### Convention

Sub-modules, sub-themes, and profiles must use the package name:

```
drupal/SUBPROJECT
```

where `SUBPROJECT` is the **machine name** of the module, theme, or profile.

### Examples

- **Module `devel_generate`** of the Devel project: `drupal/devel_generate`
- **Module `views_content`** of the ctools project: `drupal/views_content`
- **Module `page_manager`** of the ctools project (later separated): `drupal/page_manager`
- **Module `views`** in Drupal 8+ core: `drupal/views` — this **does not conflict** with the old D7 contrib `views` module because different versions resolve to different packages.

### Why no conflict with `drupal/PROJECT`

Drupal project names **share namespace** with modules, themes, and profiles because Drupal itself doesn't allow a module with the same machine name as a different project. This means `drupal/views` can resolve to:

- Views as a module of Drupal 8+ core (which replaces contrib Views).
- The contrib Views project in Drupal 7.

Composer resolves this automatically via:
1. **`replace`** property in `composer.json` of projects that absorb others.
2. **Different Composer repositories** that resolve to different versions.

### Recommendation for projects declaring their own composer.json

> To avoid dependency issues, Drupal projects that declare their own `composer.json` **should also add their submodules and subthemes to the `replace` section**.

Example:

```json
{
  "name": "drupal/ctools",
  "type": "drupal-module",
  "replace": {
    "drupal/page_manager": "*",
    "drupal/views_content": "*"
  }
}
```

This ensures that if someone has `drupal/page_manager` in their `composer.json` and later installs `drupal/ctools`, `page_manager` is not installed separately.

---

## 4. Components (PHP libraries inside Drupal projects)

Drupal projects may contain **custom components**: reusable PHP libraries not bound to any specific Drupal namespace.

Since these components **could conflict** with a Drupal project, module, theme, etc., they require their own convention.

### Convention

Package names must have the prefix `<parent>-`:

```
drupal/PARENT-COMPONENT
```

where:
- **`PARENT`** = name of the parent package (the Drupal project containing the component).
- **`COMPONENT`** = name of the component.

### Why `-` (dash)?

Drupal **does not use dashes in machine names** (uses underscores). Therefore, prefixing with a dash guarantees **no conflict with any existing namespace** of projects, modules, themes, or profiles.

### Examples

- **Datetime component** of Drupal core (`core/lib/Drupal/Component/Datetime`): `drupal/core-datetime`
- **Diff component** of Drupal core (`core/lib/Drupal/Component/Diff`): `drupal/core-diff`
- **Render component** of Drupal core: `drupal/core-render`
- **Plugin component** of Drupal core: `drupal/core-plugin`

And allows contrib to expose components:
- **Panels renderer**: `drupal/panels-renderer`
- **Display Suite builder**: `drupal/ds-builder`

---

## 5. Conflict resolution

When there's a naming collision (e.g. a module was split from one project to another), the rule is:

> **Drupal Projects** take **precedence** over (sub-)modules, themes, and profiles.

For cases that can't be resolved by precedence, use:

### `replace` in composer.json

When a project absorbs another (e.g. Views absorbed into Drupal 8 core), the absorbing project's composer.json declares `replace`:

```json
{
  "name": "drupal/core",
  "replace": {
    "drupal/views": "self.version"
  }
}
```

This tells Composer: "if someone asks for `drupal/views`, consider that I provide it".

### Different Composer repositories

Drupal projects use the official repository:

```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.drupal.org/8"
    }
  ]
}
```

For Drupal 7:

```json
{
  "repositories": [
    {
      "type": "composer",
      "url": "https://packages.drupal.org/7"
    }
  ]
}
```

---

## 6. composer.json template for custom modules

For your custom module (not published on drupal.org), follow this template:

```json
{
    "name": "weknow/my_custom_module",
    "type": "drupal-module",
    "description": "My custom module for Weknow project X.",
    "homepage": "https://www.weknow.cr",
    "license": "GPL-2.0-or-later",
    "authors": [
        {
            "name": "John",
            "email": "john@weknow.cr",
            "role": "Maintainer"
        }
    ],
    "minimum-stability": "dev",
    "prefer-stable": true,
    "require": {
        "php": ">=8.3",
        "drupal/core": "^11"
    },
    "require-dev": {
        "drupal/coder": "^8.3 || ^9.0",
        "phpstan/phpstan": "^2.0",
        "mglaman/phpstan-drupal": "^2.0"
    },
    "autoload": {
        "psr-4": {
            "Drupal\\my_custom_module\\": "src/"
        }
    }
}
```

### Rules for custom modules

- **`name`**: if NOT published on drupal.org, **don't use `drupal/`** as vendor. Use your organization's namespace (e.g. `acme/...`, `mycompany/...`).
- **`type`**: `drupal-module`, `drupal-theme`, `drupal-profile`, or `drupal-drush` as appropriate.
- **`license`**: for Drupal modules, **`GPL-2.0-or-later`** (Drupal itself is GPL-2.0+, derivative work must be compatible).
- **`require`**: declare minimum PHP version and `drupal/core` with **internally consistent** versions. Drupal 11 requires PHP 8.3+ ([source](https://www.drupal.org/docs/getting-started/system-requirements/php-requirements)); Drupal 10 requires PHP 8.1+. Pick one of:
  - **D11-only (template above):** `"php": ">=8.3"` + `"drupal/core": "^11"`.
  - **D10-only:** `"php": ">=8.1"` + `"drupal/core": "^10"`.
  - **Both (rare for new modules):** `"php": ">=8.3"` + `"drupal/core": "^10.3 || ^11"` — the floor must match the **strictest** constraint your supported D11 range imposes, otherwise installs on PHP 8.1/8.2 with `^11` will fail.
- **`require-dev`**: include coding-standards tooling. As of 2026-05 the D11-current versions are `drupal/coder: ^8.3 || ^9.0`, `phpstan/phpstan: ^2.0`, `mglaman/phpstan-drupal: ^2.0`. Re-verify on each refresh — these tools have shipped breaking changes between majors.
- **`autoload.psr-4`**: namespace maps to the `src/` folder per [PSR-4](https://project.pages.drupalcode.org/coding_standards/php/psr4/).

### If publishing on drupal.org

Change the `name` to `drupal/<machine_name>` and use the project's URL:

```json
{
    "name": "drupal/my_public_module",
    "type": "drupal-module",
    "description": "My public module published on drupal.org.",
    "homepage": "https://www.drupal.org/project/my_public_module",
    "license": "GPL-2.0-or-later"
}
```

---

## 7. Patching and installer paths

Two composer features that nearly every Drupal site uses but the official coding-standards page does not document. Both belong in the **root** `composer.json` (the site's, not a per-module file).

### `cweagans/composer-patches` — patches for core and contrib

[`cweagans/composer-patches`](https://github.com/cweagans/composer-patches) is the de-facto patching plugin for Drupal. Declare patches under `extra.patches`, keyed by the package being patched. For patches against `drupal/core` you typically need `-p2` (the patch file paths start with `a/core/...` rather than `a/...`).

```json
{
    "require": {
        "cweagans/composer-patches": "^2.0",
        "drupal/core-recommended": "^11"
    },
    "config": {
        "allow-plugins": {
            "cweagans/composer-patches": true
        }
    },
    "extra": {
        "patches": {
            "drupal/core": {
                "ISSUE-1234: fix some core bug": "patches/contrib/issue-1234-core-bug.patch"
            },
            "drupal/views_bulk_operations": {
                "ISSUE-5678: respect entity access": "patches/contrib/issue-5678-vbo.patch"
            }
        },
        "patches-file": "patches.json",
        "enable-patching": true
    }
}
```

Key options:
- `extra.patches` — map of `{ "package/name": { "Description": "path/to.patch" } }`. Description shows in install output and is for humans.
- `extra.patches-file` — alternative location (`patches.json`) for the same data; useful when you want to keep `composer.json` short.
- `extra.enable-patching: true` — required if your patches target packages that are **dependencies of dependencies** rather than direct requires. Without this, `composer install` from a downstream consumer of your project will silently skip its patches.
- The `config.allow-plugins` block above is required by Composer 2.2+; without it the plugin runs in a sandboxed mode that won't apply patches.

### `composer/installers` — `extra.installer-paths`

[`composer/installers`](https://github.com/composer/installers) places Drupal modules/themes/profiles under the standard `web/modules/contrib/...` layout. The `drupal/recommended-project` scaffold sets this up; if you build a project from scratch, replicate it:

```json
{
    "require": {
        "composer/installers": "^2.0"
    },
    "extra": {
        "installer-paths": {
            "web/core": ["type:drupal-core"],
            "web/libraries/{$name}": ["type:drupal-library"],
            "web/modules/contrib/{$name}": ["type:drupal-module"],
            "web/profiles/contrib/{$name}": ["type:drupal-profile"],
            "web/themes/contrib/{$name}": ["type:drupal-theme"],
            "drush/Commands/contrib/{$name}": ["type:drupal-drush"],
            "web/modules/custom/{$name}": ["type:drupal-custom-module"],
            "web/profiles/custom/{$name}": ["type:drupal-custom-profile"],
            "web/themes/custom/{$name}": ["type:drupal-custom-theme"]
        }
    }
}
```

Notes:
- The `web/` prefix matches `drupal/recommended-project`; sites that put Drupal at the repo root (`docroot/` or just `./`) should swap that prefix consistently across all entries.
- `type:drupal-custom-module` etc. are recognized by `composer/installers` as long as the custom module's own `composer.json` declares the matching `type`.
- Don't mix `composer/installers` with `wikimedia/composer-merge-plugin` — modern Drupal scaffolds discourage the merge-plugin pattern in favor of declaring all dependencies directly in the root `composer.json`.

---

## Composer review checklist

### Package naming
- [ ] `drupal/` vendor only for projects published on drupal.org
- [ ] Organization vendor (`weknow/`, etc.) for unpublished custom modules
- [ ] Project name = portion after `/project/` in drupal.org URL (when applicable)
- [ ] Sub-modules/themes/profiles: `drupal/SUBPROJECT` where SUBPROJECT = machine name
- [ ] Components with parent prefix + `-`: `drupal/core-datetime`, `drupal/panels-renderer`
- [ ] Lowercase, no special characters
- [ ] Only two levels (`vendor/project`), never more

### composer.json structure
- [ ] `name` correct following convention
- [ ] `type` appropriate (`drupal-module`, `drupal-theme`, etc.)
- [ ] `description` in US English, clear and concise
- [ ] `license` = `GPL-2.0-or-later` (for Drupal modules)
- [ ] `homepage` points to project URL (drupal.org or equivalent)
- [ ] `authors` with name, email, role

### Dependencies
- [ ] `require.php` declares minimum version compatible with target Drupal core
- [ ] `require.drupal/core` with compatible range (e.g. `^10 || ^11`)
- [ ] Additional dependencies with `drupal/` prefix when they are Drupal projects
- [ ] Versions using semantic versioning (`^X.Y` or `~X.Y`)

### Dev dependencies
- [ ] `drupal/coder` for PHPCS with Drupal/DrupalPractice standards
- [ ] `phpstan/phpstan` + `mglaman/phpstan-drupal` for static analysis
- [ ] Other relevant tools (drush, phpunit, etc.)

### Autoload
- [ ] Correct PSR-4 autoload: `"Drupal\\<module>\\": "src/"`
- [ ] PHP namespace matches `name` and filesystem layout
- [ ] Tests under `tests/src/` — Drupal core's test runner auto-discovers them via the `Drupal\Tests\<module>\` namespace; an explicit `autoload-dev` block is optional, not the convention

### `replace` (when applicable)
- [ ] Drupal projects with submodules: `replace` declared
- [ ] Absorbing project: declares replace of the absorbed
- [ ] `self.version` or `*` appropriate per case

### Repositories
- [ ] Drupal official repository included in projects consuming contrib
- [ ] Correct URL per Drupal version (`/8` for D8/D9/D10/D11)

---

## Official resources

- [Composer package name convention (Drupal)](https://project.pages.drupalcode.org/coding_standards/composer/package-name/)
- [Composer Basic usage: Package names](https://getcomposer.org/doc/01-basic-usage.md#package-names)
- [Composer schema reference](https://getcomposer.org/doc/04-schema.md)
- [Replace property in Composer](https://getcomposer.org/doc/04-schema.md#replace)
- [Add a composer.json file to define your module as a PHP package](https://www.drupal.org/node/2514612)
- [Using Composer with Drupal](https://www.drupal.org/docs/develop/using-composer)
