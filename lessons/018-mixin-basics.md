# 018 - Mixin Basics

## Learning goals

After this lesson, you should be able to:

- Explain what a Sass mixin is and when it is useful.
- Define a mixin with @mixin and include it with @include.
- Reuse a collection of declarations in multiple selectors.
- Include one mixin from another mixin.
- Understand where nested rules from a mixin are emitted.
- Predict the CSS output and recognize repeated output from repeated includes.
- Choose a mixin only when it makes the source clearer and easier to maintain.

## 1. What a mixin does

A mixin is a named block of Sass code that can be included in one or more places. It helps reuse a group of styles without copying the same declarations into each selector.

Define a mixin with @mixin:

~~~scss
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
~~~

Including the mixin uses @include:

~~~scss
.skip-link__text {
  @include visually-hidden;
}

.visually-hidden {
  @include visually-hidden;
}
~~~

Generated CSS:

~~~css
.skip-link__text {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
  border: 0;
}
~~~

The mixin definition describes reusable Sass code. It does not emit CSS by itself. CSS is emitted where the mixin is included.

## 2. The basic syntax

The two parts have different jobs:

~~~scss
@mixin name {
  // Reusable Sass statements go here.
}

.selector {
  @include name;
}
~~~

- @mixin introduces the definition and gives it a name.
- @include inserts the mixin's contents at that location.
- The braces around the mixin body group its reusable statements.
- A simple @include statement ends with a semicolon in SCSS.

Names are usually written in lowercase kebab-case, such as focus-ring or visually-hidden. Choose a name that describes the styles or behavior being reused.

## 3. Reuse declarations across selectors

A mixin can collect declarations that belong together.

~~~scss
@mixin control-base {
  box-sizing: border-box;
  min-height: 2.75rem;
  padding: 0.625rem 0.875rem;
  border: 1px solid #98a2b3;
  border-radius: 0.5rem;
  font: inherit;
}

.search-input {
  @include control-base;
  width: 100%;
}

.select-input {
  @include control-base;
  width: 100%;
  background-color: white;
}
~~~

Generated CSS:

~~~css
.search-input {
  box-sizing: border-box;
  min-height: 2.75rem;
  padding: 0.625rem 0.875rem;
  border: 1px solid #98a2b3;
  border-radius: 0.5rem;
  font: inherit;
  width: 100%;
}
.select-input {
  box-sizing: border-box;
  min-height: 2.75rem;
  padding: 0.625rem 0.875rem;
  border: 1px solid #98a2b3;
  border-radius: 0.5rem;
  font: inherit;
  width: 100%;
  background-color: white;
}
~~~

Each selector gets its own declarations. The include does not make the two elements share a CSS rule.

## 4. Include a mixin from another mixin

A mixin can include another mixin. This lets a larger reusable pattern build on smaller, named pieces.

~~~scss
@mixin list-reset {
  margin: 0;
  padding: 0;
  list-style: none;
}

@mixin horizontal-list {
  @include list-reset;

  li {
    display: inline-block;
  }
}

.navigation ul {
  @include horizontal-list;
}
~~~

Generated CSS:

~~~css
.navigation ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
.navigation ul li {
  display: inline-block;
}
~~~

The horizontal-list mixin reuses list-reset and adds its own nested rule.

Keep the chain of mixin calls easy to follow. If every include leads through many other definitions, it becomes harder to find the declarations that will be emitted.

## 5. Nested rules are emitted in the include context

A mixin can contain style rules in addition to declarations. Those rules are emitted where the mixin is included.

~~~scss
@mixin form-control {
  box-sizing: border-box;
  width: 100%;

  &::placeholder {
    color: #667085;
    opacity: 1;
  }
}

.search-field {
  @include form-control;
}
~~~

Generated CSS:

~~~css
.search-field {
  box-sizing: border-box;
  width: 100%;
}
.search-field::placeholder {
  color: #667085;
  opacity: 1;
}
~~~

Here the mixin is included in .search-field. Its nested parent selector resolves in that context. The next lesson on mixin content blocks covers a different pattern where the caller supplies a block of styles.

If a mixin contains a standalone selector rather than a nested selector, that rule is also emitted at the location where the include is processed. Check the output when including such a mixin inside another style rule.

## 6. Mixins can contain conditional CSS at-rules

A mixin can package a fixed media rule or another CSS at-rule with related declarations.

~~~scss
@mixin reduce-motion-transition {
  transition: transform 180ms ease-out;

  @media (prefers-reduced-motion: reduce) {
    transition: none;
  }
}

.dialog {
  @include reduce-motion-transition;
}
~~~

Generated CSS:

~~~css
.dialog {
  transition: transform 180ms ease-out;
}
@media (prefers-reduced-motion: reduce) {
  .dialog {
    transition: none;
  }
}
~~~

The conditional rule is emitted along with the other styles. A mixin with a hard-coded condition is useful only if that same condition and behavior belong in every place it is included.

## 7. Mixin output is duplicated at each include

A mixin does not create a runtime reference or a special class in CSS. Sass inserts its contents at every include.

~~~scss
@mixin border-box {
  box-sizing: border-box;
}

.input {
  @include border-box;
}

.textarea {
  @include border-box;
}
~~~

