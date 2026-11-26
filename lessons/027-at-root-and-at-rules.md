# 027 — @at-root and Nested At-Rules

## Learning goals

After this lesson, you should be able to:

- Explain how Sass bubbles nested CSS at-rules into valid top-level CSS.
- Read the selector and at-rule structure produced by nested `@media` and `@supports` rules.
- Use `@at-root` to emit styles outside the current selector context.
- Control which style rules and at-rules `@at-root` removes with `with` and `without` queries.
- Use `@at-root` with the parent selector and selector functions when generating selectors.
- Choose nesting that keeps the source clear and the generated CSS predictable.

## 1. Why nested at-rules are useful

A component's base styles and its conditional styles often belong together in the source. Sass lets you nest an at-rule such as `@media` inside the component rule:

~~~scss
.card {
  padding: 1rem;

  @media (min-width: 48rem) {
    padding: 1.5rem;
  }
}
~~~

The browser does not receive a media rule inside a style rule. Sass moves, or **bubbles**, the at-rule outward and places the component selector inside it:

~~~css
.card {
  padding: 1rem;
}

@media (min-width: 48rem) {
  .card {
    padding: 1.5rem;
  }
}
~~~

Nesting is a source-organization feature. The compiled CSS still needs valid CSS rule structure.

## 2. Nested @media rules

A nested media rule applies its declarations to the selector being written around it. Sass also combines compatible nested media conditions when it can.

~~~scss
.panel {
  color: #263238;

  @media (width >= 40rem) {
    padding: 1.5rem;

    @media (orientation: landscape) {
      max-width: 70rem;
    }
  }
}
~~~

The nested conditions are represented as a combined media condition in the output, conceptually:

~~~css
@media (width >= 40rem) {
  .panel {
    padding: 1.5rem;
  }
}

@media (width >= 40rem) and (orientation: landscape) {
  .panel {
    max-width: 70rem;
  }
}
~~~

Use nesting when a condition is a variation of the current component. Put a media rule at the root when it coordinates many unrelated selectors or when that makes the stylesheet easier to scan.

Media-query range syntax and other CSS features depend on the target browsers and Sass compiler version. Sass passes CSS features through; it does not make an unsupported browser feature work.

## 3. Nested @supports rules

`@supports` follows the same bubbling model. It can keep a progressive enhancement beside a component's fallback:

~~~scss
.sticky-banner {
  position: fixed;
  inset-block-start: 0;

  @supports (position: sticky) {
    position: sticky;
  }
}
~~~

The generated CSS keeps the feature query at the top level:

~~~css
.sticky-banner {
  position: fixed;
  inset-block-start: 0;
}

@supports (position: sticky) {
  .sticky-banner {
    position: sticky;
  }
}
~~~

The declaration order is intentional: browsers that do not support the enhancement retain the fallback. Make sure the supported declaration actually expresses the behavior you want; `position: sticky` also needs a suitable scrolling container and inset threshold to become sticky.

## 4. Other at-rules and their contents

Sass supports CSS at-rules generally, including `@font-face`, `@keyframes`, and future at-rule syntax. Many at-rules can be nested and bubbled, but their child grammar is not always an ordinary style rule.

For example, keyframes contain keyframe selectors such as `from`, percentages, and `to`:

~~~scss
@keyframes fade-in {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}
~~~

Do not put ordinary component selectors where a special at-rule expects a different kind of child. Check the CSS rule's grammar when nesting unfamiliar at-rules. At-rule-name interpolation is supported by modern Dart Sass; older compilers may not support it.

## 5. What @at-root does

`@at-root` emits its contents outside the current style-rule nesting. It is useful when the desired output selector has already been constructed and should not receive the surrounding selector again.

~~~scss
.theme-dark {
  .button {
    @at-root .button--contrast {
      color: white;
      background: #172033;
    }
  }
}
~~~

The output selector is `.button--contrast`, not `.theme-dark .button .button--contrast`:

~~~css
.button--contrast {
  color: white;
  background: #172033;
}
~~~

The selector after `@at-root` is still emitted as written. `@at-root` removes the enclosing style-rule context; it does not automatically preserve or recreate the ancestors you leave.

A block form can emit several root-level style rules:

~~~scss
.component {
  @at-root {
    .component__icon { inline-size: 1em; }
    .component__label { font-weight: 600; }
  }
}
~~~

