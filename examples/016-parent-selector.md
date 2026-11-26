# 016 - The Parent Selector: Examples

These examples show how Dart Sass substitutes the parent selector while compiling nested SCSS.

## 1. Add a state to the parent

Source SCSS:

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

Each pseudo-class applies to the button itself.

## 2. Compare a parent state with a descendant state

Source SCSS:

~~~scss
.button {
  :hover {
    text-decoration: underline;
  }

  &:hover {
    background-color: #1849a9;
  }
}
~~~

Generated CSS:

~~~css
.button :hover {
  text-decoration: underline;
}
.button:hover {
  background-color: #1849a9;
}
~~~

The selector with a space targets a hovered descendant. The selector with & targets the hovered button.

## 3. Attach a pseudo-element and attribute condition

Source SCSS:

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

The parent and the condition apply to the same element in both rules.

## 4. Put the parent in a context selector

Source SCSS:

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

The card must be inside an element with data-theme="dark".

## 5. Compare context with a class on the same element

Source SCSS:

~~~scss
.panel {
  .theme-dark & {
    color: white;
  }

  &.theme-dark {
    background-color: #101828;
  }
}
~~~

Generated CSS:

~~~css
.theme-dark .panel {
  color: white;
}
.panel.theme-dark {
  background-color: #101828;
}
~~~

In the first selector, .theme-dark is an ancestor of .panel. In the second, both classes belong to the same element.

## 6. Build BEM element and modifier classes

Source SCSS:

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

The suffix is part of the class name. It does not add a descendant relationship.

## 7. Compare a BEM suffix with a descendant class

Source SCSS:

~~~scss
.menu {
  &__item {
    display: block;
  }

  .icon {
    inline-size: 1rem;
  }

  & .help-text {
    color: #667085;
  }
}
~~~

Generated CSS:

~~~css
.menu__item {
  display: block;
}
.menu .icon {
  inline-size: 1rem;
}
.menu .help-text {
  color: #667085;
}
~~~

.menu__item is one class name. .menu .icon and .menu .help-text are descendant selectors.

## 8. Use the parent with a sibling combinator

Source SCSS:

~~~scss
.list-item {
  & + & {
    border-block-start: 1px solid #d0d5dd;
  }
}
~~~

Generated CSS:

~~~css
.list-item + .list-item {
  border-block-start: 1px solid #d0d5dd;
}
~~~

This matches a .list-item immediately following another .list-item under the same parent.

## 9. Use a selector list

Source SCSS:

~~~scss
.notice,
.warning {
  &:hover,
  &:focus-visible {
    cursor: pointer;
  }
}
~~~

Generated CSS:

~~~css
.notice:hover,
.notice:focus-visible,
.warning:hover,
.warning:focus-visible {
  cursor: pointer;
}
~~~

Each parent selector is combined with each nested state selector. Check the expanded list when a parent rule contains many selectors.

## 10. Use & inside :not() carefully

Source SCSS:

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

This selector matches elements that are not .avatar; it is not limited to descendants of .avatar. It can therefore affect a broad set of elements. Use this pattern only when that broad matching is intentional.

## 11. Inspect the parent selector as a SassScript value

Source SCSS:

~~~scss
.card {
  parent-selector: &;
}
~~~

Generated CSS:

~~~css
.card {
  parent-selector: .card;
}
~~~

This illustrates that & can represent the current selector as a SassScript expression. The declaration is for learning; parent-selector is not a standard CSS property.

## 12. Parent selector outside a nested Sass rule

Modern Dart Sass 1.100.0 and later can emit & outside a nested Sass rule for the browser's native CSS nesting behavior.

Source SCSS:

~~~scss
& {
  color: inherit;
}
~~~

Emitted CSS:

~~~css
& {
  color: inherit;
}
~~~

There is no outer Sass selector to substitute. This requires browser support for native CSS nesting, so check project browser targets before using it.

## 13. Compile a small file

Save this source as src/parent-selector.scss:

~~~scss
.control {
  &:focus-visible {
    outline: 3px solid #84caff;
  }

  &__label {
    font-weight: 600;
  }

  .theme-dark & {
    color: white;
  }
}
~~~

Compile it with Dart Sass:

~~~sh
sass src/parent-selector.scss dist/parent-selector.css
~~~

Expected CSS:

~~~css
.control:focus-visible {
  outline: 3px solid #84caff;
}
.control__label {
  font-weight: 600;
}
.theme-dark .control {
  color: white;
}
~~~

Check the generated selector in each case: same-element state, appended class name, and ancestor context.

## 14. Practice tasks

1. Add hover, focus-visible, and disabled rules to .button.
2. Create .menu__item and .menu--expanded from a .menu parent.
3. Put .theme-dark before the parent selector for a .panel.
4. Create a direct-child selector for .icon inside .card using &.
5. Make .row + .row from one .row rule.
6. Given .notice and .warning parents, predict the selector list produced by &:hover, &:focus-visible.
7. Explain why :hover and &:hover compile to different relationships inside .button.
8. Compile a rule using :not(&) and describe exactly which elements it can match.
9. Compare a Sass suffix such as &__title with the corresponding native CSS nesting interpretation.
10. For every exercise, read the generated CSS and compare it with the intended markup.

## Output review checklist

- Does each state apply to the parent or to a descendant as intended?
- Does a context selector appear before the parent or on the same element as intended?
- Does a suffix form one class name rather than a descendant selector?
- Does using & with a selector list create all the expected combinations?
- Could a selector function such as :not() make the rule broader than expected?
- Does a root-level & rely on native CSS nesting support?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="015-nesting-rules-and-declarations.md">015 - Nesting Rules and Declarations: Examples</a></td><td><a href="../lessons/016-parent-selector.md">Lesson</a></td><td><a href="../qnas/016-parent-selector.md">Q&amp;A</a></td><td><a href="017-interpolation-in-selectors-and-property-names.md">017 - Interpolation in Selectors and Property Names: Examples</a></td></tr>
  </tbody>
</table>
