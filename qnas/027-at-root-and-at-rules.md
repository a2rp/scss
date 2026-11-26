# 027 - @at-root and Nested At-Rules: Q&A

## Core behavior

### 1. What does it mean for a nested at-rule to bubble?

Sass moves the at-rule outside the surrounding style rule in the generated CSS and places that style rule inside the at-rule.

~~~scss
.card {
  @media (min-width: 48rem) {
    padding: 1.5rem;
  }
}
~~~

The output is structurally equivalent to:

~~~css
@media (min-width: 48rem) {
  .card {
    padding: 1.5rem;
  }
}
~~~

### 2. Does nesting @media put a media rule inside a CSS style rule?

No. Sass source can nest it for organization, but Sass emits a top-level media rule containing the applicable selector.

### 3. Can @supports be nested in a component rule?

Yes. Sass bubbles the `@supports` rule and puts the component selector inside it. This is useful for colocating a progressive enhancement with its fallback.

### 4. Can other CSS at-rules be nested?

Sass has general support for CSS at-rules, but each at-rule has its own grammar. Some accept declarations or style rules; others expect special children. For example, `@keyframes` expects `from`, `to`, or percentage keyframe selectors. Follow the at-rule's CSS grammar.

### 5. Does Sass guarantee that every browser supports the emitted at-rule or feature?

No. Sass compiles stylesheet syntax. Browser support for the resulting CSS feature is a separate question.

## Media queries

### 6. What happens when a media rule is nested inside another media rule?

Sass can combine compatible nested media conditions in the output. For example, a width condition nested inside a hover-capability condition can become a combined condition joined with `and`.

### 7. Does Sass combine every possible media query?

No. Only compatible queries can be combined. Do not assume a particular transformation for arbitrary query syntax; inspect the compiled CSS for the compiler version and query you use.

### 8. Should every responsive rule be nested in its component?

No. Nest a condition beside a component when it is a component-specific variation. A root-level media query can be clearer when it coordinates several unrelated components or groups a page-wide breakpoint.

### 9. Is a nested media rule always easier to maintain?

Not necessarily. Nesting can make local styles easier to locate, but deeply nested or cross-component conditions can hide the final selector and media conditions. Keep the source structure readable.

### 10. Will modern range syntax work in every browser?

No. Sass can pass CSS media-query syntax through, but browser support depends on the browsers you target. Verify the CSS feature's compatibility separately.

## @at-root

### 11. What does @at-root do?

It emits its contents outside the current style-rule nesting. It is useful when a selector is already complete and should not be nested under the surrounding selectors.

### 12. Does @at-root remove all surrounding context?

By default, it removes enclosing style rules. It leaves enclosing at-rules such as `@media` and `@supports` in place.

### 13. How can I leave an enclosing @media rule too?

Use an `@at-root` query such as `(without: media)`:

~~~scss
@media print {
  .report {
    @at-root (without: media) {
      color: #111;
    }
  }
}
~~~

This moves the generated `.report` rule out of the media context as well as out of the style-rule context.

### 14. What does @at-root (with: rule) mean?

It keeps style rules and excludes enclosing at-rules. Here, `rule` is Sass's special name for a style rule.

### 15. What does @at-root (without: rule) mean?

It removes style rules while leaving applicable at-rules in place. This matches the default treatment of style rules, but naming the query can make the intention explicit.

### 16. What does @at-root (without: all) mean?

It excludes all enclosing style rules and at-rules. Use it only when the output should leave all of those contexts.

### 17. What is the difference between with and without queries?

A `without` query names rule types to remove. A `with` query names the rule types to retain. For example, `without: media` removes media rules, while `with: rule` keeps style rules and excludes at-rules.

### 18. Is @at-root .selector { ... } different from @at-root { .selector { ... } }?

The selector form is shorthand for a block-form `@at-root` containing that selector rule. Use the block form when several rules should leave the current style-rule nesting together.

### 19. Does @at-root add the parent selector back automatically?

No. It removes the relevant nesting context. Write the full selector you want or deliberately build one with the parent selector and selector functions.

### 20. Why use @at-root with selector interpolation?

Sass performs selector nesting based on its syntax tree. It cannot infer that interpolated text already includes the parent selector. `@at-root` prevents Sass from nesting that generated selector a second time.

## Selector generation and mixins

### 21. What does selector.unify() do in the lesson's mixin?

The `sass:selector` module's `selector.unify()` combines the current selector with another selector when a selector matching both can be formed. If they cannot be unified, the result can be `null`; robust reusable code may need to account for that possibility.

### 22. Why is & passed to selector.unify()?

Inside a style rule, `&` as a SassScript expression represents the current selector. Passing it to a selector function allows Sass to combine it with another selector as selector data rather than relying on textual guesses.

### 23. What is @content doing in the mixin example?

It inserts the declarations supplied by the `@include` block at the mixin's content position. In the example, the mixin generates a selector and yields the included declarations inside that selector.

### 24. Should @at-root be used for ordinary component styles?

Usually no. Prefer regular nesting when it already produces the desired selector. `@at-root` is a deliberate escape from normal selector nesting and should be reserved for cases where that escape is needed.

## Troubleshooting

### 25. Why did my rule stay inside @media after plain @at-root?

Plain `@at-root` removes style-rule nesting by default but preserves enclosing at-rules. Add a query such as `(without: media)` if the rule must leave the media context.

### 26. Why did my selector lose its component ancestor?

`@at-root` removed the parent style-rule context. Include the ancestor in the selector explicitly, or build the selector from `&` with the selector module when that is the intended relationship.

### 27. Why does the compiled selector contain the parent twice?

The interpolated selector may already include the parent, while Sass still applies normal nesting. Emit that generated selector at the root with `@at-root` so normal nesting does not append the parent again.

### 28. Why did nested CSS at-rules produce an unexpected selector or query?

Check the full source nesting and the generated CSS. Each style rule contributes selector context; each at-rule can contribute conditional context. Then simplify the nesting or move cross-component grouping to the root.

### 29. Why is an @keyframes child invalid?

Keyframes do not use ordinary component selectors as their child rule names. Use valid keyframe selectors such as `from`, `to`, or a percentage.

### 30. How should I review a complex @at-root result?

Compile the stylesheet and verify both the selector and the at-rule context. Check that the intended parent selector remains or is removed, and confirm that media or feature-query conditions are preserved as required.

## Quick reference

| Source construct | Main effect |
| --- | --- |
| Nested `@media` | Bubbles the media rule and places the nested selector inside |
| Nested `@supports` | Bubbles the feature query and places the nested selector inside |
| Plain `@at-root` | Removes enclosing style rules, preserves enclosing at-rules |
| `@at-root (without: media)` | Also removes enclosing media rules |
| `@at-root (with: rule)` | Keeps style rules and excludes at-rules |
| `@at-root (without: all)` | Removes all enclosing style rules and at-rules |
| Interpolated selector plus `@at-root` | Emits the generated selector without automatic parent nesting |
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="026-extend-placeholder-selectors.md">026 - @extend and Placeholder Selectors: Q&amp;A</a></td><td><a href="../lessons/027-at-root-and-at-rules.md">Lesson</a></td><td><a href="../examples/027-at-root-and-at-rules.md">Example</a></td><td><a href="028-module-system-with-use.md">028 - The Module System with @use: Q&amp;A</a></td></tr>
  </tbody>
</table>
