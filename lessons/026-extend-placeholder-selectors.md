# 026 - @extend and Placeholder Selectors

## Learning goals

After this lesson, you should be able to:

- Explain what **@extend** changes in compiled selectors.
- Use a placeholder selector to share styles without emitting an unused base selector.
- Decide when a semantic class relationship is a good fit for **@extend**.
- Compare **@extend** with a mixin.
- Recognize common extension limitations involving selector types, scope, and media queries.

## 1. What does @extend do?

The **@extend** rule tells Sass that the selector where it appears should also match rules that use another selector.

~~~scss
.message {
  border: 1px solid #8a94a6;
  padding: 1rem;
}

.message--warning {
  @extend .message;
  border-color: #a45b00;
}
~~~

Sass combines the selectors for the shared declarations:

~~~css
.message,
.message--warning {
  border: 1px solid #8a94a6;
  padding: 1rem;
}

.message--warning {
  border-color: #a45b00;
}
~~~

An element with class **message--warning** receives the shared styles as if it also matched **.message**. The HTML does not need both classes.

## 2. Extension changes selectors, not declaration placement

A mixin copies declarations into each place it is included. **@extend** instead updates rules that contain the selector being extended, adding the extending selector to those rules.

~~~scss
.notice {
  border: 1px solid #8a94a6;
  padding: 1rem;
}

.notice--urgent {
  @extend .notice;
  border-width: 3px;
}
~~~

The shared declarations remain in the rule for **.notice**; Sass groups the selectors that should match it. The extra border width remains specific to **.notice--urgent**.

This affects the cascade. The shared rule's position is based on where the original selector's rule appears, not where the **@extend** appears. Read the generated selectors and rule order when cascade behavior matters.

## 3. Use a placeholder selector for abstract shared styles

A placeholder selector starts with **%** instead of **.**. It can be extended, but is not emitted as a selector on its own.

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

The output contains the extending classes, not **%control-base**:

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

If no selector extends a placeholder, rules that only contain that placeholder are omitted from the CSS output. This makes placeholders useful for shared Sass-only patterns that should not require a class in the markup.

## 4. Extend a class when the relationship is semantic

A class selector is emitted normally whether or not anything extends it. Extending one class with another can express a relationship where the second class is a specialized version of the first.

~~~scss
.status {
  padding: 0.5rem 0.75rem;
  border-radius: 0.25rem;
}

.status--success {
  @extend .status;
  color: #14532d;
  background-color: #dcfce7;
}

.status--error {
  @extend .status;
  color: #7f1d1d;
  background-color: #fee2e2;
}
~~~

The modifier classes inherit the base status styles while adding their own declarations. The relationship should make sense: a success status is still a status.

## 5. Choose between @extend and mixins

Both features can share style logic, but they change the output differently.

| Use **@extend** when… | Use a mixin when… |
| --- | --- |
| One semantic selector should match rules written for another selector. | Each include needs declarations inserted at its own location. |
| A placeholder represents an abstract base pattern. | The style needs arguments, configuration, or conditional output. |
| A shared selector relationship is clear and predictable. | A collection of declarations is reused without a selector relationship. |

A mixin copies declarations:

~~~scss
@mixin control-base {
  box-sizing: border-box;
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 0.5rem 0.75rem;
}

.text-input {
  @include control-base;
}

.select-input {
  @include control-base;
}
~~~

A placeholder extends a shared rule and allows Sass to group the selectors:

~~~scss
%control-base {
  box-sizing: border-box;
  border: 1px solid #8a94a6;
  border-radius: 0.375rem;
  padding: 0.5rem 0.75rem;
}

.text-input {
  @extend %control-base;
}

.select-input {
  @extend %control-base;
}
~~~

Choose based on meaning and cascade behavior, not only on how many CSS bytes the source seems to save. Mixins are often clearer for generic collections of declarations; extends are most useful when selectors have a real relationship.

## 6. Extend only a simple selector

In modern Dart Sass, **@extend** targets a simple selector, such as **.notice** or **a**. A compound selector like **.notice.important** or a complex selector like **.page .notice** cannot be extended directly.

