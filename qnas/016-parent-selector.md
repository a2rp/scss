# 016 - The Parent Selector: Q&A

## The basic meaning of &

### 1. What is the Sass parent selector?

The ampersand, &, is a special selector that refers to the selector of the surrounding style rule. Sass replaces it with that parent selector while compiling nested rules.

### 2. Is & a Sass variable?

No. It is a special selector token, not a variable declaration. It can also be used as a SassScript expression in advanced Sass code.

### 3. What is the difference between a nested selector with & and one without it?

A nested selector without & normally becomes a descendant selector. A nested selector with & inserts the complete parent selector at the position of the ampersand.

### 4. How do I add hover styling to the parent itself?

Use &:hover inside the parent rule. It compiles to the parent selector followed by :hover.

### 5. What does a plain :hover nested inside .button select?

It selects a hovered descendant inside .button. It does not select .button when the button itself is hovered.

### 6. How do I attach a pseudo-element to the parent?

Use a nested selector such as &::before or &::after. Sass substitutes the parent before the pseudo-element.

### 7. Can I attach an attribute condition to the parent?

Yes. A selector such as &[disabled] combines the parent and attribute selector on the same element.

### 8. Does &:hover add a descendant relationship?

No. There is no space between the parent selector and :hover, so the pseudo-class applies to the same element.

## Context selectors and combinators

### 9. What does [data-theme="dark"] & mean inside .card?

It compiles to [data-theme="dark"] .card. It matches a .card descendant inside an element whose data-theme attribute is dark.

### 10. How is .theme-dark & different from &.theme-dark?

.theme-dark & places .theme-dark before the parent, so the parent is a descendant. &.theme-dark puts both selectors on the same element.

### 11. Can & be used with sibling combinators?

Yes. A selector such as & + & can match one parent-styled element immediately following another element with the same selector.

### 12. What does & > .icon select?

It selects an element with class icon that is a direct child of the parent selector.

### 13. Can & appear inside a selector function such as :not()?

Yes. For example, :not(&) substitutes the parent selector inside :not(). The resulting selector may have a different relationship than a descendant selector, so inspect it carefully.

## Suffixes and BEM naming

### 14. How do I create a BEM element class from a parent class?

Inside a rule such as .card, write &__title. Sass appends the suffix to produce the single class .card__title.

### 15. Does &__title create a child element selector?

No. There is no whitespace, so the suffix becomes part of the same class name. Use a nested .title selector or & .title when a descendant is intended.

### 16. What does &--featured create?

Inside .card, it creates the class .card--featured. This is a class-name suffix, not a descendant selector.

### 17. Can I append a suffix to any selector?

No. Sass can append a suffix when the parent selector ends in a name that can accept it, such as a class, ID, or element name. Some selector shapes cannot take a suffix.

### 18. Why is span& invalid?

The parent selector can be replaced by an element selector, so Sass only permits it where an element selector could appear at the beginning of a compound selector. Putting a type selector before it, as in span&, is not allowed.

### 19. Can a comma-separated parent list receive a suffix?

Sass applies the suffix to each parent selector that can accept it. For example, .card, .panel with &__title generates .card__title and .panel__title.

### 20. Is Sass suffixing the same as string interpolation?

No. The parent selector is parsed as a selector. Sass suffixing follows selector rules and is not unrestricted text concatenation.

## Selector lists and SassScript

### 21. What happens when the parent contains multiple selectors and I write &:hover?

Sass substitutes each parent selector, so each selector gets its own hovered form in the generated selector list.

### 22. What value does & represent in SassScript inside a style rule?

It represents the current parent selector list in Sass's selector value format. This is mainly useful for advanced reusable Sass code.

### 23. What does & evaluate to outside a style rule in SassScript?

It evaluates to null because there is no current parent selector. A mixin can use this to detect whether it is included from inside a style rule.

### 24. Should I use & as a SassScript value in ordinary component styles?

Usually not. Common component styles only need selector nesting. SassScript use of & is more suitable for mixins and selector-function utilities.

### 25. Can Sass resolve & outside a nested rule?

Modern Dart Sass supports & outside a nested rule as native CSS nesting syntax, starting with Dart Sass 1.100.0. In this case it is emitted as CSS for the browser, not replaced with a Sass parent.

### 26. Can I append a BEM suffix to a root-level &?

No. Outside a Sass nested rule, there is no parent Sass selector to receive the suffix. The emitted ampersand is for the browser's native CSS nesting behavior.

## Compatibility and maintainability

### 27. Is Sass's parent selector the same as the ampersand in native CSS nesting?

They share familiar syntax but are not equivalent in every case. Sass resolves its selector during compilation, and Sass suffixing such as &__item does not have the same meaning in native CSS.

### 28. Why should I inspect CSS generated from complex parent lists?

The parent may contain several selectors, and inserting it into another selector can produce multiple results. The final selectors and their specificity can be harder to predict from deeply nested source.

### 29. Does using & automatically make a selector more specific?

No. Sass substitutes selectors into the output. The resulting CSS selector's specificity follows normal CSS selector rules.

### 30. How do I check whether an & rule is correct?

Read the generated selector, identify the element or state it matches, then compare that with the markup. Compile it if the parent list, combinator, or selector function makes the result unclear.

### 31. What is a common mistake when moving Sass code into a CSS file?

Assuming Sass suffix behavior is native CSS behavior. A pattern such as &__item is Sass-specific suffixing; compile SCSS with Sass or rewrite the CSS rule deliberately.

### 32. When should I prefer a separate selector over a complex & expression?

Prefer a direct selector when it is easier to understand, reuse, and match to the markup. Use & when it clearly expresses a state, context, suffix, or relationship of the current component.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="015-nesting-rules-and-declarations.md">015 - Nesting Rules and Declarations: Q&amp;A</a></td><td><a href="../lessons/016-parent-selector.md">Lesson</a></td><td><a href="../examples/016-parent-selector.md">Example</a></td><td><a href="017-interpolation-in-selectors-and-property-names.md">017 - Interpolation in Selectors and Property Names: Q&amp;A</a></td></tr>
  </tbody>
</table>
