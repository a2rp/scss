# Maps for design systems

Sass maps store related values as key/value pairs. They are useful for organizing design tokens, component variants, and configuration. Maps can also drive generated CSS. Remember that Sass data exists during compilation; use CSS custom properties when a value must change in the browser at runtime.

## Learning goals

After this lesson, you should be able to:

- Create flat and nested maps with clear, quoted keys.
- Read values safely through the `sass:map` module.
- Update and combine maps without expecting in-place mutation.
- Use `@each` to generate CSS custom properties and component variants.
- Validate token lookups and choose sensible design-system boundaries.
- Explain when to use Sass maps and when to use CSS custom properties.

## 1. Map syntax and keys

A map is written with parentheses. Each entry is a key, a colon, and a value; entries are separated by commas.

~~~scss
$brand-colors: (
  "primary": #3157d5,
  "accent": #d64b85,
  "surface": #ffffff,
  "text": #182033
);
~~~

Quote keys consistently. Names such as `"red"`, `"primary"`, and `"small"` are intended to be strings; an unquoted color-like key can be parsed as a Sass color. Quoted keys also make the map’s purpose easier to understand.

Map values can be colors, numbers, lengths, strings, lists, booleans, null, or other maps. Sass permits mixed types, but a map with a clear purpose is easier to validate and use.

~~~scss
$button-settings: (
  "radius": 0.5rem,
  "padding": 0.75rem 1rem,
  "uppercase": false
);
~~~

An empty map is `()`. Sass also treats `()` as an empty list in some contexts, so use it intentionally and do not rely on type checks to distinguish those empty values.

## 2. Nested maps for token groups

Nested maps group values by role. A design system may keep semantic colors, spacing, and radius tokens in separate groups:

~~~scss
$tokens: (
  "color": (
    "text-primary": #182033,
    "text-muted": #596579,
    "surface": #ffffff,
    "brand": #3157d5
  ),
  "space": (
    "small": 0.5rem,
    "medium": 1rem,
    "large": 1.5rem
  ),
  "radius": (
    "control": 0.375rem,
    "card": 0.75rem
  )
);
~~~

Semantic names describe how a value is used. For example, `"text-muted"` communicates a role more clearly than an unnamed raw color value. If the palette changes, components can keep using the semantic token.

Do not add nesting just because maps support it. A few purposeful levels are usually easier to maintain than a deeply nested configuration tree.

## 3. Read map values

Load Sass’s built-in map module with the module system:

~~~scss
@use "sass:map";

$brand: map.get($tokens, "color", "brand");
$large-space: map.get($tokens, "space", "large");
~~~

The multi-key form looks through nested maps from left to right. Dart Sass 1.27.0 and later supports nested lookup with `map.get($map, $key, $keys...)`. If a key or an intermediate map is missing, `map.get()` returns `null`.

Use `map.has-key()` to check whether a path exists before using its value:

~~~scss
@if map.has-key($tokens, "color", "brand") {
  .link {
    color: map.get($tokens, "color", "brand");
  }
}
~~~

For a required token, fail with a useful compile-time message instead of silently generating an incomplete declaration:

~~~scss
@use "sass:map";

@function token($group, $name) {
  @if not map.has-key($tokens, $group, $name) {
    @error "Unknown token #{$group}.#{$name}.";
  }

  @return map.get($tokens, $group, $name);
}

.button {
  background-color: token("color", "brand");
}
~~~

The function returns a Sass value; it does not emit CSS by itself. The calling rule uses the returned value. Keep lookup helpers focused and report errors where they can include useful context.

## 4. Maps are immutable values

Sass maps are immutable. Functions such as `map.set()` and `map.merge()` return a new map. Assign the result if you want to keep it:

~~~scss
@use "sass:map";

$base-colors: (
  "brand": #3157d5,
  "text": #182033
);

