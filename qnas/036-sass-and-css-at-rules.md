# 036 - Sass with CSS At-Rules: Q&A

## At-rule basics

### 1. What is a CSS at-rule?

A CSS at-rule is a statement that begins with `@`, such as `@media`, `@supports`, `@font-face`, or `@layer`. It can have a value, a block, or both.

### 2. Does Sass understand every CSS at-rule's meaning?

No. Sass has special behavior for some rules, such as media-query nesting, and general support for CSS at-rules it does not interpret specially. General support lets Sass emit CSS features it does not need to understand.

### 3. Which at-rules are Sass instructions rather than CSS output?

Rules such as `@use`, `@forward`, `@mixin`, `@include`, `@function`, and Sass flow-control rules are evaluated by Sass. They do not appear as those Sass instructions in the final CSS.

### 4. Does Sass make a browser support an at-rule?

No. Sass compiles the stylesheet. The browser decides whether it supports and applies the emitted CSS feature.

## Nesting and bubbling

### 5. What does it mean when a CSS at-rule bubbles?

When an at-rule is nested inside a style rule, Sass can move the at-rule to the top level of the generated CSS and place the selector inside the at-rule.

~~~scss
.card {
  @media (min-width: 48rem) {
    padding: 1.5rem;
  }
}
~~~

This becomes a top-level media rule containing the .card selector.

### 6. Why is nesting an at-rule inside a selector useful?

It keeps a conditional style near the selector it changes in the Sass source, while Sass emits the CSS structure browsers expect.

### 7. Are all at-rules automatically moved to the top level?

No. Sass preserves CSS structures according to the rule's syntax and its nesting behavior. General at-rules can remain nested when the CSS syntax allows it. Review the generated CSS for the exact result.

### 8. Can at-rules be nested inside other at-rules?

Yes, where the CSS grammar permits it. For example, a supports condition can be placed inside a media query to require both conditions.

### 9. Does nesting change the browser meaning of a condition?

Sass compiles the source into CSS conditions. The browser evaluates those conditions. Confirm that the generated query expresses the intended logic.

## Media queries

### 10. Can I use Sass variables inside a media query?

Yes. The @media rule supports SassScript in feature queries:

~~~scss
$wide: 68rem;

@media (min-width: $wide) {
  .page {
    max-width: 72rem;
  }
}
~~~

Sass writes the variable's compile-time value into the output.

### 11. Do I need interpolation to use a Sass variable in a media feature?

Usually not. If the media-query position accepts a SassScript expression, use the variable directly. Interpolation is for inserting text where the grammar requires it, such as a dynamic identifier.

### 12. Can I build a reusable breakpoint mixin?

Yes. A mixin can wrap an @media rule and insert caller-provided styles with @content. The mixin should use the project's breakpoint values consistently and handle unknown keys clearly.

### 13. Does Sass combine nested media queries?

When it can, Sass combines nested media queries to produce a form that works in browsers without native nested-media support. The exact output depends on whether the conditions can be combined. Inspect the generated CSS for complex query logic.

### 14. What does a nested media condition mean?

A style inside both @media screen and @media (min-width: 48rem) requires both conditions.

### 15. Can I use modern range syntax in a media query?

Dart Sass supports modern CSS media-feature range syntax. Sass generally passes CSS syntax through, but browser support depends on the project's target browsers. Use the project's browser policy to choose query syntax.

### 16. Can I use a CSS custom property as a breakpoint in @media?

No. CSS custom properties are resolved in declarations by the browser; they are not generally available as media-query feature values. A Sass variable can generate a literal breakpoint at compile time.

## Supports and container queries

### 17. What is @supports for?

It asks the browser whether a CSS declaration or feature is supported, then applies the enclosed CSS only when the test passes.

### 18. Can a Sass variable appear in a supports declaration query?

Yes. Sass can evaluate SassScript in @supports declaration queries:

~~~scss
$display-mode: grid;

@supports (display: $display-mode) {
  .layout {
    display: $display-mode;
  }
}
~~~

The browser receives the compiled declaration query.

### 19. When should I use @supports versus Sass @if?

Use @supports when the browser should decide at runtime whether a CSS feature is supported. Use Sass @if when Sass should decide at compile time whether to generate the CSS.

### 20. Does Sass evaluate container-query sizes?

No. Sass emits container-query CSS. The browser evaluates the query using the relevant container at runtime.

### 21. Does Sass know whether a container exists?

No. Sass does not inspect the HTML or browser layout. It compiles the CSS declarations; the browser determines which element is the container and whether the query matches.

### 22. Can @media, @supports, and @container be combined?