Generated CSS:

~~~css
.input {
  box-sizing: border-box;
}
.textarea {
  box-sizing: border-box;
}
~~~

The declaration appears in both rules. This can be convenient because each selector is self-contained, but a very large mixin included across many selectors can produce a lot of repeated CSS.

Think about both source clarity and generated output. Reuse is useful when it simplifies maintenance; it does not automatically make the CSS output smaller.

## 8. Mixin or ordinary class?

Use a mixin when a reusable set of declarations should be inserted into selectors that have different meanings or need to stay separate in the output.

Use a regular semantic class in the HTML when multiple elements should share the same class-based rule and adding that class is appropriate for the markup.

For example, an HTML class is often clearer for a site-wide utility intended to be applied directly to elements. A mixin is useful when the styles belong in a component's own selector or require a Sass-specific composition.

Do not add mixins for every single declaration. A name and an include have a small maintenance cost; use them when the named group makes the design easier to understand or update.

## 9. Mixin or @extend?

Both features can reuse styles, but they do different things:

- A mixin copies its declarations and nested rules into the include location.
- @extend changes selectors so multiple selectors share a CSS rule.

Use a mixin when the reusable styles should be included in each component rule, or when the pattern will need customization in a later step. Use @extend only when the selectors have a meaningful relationship and the shared rule is intended.

The @extend rule is covered in a later chapter. Avoid choosing either feature just to minimize the number of CSS bytes without considering the resulting cascade and selectors.

## 10. Mixin names and module use

Mixins are Sass members. A mixin declared in the same file can be included by its name. A mixin provided by a module is loaded and referenced through Sass's module system, which is covered in later module chapters.

Sass treats hyphens and underscores as equivalent in identifiers, so a mixin named reset-list can also be referenced as reset_list. For consistency and readability, choose one naming style and use it throughout a project.

## 11. Arguments are the next step

This lesson uses mixins without parameters so the definition and include behavior stay clear.

A later chapter adds arguments, defaults, and keyword arguments. Arguments let a mixin change its output based on values supplied at each include.

~~~scss
// The structure of a parameterized mixin is introduced in the next lesson.
@mixin example {
  display: block;
}
~~~

For now, define a reusable block of fixed styles and include it in one or more selectors.

## 12. Common mistakes

### Mistake: defining a mixin but never including it

The definition alone emits no CSS. Add an @include in a stylesheet location where its styles belong.

### Mistake: including a mixin with a misspelled name

Sass reports that it cannot find the mixin. Keep names consistent and use the same spelling in the definition and include.

### Mistake: assuming a mixin creates a class

A mixin is a compile-time Sass feature. It does not create an HTML class or a reusable runtime object.

### Mistake: assuming the include is a CSS function call

The browser never sees @include. Sass expands the mixin during compilation and emits its CSS.

### Mistake: adding every shared declaration to a universal mixin

A large catch-all mixin makes it difficult to tell what each component needs. Prefer focused mixins with a clear purpose.

### Mistake: using mixins without checking their output

Nested selectors, media queries, and repeated includes all affect generated CSS. Compile and inspect the output when a mixin does more than insert a few obvious declarations.

## 13. A practical example: focus treatment

Source SCSS:

~~~scss
@mixin keyboard-focus {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}

.primary-action {
  &:focus-visible {
    @include keyboard-focus;
  }
}

.search-input {
  &:focus-visible {
    @include keyboard-focus;
  }
}
~~~

Generated CSS:

~~~css
.primary-action:focus-visible {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
.search-input:focus-visible {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
~~~

The mixin reuses the focus declarations, while each selector retains its own semantic meaning. In real designs, focus styles may need different colors or offsets for different surfaces; the next chapter's arguments make those choices configurable.

## 14. Exercises

1. Define a list-reset mixin and include it in a navigation list.
2. Define a control-base mixin and include it in an input and a select.
3. Create a focus treatment mixin containing outline and outline-offset declarations.
4. Include one mixin from inside another, then predict the output order.
5. Put a nested pseudo-element in a mixin and include it inside a component selector.
6. Include the same mixin in two selectors and count how many copies of its declarations are generated.
7. Compare a reusable mixin with a semantic HTML class for the same style group.
8. Inspect a mixin definition and determine whether it emits CSS before any @include exists.

## Summary

- Define reusable Sass code with @mixin and insert it with @include.
- A mixin definition does not emit CSS until it is included.
- Includes expand declarations and nested rules at their use sites.
- Repeated includes generally produce repeated CSS declarations or rules.
- Mixins can include other mixins and can contain CSS at-rules.
- Choose a mixin when named Sass composition improves clarity; do not create one for every declaration.
- Mixin arguments and caller-provided content blocks are separate topics covered in the next chapters.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="017-interpolation-in-selectors-and-property-names.md">017 - Interpolation in Selectors and Property Names</a></td><td><a href="../qnas/018-mixin-basics.md">Q&amp;A</a></td><td><a href="../examples/018-mixin-basics.md">Example</a></td><td><a href="019-mixin-arguments-defaults-and-keyword-args.md">019 - Mixin Arguments, Defaults, and Keyword Arguments</a></td></tr>
  </tbody>
</table>
