# Spelling — Drupal Coding Standards Reference

Based on:
- [Drupal Spelling coding standards (official)](https://project.pages.drupalcode.org/coding_standards/spelling/spelling/)
- [Drupal cspell development tool](https://www.drupal.org/node/3352552)

---

## Table of contents

1. [Master rule: US English](#1-master-rule-us-english)
2. [Scope: where it applies](#2-scope-where-it-applies)
3. [Common UK vs US differences](#3-common-uk-vs-us-differences)
4. [CSpell: official tool](#4-cspell-official-tool)
5. [CSpell inline directives](#5-cspell-inline-directives)
6. [Types of errors to look for](#6-types-of-errors-to-look-for)
7. [Spelling review checklist](#spelling-review-checklist)

---

## 1. Master rule: US English

> **Drupal Core uses US English spelling for all source code, including comments and names.**

This rule is **absolute** and applies to source code. The user interface can later be translated via the localization system (`.po` files), but **code is always in US English**.

### Why

- **Global consistency**: Drupal is an international project with thousands of contributors.
- **Predictability**: when you search for a function, identifier, or word in code, there's ONE correct form.
- **Tooling**: tools like CSpell, IDE dictionaries, and autocompleters assume US English.

---

## 2. Scope: where it applies

US English spelling applies to **ALL** Drupal source code, including:

- **Function names**: `my_module_color_helper()` (not `colour`).
- **Variable names**: `$default_behavior` (not `behaviour`).
- **Class, method, property names**: `ColorPicker`, `getCustomBehavior()`.
- **Comments**: all inline comments and docblocks.
- **String literals in code**: hardcoded strings in arrays, configs, exception messages.
- **Translatable strings** (`t()` / `$this->t()`): even if they get translated later to Spanish/French/etc., the **source string** must be in US English.
- **Markup files**: HTML/Twig templates.
- **Configuration files**: YAML files, JSON files.
- **Documentation**: README, CHANGELOG, INSTALL, `.md`/`.txt` files.
- **File names**: `color-picker.css`, not `colour-picker.css`.
- **Machine names**: machine names in `.info.yml`, route names, service IDs.

### Legitimate exceptions

- **Strings that are proper names** or upstream API/library names that ship a British spelling (e.g. the package name `colour-picker.js` if that's what the upstream releases — match the source, don't "fix" it).
- **Test strings** that verify behavior with other languages. Can be marked with `// cspell:disable-next-line`.

---

## 3. Common UK vs US differences (editorial reference)

> **Source authority note.** The [official Drupal spelling page](https://project.pages.drupalcode.org/coding_standards/spelling/spelling/) only states the master rule ("Drupal Core uses US English spelling for all source code, including comments and names") plus CSpell mechanics. It does **not** publish a UK→US lookup table. The table below is an **editorial reference** — useful for review, but a finding against it is only as strong as the dictionary it cites (CSpell `en-US` plus standard US English usage). Don't flag entries where US English itself accepts both spellings.

Words where standard British English differs from US English. **Drupal source code uses the US column.**

| ❌ UK English | ✅ US English |
|---|---|
| colour | color |
| behaviour | behavior |
| favourite | favorite |
| organisation | organization |
| organise | organize |
| customise | customize |
| customisation | customization |
| analyse | analyze |
| categorise | categorize |
| catalogue | catalog |
| centre | center |
| theatre | theater |
| metre | meter |
| litre | liter |
| programme | program |
| grey | gray |
| labelled | labeled |
| modelled | modeled |
| travelling | traveling |
| defence | defense |
| licence (noun) | license |
| practise (verb) | practice |
| enrol | enroll |
| fulfil | fulfill |
| skilful | skillful |
| jewellery | jewelry |
| ageing | aging |
| storey (floor) | story |
| tyre | tire |
| aluminium | aluminum |
| sulphur | sulfur |
| moustache | mustache |

**Dual-acceptable in US English** (don't flag as a violation unless your project's `.cspell.json` picks a side): `cancelled`/`canceled`, `acknowledgement`/`acknowledgment`, `judgement`/`judgment`, `dialogue`/`dialog`. Drupal core code tends to use the shorter form (`canceled`, `acknowledgment`, `judgment`, `dialog` — the last especially as the UI noun for a modal), but neither side has been formalized.

### Common misspellings (not UK/US, just typos)

Beyond UK/US, watch out for these:

| ❌ Incorrect | ✅ Correct |
|---|---|
| occured | occurred |
| recieve | receive |
| seperate | separate |
| definately | definitely |
| accomodate | accommodate |
| neccessary | necessary |
| occassion | occasion |
| begining | beginning |
| writting | writing |
| commited | committed |
| existance | existence |
| independant | independent |
| persistant | persistent |
| consistant | consistent |
| sucess | success |
| sucessful | successful |

---

## 3a. User-interface terminology (Drupal UI text standards)

> **Source authority note.** These conventions come from Drupal's user-interface text standards and consistent core practice — **not** from the official Spelling page (which is silent on terminology). Cite *"Drupal UI text standards / core practice"* when flagging.

Compound vs two-word forms (the most common review trip-ups):

| Use as verb (two words) | Use as noun/adjective (one word or hyphenated) |
|---|---|
| Please **log in** to continue. | Use your **login** to access the site. |
| **Set up** your account. | Complete the **setup**. |
| **Sign in** here. | The **sign-in** page. |
| **Log out**. | After **logout** completes… |
| **Back up** your data. | Restore from a **backup**. |
| **Check out** the cart. | The **checkout** page. |

Modern compound spellings (Drupal core uses the closed form):

| ❌ Avoid | ✅ Use |
|---|---|
| e-mail | email |
| web site | website |
| on-line | online |
| home page | homepage (one word for the page; "home page" two words is also acceptable) |
| user name | username |
| filename, file name | filename (one word in code; "file name" acceptable in UI prose) |
| pop-up (noun) | pop-up *or* popup — Drupal core uses **popup** in JS/CSS class names, hyphenated in user-facing prose |

Capitalization conventions:
- **"Drupal"** is always capitalized — never `drupal` in prose.
- **"WordPress", "GitHub", "JavaScript"** — preserve the source-of-truth capitalization.
- **Module / theme machine names** stay `snake_case` (e.g. `views_ui`, not `Views UI`) when referring to the machine name; use Title Case when referring to the human-readable label.
- **Sentence case** for UI strings (page titles, button labels, menu items): "Edit user" not "Edit User".

---

## 4. CSpell: official tool

Drupal core uses **[CSpell](https://cspell.org/)** (Code Spell Checker) to validate spelling automatically.

### Installation

```bash
# Via npm (Drupal core has it as a devDependency)
npm install -g cspell

# Or via npx (without installing globally)
npx cspell "**/*.{php,module,inc,install,js,css,twig,yml}"
```

### Typical usage

```bash
# Check a whole custom module
npx cspell "modules/custom/my_module/**/*"

# Check a single file
npx cspell src/Controller/MyController.php

# Check and generate report
npx cspell --no-progress "modules/custom/**/*.{php,module,inc,install,js,css,twig,yml}" > spell-report.txt
```

### Configuration

Drupal core ships with a `.cspell.json` that defines:

- Dictionaries used (`en-US`, technical terms, project-specific words).
- Words ignored (valid Drupal-specific words).
- Files included/excluded.

For your custom module you can create your own `.cspell.json`:

```json
{
  "version": "0.2",
  "language": "en-US",
  "dictionaries": ["en-US", "drupal"],
  "words": [
    "Acme",
    "drupalSettings",
    "tableselect"
  ],
  "ignorePaths": [
    "node_modules/",
    "vendor/",
    "*.min.js"
  ]
}
```

---

## 5. CSpell inline directives

CSpell can be silenced locally with inline directives. **Always lowercase `cspell`** (the official page emphasizes this).

### Ignore specific words in a file

`cspell:ignore` adds words to the local dictionary of the file. The words go **alphabetically, separated by a single space**, on multiple lines if they exceed the character limit:

```php
<?php

// cspell:ignore first-word second-word
```

```php
<?php

// cspell:ignore tabledrag tableselect tableselectaccessibility
```

### Disable for the NEXT line only

When a line contains a specific invalid word (e.g. random token, string in another language):

```php
<?php

// cspell:disable-next-line
$token = 'example_non_secret_token_for_docs_only';
use Drupal\foo\Bar;
```

### Disable for multiple lines

Use `cspell:disable` before the block and `cspell:enable` after:

```php
<?php

// cspell:disable
$lorem1 = 'Lorem ipsum dolor sit amet in libero.';
$lorem2 = 'Ut fermentum est vitae metus orci.';
// cspell:enable
```

### Rules for using directives

1. **Last resort**: first verify the word isn't a typo or a UK spelling.
2. **`cspell:ignore` for words that really are valid** and repeat in the file (e.g. names of third-party libraries, specific machine names).
3. **`cspell:disable-next-line` for tokens, hashes, randoms** that appear once.
4. **`cspell:disable` / `cspell:enable` only for contiguous blocks** of non-English text (e.g. tests validating i18n).
5. **Never `cspell:disable` a whole file** — defeats the tool's purpose.

---

## 6. Types of errors to look for

When reviewing code, validate:

### In code (PHP/JS/CSS)

- **Function, method, class names**: are they in US English?
- **Variable names**: do they use US spelling?
- **Constants**: same rule applies.
- **Comments**: `// Setup the colour palette` → `// Set up the color palette`.

### In docblocks

- **Summary and descriptions**: complete, US English.
- **Param and return descriptions**: same rule.

### In translatable strings

- **Source string** is in US English: `t('Save changes')`, not `t('Save changes (UK style)')`.
- The Spanish (or other) translation can use localisms, but the source is US English.

### In markup/Twig

- **Hardcoded strings**: ideally wrapped in `|t`, but if loose, US English.
- **Twig comments**: `{# Display the colour picker #}` → `{# Display the color picker #}`.

### In YAML / config

- **Labels and descriptions**: in US English.
- **Machine names**: snake_case in US English (`my_color_field`, not `my_colour_field`).

### In documentation

- **README, CHANGELOG, etc.**: US English.
- **Commit messages**: project convention; Drupal core uses US English.

---

## Spelling review checklist

### Dictionary and tools
- [ ] CSpell installed and run on the code
- [ ] No unjustified CSpell errors
- [ ] `cspell:ignore` only for truly valid words (not to cover UK spelling)
- [ ] `cspell:disable*` used minimally and with clear reason

### UK vs US English
- [ ] No `colour` → `color`
- [ ] No `behaviour` → `behavior`
- [ ] No `organise/organisation` → `organize/organization`
- [ ] No `customise/customisation` → `customize/customization`
- [ ] No `analyse` → `analyze`
- [ ] No `centre` → `center`
- [ ] No `dialogue` (in code) → `dialog`
- [ ] No `grey` → `gray`
- [ ] No `licence` (noun) → `license`
- [ ] Other UK forms detected and corrected

### Common typos
- [ ] No `occured` → `occurred`
- [ ] No `recieve` → `receive`
- [ ] No `seperate` → `separate`
- [ ] No `definately` → `definitely`
- [ ] No `accomodate` → `accommodate`
- [ ] No `neccessary` → `necessary`
- [ ] No `sucess` → `success`

### Per file type

#### PHP
- [ ] Function/method names in US English
- [ ] Variable names in US English
- [ ] Class names in US English
- [ ] Comments in US English
- [ ] Docblocks in US English
- [ ] `t()` source strings in US English
- [ ] String literals in arrays in US English

#### JavaScript
- [ ] Function/variable names in US English
- [ ] Comments in US English
- [ ] JSDoc in US English
- [ ] `Drupal.t()` source strings in US English

#### CSS
- [ ] Class names in US English (`.color-picker`, not `.colour-picker`)
- [ ] Comments in US English
- [ ] CSS custom property names in US English (`--primary-color`)

#### Twig
- [ ] Comments `{# ... #}` in US English
- [ ] Hardcoded strings in US English
- [ ] Translation filters with US English source

#### YAML
- [ ] Machine names in US English
- [ ] Labels and descriptions in US English
- [ ] Permissions descriptions in US English

#### Markdown / Text
- [ ] README in US English
- [ ] CHANGELOG in US English
- [ ] INSTALL in US English
- [ ] Technical comments in US English

### Naming machine names
- [ ] No UK spelling in file names
- [ ] No UK spelling in module names, theme names, route names
- [ ] No UK spelling in service IDs, plugin IDs

---

## Official resources

- [Drupal Spelling guide](https://project.pages.drupalcode.org/coding_standards/spelling/spelling/)
- [CSpell official site](https://cspell.org/)
- [CSpell directives documentation](https://cspell.org/docs/Configuration/document-settings/)
- [Drupal cspell tool page](https://www.drupal.org/node/3352552)
