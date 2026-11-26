# 012 - Maps and Map Functions: Examples

Each section is a focused Sass example. If you combine examples in one stylesheet, put each @use at the top and include each module only once.

## 1. Define a map of named tokens

A map associates each name with a value.

~~~scss
$colors: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);
~~~

The map is Sass data. It does not emit CSS until a Sass operation uses its values.

## 2. Retrieve individual values

Use map.get() for a named lookup.

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);

$brand-color: map.get($colors, "brand");
$text-color: map.get($colors, "text");

.button {
  background-color: $brand-color;
  color: $text-color;
}
~~~

Compiled CSS:

~~~css
.button {
  background-color: #2457c5;
  color: #172033;
}
~~~

## 3. Check a key and provide a fallback

map.get() returns null for a missing key. Check map.has-key() when you need a fallback.

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033
);

$requested-color: "border";
$fallback-color: #d0d5dd;

$border-color: if(
  map.has-key($colors, $requested-color),
  map.get($colors, $requested-color),
  $fallback-color
);

.card {
  border-color: $border-color;
}
~~~

The result uses the fallback because "border" is not a key in $colors.

## 4. Create a helper that reports unknown tokens

Use a function to apply one consistent lookup rule throughout a stylesheet.

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);

@function color-token($name) {
  @if not map.has-key($colors, $name) {
    @error "Unknown color token: #{$name}.";
  }

  @return map.get($colors, $name);
}

.card {
  color: color-token("text");
  background-color: color-token("surface");
}
~~~

A misspelled token fails during compilation with an explicit error.

## 5. Use quoted string keys

Some unquoted words are interpreted as other Sass values. Quote names to make their type unambiguous.

~~~scss
@use "sass:map";

$labels: (
  "red": "Danger",
  "green": "Success"
);

$danger-label: map.get($labels, "red");

.badge::before {
  content: $danger-label;
}
~~~

The key "red" is text. The map value remains a quoted string for the CSS content property.

## 6. Read from a nested map

Group related values under a parent key.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033,
    "surface": #ffffff
  ),
  "spacing": (
    "small": 0.5rem,
    "medium": 1rem,
    "large": 1.5rem
  )
);

$brand: map.get($theme, "colors", "brand");
$large-gap: map.get($theme, "spacing", "large");

.card {
  border-color: $brand;
  padding: $large-gap;
}
~~~

Nested lookups read one key at a time: first the group, then the token inside it.

## 7. Check that a nested path exists

Use map.has-key() with the entire key path before relying on a nested value.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033
  )
);

@if map.has-key($theme, "colors", "brand") {
  .brand-text {
    color: map.get($theme, "colors", "brand");
  }
}
~~~

The rule is emitted only if that nested key exists.

## 8. Generate classes from a map

Use @each to assign each key and value to a variable.

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

Representative output:

~~~css
.text-brand {
  color: #2457c5;
}

.text-success {
  color: #147d4b;
}

.text-danger {
  color: #b42318;
}
~~~

The key becomes part of the selector, so it uses interpolation. The color remains a typed Sass value in the declaration.

## 9. Generate custom properties from a nested color map

A theme map can generate CSS custom properties for runtime use.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "surface": #ffffff,
    "text": #172033
  )
);

:root {
  @each $name, $value in map.get($theme, "colors") {
    --color-#{$name}: #{$value};
  }
}
~~~

Compiled CSS:

~~~css
:root {
  --color-brand: #2457c5;
  --color-surface: #ffffff;
  --color-text: #172033;
}
~~~

Interpolation inserts the Sass value into the custom property value.

## 10. Add and replace entries with map.set()

map.set() returns an updated map. Save the result in a variable.

~~~scss
@use "sass:map";

$weights: (
  "regular": 400,
  "bold": 700
);

$with-medium: map.set($weights, "medium", 500);
$with-updated-bold: map.set($with-medium, "bold", 750);

@debug $weights;
// ("regular": 400, "bold": 700)

@debug $with-updated-bold;
// ("regular": 400, "bold": 750, "medium": 500)
~~~

The source map remains unchanged.

## 11. Add an entry to an empty map

An empty map is useful as a starting value for building configuration.

~~~scss
@use "sass:map";

$tokens: ();
$tokens: map.set($tokens, "brand", #2457c5);
$tokens: map.set($tokens, "spacing", 1rem);

@debug $tokens;
// ("brand": #2457c5, "spacing": 1rem)
~~~

## 12. Merge base tokens with a theme override

map.merge() combines top-level entries. The override map's values take precedence for duplicate keys.

~~~scss
@use "sass:map";

$base: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);

$dark-overrides: (
  "text": #f5f7fb,
  "surface": #172033
);

$dark-theme: map.merge($base, $dark-overrides);

@debug $dark-theme;
// ("brand": #2457c5, "text": #f5f7fb, "surface": #172033)
~~~

