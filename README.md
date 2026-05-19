# drupal-coding-standards

A [Claude Code](https://docs.claude.com/en/docs/claude-code) skill that reviews, generates, and fixes Drupal code against the **official [Drupal Coding Standards](https://project.pages.drupalcode.org/coding_standards/)**.

Targets **Drupal 11.x on PHP 8.3**, but the standards themselves are version-independent and apply to any Drupal 8+ codebase.

## What it does

When the skill is active, Claude:

1. **Identifies** which code types are present (PHP, Twig, YAML, JS, CSS, SQL, Composer, markup).
2. **Loads only the relevant sub-skill(s)** from `references/` — it does not rely on memory for the hundreds of specific rules in the official docs.
3. **Reviews** every file against the matching reference, severity-tiered:
   - 🔴 **Blocker** — runtime / security / WCAG AA failure.
   - 🟡 **Standards violation** — `phpcs --standard=Drupal,DrupalPractice`, `eslint`, or `cspell` would flag it.
   - 🟢 **Suggestion** — readability / supplementary best practice.
4. **Offers** to generate a fixed file, a `.patch`, or the equivalent `phpcbf` / `rector` commands.

## Installation

This skill follows the standard Claude Code skill layout. Drop it into one of:

| Scope | Path |
|---|---|
| Global (all projects) | `~/.claude/skills/drupal-coding-standards/` |
| Project-only | `<repo>/.claude/skills/drupal-coding-standards/` |

### Clone

```bash
# Global install (default)
git clone git@github.com:jalvarez-tech/drupal-coding-standards.git \
  ~/.claude/skills/drupal-coding-standards

# Project install — overrides the global copy when both exist
git clone git@github.com:jalvarez-tech/drupal-coding-standards.git \
  ./.claude/skills/drupal-coding-standards
```

Restart Claude Code (or run `/init`) so the new skill is indexed.

### Verify it's loaded

The skill registers under the slug `drupal-coding-standards`. It appears in the available-skills list once the directory exists with a valid `SKILL.md`.

## Activation

The skill auto-triggers on edits to any of these files (full canonical list in [`triggers.md`](triggers.md)):

- PHP family: `*.php`, `*.module`, `*.theme`, `*.install`, `*.inc`, `*.engine`, `*.profile`, `*.test`, `*.api.php`
- Twig: `*.html.twig`, `*.twig`
- Drupal YAML: `*.info.yml`, `*.routing.yml`, `*.services.yml`, `*.libraries.yml`, `*.permissions.yml`, `*.links.*.yml`, `*.schema.yml`, `*.breakpoints.yml`, plus any YAML under `config/{default,install,schema}/`, `modules/custom/`, `themes/custom/`, `docroot/modules/custom/`, `docroot/themes/custom/`, `web/modules/custom/`, `web/themes/custom/`
- CSS: `*.css`, `*.pcss.css`, `*.scss`
- JavaScript: `*.js`
- Composer: `composer.json`

It also activates on keywords like `phpcs`, `drupal/coder`, `phpstan drupal`, `ESLint Drupal`, `hook_`, `render array`, `preprocess function`, `Drupal.behaviors`, `Drupal.t()`, `Schema API`, `Drupal coding standards`, `Drupal naming`.

## What's covered

Modular sub-skills under [`references/`](references/):

| Sub-skill | File | Scope |
|---|---|---|
| Naming (cross-cutting) | [`NAMING.md`](references/NAMING.md) | Modules, themes, functions, classes, services, plugins, routes, fields, content types, libraries, permissions, JS behaviors, CSS classes, DB tables, Twig files, tests |
| PHP (orchestrator) | [`PHP.md`](references/PHP.md) | Loads the relevant sub-files below |
| PHP coding | [`php/PHP-CODING.md`](references/php/PHP-CODING.md) | Formatting, whitespace, control structures, arrays, strings, placeholders, E_ALL |
| PHP docs | [`php/PHP-DOCS.md`](references/php/PHP-DOCS.md) | Docblocks, `@param` / `@return` / `@throws`, hook docs, inline comments |
| PHP OO / PSR-4 | [`php/PHP-OO-PSR4.md`](references/php/PHP-OO-PSR4.md) | Classes, interfaces, traits, enums, namespaces, type hints, services |
| PHP exceptions | [`php/PHP-EXCEPTIONS.md`](references/php/PHP-EXCEPTIONS.md) | Exception naming, messages, subclass strategy, try/catch |
| PHP legacy | [`php/PHP-LEGACY.md`](references/php/PHP-LEGACY.md) | D6/D7 patterns to recognize but never introduce |
| Accessibility | [`ACCESSIBILITY.md`](references/ACCESSIBILITY.md) | WCAG 2.2 AA on rendered output, forms, modals, tables |
| CSS | [`CSS.md`](references/CSS.md) | Formatting, architecture, file organization, CSScomb, review |
| JavaScript | [`JAVASCRIPT.md`](references/JAVASCRIPT.md) | Coding style, ES6+, `Drupal.behaviors`, ESLint, docs |
| SQL | [`SQL.md`](references/SQL.md) | Conventions, keywords, `SELECT *` avoidance |
| Twig | [`TWIG.md`](references/TWIG.md) | Template formatting and Drupal-specific patterns |
| Markup | [`MARKUP.md`](references/MARKUP.md) | Structural HTML and semantics |
| Spelling | [`SPELLING.md`](references/SPELLING.md) | US-English rule, cspell integration |
| YAML | [`YAML.md`](references/YAML.md) | Drupal config and `*.yml` conventions |
| Composer | [`COMPOSER.md`](references/COMPOSER.md) | `composer.json` rules, patches, installer paths |

The full router and review workflow live in [`SKILL.md`](SKILL.md).

## Tooling it expects to find

The skill recommends commands appropriate to your codebase. Defaults:

```bash
# PHPCS + DrupalPractice
composer require --dev drupal/coder
vendor/bin/phpcs  --standard=Drupal,DrupalPractice <path>
vendor/bin/phpcbf --standard=Drupal,DrupalPractice <path>

# PHPStan
composer require --dev phpstan/phpstan mglaman/phpstan-drupal
vendor/bin/phpstan analyse <path>

# Modernizations (D9→D10→D11)
composer require --dev palantirnet/drupal-rector
vendor/bin/rector process modules/custom --dry-run

# ESLint
npm install --save-dev eslint eslint-config-drupal
npx eslint modules/custom/

# CSpell
npx cspell "modules/custom/**/*.{php,module,inc,install,js,css,twig,yml}"
```

If the project ships a DDEV / Lando / Docksal / Makefile wrapper, the skill recommends the wrapped form so versions match CI. See [`SKILL.md` § Complementary validation tools](SKILL.md#complementary-validation-tools) for the full guidance, including two common defects in `git status`-driven wrappers.

## Freshness

The skill carries `last_verified: 2026-05-18` in the [`SKILL.md`](SKILL.md) frontmatter. The official docs occasionally reshuffle URLs or revise rules — if a citation looks stale, re-verify against [project.pages.drupalcode.org/coding_standards](https://project.pages.drupalcode.org/coding_standards/) and bump the date.

## Contributing

The standard contribution loop is:

1. Edit a reference file under `references/`.
2. If you changed activation surface (extensions or keywords), update [`triggers.md`](triggers.md) — `SKILL.md`'s frontmatter `description` paraphrases it for the runtime indexer.
3. Bump `last_verified` in `SKILL.md` if you re-verified against the official source.
4. Open a PR.

## License

Unspecified — the content paraphrases / cites the official [Drupal Coding Standards](https://project.pages.drupalcode.org/coding_standards/), which are part of the Drupal community documentation under [CC BY-SA 2.0](https://creativecommons.org/licenses/by-sa/2.0/). Treat this skill's prose as compatible with that license unless this section is replaced.
