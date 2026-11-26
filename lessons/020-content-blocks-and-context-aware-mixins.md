# 020 - Content Blocks and Context-Aware Mixins

## Learning goals

After this lesson, you should be able to:

- Pass a block of Sass statements to a mixin with an @include content block.
- Insert that block where the mixin uses @content.
- Use a mixin to place caller styles inside a selector state or CSS at-rule.
- Explain the lexical scope of a content block.
- Pass values from a mixin to its content block with @content(...) and using (...).
- Understand what happens when a mixin inserts the same content block more than once.
- Decide whether a mixin needs a content block or only ordinary arguments.

## 1. What is a content block?

A content block is a block of Sass statements supplied at an @include site. The mixin receives that block at an @content statement.

~~~scss
@mixin wrapper {
  .wrapper {
    @content;
  }
}

@include wrapper {
  color: #202124;
  padding: 1rem;
}
~~~

Generated CSS:

~~~css
.wrapper {
  color: #202124;
  padding: 1rem;
}
~~~

The @content statement is a placeholder for the caller's block. Sass inserts the supplied statements at that location while compiling.

A content block is useful when the mixin controls the selector or at-rule context, while the caller chooses the declarations or rules that belong inside it.

## 2. Define a mixin that accepts content

A mixin can use @content inside its body. At the include site, put the caller-provided Sass inside braces.

~~~scss
@mixin card-surface {
  padding: 1rem;
  border: 1px solid #d0d5dd;
  border-radius: 0.75rem;

  @content;
}

.profile-card {
  @include card-surface {
    background-color: white;
    color: #202124;
  }
}
~~~

Generated CSS:

~~~css
.profile-card {
  padding: 1rem;
  border: 1px solid #d0d5dd;
  border-radius: 0.75rem;
  background-color: white;
  color: #202124;
}
~~~

The declarations in the content block are emitted at the position of @content, after the mixin's first three declarations.

The block can contain more than declarations. It can contain nested selectors, CSS at-rules, and other valid Sass statements.

## 3. A context-aware state mixin

A context-aware mixin chooses where the supplied styles apply. A common example is a hover rule that also excludes disabled controls.

~~~scss
@mixin interactive-hover {
  &:not([disabled]):hover {
    @content;
  }
}

.button {
  background-color: #175cd3;

  @include interactive-hover {
    background-color: #1849a9;
  }
}
~~~

Generated CSS:

~~~css
.button {
  background-color: #175cd3;
}
.button:not([disabled]):hover {
  background-color: #1849a9;
}
~~~

The mixin owns the state selector. The caller supplies the hover declarations. Because the mixin is included inside .button, the parent selector resolves to .button.

This separation can help keep a shared interaction rule consistent while leaving each component's actual hover styling close to that component.

## 4. Use a content block inside a media query

A mixin can wrap caller styles in a CSS at-rule.

~~~scss
@mixin medium-screen {
  @media (min-width: 48rem) {
    @content;
  }
}

.article {
  padding-inline: 1rem;

  @include medium-screen {
    max-width: 48rem;
    margin-inline: auto;
  }
}
~~~

Generated CSS:

~~~css
.article {
  padding-inline: 1rem;
}
@media (min-width: 48rem) {
  .article {
    max-width: 48rem;
    margin-inline: auto;
  }
}
~~~

The mixin places the supplied declarations inside the media query. Since the include is within .article, the declarations remain associated with that selector in the generated CSS.

The next part of the example gives the condition a parameter so it can be reused at different breakpoints.

## 5. Combine mixin arguments and a content block

A mixin can receive ordinary arguments as well as a content block.

~~~scss
@mixin at-min-width($minimum) {
  @media (min-width: $minimum) {
    @content;
  }
}

.grid {
  @include at-min-width(48rem) {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1.5rem;
  }
}

.gallery {
  @include at-min-width(72rem) {
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 2rem;
  }
}
~~~

Generated CSS:

~~~css
@media (min-width: 48rem) {
  .grid {
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1.5rem;
  }
}
@media (min-width: 72rem) {
  .gallery {
    grid-template-columns: repeat(4, minmax(0, 1fr));
    gap: 2rem;
  }
}
~~~

