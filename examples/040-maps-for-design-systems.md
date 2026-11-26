# Maps for design systems - Examples

These examples use Dart Sass and the built-in `sass:map` module. Treat each numbered code block as a separate example unless a section says it continues a previous one. Save Sass code in a `.scss` file and compile it with Dart Sass, for example:

~~~sh
sass input.scss output.css
~~~

## 1. A flat color map

Use a flat map when a group of values has one shared purpose.

~~~scss
$colors: (
  "brand": #3157d5,
  "text": #182033,
  "surface": #ffffff,
  "border": #d6dce8
);
~~~

The keys are quoted semantic names. The values remain Sass colors until they are used in a declaration or emitted as CSS custom properties.

## 2. Read a value by key

~~~scss
@use "sass:map";

$colors: (
  "brand": #3157d5,
  "text": #182033
);

.button {
  background-color: map.get($colors, "brand");
  color: map.get($colors, "text");
}
~~~

Expected CSS:

~~~css
.button {
  background-color: #3157d5;
  color: #182033;
}
~~~

## 3. Nested semantic tokens

~~~scss
$tokens: (
  "color": (
    "brand": #3157d5,
    "text-primary": #182033,
    "text-muted": #596579
  ),
  "space": (
    "small": 0.5rem,
    "medium": 1rem,
    "large": 1.5rem
  )
);
~~~

This structure separates token categories while keeping names about use and intent.

## 4. Read a nested value

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #3157d5,
    "text-primary": #182033
  )
);

.alert {
  border-color: map.get($tokens, "color", "brand");
  color: map.get($tokens, "color", "text-primary");
}
~~~

The nested lookup uses the group key followed by the token key.

## 5. Required token accessor with a useful error

This file is self-contained. It checks that the group is a map and that the requested name exists before returning the value.

~~~scss
@use "sass:map";
@use "sass:meta";

$tokens: (
  "color": (
    "brand": #3157d5,
    "text": #182033
  )
);

@function token($source, $group, $name) {
  $group-map: map.get($source, $group);

  @if meta.type-of($group-map) != "map" {
    @error "Unknown token group #{$group}.";
  }

  @if not map.has-key($group-map, $name) {
    @error "Unknown token #{$group}.#{$name}.";
  }

  @return map.get($group-map, $name);
}

.button {
  background-color: token($tokens, "color", "brand");
}
~~~

A lookup typo now stops compilation with a message instead of producing a missing declaration.

## 6. Check optional tokens

Use `map.has-key()` when a token is optional and should only produce a declaration when it exists.

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #3157d5
  )
);

.card {
  @if map.has-key($tokens, "color", "outline") {
    outline-color: map.get($tokens, "color", "outline");
  }
}
~~~

If the map has no `"outline"` key, Sass emits no outline declaration. For required tokens, prefer an accessor that reports an error.

## 7. Inspect keys and values

~~~scss
@use "sass:map";

$spacing: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 1.5rem
);

$spacing-names: map.keys($spacing);
$spacing-values: map.values($spacing);

@debug $spacing-names;
@debug $spacing-values;
~~~

The compiler prints the Sass lists in its debug output. Use `@each` directly when each key needs to stay paired with its value.

## 8. Add or replace a top-level entry

~~~scss
@use "sass:map";

$colors: (
  "brand": #3157d5,
  "text": #182033
);

$colors: map.set($colors, "focus", #9a42c8);

.focus-ring {
  outline-color: map.get($colors, "focus");
}
~~~

`map.set()` returns a new map. Reassigning that result retains the new entry.

## 9. Update a nested entry

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #3157d5,
    "text": #182033
  )
);

$tokens: map.set($tokens, "color", "brand", #2747b8);

.brand-surface {
  background-color: map.get($tokens, "color", "brand");
}
~~~

The original map value is not changed in place; the returned map becomes the new value of `$tokens`.

## 10. Compare shallow and deep merges

~~~scss
@use "sass:map";

$base: (
  "color": (
    "brand": #3157d5,
    "text": #182033
  )
);

$override: (
  "color": (
    "brand": #2747b8
  )
);

$shallow: map.merge($base, $override);
$deep: map.deep-merge($base, $override);

@debug map.get($shallow, "color", "text"); // null
@debug map.get($deep, "color", "text");    // #182033
~~~

The shallow merge replaces the whole conflicting `"color"` entry. The deep merge replaces only `"brand"` and retains `"text"`.

## 11. Remove an entry

~~~scss
@use "sass:map";

$colors: (
  "brand": #3157d5,
  "accent": #d64b85,
  "text": #182033
);

$colors: map.remove($colors, "accent");
~~~

`map.remove()` returns a map without the requested top-level key. The `sass:map` module also provides `map.deep-remove()` for a nested key path.

## 12. Generate custom properties from a flat map

~~~scss
$colors: (
  "brand": #3157d5,
  "text": #182033,
  "surface": #ffffff
);

:root {
  @each $name, $value in $colors {
    --color-#{$name}: #{$value};
  }
}
~~~

Expected CSS:

~~~css
:root {
  --color-brand: #3157d5;
  --color-text: #182033;
  --color-surface: #ffffff;
}
~~~

The browser sees the custom properties, not the original Sass map.

## 13. Generate custom properties from grouped maps

~~~scss
$tokens: (
  "color": (
    "brand": #3157d5,
    "text": #182033
  ),
  "space": (
    "small": 0.5rem,
    "large": 1.5rem
  )
);

:root {
  @each $group, $values in $tokens {
    @each $name, $value in $values {
      --#{$group}-#{$name}: #{$value};
    }
  }
}
~~~

This expects every top-level value to be a map. If some groups have a different shape, split the data or add validation before using the nested loop.

## 14. Generate button variant classes

~~~scss
@use "sass:map";

$button-variants: (
  "primary": (
    "background": #3157d5,
    "foreground": #ffffff
  ),
  "secondary": (
    "background": #edf1ff,
    "foreground": #253b80
  ),
  "danger": (
    "background": #b42335,
    "foreground": #ffffff
  )
);

.button {
  border: 0;
  border-radius: 0.375rem;
  padding: 0.75rem 1rem;
}

@each $name, $variant in $button-variants {
  .button--#{$name} {
    background-color: map.get($variant, "background");
    color: map.get($variant, "foreground");
  }
}
~~~

