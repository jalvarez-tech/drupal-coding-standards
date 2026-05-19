# PHP — Exceptions

Scope: exception class naming, message conventions, subclass strategy, try/catch formatting, inheritance hierarchy.

Source:
- [Exceptions](https://project.pages.drupalcode.org/coding_standards/php/exceptions/)

For formatting/syntax: see `PHP-CODING.md`.
For naming/OO: see `PHP-OO-PSR4.md`.

---

## Naming

1. Follow **OO coding** standards (classes).
2. **`Exception` suffix ALWAYS**.
3. Message: include **hint about the values that caused the exception**. Format with concatenation or `sprintf()`. Values in single quotes.
4. **Do NOT translate** the message. Only install and update system are translated (user-facing).
5. **Do NOT use `SafeMarkup::format()`** (deprecated in modern Drupal; see `PHP-LEGACY.md`).
6. Naming: `[Subsystem][ErrorType]Exception`.

## Subclasses

Preferred **specific subclasses** vs a generic one with different messages:

```php
class WidgetNotFoundException extends \Exception {}

function use_widget($widget_name) {
  $widget = find_widget($widget_name);

  if (!$widget) {
    throw new WidgetNotFoundException("Widget '$widget_name' not found.");
  }
}
```

## Try-catch

Follows the if-else pattern: each `catch` on a new line, separated from the previous `}`:

```php
try {
  $widget = 'thingies';
  $result = use_widget($widget);
}
catch (WidgetNotFoundException $e) {
  // Specific handling.
}
catch (\Exception $e) {
  // Generic handling.
  \Drupal::logger('widget')->error($e->getMessage());
}
```

## Inheritance

PHP requires all exceptions to inherit from `\Exception` (directly or indirectly).

If a subsystem has multiple exceptions, **all extend a common base**:

```php
class FelineException extends \Exception {}
class FelineHairBallException extends FelineException {}
class FelineKittenTooCuteException extends FelineException {}

try {
  $normal = new Kitten();
  $normal->playWith($string);
}
catch (FelineHairBallException $e) {
  // ...
}
catch (FelineKittenTooCuteException $e) {
  // ...
}
catch (FelineException $e) {
  // Generic feline.
}
```