The argument controls the query. The content block controls the styles placed inside it.

This example uses the include location to keep each component's responsive declarations next to that component's base rule.

## 6. Content blocks are lexically scoped

A content block can access variables from the scope where the @include appears. It does not gain access to local variables defined inside the mixin that invokes @content.

~~~scss
@mixin with-content {
  $mixin-color: red;
  @content;
}

.card {
  $card-color: #175cd3;

  @include with-content {
    border-color: $card-color;
  }
}
~~~

Generated CSS:

~~~css
.card {
  border-color: #175cd3;
}
~~~

The content block can read $card-color because that variable is in the include site's scope.

It cannot read $mixin-color just because the mixin defined it before @content. That variable belongs to the mixin's scope, not the lexical scope where the content block was written.

If content needs a value chosen by the mixin, pass it explicitly through @content(...) and receive it with using (...).

## 7. Pass a value to the content block

A mixin can call @content with arguments. The include site declares which values it accepts with using (...).

~~~scss
@mixin at-min-width($minimum) {
  @media (min-width: $minimum) {
    @content($minimum);
  }
}

.card {
  @include at-min-width(48rem) using ($breakpoint) {
    min-width: $breakpoint;
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
~~~

Generated CSS:

~~~css
@media (min-width: 48rem) {
  .card {
    min-width: 48rem;
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }
}
~~~

The mixin passes $minimum to the block. The content block receives it as $breakpoint and can use it in its own statements.

The value passed to @content is an argument, just like a value passed to a mixin. If the mixin passes more than one value, the using clause must declare matching parameters.

## 8. A content block can contain selectors

The caller can provide nested rules as well as declarations.

~~~scss
@mixin state-styles {
  &:focus-visible {
    @content;
  }
}

.button {
  @include state-styles {
    outline: 3px solid #84caff;
    outline-offset: 3px;

    .icon {
      opacity: 0.8;
    }
  }
}
~~~

Generated CSS:

~~~css
.button:focus-visible {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
.button:focus-visible .icon {
  opacity: 0.8;
}
~~~

The caller's nested .icon selector is inserted inside the focus-visible context. The context-aware mixin controls the state; the caller controls the styles inside that state.

## 9. A mixin can insert the same content more than once

If a mixin contains multiple @content statements, Sass inserts the caller's block at each one.

~~~scss
@mixin duplicate-content {
  .first {
    @content;
  }

  .second {
    @content;
  }
}

@include duplicate-content {
  color: #202124;
}
~~~

Generated CSS:

~~~css
.first {
  color: #202124;
}
.second {
  color: #202124;
}
~~~

Repeated content insertion can be useful in a deliberate abstraction, but it may duplicate declarations or expensive rules. Make the repeated output clear to anyone using the mixin.

A content block is not automatically evaluated once and shared. Each @content statement is an insertion point.

## 10. Use content arguments for explicit context

A mixin can pass information about its context to the block rather than relying on a variable from its own scope.

~~~scss
@mixin responsive-rule($minimum) {
  @media (min-width: $minimum) {
    @content($minimum);
  }
}

.layout {
  @include responsive-rule(60rem) using ($breakpoint) {
    max-width: $breakpoint;
    margin-inline: auto;
  }
}
~~~

Generated CSS:

~~~css
@media (min-width: 60rem) {
  .layout {
    max-width: 60rem;
    margin-inline: auto;
  }
}
~~~

This can make a reusable mixin's contract explicit: it establishes a context and tells the content block which value it used.

Use this pattern only when the caller needs that information. If the content block can be written without receiving values, a simple @content; is easier.

## 11. Content arguments and compatibility

Passing arguments to a content block with @content(...) and accepting them with using (...) is supported by Dart Sass. This feature was added in Dart Sass 1.15.0 and is not supported by LibSass.

This repository follows Dart Sass. If a project uses an old compiler or a different Sass implementation, check its compatibility before using this syntax.

When a reusable mixin passes values to content, prefer positional content arguments. The content block must declare the arguments it accepts, so adding or changing passed values can require updates at every include site.

## 12. Content block or ordinary arguments?

Use ordinary mixin arguments when the mixin needs a value, such as a breakpoint or color.

Use a content block when the caller needs to provide a group of statements that should run inside a context chosen by the mixin.

For example:

- A mixin that sets border-radius from a value needs an argument.
- A mixin that wraps arbitrary styles in a focus state can use a content block.
- A responsive mixin can combine a breakpoint argument with a content block.
- A mixin that inserts a fixed declaration group may not need either a content block or additional parameters.

Prefer the smallest API that expresses what the caller needs.

## 13. Common mistakes

### Mistake: defining @content but never supplying a block

If a mixin expects caller styles, include it with a block and make sure the block is placed where those styles belong.

### Mistake: passing a value without accepting it

If the mixin calls @content($value), the include must accept that argument with using (...). The content block does not receive it automatically as a variable with the same name.

### Mistake: expecting content to see mixin-local variables

Content blocks use the lexical scope where the include is written. Pass a mixin-local value explicitly through @content(...) when needed.

### Mistake: declaring the wrong number of using parameters

The number of parameters in using (...) must match the values passed to @content(...). Keep that interface in sync.

### Mistake: repeating @content accidentally

Every @content statement inserts the block. Check how many insertion points the mixin contains and inspect the output for duplication.

### Mistake: using a content block for a simple value

If the caller only needs to provide one value, a regular mixin argument is usually clearer than a block and @content.

## 14. Practical example: responsive card

Source SCSS:

~~~scss
@mixin at-min-width($minimum) {
  @media (min-width: $minimum) {
    @content($minimum);
  }
}

.card {
  padding: 1rem;

  @include at-min-width(48rem) using ($breakpoint) {
    display: grid;
    grid-template-columns: minmax(0, 1fr) 12rem;
    gap: 1.5rem;
    max-width: $breakpoint;
    margin-inline: auto;
  }
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
}
@media (min-width: 48rem) {
  .card {
    display: grid;
    grid-template-columns: minmax(0, 1fr) 12rem;
    gap: 1.5rem;
    max-width: 48rem;
    margin-inline: auto;
  }
}
~~~

The mixin provides the media context and passes the breakpoint into the content block. The include site keeps the responsive layout next to the card's base styles.

## 15. Exercises

1. Define a mixin that places a content block inside &:hover.
2. Add a disabled exclusion to the hover selector.
3. Define a mixin that wraps content in a media query at a fixed breakpoint.
4. Add a breakpoint argument and use the mixin at two sizes.
5. Pass that breakpoint to the content block with @content(...) and using (...).
6. Create a mixin whose content block contains both declarations and a nested selector.
7. Make a mixin with two @content statements, then identify the duplicated CSS.
8. Write a mixin that declares a local variable and confirm that the content block cannot see it.
9. Rewrite the previous exercise to pass the value explicitly to the content block.
10. Decide whether a component's custom styles should be a mixin argument or a content block, and explain why.

## Summary

- @content marks where a caller-provided block is inserted into a mixin.
- An @include can provide a block of declarations, selectors, and other valid Sass statements.
- Context-aware mixins can wrap that block in a selector state, media query, or other context.
- Content blocks are lexically scoped at the include site; they cannot read a mixin's local variables.
- @content(value) and using (value) pass values explicitly from the mixin to its content block.
- Every @content statement inserts the supplied block, so multiple statements produce repeated output.
- Use regular parameters for values and content blocks for caller-supplied Sass statements.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="019-mixin-arguments-defaults-and-keyword-args.md">019 - Mixin Arguments, Defaults, and Keyword Arguments</a></td><td><a href="../qnas/020-content-blocks-and-context-aware-mixins.md">Q&amp;A</a></td><td><a href="../examples/020-content-blocks-and-context-aware-mixins.md">Example</a></td><td><a href="021-custom-functions-and-return-values.md">021 - Custom Functions and Return Values</a></td></tr>
  </tbody>
</table>