The selector form is shorthand for placing a style rule inside an `@at-root { ... }` block.

## 6. @at-root with selector interpolation

A common advanced use is to combine the current selector with another selector. Sass cannot infer from interpolated selector text that the generated selector already contains the parent. Without `@at-root`, normal nesting may add that parent again.

Use the `sass:selector` module's `selector.unify()` to combine selectors deliberately:

~~~scss
@use "sass:selector";

@mixin with-input-type($type) {
  @at-root #{selector.unify(&, $type)} {
    @content;
  }
}

.form-field {
  @include with-input-type("input") {
    border: 1px solid #64748b;
  }

  @include with-input-type("select") {
    border: 1px solid #64748b;
  }
}
~~~

Here `&` is the current selector value passed to the selector function. The result is emitted at the root, so Sass does not append `.form-field` a second time. The content block supplies declarations for each generated selector.

This pattern belongs in reusable selector-generating abstractions, not routine styling. If a plain nested rule expresses the result, prefer the simpler rule.

## 7. Controlling what @at-root removes

By default, `@at-root` removes enclosing **style rules**, while leaving enclosing at-rules such as `@media` in place.

~~~scss
@media print {
  .report {
    @at-root {
      color: #111;
    }
  }
}
~~~

This emits the `.report` rule outside the style-rule nesting but still inside `@media print`. To exclude a particular at-rule too, use a query:

~~~scss
@media print {
  .report {
    @at-root (without: media) {
      color: #111;
    }
  }
}
~~~

The resulting `.report` rule is outside the print media rule.

The query can also select what remains:

- `@at-root (with: rule)` keeps style rules and excludes at-rules.
- `@at-root (without: rule)` removes style rules and keeps at-rules.
- `@at-root (without: media)` removes media rules while preserving other applicable context.
- `@at-root (without: all)` removes all enclosing style rules and at-rules.

The special name `rule` means a style rule, and `all` means all style rules and at-rules. Use these queries only when you can explain which context the output must keep.

## 8. A practical decision process

Before adding `@at-root`, ask:

1. Is ordinary nesting producing an unwanted ancestor selector?
2. Is the intended selector already fully formed, perhaps by interpolation or a selector function?
3. Should the result stay inside a surrounding `@media`, `@supports`, or another at-rule?
4. Would moving the rule to the stylesheet root make its relationship to the component harder to understand?
5. Can a simpler root-level rule or mixin avoid this special output behavior?

Before deeply nesting at-rules, ask whether the source is easier to maintain with one root-level query that groups related selectors. Read the compiled CSS when nesting affects selector scope, media conditions, or source order.

## 9. Common mistakes

### Assuming every enclosing rule is removed

Plain `@at-root` removes style rules by default, but preserves at-rules. Add a `with` or `without` query only if the output should leave or retain a particular at-rule.

### Losing a selector unintentionally

When `@at-root` removes the parent context, it does not add that parent back. Write the full intended selector or deliberately combine it with `&` using selector functions.

### Duplicating the parent selector

Interpolation can create a selector that contains the parent, but Sass may still perform ordinary nesting. Put the generated selector at the root when that extra nesting is not wanted.

### Nesting without a clear component relationship

Deeply nested conditions make emitted CSS harder to predict and source order harder to inspect. Keep nesting shallow and group global or cross-component conditions at the root.

### Assuming Sass changes browser support

Sass transforms the stylesheet structure. The browser still decides whether it understands the emitted CSS feature or condition.

## 10. Summary

- Nested CSS at-rules bubble outward, with their nested style rules placed inside them.
- Compatible nested media conditions may be combined in compiled CSS.
- `@at-root` removes style-rule nesting by default but keeps surrounding at-rules.
- `with` and `without` queries control which kinds of enclosing rules remain.
- Selector interpolation paired with `@at-root` is useful for advanced selector generation.
- Inspect the compiled CSS whenever nesting or `@at-root` changes selector scope or conditional context.

## Further reading

- [Sass: @at-root](https://sass-lang.com/documentation/at-rules/at-root/)
- [Sass: CSS At-Rules](https://sass-lang.com/documentation/at-rules/css/)
- [Sass: Parent Selector](https://sass-lang.com/documentation/style-rules/parent-selector/)
- [Sass: sass:selector module](https://sass-lang.com/documentation/modules/selector/)