Yes, by nesting them where valid CSS permits. The nested declaration applies only when all enclosing conditions match. Avoid excessive nesting, and inspect the compiled structure.

## Interpolation and special at-rules

### 23. Where can interpolation be used in CSS at-rules?

Sass interpolation can be used in at-rule names and values, as well as in many other CSS positions. Use it to create dynamic CSS text where the grammar needs it.

### 24. Should I interpolate every Sass variable inside CSS?

No. Use normal Sass expressions where Sass already supports them. Interpolation turns an expression into text, so it can remove useful type and unit checks.

### 25. How do keyframes differ from ordinary selector nesting?

The children of @keyframes are keyframe selectors such as from, to, or percentages. They are not ordinary CSS selectors like .card.

### 26. Can I generate a keyframe name dynamically?

Yes, with interpolation in the at-rule name. Ensure that the generated identifier is valid CSS and that the matching animation-name uses the same value.

### 27. Do repeated mixin calls deduplicate generated keyframes?

No. Sass does not generally deduplicate repeated emitted CSS. If a mixin emits the same named keyframes several times, inspect the compiled result and consider defining shared keyframes once.

### 28. What does @at-root do?

@at-root emits its contents at the root of the document instead of using normal Sass style-rule nesting. It is useful when a selector generated inside a rule should not remain nested beneath that rule.

### 29. Does @at-root remove enclosing @media and @supports rules too?

Not by default. On its own, @at-root removes style-rule nesting. Enclosing at-rules such as @media remain unless an explicit with or without filter says otherwise.

### 30. Why use @at-root with the parent selector?

It can build a selector from the current parent and another selector, then emit the result outside the parent's normal nesting. This helps generate selectors that relate to a component but should not be descendants of it.

### 31. Does @font-face use normal selector nesting?

@font-face contains descriptors such as font-family and src, not ordinary selector rules. Sass can compile values inside the block, but the block must follow the CSS at-rule's grammar.

### 32. Does Sass validate the contents of every unknown at-rule?

Sass handles CSS syntax and Sass features, but it does not act as a full browser validator for every CSS feature or at-rule-specific grammar. Use appropriate CSS tooling and browser targets to validate the output.

## CSS output and common mistakes

### 33. Will nested source always produce nested CSS?

No. Sass may bubble at-rules, combine media conditions, and expand selectors. Source indentation is not a promise about the final CSS shape.

### 34. Does Sass merge all identical rules inside at-rules?

No. Sass may combine nested media conditions when possible, but it does not automatically optimize every duplicate selector or declaration into one rule.

### 35. Should I put fallback styles outside @supports?

If browsers without the feature need a usable presentation, yes. Put the fallback in ordinary CSS and the enhanced declarations in the supports block.

### 36. Should vendor-prefixed at-rules be written by hand?

Only when the project's browser-support strategy requires them. Otherwise, use the project's documented tooling or omit unnecessary prefixes. Hand-written prefixes can become stale or generate duplicate output.

### 37. How do I know what Sass emitted?

Compile the relevant entrypoint using the project's normal build command, then inspect the generated CSS. For a nested rule, confirm the selector, condition, rule order, and fallback behavior.

### 38. Is a CSS at-rule the same thing as a CSS custom property?

No. An at-rule is a CSS statement that starts with @. A custom property is a declaration whose property name starts with --, such as --brand-color: blue.

## Quick reference

~~~scss
$wide: 68rem;

.card {
  padding: 1rem;

  @media (min-width: $wide) {
    padding: 2rem;
  }

  @supports (display: grid) {
    display: grid;
  }
}

@layer components {
  .button {
    border-radius: 0.5rem;
  }
}
~~~

- Use browser conditions such as @media, @supports, and @container for runtime CSS decisions.
- Use Sass flow control for compile-time decisions.
- Use interpolation for dynamic at-rule names and values.
- Use @at-root to escape style-rule nesting intentionally.
- Inspect generated CSS when conditions are nested.

## Related material

- [Lesson: Sass with CSS at-rules](../lessons/036-sass-and-css-at-rules.md)
- [Examples: Sass with CSS at-rules](../examples/036-sass-and-css-at-rules.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="035-migrating-to-sass-modules.md">035 - Migrating to Sass Modules: Q&amp;A</a></td><td><a href="../lessons/036-sass-and-css-at-rules.md">Lesson</a></td><td><a href="../examples/036-sass-and-css-at-rules.md">Example</a></td><td><a href="037-sass-variables-vs-css-custom-properties.md">037 - Sass Variables versus CSS Custom Properties: Q&amp;A</a></td></tr>
  </tbody>
</table>
