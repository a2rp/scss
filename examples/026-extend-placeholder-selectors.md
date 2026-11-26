# 026 — @extend and Placeholder Selectors: Examples

These examples compare ordinary class extensions, placeholders, and mixins. Sass resolves **@extend** during compilation and writes the resulting selectors into CSS.

## 1. Extend a semantic base class

A modifier can extend a base class when the modifier is still conceptually that kind of component.

~~~scss
.message {
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 1rem;
}

.message--warning {
  @extend .message;
  border-color: #a45b00;
  background-color: #fff7ed;
}
~~~

Generated CSS:

~~~css
.message,
.message--warning {
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 1rem;
}

.message--warning {
  border-color: #a45b00;
  background-color: #fff7ed;
}
~~~

An element with only **message--warning** receives the shared declarations. It does not need both classes in its HTML.

## 2. Share internal styles with a placeholder

A placeholder is Sass-only. It contributes styles when extended, and its own name does not appear in the CSS.

~~~scss
%control-base {
  box-sizing: border-box;
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 0.5rem 0.75rem;
}

.text-input {
  @extend %control-base;
  width: 100%;
}

.select-input {
  @extend %control-base;
  background-color: #ffffff;
}
~~~

Generated CSS:

~~~css
.text-input,
.select-input {
  box-sizing: border-box;
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 0.5rem 0.75rem;
}

.text-input {
  width: 100%;
}

.select-input {
  background-color: #ffffff;
}
~~~

The placeholder is not an HTML class. If no selector extends **%control-base**, the placeholder-only styles are not emitted.

## 3. Extend shared interaction styles

A placeholder can also include nested selectors. Extenders receive the matching nested styles too.

~~~scss
%interactive-surface {
  transition: background-color 150ms ease;

  &:hover {
    background-color: #eef2f7;
  }
}

.card-link {
  @extend %interactive-surface;
  display: block;
}

.menu-button {
  @extend %interactive-surface;
  border: 0;
}
~~~

The compiled output groups the extending selectors for the shared transition and hover declarations. The placeholder itself is omitted.

~~~css
.card-link,
.menu-button {
  transition: background-color 150ms ease;
}

.card-link:hover,
.menu-button:hover {
  background-color: #eef2f7;
}
~~~

Sass may order grouped selectors differently while preserving their matching behavior.

## 4. Compare a placeholder with a mixin

A placeholder lets Sass group selectors around a shared rule:

~~~scss
%surface {
  border: 1px solid #d0d5dd;
  border-radius: 0.5rem;
  padding: 1rem;
}

.card {
  @extend %surface;
  background-color: #ffffff;
}

.notice {
  @extend %surface;
  background-color: #f5f7fa;
}
~~~

The shared declarations are associated with both selectors:

~~~css
.card,
.notice {
  border: 1px solid #d0d5dd;
  border-radius: 0.5rem;
  padding: 1rem;
}

.card {
  background-color: #ffffff;
}

.notice {
  background-color: #f5f7fa;
}
~~~

A mixin inserts declarations at each include location:

~~~scss
@mixin surface {
  border: 1px solid #d0d5dd;
  border-radius: 0.5rem;
  padding: 1rem;
}

.card {
  @include surface;
  background-color: #ffffff;
}

.notice {
  @include surface;
  background-color: #f5f7fa;
}
~~~

The mixin output keeps the declarations in each rule:

~~~css
.card {
  border: 1px solid #d0d5dd;
  border-radius: 0.5rem;
  padding: 1rem;
  background-color: #ffffff;
}

.notice {
  border: 1px solid #d0d5dd;
  border-radius: 0.5rem;
  padding: 1rem;
  background-color: #f5f7fa;
}
~~~

Use a mixin when styles need arguments or when declaration placement should be local to each rule. Use a placeholder when the selectors have a meaningful shared base and grouped output is appropriate.

## 5. Observe the cascade location

An extend adds the extending selector to the original rule; it does not move the original rule to the extender's location.

~~~scss
.alert {
  color: #7f1d1d;
}

.alert--urgent {
  @extend .alert;
  color: #ffffff;
  background-color: #b42318;
}
~~~

Generated CSS:

~~~css
.alert,
.alert--urgent {
  color: #7f1d1d;
}

.alert--urgent {
  color: #ffffff;
  background-color: #b42318;
}
~~~

The urgent alert is white because its later, more specific rule overrides the shared color. When debugging an extend, inspect the compiled rule order and selectors.

## 6. An optional extension

A normal extend is mandatory: Sass reports an error if its target is missing. **!optional** is for cases where the target may intentionally be absent.

~~~scss
.badge--custom {
  @extend .optional-badge !optional;
  font-weight: 700;
}
~~~

If no **.optional-badge** rule exists in scope, Sass skips the extension and keeps the badge's own declaration. Do not add **!optional** just to silence an unexpected missing selector.

## 7. Use a mixin inside a media query

An extension inside a media query cannot target a selector outside that media query. A mixin is a straightforward option when the declarations must be added conditionally at a breakpoint.

~~~scss
@mixin compact-message {
  padding: 0.5rem;
  font-size: 0.875rem;
}

.message {
  padding: 1rem;

  @media (max-width: 40rem) {
    @include compact-message;
  }
}
~~~

Generated CSS:

~~~css
.message {
  padding: 1rem;
}

@media (max-width: 40rem) {
  .message {
    padding: 0.5rem;
    font-size: 0.875rem;
  }
}
~~~

The mixin output remains inside the media query where the browser can apply it.

## 8. A placeholder for a component family

~~~scss
%pill-base {
  display: inline-flex;
  align-items: center;
  border-radius: 999px;
  padding: 0.25rem 0.625rem;
  font-size: 0.875rem;
}

.pill--info {
  @extend %pill-base;
  color: #1e3a8a;
  background-color: #dbeafe;
}

.pill--success {
  @extend %pill-base;
  color: #14532d;
  background-color: #dcfce7;
}
~~~

The shared declarations are grouped for **.pill--info** and **.pill--success**. The color and background remain in each modifier's own rule.

## 9. Common mistakes

### Expecting a placeholder to appear in CSS

A **%placeholder** is not emitted by itself. Extend it from a real selector to include its styles.

### Expecting @extend to copy declarations at the extender

Sass updates the selector list of the target rule. Use a mixin if you need declarations inserted at the include site.

### Extending a missing target

A required extension fails compilation when Sass cannot find the target in scope. Check spelling and module relationships; use **!optional** only when absence is expected.

### Extending a compound selector

Modern Dart Sass only allows simple selector targets for **@extend**. Refactor the shared rule to use a simple selector or placeholder, or use a mixin.

## Practice

1. Create a **%panel-base** placeholder and extend it from two panel classes.
2. Add a modifier that extends a semantic base class and adds its own declarations.
3. Rewrite a placeholder example as a mixin and compare the output location.
4. Remove every extension of a placeholder and observe that its styles disappear from CSS.
5. Explain why a mixin is easier than **@extend** for responsive conditional declarations.

## Compile an example

Save a snippet as **extend.scss** and compile it with Dart Sass:

~~~sh
sass extend.scss extend.css
~~~

Inspect the selector groups and rule order. The compiled CSS shows which selectors Sass has joined and where each declaration applies.

## Key takeaways

- **@extend** adds selectors to rules for the target; it does not copy the declarations to the extender.
- **%placeholders** do not appear in CSS unless extended.
- Use mixins for configurable or locally inserted declarations.
- A semantic relationship is a good reason to extend a class; a shared abstract base is a good reason to extend a placeholder.
- Check generated selectors and cascade order.