The generated modifier class names match the map keys. This is easiest to maintain when all variants share the same keys and value types.

## 15. Skip optional null values

~~~scss
$optional-declarations: (
  "outline": null,
  "box-shadow": 0 0.25rem 0.75rem rgb(0 0 0 / 12%)
);

.card {
  @each $property, $value in $optional-declarations {
    @if $value != null {
      #{$property}: $value;
    }
  }
}
~~~

The explicit condition documents that `null` means “do not emit this declaration.” Avoid letting a missing required map lookup silently act like an optional value.

## 16. Generate spacing utility classes

~~~scss
$spacing: (
  "small": 0.5rem,
  "medium": 1rem,
  "large": 1.5rem
);

@each $name, $value in $spacing {
  .gap-#{$name} {
    gap: $value;
  }
}
~~~

This creates `.gap-small`, `.gap-medium`, and `.gap-large`. Utility generation is useful when the naming pattern and supported values are intentional parts of the project’s CSS API.

## 17. Generate theme rules

~~~scss
$themes: (
  "light": (
    "surface": #ffffff,
    "text": #182033
  ),
  "dark": (
    "surface": #151a24,
    "text": #f3f5fa
  )
);

@each $theme-name, $values in $themes {
  @if $theme-name == "light" {
    :root {
      @each $name, $value in $values {
        --color-#{$name}: #{$value};
      }
    }
  } @else {
    [data-theme="#{$theme-name}"] {
      @each $name, $value in $values {
        --color-#{$name}: #{$value};
      }
    }
  }
}

.card {
  background: var(--color-surface);
  color: var(--color-text);
}
~~~

Sass generates both sets of CSS declarations. At runtime, the browser applies the declarations under the selected `data-theme` attribute.

## 18. Keep primitive and semantic tokens separate

~~~scss
$palette: (
  "blue-700": #2747b8,
  "blue-100": #edf1ff,
  "slate-900": #182033,
  "white": #ffffff
);

$semantic-colors: (
  "brand": map.get($palette, "blue-700"),
  "surface": map.get($palette, "white"),
  "text-primary": map.get($palette, "slate-900"),
  "surface-subtle": map.get($palette, "blue-100")
);
~~~

To compile this example, include `@use "sass:map";` as the first line. The palette names describe raw values; semantic names describe how components use them. This separation lets a theme change semantic assignments without renaming component tokens.

## 19. Share a token map through a Sass module

Example project structure:

~~~text
styles/
  _tokens.scss
  components/
    _button.scss
  app.scss
~~~

`styles/_tokens.scss`:

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #3157d5,
    "text": #182033
  )
);

@function token($group, $name) {
  @if not map.has-key($tokens, $group, $name) {
    @error "Unknown token #{$group}.#{$name}.";
  }

  @return map.get($tokens, $group, $name);
}
~~~

`styles/components/_button.scss`:

~~~scss
@use "../tokens";

.button {
  background-color: tokens.token("color", "brand");
  color: tokens.token("color", "text");
}
~~~

`styles/app.scss`:

~~~scss
@use "components/button";
~~~

The Sass module namespace makes the token helper explicit and avoids adding all of its members to a global namespace.

## 20. Debug a map lookup

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #3157d5
  )
);

@debug map.get($tokens, "color", "brand");
@debug map.get($tokens, "color", "missing");
~~~

The first debug statement prints the color value; the second prints `null`. This is a quick way to inspect a lookup while developing. Remove temporary debug statements when finished.

## 21. Common broken patterns and fixes

### Broken: discard the updated map

~~~scss
map.set($colors, "focus", #9a42c8);
~~~

Fix:

~~~scss
$colors: map.set($colors, "focus", #9a42c8);
~~~

### Broken: assume a shallow merge retains nested entries

~~~scss
$combined: map.merge($base, $override);
~~~

If nested keys must remain, use:

~~~scss
$combined: map.deep-merge($base, $override);
~~~

### Broken: use a missing required value without validation

~~~scss
.button {
  color: map.get($tokens, "color", "text-primray");
}
~~~

Fix the key and validate required paths with `map.has-key()` or a token function that raises `@error`.

## 22. Practice tasks

1. Add a `"success"` color to `$colors` and use it on a notice component.
2. Create a `"type"` group with `"body"` and `"heading"` tokens.
3. Add a third button variant and confirm the loop generates its modifier class.
4. Make a dark theme with surface, text, and brand custom properties.
5. Add an accessor that reports an error for a missing group and a missing token.
6. Create a map of card variants and generate one class for each variant.
7. Use a deep merge to override one nested token while preserving its siblings.
8. Choose one Sass map that should become CSS custom properties because it needs runtime theme switching.

## Reference links

## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="039-reusable-responsive-mixins.md">039 - Reusable Responsive Mixins: Examples</a></td><td><a href="../lessons/040-maps-for-design-systems.md">Lesson</a></td><td><a href="../qnas/040-maps-for-design-systems.md">Q&amp;A</a></td><td><a href="041-component-styles-and-architecture.md">041 - Component styles and architecture - Examples</a></td></tr>
  </tbody>
</table>
