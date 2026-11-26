# Maps for design systems — Q&A

This Q&A explains Sass maps as data structures for design tokens, component variants, and generated CSS. The examples use the built-in §sass:map§ module, so load it with §@use "sass:map";§ in each stylesheet that calls its functions.

## 1. What is a Sass map?

A Sass map is a collection of key/value pairs. Each key identifies a value, like a name identifying a design token.

~~~scss
$colors: (
  "brand": #3157d5,
  "text": #182033
);
~~~

Maps are compile-time Sass values. They do not become browser variables unless Sass uses them to emit CSS, such as CSS custom properties.

## 2. How is a map different from a list?

A list is an ordered sequence of values, while a map associates keys with values. Use a list when position is meaningful; use a map when a value should be found by a descriptive key.

~~~scss
$spacing-scale: 0.25rem, 0.5rem, 1rem; // list
$spacing-tokens: ("small": 0.5rem, "large": 1rem); // map
~~~

Sass maps can be iterated over, but their key/value structure is what makes named lookup possible.

## 3. Why should map keys usually be quoted?

Quoting clearly marks a key as a string and avoids ambiguity with Sass values. For example, an unquoted color name such as §red§ can be interpreted as a color rather than the string name §"red"§.

~~~scss
$palette: (
  "red": #c0392b,
  "blue": #3157d5
);
~~~

Use one consistent key style across a token system. Quoted, lowercase, hyphenated names are a readable convention.

## 4. Can a map contain different value types?

Yes. A map can store any Sass value, including colors, numbers, strings, lists, booleans, null, and other maps.

~~~scss
$component: (
  "radius": 0.5rem,
  "padding": 0.75rem 1rem,
  "uppercase": false
);
~~~

Sass permits this, but a map with a consistent purpose and documented shape is easier to use safely.

## 5. How do I read a value from a map?

Load §sass:map§ and use §map.get()§:

~~~scss
@use "sass:map";

