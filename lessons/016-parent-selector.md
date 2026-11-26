# 016 — The Parent Selector

## Learning goals

After this lesson, you should be able to:

- Explain what the parent selector means inside a nested Sass rule.
- Use it to add a pseudo-class or pseudo-element to the current selector.
- Add a selector before the current parent to express contextual styling.
- Build BEM-style class names by appending a suffix.
- Understand how parent selector lists are expanded.
- Recognize where a parent selector is valid and where a suffix cannot be appended.
- Use the parent selector as a SassScript expression at a basic level.
- Avoid confusing Sass parent selector behavior with native CSS nesting.

## 1. What the parent selector represents

The ampersand character, &, is Sass's parent selector. Inside a nested style rule, it refers to the selector of the surrounding rule.

A nested selector without & normally gets a descendant relationship. With &, Sass substitutes the complete parent selector where the ampersand appears.

~~~scss
.button {
  &:hover {
    background-color: #1849a9;
  }
}
~~~

Generated CSS:

~~~css
.button:hover {
  background-color: #1849a9;
}
~~~

The parent selector is not a Sass variable. It is a special selector token that Sass resolves while compiling nested rules.

## 2. Attach states to the parent

Use &: when the state belongs to the parent element itself.

~~~scss
.button {
  background-color: #175cd3;

  &:hover {
    background-color: #1849a9;
  }

  &:focus-visible {
    outline: 3px solid #84caff;
    outline-offset: 3px;
  }

  &:disabled {
    cursor: not-allowed;
    opacity: 0.6;
  }
}
~~~

Generated CSS:

