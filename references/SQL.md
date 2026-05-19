# SQL — Drupal Coding Standards Reference

Official consolidation of:
- [Coding conventions](https://project.pages.drupalcode.org/coding_standards/sql/conventions/)
- [Key words](https://project.pages.drupalcode.org/coding_standards/sql/keywords/)
- [Avoid "SELECT * FROM..."](https://project.pages.drupalcode.org/coding_standards/sql/select-from/)

> **API versions in this file:** the official Drupal coding standards still document both the legacy procedural `db_query()` / `db_select()` family (Drupal 7) and the modern `\Drupal::database()->query()` / `->select()` family (Drupal 8+). On any Drupal 8+ codebase, **new code must use the D8+ API**; `db_query()` and friends are 🟥 forbidden — see `php/PHP-LEGACY.md`. The D7-style examples below are kept for historical context only because contributors will encounter them when patching old contrib code. When you write a review, flag any `db_query()` in new code as a 🟡 standards violation (legacy API used in modern context).

---

## Table of contents

1. [Reserved words and quoting](#1-reserved-words-and-quoting)
2. [Capitalization and user-supplied data](#2-capitalization-and-user-supplied-data)
3. [Naming](#3-naming)
4. [Database server config](#4-database-server-config)
5. [Indentation](#5-indentation)
6. [Avoid `SELECT *`](#6-avoid-select)
7. [Schema API](#7-schema-api)
8. [Review checklist](#8-review-checklist)

---

## 1. Reserved words and quoting

### Drupal 8 and 9+

**ALL identifiers** in SQL queries must be quoted:

- **Table names** with **curly brackets**: `{table_name}`.
- **Other identifiers** (columns, aliases) with **square brackets**: `[column_name]`.

```sql
SELECT [u].[uid], [u].[name]
FROM {users} [u]
WHERE [u].[status] = 1
```

### Avoid reserved words (all Drupal versions)

**Do NOT use reserved words** from ANSI SQL / MySQL / PostgreSQL / MS SQL Server / etc. for column or table names. The official Drupal SQL conventions page presents this rule under its "Drupal 7" subsection because D8+ added `[bracket]` quoting that escapes most cases, but **portability across MySQL / PostgreSQL / SQLite is still a Drupal requirement** and bracket quoting does not lift it — picking `type`, `order`, `group`, etc. as a column name can break on a database your code claims to support.

#### Reserved words refs

- [Drupal SQL Keywords (consolidated list, 826 reserved terms)](https://project.pages.drupalcode.org/coding_standards/sql/keywords/)
- [MySQL 8.x Reserved Words](https://dev.mysql.com/doc/refman/8.4/en/keywords.html)
- [PostgreSQL SQL Key Words](https://www.postgresql.org/docs/current/sql-keywords-appendix.html)
- [Oracle Reserved Words](https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/Oracle-SQL-Reserved-Words.html) — `UID` is particularly problematic
- [MS SQL Server Reserved Keywords](https://learn.microsoft.com/sql/t-sql/language-elements/reserved-keywords-transact-sql)
- [IBM Db2 Reserved Schema Names and Keywords](https://www.ibm.com/docs/en/db2/11.5?topic=words-reserved-schema-names)

#### Keywords commonly misused

`UID` (Oracle), `TIMESTAMP`, `TYPE`, `TYPES`, `MODULE`, `DATA`, `DATE`, `TIME`, `ORDER`, `GROUP`, `USER`, `ROLE` and similar. The keywords list above is the canonical reference — review it before naming a new column.

---

## 2. Capitalization and user-supplied data

### Capitalization

- **SQL reserved words in UPPERCASE** — a style convention in Drupal standards for code readability (SQL keywords are case-insensitive at the database level, but the standard mandates uppercase to distinguish keywords from identifiers).
- **Column and constraint names in lowercase**.
- **Each table name wrapped in `{}`** — allows Drupal to apply the prefix.

### Variable arguments (anti-SQL injection)

Variable arguments (frequently user-supplied) **NEVER** get concatenated into the query body. They are passed as **separate parameters** to the database API.

### Drupal 8+ — named placeholders (current standard for D11)

The modern Drupal Database API uses **named placeholders** of the form `:name`:

```php
\Drupal::database()->insert('filters')
  ->fields([
    'format' => $format,
    'module' => 'php',
    'delta' => 0,
    'weight' => 0,
  ])
  ->execute();

// Or with a manual query:
\Drupal::database()->query(
  'SELECT [name] FROM {users} WHERE [uid] = :uid',
  [':uid' => $uid]
)->fetchField();
```

Notes:
- Named placeholders carry **no type prefix**. Cast in PHP before passing if a type is required.
- The placeholder name (without the leading `:`) is the array key in the arguments map.
- For `IN (...)` clauses, pass an array and Drupal expands it: `'WHERE [uid] IN (:uids[])'`, `[':uids[]' => $ids]`.

### Drupal 7 — `sprintf`-style placeholders (legacy, do not use in new code)

D7 used type-tagged placeholders passed to `db_query()` / `db_query_range()` / `db_query_temporary()`:

| Placeholder | Type | Notes |
|---|---|---|
| `%d` | integers | |
| `%f` | floats | |
| `%s` | strings | enclose in single quotes in the query |
| `%b` | binary data | **Do NOT** enclose in single quotes |
| `%%` | replaced with `%` | literal escape |

```php
// ⚠️ Drupal 7 only — `db_query()` is removed in D9+.
db_query(
  "INSERT INTO {filters} (format, module, delta, weight) VALUES (%d, 'php', 0, 0)",
  $format
);
```

For any D8+ codebase the only correct form is the named-placeholder example above.

### Literal arguments

**Constants/literals** may go directly in the query body **OR** be treated as variable args.

### String quoting

**Any string literal or `%s` placeholder** is enclosed in **single quotes** `'`. **NEVER double quotes**.

```sql
WHERE [name] = 'some_value'
```

### Schema API

Since Drupal 6.x, **table definitions and constraints** (primary keys, unique keys, indexes) **are always handled via the [Schema API](https://www.drupal.org/node/146843)**. It resolves cross-database compat automatically.

---

## 3. Naming

### Table names

- **Singular nouns** (Drupal 7+, in contrast to D6 which mixed).
- **Module prefix** to prevent namespace conflicts.

```
✅ {my_module_item}
❌ {my_module_items}  /* plural */
❌ {item}              /* no prefix */
```

### Constraints

- **Name every constraint** (primary, foreign, unique). Otherwise, system-generated names appear in obscure errors.

#### Illustrative historical bug

```sql
-- D6: `KEY (mid)` without explicit name
-- mysqldump wrote it as `KEY mid (mid)`
-- → syntax error because mid() is a mysql function.
-- Fix: name it explicitly.
KEY moderation_roles_mid (mid)
```

### Index names

Start with the **table name** + `_idx` or another descriptive suffix:

```sql
INDEX users_sid_idx (sid)
INDEX node_revision_node_idx (nid)
```

---

## 4. Database server config

Most DB servers use extensions to standard SQL. Many can be configured in **standard-compliant** mode to force portable code.

### MySQL

Enable **ANSI** and **Strict** mode:

```sql
SET sql_mode = 'ANSI,STRICT_ALL_TABLES';
```

Or in `my.cnf`:

```ini
[mysqld]
sql_mode = "ANSI,STRICT_ALL_TABLES"
```

Refs:
- [MySQL Server SQL Modes (8.x)](https://dev.mysql.com/doc/refman/8.4/en/sql-mode.html) — ANSI mode and strict-mode behavior.

---

## 5. Indentation

Drupal **has no standard method** for indentation or formatting of multi-line queries. Strategies in use:

### Strategy 1 — query in heredoc-like with concatenation

```php
if (!(db_query(
  "
    INSERT INTO {my_module_media_file_type}
    SET extension   = '%s',
    attributes      = '%s'
  ",
  $file_type_entry['extension'],
  $selected_attributes
))) {
  $errors = TRUE;
}
```

### Strategy 2 — pipe concatenation

```php
$sql = "SELECT t.*, j1.col1, j2.col2"
. " FROM {table} AS t"
. " LEFT JOIN {join1} AS j1 ON j1.id = t.jid"
. " LEFT JOIN {join2} AS j2 ON j2.id = t.jid"
. " WHERE t.col LIKE '%s'"
. " ORDER BY %s"
;
$result = db_query($sql, 'find_me', 't.weight');
```

### Strategy 3 — HEREDOC with "rivers"

```php
$sql = <<<SQL
SELECT t.id,
       t.name
FROM   {my_table} t
WHERE  t.status = :status
SQL;
```

**Pick one** and apply it consistently across the module.

---

## 6. Avoid `SELECT *`

### Rule (all Drupal versions)

Queries that generate lists of nodes **must avoid `SELECT *` in ALL cases** — list only the columns you actually consume.

**Historical rationale:** on Drupal versions prior to 7, `SELECT * FROM {node}` could **bypass the Node Access system** and leak private content to unauthorized users. That specific bypass is gone in D7+ thanks to the `node_access` tagging system, but the rule still stands because (a) `SELECT *` is wasteful when only a few columns are used, (b) it tightly couples the query to the table schema in a way that breaks on future column additions, and (c) reviewers can't audit what data leaves the database without enumerating columns.

```sql
-- ❌ Bad
SELECT * FROM {node}

-- ✅ Good
SELECT [nid], [title], [type], [status] FROM {node}
```

### General reasons to avoid `SELECT *`

1. **Less self-documenting** than listing fields explicitly.
2. **Slightly slower**.
3. Couples code to table structure — adding columns can cause side-effects.

### When `SELECT *` is OK

1. **Table fields are dynamic** and unknown at development-time (rare and generally bad practice).
2. The **field list to select is prohibitively long**.

### References

- [Drupal development list discussion (Feb 2009)](https://lists.drupal.org/pipermail/development/2009-February/thread.html#31953) — Drupal's normative rationale for avoiding `SELECT *`.
- [SELECT * IS EVIL](https://www.parseerror.com/sql/select*isevil.html) — supplementary third-party essay (not part of the official Drupal standards; provided for context only).

---

## 7. Schema API

All schema definitions (tables, columns, indexes, primary keys, foreign keys) are done via Schema API in `<module>.install`:

```php
/**
 * Implements hook_schema().
 */
function my_module_schema() {
  $schema['my_module_log'] = [
    'description' => 'Stores log entries.',
    'fields' => [
      'lid' => [
        'type' => 'serial',
        'unsigned' => TRUE,
        'not null' => TRUE,
        'description' => 'Primary Key: Unique log entry ID.',
      ],
      'uid' => [
        'type' => 'int',
        'unsigned' => TRUE,
        'not null' => TRUE,
        'default' => 0,
        'description' => 'The {users}.uid of the user who triggered the event.',
      ],
      'type' => [
        'type' => 'varchar',
        'length' => 64,
        'not null' => TRUE,
        'default' => '',
        'description' => 'Type of log entry.',
      ],
      'message' => [
        'type' => 'text',
        'not null' => TRUE,
        'size' => 'big',
        'description' => 'Text of log message.',
      ],
      'timestamp' => [
        'type' => 'int',
        'not null' => TRUE,
        'default' => 0,
        'description' => 'Unix timestamp of when event occurred.',
      ],
    ],
    'primary key' => ['lid'],
    'indexes' => [
      'my_module_log_type_idx' => ['type'],
      'my_module_log_uid_idx' => ['uid'],
      'my_module_log_timestamp_idx' => ['timestamp'],
    ],
    'foreign keys' => [
      'log_user' => [
        'table' => 'users',
        'columns' => ['uid' => 'uid'],
      ],
    ],
  ];
  return $schema;
}
```

---

## 8. Review checklist

### Quoting (D8+)

- [ ] Tables wrapped in `{table_name}`
- [ ] Columns/aliases wrapped in `[column_name]`
- [ ] String literals in single quotes `'value'`
- [ ] **No double quotes**

### SQL injection prevention

- [ ] **NEVER** concatenate user input into query body
- [ ] User input via **named placeholders** `:param` (D8+); type-tagged `%d`/`%s`/`%f`/`%b` only for legacy D7 `db_query()` calls
- [ ] Params passed as array to `->query()` (or `db_query()` in legacy D7 code)
- [ ] **Schema definitions/alterations** (create/drop/alter table, add/drop column, indexes, constraints) go through the **Schema API** — never raw `CREATE TABLE`/`ALTER TABLE`
- [ ] **Data queries** may use raw SQL through Drupal's DB APIs (`Database::getConnection()->query()`, `->select()`, `->insert()`, etc.) **as long as** identifiers use `{table}` / `[column]` quoting and all user-supplied values go through `:placeholders`

### Capitalization

- [ ] SQL keywords in **UPPERCASE**: `SELECT`, `FROM`, `WHERE`, `JOIN`, `INSERT`, `UPDATE`, `DELETE`, `INTO`, `VALUES`, `AS`, `ON`, `AND`, `OR`, `NOT`, `ORDER BY`, `GROUP BY`, `LIMIT`, etc.
- [ ] Column and constraint names in **lowercase**

### Naming

- [ ] Tables: **singular** + module prefix
- [ ] Constraints **named explicitly** (not auto-generated)
- [ ] Indexes with table prefix: `<table>_<field>_idx`

### SELECT *

- [ ] **No `SELECT *`** except in justified cases (dynamic fields or prohibitively long list)
- [ ] **NEVER `SELECT * FROM {node}`** (security — Node Access bypass)
- [ ] Explicitly list necessary fields

### Reserved words

- [ ] Verify against ANSI SQL + MySQL + PostgreSQL keyword lists
- [ ] Avoid `TIMESTAMP`, `TYPE`, `DATA`, `DATE`, `TIME` as column/table names
- [ ] If needed, quote correctly (curly/square brackets in D8+)

### Schema API

- [ ] Tables defined via `hook_schema()` in `<module>.install`
- [ ] Update functions (`hook_update_N()`) for alterations
- [ ] Foreign keys documented
- [ ] Indexes on frequent WHERE/JOIN columns
- [ ] Complete descriptions for tables and fields

### Indentation and format

- [ ] Consistent strategy within the module (heredoc, concatenation, or pipe)
- [ ] Keywords vertically aligned when it helps readability
- [ ] Line ≤ 80 chars when reasonable

### DB server config

- [ ] MySQL in ANSI + Strict mode for development
- [ ] Portable schema (no MySQL-isms if distributing contrib)

---

## Common patterns (D8+)

### Insert

```php
\Drupal::database()->insert('my_table')
  ->fields([
    'uid' => $uid,
    'name' => $name,
    'created' => time(),
  ])
  ->execute();
```

### Update

```php
\Drupal::database()->update('my_table')
  ->fields(['status' => 1])
  ->condition('uid', $uid)
  ->execute();
```

### Delete

```php
\Drupal::database()->delete('my_table')
  ->condition('uid', $uid)
  ->execute();
```

### Select with join

```php
$query = \Drupal::database()->select('users_field_data', 'u');
$query->join('node_field_data', 'n', '[n].[uid] = [u].[uid]');
$query->fields('u', ['uid', 'name']);
$query->fields('n', ['nid', 'title']);
$query->condition('n.status', 1);
$query->condition('u.status', 1);
$query->orderBy('n.created', 'DESC');
$query->range(0, 10);
$results = $query->execute()->fetchAll();
```

### Entity Query (preferred for entities)

For entities, **prefer `entity_type.manager`** over direct SQL:

```php
$query = \Drupal::entityQuery('node')
  ->accessCheck(TRUE)
  ->condition('type', 'article')
  ->condition('status', 1)
  ->sort('created', 'DESC')
  ->range(0, 10);
$nids = $query->execute();
$nodes = \Drupal::entityTypeManager()->getStorage('node')->loadMultiple($nids);
```

This **respects Node Access** automatically and abstracts the storage layer.