~~~scss
// Valid: extends one simple class selector.
.notice--urgent {
  @extend .notice;
}
~~~

If an extending selector also needs more than one independent class relationship, structure the shared rules around simple selectors or use a placeholder. Do not rely on extending a compound selector.

Sass combines compatible selectors and avoids generating selectors that cannot match an element. That means the final selector list can be more complex than a literal text replacement. Inspect compiled CSS when a selector relationship is not obvious.

## 7. Missing targets and !optional

By default, **@extend** is mandatory: if Sass cannot find a matching selector, compilation reports an error. This catches spelling mistakes and stale references.

~~~scss
.button--primary {
  @extend .button;
}
~~~

If no **.button** selector is available in the relevant extension scope, compilation fails. Add **!optional** only when it is intentional for the extension to do nothing when the target is absent:

~~~scss
.button--primary {
  @extend .button !optional;
}
~~~

Do not use **!optional** to silence an unexpected missing selector. It can hide a typo or a broken stylesheet relationship.

## 8. Extension scope and modules

With Sass's **@use** module system, an extension affects rules in the current stylesheet and upstream modules loaded through that module graph. It does not behave like a project-wide search-and-replace across unrelated stylesheets.

This scope makes extension more predictable. Keep the selector or placeholder definition in a module that the stylesheet using **@extend** loads.

When older Sass **@import** is used, extensions can be global across imported stylesheets, which makes their effects harder to trace. Prefer modules and **@use** in new Sass.

## 9. Avoid extending across media queries

An **@extend** inside a media query cannot extend a selector defined outside that media query. Sass cannot safely move the extended selector out of the media context without changing the meaning.

~~~scss
@media (max-width: 40rem) {
  .message--compact {
    @extend .message;
  }
}

.message {
  padding: 1rem;
}
~~~

This pattern is not valid because the extender is inside the media query while the target rule is outside it. If responsive behavior is needed, use a mixin inside the media query or define the relevant rules in a compatible context.

## 10. Common mistakes

### Expecting @extend to copy declarations at the include site

**@extend** updates selectors on existing rules. It does not insert a fresh copy of declarations where it appears.

### Extending a selector that does not exist

A mandatory extension to a missing selector is an error. Check the target name and whether it is available in the current module scope.

### Using a placeholder as though it were a class

A placeholder starts with **%** and does not appear in HTML or CSS output unless it is extended. Use a class selector when authors need to apply that name in markup.

### Extending only to reduce CSS size

Selector merging can affect cascade behavior and generate complex selector lists. Choose **@extend** to model a real selector relationship, not just as a compression trick.

### Extending complex or compound selectors

Extend a simple selector or use a placeholder designed for sharing. Use a mixin when the desired relationship is only a group of declarations.

## 11. Practice

1. Create a **%surface** placeholder for shared background, border, and padding styles.
2. Extend it from two component classes and inspect the grouped CSS.
3. Create a base **.notice** class and a semantic **.notice--warning** modifier that extends it.
4. Rewrite the shared declarations as a mixin and compare where the declarations appear in the output.
5. Explain why a missing mandatory target causes a Sass error.
6. Explain why an extension inside a media query cannot extend an outer selector.

## Key takeaways

- **@extend** adds the extending selector to rules that contain the target selector.
- A placeholder starts with **%** and is omitted unless extended.
- Use extends for meaningful selector relationships and mixins for configurable or generic declaration reuse.
- Extend simple selectors; avoid compound and complex extend targets.
- Mandatory extends catch missing targets; **!optional** should be used only intentionally.
- Module scope and media-query boundaries affect which rules can be extended.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="025-generating-selectors-and-utilities.md">025 - Generating Selectors and Utility Classes</a></td><td><a href="../qnas/026-extend-placeholder-selectors.md">Q&amp;A</a></td><td><a href="../examples/026-extend-placeholder-selectors.md">Example</a></td><td><a href="027-at-root-and-at-rules.md">027 - @at-root and Nested At-Rules</a></td></tr>
  </tbody>
</table>
