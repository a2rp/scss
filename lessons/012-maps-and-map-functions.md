# 012 — Maps and Map Functions

## Learning goals

After this lesson, you should be able to:

- Explain how a Sass map associates keys with values.
- Create maps with clear, reliable keys.
- Look up a value and check whether a key exists.
- Read values from nested maps.
- Iterate over key/value pairs with @each.
- Add, replace, merge, and remove map entries.
- Understand why Sass map functions return new maps.
- Use maps to organize design tokens and reusable configuration.

## 1. What is a Sass map?

A map stores key/value pairs. Each key identifies the value associated with it. Maps are useful when data should be looked up by a name rather than by its position.

~~~scss
$font-weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);
~~~

The key is before the colon; the value is after it. The keys must be unique. Values may be repeated.

Maps use parentheses, including when a map is empty:

~~~scss
$empty-map: ();
~~~

Sass maps are Sass data structures. A map by itself is not a valid CSS property value. Use map functions to retrieve values or iterate over its pairs and generate CSS.

## 2. Prefer quoted string keys

Map keys can be many kinds of Sass values, but quoted strings are usually the clearest choice for names.

~~~scss
$colors: (
  "brand": #2457c5,
  "success": #147d4b,
  "danger": #b42318
);
~~~

Some unquoted words are parsed as values of another type. For example, red is a Sass color, not an ordinary unquoted string. Quoted keys avoid accidental type differences and make map lookups predictable.

~~~scss
$named-colors: (
  "red": #b42318,
  "blue": #2457c5
);
~~~

Look up the "red" key as a string:

~~~scss
@use "sass:map";

$danger: map.get($named-colors, "red");
~~~

Keep the same key type and spelling when writing and reading a map.

## 3. Load the sass:map module

Use the built-in map module with @use:

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033
);

$brand-color: map.get($colors, "brand");
~~~

The map namespace makes each operation clear. The module form is the modern Sass approach.

## 4. Look up a value with map.get()

map.get($map, $key) returns the value associated with a key. If the map does not contain that key, it returns null.

~~~scss
@use "sass:map";

$font-weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

$medium-weight: map.get($font-weights, "medium");
$missing-weight: map.get($font-weights, "extra-bold");

@debug $medium-weight; // 500
@debug $missing-weight; // null
~~~

A missing key does not automatically provide a fallback. Decide how your stylesheet should handle it.

## 5. Check for a key with map.has-key()

map.has-key() returns true when the map contains the requested key and false otherwise.

~~~scss
@use "sass:map";

$breakpoints: (
  "compact": 30rem,
  "wide": 72rem
);

@debug map.has-key($breakpoints, "wide"); // true
@debug map.has-key($breakpoints, "tablet"); // false
~~~

Use this when a missing value should trigger a fallback or a useful Sass error.

## 6. Provide a fallback for missing keys

A small helper function can return a fallback value when a key is missing.

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033
);

@function color-token($name, $fallback: null) {
  @if map.has-key($colors, $name) {
    @return map.get($colors, $name);
  }

  @return $fallback;
}

.card {
  color: color-token("text", #333333);
  border-color: color-token("border", #d0d5dd);
}
~~~

The first declaration uses the text token. The second uses its fallback because border is not in the map.

If a missing token is always a mistake, prefer reporting an @error instead of silently falling back.

## 7. Read values from nested maps

A map can contain another map. This is useful for organizing related tokens into groups.

~~~scss
$theme: (
  "colors": (
    "brand": #2457c5,
    "surface": #ffffff,
    "text": #172033
  ),
  "spacing": (
    "small": 0.5rem,
    "large": 1.5rem
  )
);
~~~

map.get() can follow a sequence of keys through nested maps:

~~~scss
@use "sass:map";

$brand-color: map.get($theme, "colors", "brand");
$large-spacing: map.get($theme, "spacing", "large");
$missing-color: map.get($theme, "colors", "border");

@debug $brand-color; // #2457c5
@debug $large-spacing; // 1.5rem
@debug $missing-color; // null
~~~

If an intermediate key is missing or does not point to a map, the lookup returns null.

## 8. Check a nested path with map.has-key()

map.has-key() can also check a nested key path.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033
  )
);

@debug map.has-key($theme, "colors", "brand"); // true
@debug map.has-key($theme, "colors", "border"); // false
@debug map.has-key($theme, "spacing", "large"); // false
~~~

Use a nested check before retrieving a value when the path may not exist.

## 9. Iterate over all key/value pairs

@each assigns the key and value of each map entry to variables.

~~~scss
$colors: (
  "brand": #2457c5,
  "success": #147d4b,
  "danger": #b42318
);

@each $name, $color in $colors {
  .text-#{$name} {
    color: $color;
  }
}
~~~

The loop emits .text-brand, .text-success, and .text-danger. Interpolation is used for the selector name; the color value is assigned directly.

Maps are an effective way to generate related rules from a set of named tokens.

## 10. Add or replace one entry with map.set()

map.set() returns a new map with a key added or its existing value replaced.

~~~scss
@use "sass:map";

$weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

$extended-weights: map.set($weights, "extra-bold", 800);
$updated-weights: map.set($weights, "bold", 750);

@debug $weights;
// ("regular": 400, "medium": 500, "bold": 700)

@debug $extended-weights;
// ("regular": 400, "medium": 500, "bold": 700, "extra-bold": 800)

@debug $updated-weights;
// ("regular": 400, "medium": 500, "bold": 750)
~~~

The original $weights map is unchanged.

## 11. Merge two maps with map.merge()

map.merge() combines two maps into a new map. When both maps have the same key, the value from the second map wins.