$base-colors: map.set($base-colors, "focus", #9a42c8);
~~~

The original value is not modified in place. Reassignment makes the change explicit.

### Add or replace an entry with `map.set()`

Use `map.set($map, $key, $value)` to add or replace a top-level entry. For a nested entry, provide the keys leading to the map that should be updated:

~~~scss
$tokens: map.set($tokens, "color", "brand", #2747b8);
~~~

### Combine top-level entries with `map.merge()`

A normal merge combines top-level entries. If both maps have the same key, the second map’s value replaces the first. For nested maps, the replacement is shallow:

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

$shallow-result: map.merge($base, $override);
// The whole "color" value is replaced; "text" is not in this result.
~~~

### Recursively combine nested entries with `map.deep-merge()`

Use `map.deep-merge()` when nested entries should be combined recursively:

~~~scss
$combined: map.deep-merge($base, $override);
// "brand" is overridden and "text" remains.
~~~

`map.deep-merge()` is available in Dart Sass starting with 1.27.0. Use it deliberately: recursive configuration merging is useful, but can be confusing if map shapes are not documented.

## 5. Generate CSS from maps

Use `@each` to iterate over map keys and values:

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

This emits CSS custom properties such as `--color-brand`. Interpolation inserts each key into the property name and each value into the declaration.

For grouped tokens, use an outer loop for the group and an inner loop for its entries:

~~~scss
$css-tokens: (
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
  @each $group, $values in $css-tokens {
    @each $name, $value in $values {
      --#{$group}-#{$name}: #{$value};
    }
  }
}
~~~

The inner loop expects each group value to be a map. Keep generated names predictable because they become part of your CSS API.

Sass maps do not create browser-visible variables on their own. The loop emits CSS custom properties; CSS and JavaScript can then read or change those properties at runtime.

## 6. Use maps for component variants

A variant map works well when each variant has the same named properties:

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

A map is helpful because the variants share a predictable shape. If variants need substantially different markup or behavior, keep that logic explicit instead of forcing it into a large map.

## 7. Organize maps in Sass modules

A token module can expose a small, deliberate API:

~~~text
styles/
  _tokens.scss
  _index.scss
  components/
    _button.scss
  app.scss
~~~

For example, `_tokens.scss` can define the token map. A component can load it with `@use "tokens"` and refer to members through the module namespace. Sass modules load each file once and avoid the global namespace behavior of legacy `@import`.

If a module needs configurable defaults, plan that API with `!default` variables and module configuration. Configuration is applied when the module is first loaded; it is not a runtime theme switch.

For browser-selectable themes, emit CSS custom properties for each theme:

~~~scss
:root {
  --color-surface: #ffffff;
  --color-text: #182033;
}

[data-theme="dark"] {
  --color-surface: #151a24;
  --color-text: #f3f5fa;
}

.card {
  background: var(--color-surface);
  color: var(--color-text);
}
~~~

Use maps to organize and generate theme declarations. Use CSS custom properties for values that must respond to runtime state.

## 8. Handle null values and validate inputs

A map lookup may return `null` when a key does not exist. Sass omits declarations whose property value is `null`, which can hide a typo. Check required tokens and use `@error`. For optional values, branch explicitly:

~~~scss
$optional-styles: (
  "outline": null,
  "box-shadow": 0 0.25rem 0.75rem rgb(0 0 0 / 12%)
);

.card {
  @each $property, $value in $optional-styles {
    @if $value != null {
      #{$property}: $value;
    }
  }
}
~~~

Validate map shape at important boundaries. A missing nested map, unexpected value type, or unknown token should produce a clear compile-time error rather than malformed CSS.

## 9. Design-system practices

- Name tokens by purpose and role, not only by their current raw value.
- Separate primitive palette values from semantic tokens when both are useful.
- Keep keys quoted and consistent across files.
- Document which maps are public configuration and which are implementation details.
- Use `map.get()` for reads and reassign results from update functions.
- Use shallow merge for intentional replacement and deep merge when nested entries should remain.
- Avoid giant maps that combine unrelated component, theme, and application data.
- Do not use Sass maps as a substitute for browser runtime state.
- Keep generated CSS names stable because other styles may depend on them.
- Avoid duplicate sources of truth; derive generated token CSS from the map.

## Common mistakes

### Discarding the result of an update

~~~scss
map.set($tokens, "color", "brand", #2747b8);
~~~

This returns a new map, but the result is discarded. Assign the result back to a variable.

### Assuming `map.merge()` recursively combines nested maps

A shallow merge replaces a conflicting nested value as a whole. Use `map.deep-merge()` when recursive combination is intended.

### Using an unquoted color-like key

A key such as `red` may be parsed as a color. Use `"red"` when the key is meant to be a string name.

### Treating a missing lookup as a valid token

`map.get()` returns `null` for a missing path. Validate required paths so typos do not silently disappear from output.

### Expecting Sass data to exist in the browser

Sass variables and maps are compile-time values. Emit CSS custom properties when browser code or runtime selectors need access.

## Quick reference

| Need | Sass API |
| --- | --- |
| Read a value | `map.get($map, $key)` |
| Read a nested value | `map.get($map, $key, $nested-key)` |
| Check a key | `map.has-key($map, $key)` |
| Add or replace a value | `map.set($map, $key, $value)` |
| Combine top-level entries | `map.merge($map1, $map2)` |
| Recursively combine nested entries | `map.deep-merge($map1, $map2)` |
| Iterate over key/value pairs | `@each $key, $value in $map` |
| Return a computed Sass value | `@function ... { @return ...; }` |
| Emit browser-readable tokens | CSS custom properties |

## Next lesson

Continue with [Component styles and architecture](041-component-styles-and-architecture.md).
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="039-reusable-responsive-mixins.md">039 - Reusable Responsive Mixins</a></td><td><a href="../qnas/040-maps-for-design-systems.md">Q&amp;A</a></td><td><a href="../examples/040-maps-for-design-systems.md">Example</a></td><td><a href="041-component-styles-and-architecture.md">041 - Component styles and architecture</a></td></tr>
  </tbody>
</table>
