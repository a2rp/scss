# 012 — Maps and Map Functions: Q&A

## Map fundamentals

### 1. What is a Sass map?

A map is a collection of key/value pairs. Each key identifies one associated value, making maps useful for named configuration such as colors, spacing tokens, breakpoints, and font weights.

### 2. How do I write a Sass map?

Use parentheses and separate each key/value pair with a colon. Separate pairs with commas:

~~~scss
$weights: (
  "regular": 400,
  "bold": 700
);
~~~

An empty map is written as ().

### 3. Must map keys be unique?

Yes. A map can't contain two equal keys as distinct entries. Its values may be duplicated.

### 4. Can a key or value be any Sass type?

Yes. Sass maps can use Sass values as keys and values. In ordinary design systems, quoted string keys are usually easier to understand and safer to look up.

### 5. Why are quoted string keys recommended?

An unquoted word may be parsed as a different Sass type. For example, red is a color value. Writing "red" makes it a string key and avoids confusion between the text name and the color.

### 6. Is an empty map the same as an empty list?

The syntax () is both an empty map and an empty list. Its meaning depends on how it is used. Maps otherwise require key/value pairs and parentheses.

### 7. Can I write a map directly as a CSS property value?

No. A map is a Sass data structure, not a valid CSS value by itself. Use map.get() to retrieve a value or @each to create CSS from its entries.

### 8. How do I load map functions?

Use the built-in module:

~~~scss
@use "sass:map";

$color: map.get(("brand": royalblue), "brand");
~~~

The map namespace makes it clear which built-in functions operate on maps.

## Lookups and validation

### 9. What does map.get() do?

map.get($map, $key) returns the value associated with the key. If the key does not exist, it returns null.

### 10. Does map.get() accept a default value?

No. Its arguments are the map and key, with additional keys available for nested lookups. If you need a fallback, check with map.has-key() and choose the fallback yourself.

### 11. What does map.has-key() do?

It returns true if the requested key exists and false otherwise. It can also check a nested path by passing multiple keys.

### 12. How do I look up a nested value?

Pass each key in order to map.get():

~~~scss
@use "sass:map";

$theme: ("colors": ("brand": royalblue));
$brand: map.get($theme, "colors", "brand");
~~~

This returns royalblue.

### 13. How do I check whether a nested path exists?

Pass the full path to map.has-key():

~~~scss
@use "sass:map";

$exists: map.has-key($theme, "colors", "brand");
~~~

It returns false if a path key is missing or an intermediate value is not a map.

### 14. What should I do when a required key is missing?

Choose deliberately: provide a documented fallback, or raise an @error with the invalid key. For design tokens, reporting an error often catches typos during compilation.

### 15. Can map.get() return a real null value as well as indicate a missing key?

Yes. A map may associate a key with null, so a null result alone may not distinguish a missing key from a present key whose value is null. Use map.has-key() when that distinction matters.

### 16. How can I make a helper function for token lookups?

Use a shared map, check the key path with map.has-key(), raise an @error when it is absent, then return the value from map.get(). This centralizes validation and avoids repeating lookup rules.

## Iteration and inspection

### 17. How does @each iterate over a map?

Use two loop variables: the first receives the key and the second receives the value.

~~~scss
@each $name, $color in $colors {
  .text-#{$name} {
    color: $color;
  }
}
~~~

The block runs once for each map entry.

### 18. What does map.keys() return?

It returns a comma-separated list containing all keys in the map.

### 19. What does map.values() return?

It returns a comma-separated list containing all values in the map.

### 20. How can I find how many entries a map has?

Sass maps are also list-like, so list.length($map) can count the entries. If the code is about named lookups or map configuration, keep using map functions for the operation itself.

### 21. Are maps and lists interchangeable?

No. A map represents key/value associations; a list represents an ordered sequence. Sass treats each map entry as a two-item list for some list operations, but use a map for lookups by name and a list for sequence or positional work.

## Updating and merging

### 22. What does map.set() do?

It returns a new map with the requested key set to a value. If the key already exists, its value is replaced. If it does not exist, the pair is added.

### 23. Can map.set() update a nested value?

Yes. Pass the nested key path followed by the final key and new value. The function returns an updated copy with the nested entry changed.

### 24. What does map.remove() do?

It returns a new map without one or more requested keys. Missing keys are ignored.

### 25. What does map.merge() do?

It combines two maps into a new map. Keys present in only one map are retained. When both maps contain the same key, the value from the second map takes precedence.

### 26. Is map.merge() recursive?

No. It is shallow. If both maps have the same key whose values are maps, the second nested map replaces the first nested map as a whole.

### 27. How is map.deep-merge() different?

map.deep-merge() recursively merges nested maps. It keeps nested entries that occur only in one input and applies values from the second map when the same nested key exists. It is supported by Dart Sass.

### 28. What is the difference between map.set() and map.merge()?

Use map.set() to add or replace one key/value pair. Use map.merge() to combine two maps. Use map.deep-merge() when nested map values should be recursively combined.

### 29. Do map functions change the original map?

No. Sass maps are immutable. Functions such as map.set(), map.remove(), and map.merge() return new maps. Assign the result if you want to use the updated version.

### 30. What happens if I call map.set() without assigning its result?

The original variable still refers to the original map. The returned map is discarded, so the update is not retained.

## Common use cases and debugging

### 31. Should breakpoints be a list or a map?

If you refer to breakpoints by names such as compact or wide, use a map. If you only need to process an ordered sequence of values, a list may be more appropriate.

### 32. How can I use a map to generate utility classes?

Use @each to receive each name and value, interpolate the name into the selector, and use the value directly in the declaration.

### 33. Why does a map key lookup fail even though the text looks the same?

The key may have a different Sass type, spelling, or case. An unquoted word such as red can be a color instead of a string. Prefer consistently quoted string keys and look them up with the same quoted spelling.

### 34. How can I inspect a map while developing?

Use @debug to print the map, or inspect map.keys(), map.values(), map.get(), and map.has-key() for focused checks. Debug output is compiler output, not emitted CSS.

### 35. Can maps contain maps?

Yes. Nested maps are common for themes and design-system tokens. Use multiple keys with map.get() and map.has-key() to traverse them.

### 36. When should I choose map.deep-merge()?

Choose it when each map may contain nested maps and your intended behavior is to preserve nested entries from both inputs while allowing the later map to override matching keys. Use map.merge() when replacing an entire value at a top-level key is the intended behavior.

## Quick checks

### 37. What does map.get($weights, "medium") return if medium maps to 500?

It returns 500.

### 38. What does map.get() return for an absent key?

It returns null. Use map.has-key() if you need to distinguish absence from a key whose value is null.

### 39. Which value wins in map.merge() when both maps have the same key?

The value from the second map wins.

### 40. Which function adds one new entry to a map?

Use map.set(). Save its returned map if the updated value is needed later.

### 41. Which function combines two maps recursively?

Use map.deep-merge() in Dart Sass.

### 42. What is the main rule for maps?

Use maps for named key/value data. Quote string keys consistently, check missing or nested keys intentionally, and remember that map functions return new values instead of changing the original map.