~~~scss
@use "sass:map";

$base-colors: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);

$dark-overrides: (
  "text": #f5f7fb,
  "surface": #172033
);

$dark-colors: map.merge($base-colors, $dark-overrides);

@debug $dark-colors;
// ("brand": #2457c5, "text": #f5f7fb, "surface": #172033)
~~~

The brand color stays from the base map. Text and surface are replaced by the values from the override map.

## 12. Remove entries with map.remove()

map.remove() returns a new map without the listed keys. Missing keys are ignored.

~~~scss
@use "sass:map";

$weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

$without-medium: map.remove($weights, "medium");
$without-two: map.remove($weights, "regular", "bold");

@debug $without-medium; // ("regular": 400, "bold": 700)
@debug $without-two; // ("medium": 500)
~~~

The original map still contains all three entries.

## 13. Inspect map keys and values

map.keys() returns a comma-separated list of keys. map.values() returns a comma-separated list of values.

~~~scss
@use "sass:map";

$font-weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

@debug map.keys($font-weights); // "regular", "medium", "bold"
@debug map.values($font-weights); // 400, 500, 700
~~~

These are lists, so list functions can inspect them when needed. Prefer the map module for map operations and the list module when you need to work with the returned list.

## 14. Merge nested maps carefully

map.merge() is shallow: a matching nested map value from the second map replaces the first nested map value as a whole. It does not merge the nested entries.

~~~scss
@use "sass:map";

$base-theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033
  )
);

$dark-overrides: (
  "colors": (
    "text": #f5f7fb
  )
);

$shallow-result: map.merge($base-theme, $dark-overrides);

@debug $shallow-result;
// The colors map now contains only "text".
~~~

Use map.deep-merge() when nested maps should merge recursively:

~~~scss
@use "sass:map";

$deep-result: map.deep-merge($base-theme, $dark-overrides);

@debug $deep-result;
// The colors map contains both "brand" and the overridden "text".
~~~

Deep merge is available in Dart Sass. Check your Sass implementation if a project uses a legacy compiler.

## 15. Update a nested map with map.set()

map.set() can follow nested keys and return a new map with a nested value added or replaced.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033
  )
);

$theme: map.set($theme, "colors", "brand", #1745a5);

@debug map.get($theme, "colors", "brand"); // #1745a5
~~~

Reassigning the returned map makes the updated value available in $theme.

## 16. Use maps for design tokens

A nested token map groups colors, spacing, and typography in one configuration value.

~~~scss
$tokens: (
  "color": (
    "brand": #2457c5,
    "text": #172033,
    "surface": #ffffff
  ),
  "spacing": (
    "small": 0.5rem,
    "medium": 1rem,
    "large": 1.5rem
  ),
  "type": (
    "body": 1rem,
    "heading": 2rem
  )
);
~~~

A helper function can keep token lookup consistent:

~~~scss
@use "sass:map";

@function token($group, $name) {
  @if not map.has-key($tokens, $group, $name) {
    @error "Unknown token: #{$group}.#{$name}.";
  }

  @return map.get($tokens, $group, $name);
}

.page-title {
  color: token("color", "text");
  font-size: token("type", "heading");
  margin-block-end: token("spacing", "large");
}
~~~

This approach centralizes values and makes a typo fail during compilation with a clear message.

## 17. Maps and lists

Sass maps are also list-like: each map entry can be viewed as a two-item key/value list. This is why some list functions can operate on maps. However, choose the data structure that expresses the task:

- Use a map for lookup by a meaningful key.
- Use a list for ordered values accessed by position or iterated as a sequence.

For example, a font stack is naturally a list, while named breakpoints are naturally a map.

## 18. Maps are immutable

Map functions return new maps; they do not modify the original value. Assign the result when you want to retain an update.

~~~scss
@use "sass:map";

$base: ("brand": #2457c5);
$with-text: map.set($base, "text", #172033);

@debug $base; // ("brand": #2457c5)
@debug $with-text; // ("brand": #2457c5, "text": #172033)
~~~

This lets a library create derived configurations without changing the input map supplied by its caller.

## 19. Common mistakes

### Using unquoted color names as keys

An unquoted color name may be parsed as a color value. Quote string keys consistently.

### Treating map.get() as a guaranteed lookup

A missing key returns null. Use map.has-key() when you need to check first, then choose a fallback or raise an error.

### Forgetting to save the returned map

map.set(), map.remove(), map.merge(), and map.deep-merge() all return new maps. Reassign the result if you need the update.

### Expecting map.merge() to merge nested maps

map.merge() is shallow. Use map.deep-merge() when you need recursive merging.

### Using a list for named configuration

If code asks for a value by name, a map is clearer and safer than relying on a list's position.

### Trying to emit a map directly as CSS

A map is Sass data, not a CSS declaration value. Retrieve the needed value or loop over entries to generate CSS.

## 20. Practice checklist

Before moving on, make sure you can answer these questions:

1. How is a Sass map written?
2. Why are quoted string keys recommended?
3. What does map.get() return for a missing key?
4. How can you validate a nested key path?
5. How can @each iterate over map entries?
6. What happens when map.set() uses an existing key?
7. Which map wins when map.merge() sees duplicate keys?
8. How does map.deep-merge() differ from map.merge()?
9. Why must you reassign the result of a map function?
10. When should you use a map instead of a list?

## Further reading

- [Sass maps](https://sass-lang.com/documentation/values/maps/)
- [sass:map module](https://sass-lang.com/documentation/modules/map/)
- [Sass @each rule](https://sass-lang.com/documentation/at-rules/control/each/)
