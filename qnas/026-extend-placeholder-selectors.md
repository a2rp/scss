# 026 — @extend and Placeholder Selectors: Q&A

## Core behavior

### 1. What does @extend do?

**@extend** tells Sass that the selector where it appears should also match rules written for another selector. Sass updates the relevant selector lists in the compiled CSS.

### 2. Does @extend copy declarations to the current location?

No. A mixin inserts declarations at each include site. **@extend** updates selectors on existing rules that use the extended selector.

### 3. Does the HTML need both classes after one class extends another?

Usually no. If **.message--warning** extends **.message**, an element with **message--warning** receives the styles from **.message** as though it also matched that base class.

### 4. How does Sass represent shared declarations after an extend?

Sass typically groups the original selector and the extender in a selector list for the shared declarations.

~~~scss
.message {
  padding: 1rem;
}

.message--warning {
  @extend .message;
}
~~~

The output includes a rule similar to:

~~~css
.message,
.message--warning {
  padding: 1rem;
}
~~~

### 5. Does the location of @extend determine where the shared declarations appear?

No. The extended rule remains at its original place in the stylesheet. Sass adds the extending selector to that rule. This means cascade order follows the original rule's position, which can matter when inspecting conflicting declarations.

### 6. Does @extend always create a simple selector list?

Not necessarily. Sass performs selector unification to preserve which elements should match the rules. It may generate combined selectors in complex situations, so inspect compiled CSS when the result is not obvious.

## Placeholder selectors

### 7. What is a placeholder selector?

A placeholder selector begins with **%** instead of a dot or hash, such as **%control-base**. It can be extended, but is not emitted in CSS by itself.

### 8. Why use a placeholder instead of an ordinary class?

A placeholder represents Sass-only shared styles without creating a class name that someone might try to put in HTML. If it is not extended, its placeholder-only rule is omitted from CSS output.

### 9. What happens when several selectors extend one placeholder?

Sass can group the extenders in the generated selector list for the placeholder's declarations. Each extender receives the shared styles without the placeholder appearing in the CSS.

### 10. Can a placeholder be used in HTML?

No. The **%name** syntax is for Sass. It is not a CSS class or ID selector for markup.

### 11. When is an ordinary class better than a placeholder?

Use an ordinary class when it is a useful standalone selector for the application and authors may apply it directly in markup. Use a placeholder when the shared base is only an internal Sass abstraction.

## Choosing between features

### 12. When is @extend a good fit?

Use it to express a meaningful relationship between selectors, such as one semantic class being a specialized form of another, or when several selectors share an abstract placeholder.

### 13. When should I use a mixin instead?

Use a mixin when you want declarations inserted at the include site, when the styles need arguments, or when there is no meaningful selector relationship.

~~~scss
@mixin control-base($radius) {
  border: 1px solid #8a94a6;
  border-radius: $radius;
  padding: 0.5rem 0.75rem;
}

.text-input {
  @include control-base(0.25rem);
}
~~~

The mixin can be configured for this specific include.

### 14. Is @extend a CSS compression technique?

It can group rules, but CSS byte savings should not be the only reason to use it. Selector expansion can affect cascade behavior and generate complex output. Choose the feature that best represents the code's intent.

### 15. How do extends and mixins differ in the compiled CSS?

A mixin generally copies declarations into each include site. An extend generally adds the extender to selector lists for existing rules. The resulting output and cascade behavior are different.

## Targets, limitations, and scope

### 16. Which selectors can be extended?

Modern Dart Sass allows extending simple selectors such as **.notice** or **a**. Compound selectors such as **.notice.important** and complex selectors such as **.page .notice** cannot be extended directly.

### 17. What happens if a mandatory extend target is missing?

Sass reports a compilation error. This helps catch typos or a target selector that was removed.

### 18. What does !optional do?

Adding **!optional** tells Sass not to report an error if the target selector is missing.

~~~scss
.notice--warning {
  @extend .notice !optional;
}
~~~

Use it only when doing nothing is an intentional and safe outcome. Otherwise, it can hide mistakes.

### 19. Can an extend inside a media query target a selector outside it?

No. An extender inside a media query cannot extend a target selector outside that media query. Sass cannot safely move the resulting selector out of its media context without changing behavior. Use a mixin in the media query or organize the styles in a compatible context.

### 20. How does @use affect extension scope?

With the module system, extensions affect relevant rules in the current stylesheet and upstream modules loaded through its **@use** or **@forward** graph. They do not act as a project-wide search across unrelated modules.

### 21. Does @import have the same extension scope as @use?

No. Legacy Sass **@import** makes extensions global across imported stylesheets, which can make the effects harder to predict. Prefer the module system with **@use** for new code.

## Debugging and design

### 22. Why did my placeholder rule disappear from the CSS?

A placeholder is omitted unless an emitted selector extends it. If it should contribute styles, check that an extender exists and that the extension is in scope.

### 23. Why did my extender not receive the expected styles?

Check that the target selector is a simple selector, exists in the extension scope, and is not separated by a media-query boundary. Also check whether **!optional** caused a missing target to be ignored.

### 24. Why did an extend change cascade behavior?

The extender is added to the original selector's rule. The shared declarations stay in the original rule's position, so rule order may differ from what you expected if you assumed declarations would be copied to the extend location.

### 25. Should I replace every repeated declaration block with @extend?

No. Repeated declarations alone do not necessarily represent a selector relationship. Use a mixin for generic reusable declarations, especially if arguments or local configuration may be useful.

## Quick review

- **@extend** changes selector lists on matching rules; it does not copy declarations at the extender location.
- A **%placeholder** can be extended without being emitted by itself.
- Use **@extend** for meaningful selector relationships and mixins for configurable style reuse.
- Extend simple selectors and keep targets in the correct module scope.
- Mandatory extends catch missing targets; **!optional** intentionally suppresses that error.
- Review compiled selector lists and cascade order when using extends.

## Further reading

- [Sass documentation: @extend](https://sass-lang.com/documentation/at-rules/extend/)
- [Sass documentation: placeholder selectors](https://sass-lang.com/documentation/style-rules/placeholder-selectors/)
