# 020 - Content Blocks and Context-Aware Mixins: Examples

Each example shows a mixin that chooses a context and a caller-supplied block inserted into that context.

## 1. Insert caller declarations

Source SCSS:

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

The caller block is inserted where @content appears.

## 2. Put custom declarations inside a hover state

Source SCSS:

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
    border-color: #1849a9;
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
  border-color: #1849a9;
}
~~~

The mixin controls the hover selector, while the include supplies the hover declarations.

## 3. Wrap a block in a media query

Source SCSS:

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

The include occurs inside .article. Sass emits that selector inside the media query.

## 4. Reuse a responsive context at different breakpoints

Source SCSS:

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

The breakpoint is an ordinary mixin argument. The responsive declarations are the content block.

## 5. Pass the breakpoint to the content block

Source SCSS:

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

The mixin passes $minimum with @content($minimum). The content block receives that value under the name $breakpoint.

## 6. Put nested selectors in the content block

Source SCSS:

~~~scss
@mixin focus-styles {
  &:focus-visible {
    @content;
  }
}

.button {
  @include focus-styles {
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

Both the declarations and nested .icon rule are placed inside the focus-visible context.

## 7. Confirm lexical scope at the include site

Source SCSS:

~~~scss
@mixin insert-content {
  $mixin-color: #b42318;
  @content;
}

.card {
  $card-color: #175cd3;

  @include insert-content {
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

The content block can read $card-color because it is declared at the include site. It cannot read $mixin-color just because that variable exists inside the mixin. Pass mixin-local values explicitly with @content(...) if the content block needs them.

## 8. Insert the same content at two locations

Source SCSS:

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

The content is inserted for each @content statement. This behavior can produce duplicate output, so use multiple insertion points deliberately.

## 9. Combine mixin declarations with a content block

Source SCSS:

~~~scss
@mixin alert-frame {
  padding: 1rem;
  border: 1px solid currentColor;

  @content;
}

.alert {
  @include alert-frame {
    color: #b42318;
    background-color: #fef3f2;
  }
}
~~~

Generated CSS:

~~~css
.alert {
  padding: 1rem;
  border: 1px solid currentColor;
  color: #b42318;
  background-color: #fef3f2;
}
~~~

The mixin provides the shared frame. The caller supplies the alert-specific colors.

## 10. Compile and inspect

Save a practice file as src/content-mixins.scss and compile it with Dart Sass:

~~~sh
sass src/content-mixins.scss dist/content-mixins.css
~~~

Find every include and check:

- Which selector or at-rule contains the inserted block.
- Whether the include site contributes a parent selector.
- Whether values passed by @content(...) are declared with using (...).
- Whether the caller's variables are in the include site's lexical scope.
- Whether multiple @content statements duplicate the output.

## 11. Practice tasks

1. Write a hover mixin that excludes disabled controls and accepts custom hover declarations.
2. Wrap a content block in a media query at a fixed breakpoint.
3. Add a breakpoint parameter and reuse the mixin at two widths.
4. Pass the breakpoint to the caller block and use it in a declaration.
5. Put a nested selector inside a focus-visible content block.
6. Define a caller-local variable and use it from a content block.
7. Make a mixin-local value available by passing it explicitly to @content.
8. Create two @content insertion points and locate both generated rules.
9. Compare a value-only mixin argument with a block of caller statements.
10. Compile the stylesheet and read the generated CSS as ordinary browser CSS.

## Output review checklist

- Is the content block inserted at the intended @content location?
- Is it nested in the intended selector or at-rule?
- Does a content argument have a matching using parameter?
- Are variables read from the include site's scope?
- Does the mixin insert the content once or multiple times intentionally?
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous example</th><th>Chapter lesson</th><th>Chapter Q&amp;A</th><th>Next example</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="019-mixin-arguments-defaults-and-keyword-args.md">019 - Mixin Arguments, Defaults, and Keyword Arguments: Examples</a></td><td><a href="../lessons/020-content-blocks-and-context-aware-mixins.md">Lesson</a></td><td><a href="../qnas/020-content-blocks-and-context-aware-mixins.md">Q&amp;A</a></td><td><a href="021-custom-functions-and-return-values.md">021 - Custom Functions and Return Values: Examples</a></td></tr>
  </tbody>
</table>