$colors: ("brand": #3157d5);
$brand: map.get($colors, "brand");
~~~

The result is the Sass value stored under that key.

## 6. How do I read a value from a nested map?

Pass the map followed by the nested keys:

~~~scss
@use "sass:map";

$tokens: (
  "color": (
    "text": (
      "primary": #182033
    )
  )
);

$primary-text: map.get($tokens, "color", "text", "primary");
~~~

The multi-key lookup form is supported by modern Dart Sass (Dart Sass 1.27.0 or later). Each key is resolved against the map returned by the previous lookup.

## 7. What does map.get() return if a key is missing?

§map.get()§ returns §null§ when the requested key or an intermediate nested map is missing. That can cause a required declaration to be omitted, so validate required tokens rather than relying on the result.

~~~scss
$value: map.get($tokens, "color", "typo");
~~~

For required configuration, use §map.has-key()§ and issue §@error§ with the missing token name.

## 8. How do I check if a key exists?

Use §map.has-key()§. It accepts nested keys as well:

~~~scss
@use "sass:map";

@if map.has-key($tokens, "color", "brand") {
  .link {
    color: map.get($tokens, "color", "brand");
  }
}
~~~

Use this for optional lookups or for implementing a helper that gives a useful error when a required path is absent.

## 9. What are map.keys() and map.values() for?

They return the keys or values of a map as Sass lists. For example, keys can drive a separate loop when a task needs only names:

~~~scss
@use "sass:map";

$names: map.keys($tokens);
~~~

If the task needs each name with its corresponding value, use §@each $key, $value in $map§ directly.

## 10. Are maps mutable?

No. Sass maps are immutable. A map function returns a new value and does not change the original map in place. Keep the result by assigning it:

~~~scss
$colors: map.set($colors, "focus", #9a42c8);
~~~

If the return value is not assigned or passed elsewhere, the update has no effect on the map variable.

## 11. How do I add or replace an entry?

Use §map.set()§:

~~~scss
$colors: map.set($colors, "brand", #2747b8);
~~~

If the key exists, its value is replaced in the returned map. If it does not, the returned map includes the new entry. For a nested map, provide the path keys before the new value:

~~~scss
$tokens: map.set($tokens, "color", "brand", #2747b8);
~~~

## 12. What is the difference between map.merge() and map.deep-merge()?

§map.merge()§ combines entries at the top level. When both inputs contain the same key, the second map replaces the first map’s value for that key. If that value is itself a map, the entire nested map is replaced.

§map.deep-merge()§ recursively combines nested map values. Use it when an override should replace selected nested tokens while retaining other nested entries. It is available in Dart Sass 1.27.0 and later.

~~~scss
$base: ("color": ("brand": blue, "text": black));
$override: ("color": ("brand": navy));

$shallow: map.merge($base, $override);
$deep: map.deep-merge($base, $override);
~~~

Choose the operation based on the intended merge behavior; do not assume a normal merge is recursive.

## 13. How do I remove a map entry?

The §sass:map§ module provides §map.remove()§ to return a map without specified top-level keys, and §map.deep-remove()§ for removing a nested key path. These functions also return new maps, so assign the result if it should be retained.

Check the Sass map module reference for exact signatures and compiler compatibility.

## 14. How do I loop through a map?

Use §@each§ with two variables: the first receives a key and the second its value.

~~~scss
@each $name, $value in $colors {
  // use $name and $value
}
~~~

For a map of maps, an outer loop receives each group and inner map; an inner loop visits that group’s entries.

## 15. How can a map generate CSS custom properties?

Iterate through the map and interpolate its key and value:

~~~scss
:root {
  @each $name, $value in $colors {
    --color-#{$name}: #{$value};
  }
}
~~~

This generates browser-readable CSS properties, such as §--color-brand§. The Sass map itself remains compile-time data.

## 16. Can I generate custom properties from a nested map?

Yes, if each top-level value is itself a map with the expected shape:

~~~scss
:root {
  @each $group, $values in $tokens {
    @each $name, $value in $values {
      --#{$group}-#{$name}: #{$value};
    }
  }
}
~~~

This assumes the map is exactly two levels deep. For more complex shapes, use explicit generation logic or helper functions and validate the structure instead of blindly iterating.

## 17. Do Sass maps automatically create CSS variables?

No. A map is Sass data and disappears after compilation. Use Sass to generate declarations such as §--color-brand: #3157d5;§ if the browser needs a runtime value.

CSS custom properties are especially useful for theme switching, inheritance, and values changed by selectors or JavaScript.

## 18. Can I use Sass maps to switch themes at runtime?

Not directly. Sass chooses and compiles map values before the page runs. To switch themes in the browser, emit CSS custom properties for each theme and select them with a class, data attribute, media query, or another CSS condition.

Sass maps can still be the source data used to generate those theme rules.

## 19. What is a semantic design token?

A semantic token names a value by its intended role, such as §"text-muted"§, §"surface"§, or §"action-primary"§. A primitive token describes a raw value, such as a palette entry §"blue-600"§.

Semantic names reduce the number of components that need changes when the visual palette changes. A system may have both layers and map semantic roles to primitive values.

## 20. Should every token be in one giant map?

Usually not. Group related tokens, keep the public API understandable, and avoid nesting so deeply that lookups are hard to follow. A token map should have documented groups and a predictable shape.

Separate maps or Sass modules can be appropriate for colors, typography, component configuration, and themes if those areas evolve independently.

## 21. How can I make a token lookup fail with a useful message?

Write a Sass function that validates the path, then returns the value:

~~~scss
@use "sass:map";

@function token($group, $name) {
  @if not map.has-key($tokens, $group, $name) {
    @error "Unknown token #{$group}.#{$name}.";
  }

  @return map.get($tokens, $group, $name);
}
~~~

Then use the returned value in a declaration. A Sass function returns a Sass value; it should not be used to emit CSS rules.

## 22. When should I use @error, @warn, or a null result?

Use §@error§ when compilation cannot safely continue, such as when a required token is missing. Use §@warn§ for a recoverable situation where compilation can continue but the author should review something. Use §null§ for a deliberate optional value only when callers handle it clearly.

Do not use a silent null result for a required design token: it can hide a typo.

## 23. What happens if a CSS declaration has a null value?

Sass omits declarations whose value is §null§. That behavior is convenient for optional output, but it can also make a missing map lookup disappear without an obvious error. Explicitly check optional values and validate required ones.

~~~scss
@if $value != null {
  box-shadow: $value;
}
~~~

## 24. Is false the same as null?

No. §false§ is a boolean value; §null§ represents the absence of a value. A component option such as §"uppercase": false§ can be a meaningful setting. Do not treat every false value as missing.

When testing an optional token for absence, compare it to §null§ or check whether the key exists.

## 25. What if the map has a key whose value is null?

The key exists, so §map.has-key()§ can report it as present, while §map.get()§ returns §null§ as its value. Decide whether a null entry means “intentionally unset” or indicates invalid configuration, and validate accordingly.

Key presence and value usability are separate questions.

## 26. Can map keys be numbers or other values?

Sass map keys can be Sass values, but descriptive string keys are generally the clearest choice for design tokens. Non-string keys can make iteration, interpolation, and maintenance less predictable.

Use stable quoted names unless there is a specific reason for a different key type.

## 27. Can two map keys have the same value?

Yes. Keys identify entries; values do not need to be unique. Multiple semantic tokens may intentionally point to the same primitive color or size. Keeping separate semantic names allows those roles to change independently later.

Avoid duplicate copies of the same token under multiple names if they represent the same role and should always change together.

## 28. Can a Sass map contain another map?

Yes. That is how nested token groups and component variants are represented. Define the shape clearly and validate it before running loops or nested lookups.

If nested maps become difficult to inspect, simplify the structure or split it into smaller maps.

## 29. Why can an empty map be confusing?

The empty map syntax §()§ is also interpreted as an empty list in some Sass contexts. Non-empty maps have key/value structure, but code that only checks an empty value’s type may not distinguish the two as expected.

Do not rely on an empty map to signal a distinct configuration state unless the surrounding API makes that state clear.

## 30. How do maps work with Sass modules and @use?

A module can define and expose token maps. Another stylesheet loads it with §@use "tokens"§ and accesses public members through the module namespace. This helps avoid global name collisions and the legacy behavior of §@import§.

Keep module boundaries purposeful. A design-system module should expose documented tokens and helpers rather than every internal implementation detail.

## 31. What does !default do for map configuration?

§!default§ assigns a variable’s default only if it has not already been assigned. It can support configurable Sass modules, but consumers must configure the module when it is first loaded through the Sass module system.

It does not let a running browser mutate the Sass map. Use CSS custom properties for runtime customization.

## 32. Should a Sass function or mixin be used with a map?

Use a function when the result is a Sass value, such as one token value. Use a mixin when the operation should emit CSS declarations or rules. Use a direct §@each§ loop when it is short and clear.

Choose the smallest abstraction that makes the token usage understandable.

## 33. How do I debug the contents of a map?

During development, §@debug $tokens;§ can print a Sass value to the compiler’s debug output. You can also inspect a focused lookup rather than dumping a large token map.

Remove temporary debug statements when they are no longer useful, especially if they clutter normal compilation logs.

## 34. What are common map mistakes?

- Forgetting to assign the result of §map.set()§ or a merge function.
- Assuming §map.merge()§ recursively combines nested maps.
- Using an unquoted color-like key.
- Expecting a missing §map.get()§ lookup to throw an error.
- Iterating a nested map as though every group had the same shape.
- Expecting Sass data to remain available at browser runtime.
- Using a deeply nested map that is harder to maintain than smaller modules.

## 35. How can I choose between a map and CSS custom properties?

Use a Sass map to organize compile-time values, iterate over them, and generate stylesheet output. Use CSS custom properties when values need to inherit, respond to runtime conditions, or be read and changed in the browser.

They can work together: keep authoring tokens in a Sass map and compile them into CSS custom properties.

## Official references

- [Sass maps](https://sass-lang.com/documentation/values/maps/)
- [The built-in §sass:map§ module](https://sass-lang.com/documentation/modules/map/)
- [The §@each§ control rule](https://sass-lang.com/documentation/at-rules/control/each/)
- [Sass functions](https://sass-lang.com/documentation/at-rules/function/)
- [Sass modules and §@use§](https://sass-lang.com/documentation/at-rules/use/)
