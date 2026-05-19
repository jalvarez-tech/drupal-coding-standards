# Drupal Coding Standards — Trigger Surface (canonical)

**Single source of truth** for "which files / keywords activate the `drupal-coding-standards` skill". The previous duplication across `SKILL.md` description, `openspec-apply-change` Step 6, and `openspec-verify-change` Step 7 created drift; consumers now reference this file instead of maintaining their own copies.

## Consumers (must read this file)

- `.claude/skills/drupal-coding-standards/SKILL.md` — frontmatter `description` field rephrases this list as activation keywords for the runtime indexer.
- `.claude/skills/openspec-apply-change/SKILL.md` Step 6 — "Drupal standards gate" trigger condition.
- `.claude/skills/openspec-verify-change/SKILL.md` Step 7 — "Drupal standards compliance" sub-check trigger condition.
- `.codex/skills/drupal-coding-standards/triggers.md` — optional mirror for projects that also use the Codex skill layout (refresh with `rsync -a --delete`).

When you change either list below, **update every consumer in the same commit** so they don't drift. The standard smoke test is to run `/opsx:verify` against a change that edits a `.module` file and confirm the Drupal-standards gate fires.

---

## 1. File-extension triggers

The skill activates on edits to **any** file matching these patterns.

```yaml
extensions:
  # PHP family — Drupal procedural + OO code.
  - "*.php"
  - "*.module"
  - "*.theme"
  - "*.install"
  - "*.inc"
  - "*.engine"
  - "*.profile"
  - "*.test"
  - "*.api.php"
  # Twig templates.
  - "*.html.twig"
  - "*.twig"
  # Drupal YAML — info, routing, services, libraries, links, permissions, config exports.
  - "*.info.yml"
  - "*.routing.yml"
  - "*.services.yml"
  - "*.libraries.yml"
  - "*.links.menu.yml"
  - "*.permissions.yml"
  - "*.schema.yml"
  - "*.links.action.yml"
  - "*.links.contextual.yml"
  - "*.links.task.yml"
  - "*.breakpoints.yml"
  # Drupal config exports — explicit paths only, NOT a global *.yml glob.
  - "config/default/**/*.yml"
  - "config/install/**/*.yml"
  - "config/schema/**/*.yml"
  - "modules/custom/**/*.yml"
  - "themes/custom/**/*.yml"
  - "docroot/modules/custom/**/*.yml"
  - "docroot/themes/custom/**/*.yml"
  - "web/modules/custom/**/*.yml"
  - "web/themes/custom/**/*.yml"
  # CSS family.
  - "*.css"
  - "*.pcss.css"
  - "*.scss"
  # JavaScript.
  - "*.js"
  # Composer.
  - "composer.json"
```

> **Note on YAML scoping**: the previous version of this file used a broad `*.yml` glob, which over-matched CI / GitHub Actions / Composer / Docker YAML files that have nothing to do with Drupal. The list above now matches **only** files whose name follows a Drupal convention (`*.info.yml`, `*.routing.yml`, …, `*.schema.yml`, `*.breakpoints.yml`) plus YAML files inside the Drupal config-export directories or custom module/theme trees. Repo-root YAML such as `.github/workflows/*.yml`, `bitbucket-pipelines.yml`, `phpstan.neon` (not YAML, just a note), `.gitlab-ci.yml`, and `.ddev/config.*.yaml` are intentionally **not** matched.

## 2. Keyword triggers (description-only)

The runtime indexer also activates the skill when the user types any of these keywords, even if no matching file is in scope:

```yaml
keywords:
  - "phpcs"
  - "drupal/coder"
  - "Coder Sniffer"
  - "phpstan drupal"
  - "ESLint Drupal"
  - "hook_"
  - "render array"
  - "preprocess function"
  - "Drupal.behaviors"
  - "Drupal.t()"
  - "Schema API"
  - "Drupal coding standards"
  - "Drupal naming"
```

## 3. How a consumer should use this

Pseudo-code for a consumer's trigger check:

```python
touched_files = git_diff_files(base="HEAD", head="WORKING")

# triggers.md is Markdown, not YAML; consumers extract the first fenced ```yaml block
# under § 1 and parse THAT. (Equivalently, mirror the patterns into a sibling triggers.yaml
# if you prefer to call load_yaml() directly — keep the two in sync.)
patterns = parse_yaml_block(
    read(".claude/skills/drupal-coding-standards/triggers.md"),
    section="1. File-extension triggers",
)["extensions"]

triggered = any(
    fnmatch(file, pattern)
    for file in touched_files
    for pattern in patterns
)

if triggered:
    invoke("drupal-coding-standards", touched_files)
```

In practice, the agent reads this file when it executes `openspec-apply-change` Step 6 or `openspec-verify-change` Step 7, matches the touched-files set against the YAML-block patterns above, and invokes the skill only if at least one file matches. If zero files match, the agent records "Drupal standards compliance: not applicable (no Drupal files touched)" in the verify report.

## 4. Updating the trigger surface

1. Edit this file. Add or remove a pattern in the appropriate YAML block.
2. Re-read each consumer listed in § 1 and confirm they say "see `triggers.md`" rather than re-listing the patterns inline. The acceptable inline form is the SKILL.md frontmatter `description` field, which **paraphrases** this list for the indexer (the indexer doesn't read this file directly).
3. Mirror to `.codex/`:
   ```bash
   rsync -a --delete .claude/skills/drupal-coding-standards/ .codex/skills/drupal-coding-standards/
   diff -r .claude/skills/drupal-coding-standards .codex/skills/drupal-coding-standards
   ```
4. Run the smoke test (`/opsx:apply` on a throwaway change that edits the newly-added extension) to confirm the gate fires.
