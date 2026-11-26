# 011 — Lists and List Functions: Q&A

## List fundamentals

### 1. What is a Sass list?

A Sass list is an ordered sequence of values. A list can hold numbers, strings, colors, other lists, and other Sass values. Lists are used both as CSS property values and as data for Sass logic.

### 2. Do Sass lists need brackets?

No. Space-separated and comma-separated expressions are lists even without parentheses or square brackets. Parentheses can group or nest values; square brackets create a bracketed list when the brackets are part of the intended value.

### 3. What separators can a Sass list use?

A list can be space-separated, comma-separated, or slash-separated. Use the separator expected by the CSS property or function you are writing.

### 4. When should I use a space-separated list?

Use a space-separated list when CSS expects values separated by spaces, such as padding shorthand or a group of grid track values.

### 5. When should I use a comma-separated list?

Use a comma-separated list when CSS expects alternatives or multiple entries, such as a font stack or multiple box shadows.

### 6. What do parentheses do in a list?

Parentheses group an expression and can create nested lists. They are especially useful when an inner comma-separated list is one item inside another list, or when passing a comma-separated list as a single function argument.

### 7. What does a square-bracketed list do?

Square brackets mark a list as bracketed, and that bracketed structure is preserved when Sass serializes the value. This can be useful for CSS syntax that uses brackets, such as grid line names.

### 8. Can a list contain another list?

Yes. Sass lists can contain nested lists. Use parentheses to make the nesting clear, especially when both the inner and outer lists use commas.

### 9. Can a Sass list have only one item or no items?

Yes. Single-value inputs are already treated as one-item lists by list functions. Explicit one-item lists can be represented with a trailing comma or square brackets. Empty lists can be written as () or [], but an unbracketed empty list is not a valid CSS property value.

### 10. Why does a comma-separated list sometimes cause an argument error?

Commas separate function arguments in Sass. To pass a comma-separated list as one argument, wrap it in parentheses:

~~~scss
@use "sass:list";

$fonts: ("Inter", "Segoe UI", sans-serif);
$second-font: list.nth($fonts, 2);
~~~

The parentheses group the full font list as one argument to nth().

## Indexing and inspecting

### 11. Are Sass list indexes zero-based?

No. Sass indexes start at 1. Index 1 is the first item, and index 2 is the second item.

### 12. Can I count from the end of a list?

Yes. Negative indexes count from the end. Index -1 is the final item, -2 is the second-to-last item, and so on.

### 13. What does list.nth() do?

list.nth($list, $index) returns the item at the requested one-based or negative index. An index outside the list's range produces an error.

### 14. What does list.length() return?

It returns the number of items in a list. It also counts a single Sass value as a one-item list and can count the entries in a map.

### 15. What does list.index() return when it finds no match?

It returns null. When a value appears multiple times, it returns the index of the first occurrence. Since null is falsey, list.index() can be used in an @if condition.

### 16. What does list.separator() report?

It returns the list's separator name: space, comma, or slash. If a value has no explicit separator, it reports space.

### 17. What does list.is-bracketed() report?

It returns true when the list has square brackets and false when it does not.

### 18. Can I pass a number, string, or color to a list function?

Yes. Sass treats an individual value as a one-item list for list functions. For example, list.length(1rem) returns 1.

## Iteration and list operations

### 19. How does @each iterate over a list?

@each assigns each list item to a variable and runs the block once per item:

~~~scss
$sizes: small, medium, large;

@each $size in $sizes {
  .size-#{$size} {
    display: block;
  }
}
~~~

The loop creates one rule for each size.

### 20. Can @each unpack nested list values into multiple variables?

Yes. If each outer list item is itself a list, declare a variable for each inner value:

~~~scss
$themes: (light, white, black), (dark, black, white);

@each $name, $background, $foreground in $themes {
  .theme-#{$name} {
    background-color: $background;
    color: $foreground;
  }
}
~~~

Each inner item supplies a name, background, and foreground value.

### 21. What is the difference between list.append() and list.join()?

list.append() adds one value to the end of a list. If that value is itself a list, it remains one nested item.

list.join() combines the elements of two lists into a single list. Use append() for one value and join() for two lists.

### 22. Why shouldn't I use list.join() to append a list as one item?

join() flattens the second list into the result. If the value should remain nested, use append().

### 23. What does list.set-nth() do?

It returns a copy of a list with the item at a given index replaced. It does not modify the original list.

### 24. What does list.zip() do?

It combines items at matching positions from multiple lists into nested lists. It is useful for pairing parallel values, such as breakpoint names and widths.

### 25. Do list functions mutate the original list?

No. Sass lists are immutable. List functions return new values. Assign the result if you want to use the updated list.

### 26. How do I control the separator in a new list?

Functions such as append() and join() accept a separator option. Supported values include comma, space, and slash. If omitted, the function usually derives the result separator from its input list or uses the documented default.

### 27. Can list.join() control whether its result has brackets?

Yes. Its bracketed option can request a bracketed or unbracketed result. By default, it follows the first list's bracketed state.

## CSS syntax and edge cases

### 28. How do I make a slash-separated CSS value?

Use list.slash() from sass:list. Sass historically parsed slash as division, so list.slash() clearly expresses a slash-separated CSS list.

### 29. When should I use math.div() instead of list.slash()?

Use math.div() when the slash means arithmetic division. Use list.slash() when the slash is part of the CSS value syntax, such as a font-size/line-height pair.

### 30. Are a map and a list the same thing?

No. A map associates keys with values and should normally be accessed with sass:map functions. Sass also treats each map entry as a two-item list for some list operations, but use a map when the code is organized around named keys.

### 31. Why use a list for a font stack instead of separate variables?

A font stack is naturally an ordered, comma-separated group of fallback choices. Keeping it in one list makes it easy to pass to a property or share across styles.

### 32. Can a list have nested values with different separators?

Yes. Each nested list has its own separator and bracketed state. Parentheses help keep the nested structure explicit.

### 33. How should I validate a value before generating a property from it?

Keep an allowed-values list and check it with list.index(). If the returned value is null, report an @error. This prevents accidental generation of invalid property names.

### 34. Why do Sass list indexes differ from JavaScript arrays?

Sass uses one-based indexing to align with CSS and Sass list conventions. Remember that the first item is index 1, not index 0.

### 35. How can I debug an unexpected list value?

Use @debug with the value and, when useful, inspect list.length(), list.separator(), list.is-bracketed(), or list.index(). These checks reveal whether Sass parsed the separators and nesting as intended.

## Quick checks

### 36. What is the first item in 1rem 2rem 3rem?

list.nth(1rem 2rem 3rem, 1) returns 1rem.

### 37. What is the last item in red, green, blue?

list.nth((red, green, blue), -1) returns blue.

### 38. What does list.append((a, b), (c, d)) preserve?

It returns a comma-separated outer list whose final item is the nested list (c, d).

### 39. What does list.join((a, b), (c, d)) return?

It combines the elements into one comma-separated list: a, b, c, d.

### 40. What is the key rule for list functions?

Know the list's separator, nesting, and brackets. Use the proper index, choose append() or join() based on whether the new value should stay nested, and assign a function's returned value when you need the updated list.