~~~css
.button {
  background-color: #175cd3;
}
.button:hover {
  background-color: #1849a9;
}
.button:focus-visible {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
.button:disabled {
  cursor: not-allowed;
  opacity: 0.6;
}
~~~

The output selectors all match the button itself in the specified state.

Without the ampersand, a nested :hover would be a descendant selector. For example, :hover nested under .button would target a hovered descendant inside the button, not the button itself.

## 3. Add pseudo-elements and attribute conditions

The parent selector can be followed by a pseudo-element or another selector condition that belongs to the same element.

~~~scss
.external-link {
  &::after {
    content: " ↗";
  }

  &[target="_blank"] {
    text-decoration-style: dotted;
  }
}
~~~

Generated CSS:

~~~css
.external-link::after {
  content: " ↗";
}
.external-link[target="_blank"] {
  text-decoration-style: dotted;
}
~~~

Both nested selectors reuse the parent selector and add another condition to that same element.

## 4. Place the parent inside a contextual selector

The parent selector can appear after another selector. This is useful when the current component changes in a surrounding context.

~~~scss
.card {
  color: #202124;

  [data-theme="dark"] & {
    color: #f2f4f7;
    background-color: #101828;
  }
}
~~~

Generated CSS:

~~~css
.card {
  color: #202124;
}
[data-theme="dark"] .card {
  color: #f2f4f7;
  background-color: #101828;
}
~~~

The context selector appears before the parent in the generated selector. The rule matches a .card inside an element marked with data-theme="dark".

A theme class is another common context:

~~~scss
.button {
  background-color: #175cd3;

  .theme-dark & {
    background-color: #528bff;
  }
}
~~~

Generated selector:

~~~css
.theme-dark .button {
  background-color: #528bff;
}
~~~

This is different from &:hover. A state selector modifies the parent selector; a context selector places another selector before it.

## 5. Build BEM-style names with a suffix

Sass can append a suffix directly to the end of a parent selector. This is useful for naming related classes with BEM-style element and modifier names.

~~~scss
.product-card {
  &__title {
    font-size: 1.25rem;
  }

  &__image {
    display: block;
    width: 100%;
  }

  &--featured {
    border-color: #175cd3;
  }
}
~~~

Generated CSS:

~~~css
.product-card__title {
  font-size: 1.25rem;
}
.product-card__image {
  display: block;
  width: 100%;
}
.product-card--featured {
  border-color: #175cd3;
}
~~~

The ampersand and suffix form a single class name. They do not create a descendant selector.

This works when the outer selector ends in a name that can take the suffix, such as a class, ID, or element name. It is not a general string append operation for every possible selector shape.

### Avoid accidental concatenation

~~~scss
.nav {
  &__item {
    display: block;
  }

  &--compact {
    gap: 0.5rem;
  }
}
~~~

This produces .nav__item and .nav--compact. If the intention is to select a descendant with a separate class, write a normal nested selector instead:

~~~scss
.nav {
  .item {
    display: block;
  }
}
~~~

This produces .nav .item. Choose the form that matches the actual class names and markup.

## 6. Combine the parent with combinators

The parent selector can be used on either side of a combinator.

~~~scss
.list-item {
  & + & {
    border-block-start: 1px solid #d0d5dd;
  }

  & > .list-item__icon {
    margin-inline-end: 0.5rem;
  }
}
~~~

Generated CSS:

~~~css
.list-item + .list-item {
  border-block-start: 1px solid #d0d5dd;
}
.list-item > .list-item__icon {
  margin-inline-end: 0.5rem;
}
~~~

The first selector matches a list item that immediately follows another list item. The second selects a direct child icon element inside the parent.

Two ampersands in one selector each refer to the parent selector. This pattern can describe repeated siblings, but check the generated selector to confirm it matches the intended structure.

## 7. Parent selector lists

If the parent rule has multiple selectors, Sass substitutes each selector where & appears.

~~~scss
.notice,
.warning {
  &:hover {
    cursor: pointer;
  }
}
~~~

Generated CSS:

~~~css
.notice:hover,
.warning:hover {
  cursor: pointer;
}
~~~

A suffix is also applied to every parent selector that can accept it:

~~~scss
.card,
.panel {
  &__title {
    font-weight: 700;
  }
}
~~~

Generated CSS:

~~~css
.card__title,
.panel__title {
  font-weight: 700;
}
~~~

When a complex parent selector list is combined with other selectors, inspect the generated output. Selector lists can affect matching and specificity in ways that are easy to miss when rules are deeply nested.

## 8. Use the parent inside selector functions

The parent selector can be placed inside a selector-taking pseudo-class such as :not().

~~~scss
.avatar {
  :not(&) {
    opacity: 0.7;
  }
}
~~~

Generated CSS:

~~~css
:not(.avatar) {
  opacity: 0.7;
}
~~~

This selector means an element that does not match .avatar. It is not a descendant of .avatar. Selector functions can change the relationship in non-obvious ways, so inspect the output and make sure the rule is scoped safely.

A simpler state rule should usually use &: directly:

~~~scss
.avatar {
  &:hover {
    opacity: 0.85;
  }
}
~~~

## 9. Parent selector in SassScript

The parent selector can also be used as a SassScript expression, for example when inspecting or combining selectors with the sass:selector module. Inside a style rule, it represents the current selector list.

~~~scss
.card {
  current-selector: &;
}
~~~

This produces a declaration whose value contains the current selector. Such declarations are mainly useful in advanced Sass APIs and are not ordinary styling patterns.

Outside a style rule, the parent selector expression evaluates to null. A mixin can use this fact to behave differently depending on whether it is included inside another style rule. Keep this technique for reusable library code; ordinary component styles rarely need it.

## 10. The parent selector outside a nested rule

Modern Dart Sass supports writing & outside a nested rule as plain CSS for the browser to interpret. Dart Sass added this support in version 1.100.0.

~~~scss
& {
  color: inherit;
}
~~~

This compiles as a CSS rule containing &, where the browser interprets & as the scoping root for native CSS nesting. There is no parent Sass selector to append a suffix to in this case.

This syntax is distinct from the common Sass use of & inside a nested rule. Confirm browser support and project Sass version before relying on native CSS nesting.

## 11. Sass and native CSS nesting differ

Sass substitutes the parent selector during compilation. Native CSS nesting is interpreted by the browser. Their syntax overlaps, but their behavior is not identical.

One notable difference is a suffix such as &__title. Sass appends the suffix to form a BEM class name. Native CSS interprets the ampersand as a CSS nesting selector; it does not provide the same Sass suffix behavior.

Sass and native CSS can also differ when a parent selector list is nested, including how selector specificity is calculated. Treat SCSS and CSS as separate languages when moving nested rules between files.

## 12. Common mistakes

### Mistake: leaving out the ampersand for a parent state

~~~scss
.button {
  :focus-visible {
    outline: 3px solid #84caff;
  }
}
~~~

This targets a focused descendant, not the button itself.

Use:

~~~scss
.button {
  &:focus-visible {
    outline: 3px solid #84caff;
  }
}
~~~

### Mistake: using a suffix when a descendant was intended

~~~scss
.card {
  & .title {
    font-weight: 700;
  }
}
~~~

This is valid and produces .card .title: the space makes .title a descendant. Without the space, &__title would form the single class .card__title.

### Mistake: appending a suffix to a selector that cannot accept it

Suffix appending is not valid for every parent selector. Use a separate nested selector or an explicit class when the selector does not end in a name that can take the suffix.

### Mistake: assuming context is implicit

A rule written as .theme-dark & means the parent is inside .theme-dark. It does not mean the parent itself has the theme class. To add a class to the same element, use a compound selector such as &.theme-dark.

### Mistake: treating the parent selector as text

The ampersand represents the parsed parent selector. Sass understands selector structure, so the result is not always equivalent to string concatenation. For class-name suffixes, use only the supported suffix pattern.

### Mistake: assuming Sass and browser nesting are interchangeable

A selector such as &__item is Sass-specific suffixing. Native CSS nesting has different parsing and semantics. Compile SCSS before serving it as CSS.

## 13. Practical component example

Source SCSS:

~~~scss
.alert {
  padding: 1rem;
  border: 1px solid #d0d5dd;

  &__title {
    margin-block: 0 0.5rem;
    font-weight: 700;
  }

  &__message {
    margin: 0;
  }

  &--success {
    border-color: #12b76a;

    .alert__title {
      color: #027a48;
    }
  }

  &:focus-within {
    outline: 3px solid #84caff;
    outline-offset: 3px;
  }

  [data-density="compact"] & {
    padding: 0.5rem;
  }
}
~~~

Generated CSS:

~~~css
.alert {
  padding: 1rem;
  border: 1px solid #d0d5dd;
}
.alert__title {
  margin-block: 0 0.5rem;
  font-weight: 700;
}
.alert__message {
  margin: 0;
}
.alert--success {
  border-color: #12b76a;
}
.alert--success .alert__title {
  color: #027a48;
}
.alert:focus-within {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
[data-density="compact"] .alert {
  padding: 0.5rem;
}
~~~

This example uses the parent selector for BEM class names, a modifier, a state, and an external context. In a production stylesheet, verify that each selector matches the markup and does not create unnecessary specificity.

## 14. Exercises

1. Add hover and focus-visible styles to .button using the parent selector.
2. Write a nested selector that places .theme-dark before .panel.
3. Build .menu__item and .menu__item--current from a .menu parent.
4. Make .row + .row rules from one .row parent using two ampersands.
5. Given .card, .panel as parent selectors, predict the output of &:hover and &__title.
6. Explain the difference between & .title and &__title.
7. Compile the alert example and check every generated selector against its source.
8. Write one example where omitting & would accidentally target a descendant instead of the parent.

## Summary

- Inside a nested rule, & refers to the complete parent selector and is substituted in its position.
- Use &:hover, &:focus-visible, and &:disabled to add a state to the parent itself.
- Place another selector before & to express a context, such as [data-theme="dark"] .card.
- Append a suffix such as __title to create a BEM-style class name when the parent can accept it.
- Use & with combinators when the parent participates in a child or sibling relationship.
- Sass's parent selector is resolved during compilation and does not behave exactly like native CSS nesting.
- Read the generated selector to verify its meaning and specificity.

## Official references

- [Sass parent selector](https://sass-lang.com/documentation/style-rules/parent-selector/)
- [Sass style rules and selector nesting](https://sass-lang.com/documentation/style-rules/)
- [Sass and native CSS nesting](https://sass-lang.com/blog/sass-and-native-nesting/)
- [Sass interpolation](https://sass-lang.com/documentation/interpolation/)
- [Sass selector module](https://sass-lang.com/documentation/modules/selector/)