Use the base value when a key is not overridden. The second map wins when both maps have the same key.

## 13. Compare shallow merge and deep merge

map.merge() replaces a matching nested map as a whole. map.deep-merge() combines nested maps recursively.

~~~scss
@use "sass:map";

$base: (
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

$shallow: map.merge($base, $dark-overrides);
$deep: map.deep-merge($base, $dark-overrides);

@debug $shallow;
// "colors" contains only "text".

@debug $deep;
// "colors" contains "brand" and the overridden "text".
~~~

map.deep-merge() is available in Dart Sass. Confirm compiler support when maintaining a project on legacy Sass tooling.

## 14. Remove one or more entries

map.remove() returns a new map without the requested keys.

~~~scss
@use "sass:map";

$weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

$without-medium: map.remove($weights, "medium");
$without-regular-and-bold: map.remove($weights, "regular", "bold");

@debug $without-medium;
// ("regular": 400, "bold": 700)

@debug $without-regular-and-bold;
// ("medium": 500)
~~~

Removing a key that is absent is safe; it is ignored.

## 15. Read all keys or values

map.keys() and map.values() return lists.

~~~scss
@use "sass:map";

$weights: (
  "regular": 400,
  "medium": 500,
  "bold": 700
);

$names: map.keys($weights);
$values: map.values($weights);

@debug $names; // "regular", "medium", "bold"
@debug $values; // 400, 500, 700
~~~

Use these lists when another Sass operation needs the keys or values as a sequence.

## 16. Update a nested value

Pass the nested path followed by the final key and new value to map.set().

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

The returned map contains the updated nested value. The original map value was not modified.

## 17. Remove a nested value

map.deep-remove() removes a key from a nested map and returns a new map.

~~~scss
@use "sass:map";

$theme: (
  "colors": (
    "brand": #2457c5,
    "text": #172033
  )
);

$without-text: map.deep-remove($theme, "colors", "text");

@debug $without-text;
// "colors" contains only "brand".
~~~

Use this when you need to remove an entry below the top level.

## 18. Use a nested map as a design-token source

This example combines nested lookup with a token helper.

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "brand": #2457c5,
    "text": #172033,
    "surface": #ffffff
  ),
  "space": (
    "small": 0.5rem,
    "large": 1.5rem
  ),
  "type": (
    "body": 1rem,
    "heading": 2rem
  )
);

@function token($group, $name) {
  @if not map.has-key($tokens, $group, $name) {
    @error "Unknown token: #{$group}.#{$name}.";
  }

  @return map.get($tokens, $group, $name);
}

.card {
  color: token("color", "text");
  background-color: token("color", "surface");
  padding: token("space", "large");
}

.page-title {
  color: token("color", "brand");
  font-size: token("type", "heading");
}
~~~

A token typo is reported when Sass compiles, instead of silently producing a missing value.

## 19. Practice tasks

1. Create a map for three named colors and apply the brand value to a button.
2. Add a key and replace an existing key with map.set().
3. Check whether a spacing token exists before using it.
4. Create a nested theme map and look up its text color.
5. Iterate over the theme's colors to generate custom properties.
6. Merge a dark theme override with a base map.
7. Compare a shallow merge with a deep merge for nested colors.
8. Remove a deprecated token.
9. Use map.keys() and map.values() to inspect a map.

### Sample answers

~~~scss
@use "sass:map";

$colors: (
  "brand": #2457c5,
  "text": #172033,
  "surface": #ffffff
);

$extended-colors: map.set($colors, "danger", #b42318);
$updated-colors: map.set($extended-colors, "brand", #1745a5);

@if map.has-key($updated-colors, "brand") {
  .button {
    background-color: map.get($updated-colors, "brand");
  }
}

$theme: (
  "colors": $updated-colors,
  "spacing": (
    "large": 1.5rem
  )
);

$text-color: map.get($theme, "colors", "text");

:root {
  @each $name, $value in map.get($theme, "colors") {
    --color-#{$name}: #{$value};
  }
}

$dark: map.merge($updated-colors, ("text": #f5f7fb));
$without-surface: map.remove($dark, "surface");
$color-names: map.keys($without-surface);
$color-values: map.values($without-surface);

@debug $text-color;
@debug $color-names;
@debug $color-values;
~~~

The button uses the updated brand color. The generated root variables contain each color from the updated map.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="011-lists-and-list-functions.md">011 - Lists and List Functions: Examples</a></td><td><a href="../lessons/012-maps-and-map-functions.md">Lesson</a></td><td><a href="../qnas/012-maps-and-map-functions.md">Q&amp;A</a></td><td><a href="013-colors-and-sass-color-module.md">013 - Colors and the Sass Color Module: Examples</a></td></tr>
  </tbody>
</table>
