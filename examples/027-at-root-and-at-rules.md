# 027 — @at-root and Nested At-Rules: Examples

These examples show how nested at-rules and `@at-root` affect compiled CSS. Compile each example on its own so that the output is easy to compare.

## 1. Bubble a component-specific media rule

### SCSS

~~~scss
.notice {
  display: block;
  padding: 1rem;

  @media (min-width: 48rem) {
    display: grid;
    grid-template-columns: 12rem 1fr;
    gap: 1.5rem;
  }
}
~~~

### CSS output

~~~css
.notice {
  display: block;
  padding: 1rem;
}

@media (min-width: 48rem) {
  .notice {
    display: grid;
    grid-template-columns: 12rem 1fr;
    gap: 1.5rem;
  }
}
~~~

The source keeps the responsive variation beside `.notice`. Sass moves the media rule outward and nests the selector inside it.

## 2. Combine nested media conditions

### SCSS

~~~scss
.gallery {
  display: block;

  @media (hover: hover) {
    @media (min-width: 60rem) {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
    }
  }
}
~~~

### CSS output

Sass can combine compatible nested conditions:

~~~css
.gallery {
  display: block;
}

@media (hover: hover) and (min-width: 60rem) {
  .gallery {
    display: grid;
    grid-template-columns: repeat(3, minmax(0, 1fr));
  }
}
~~~

Confirm the output when combining complex queries or targeting a particular compiler version.

## 3. Keep a @supports enhancement beside its fallback

### SCSS

~~~scss
.site-header {
  position: relative;
  background-color: #172033;

  @supports (backdrop-filter: blur(0.5rem)) {
    position: sticky;
    inset-block-start: 0;
    backdrop-filter: blur(0.5rem);
    background-color: rgb(23 32 51 / 85%);
  }
}
~~~

### CSS output

~~~css
.site-header {
  position: relative;
  background-color: #172033;
}

@supports (backdrop-filter: blur(0.5rem)) {
  .site-header {
    position: sticky;
    inset-block-start: 0;
    backdrop-filter: blur(0.5rem);
    background-color: rgb(23 32 51 / 85%);
  }
}
~~~

The fallback stays outside the feature query. Browsers that do not support the tested property can still use the base declarations.

## 4. Put an already-complete selector at the root

### SCSS

~~~scss
.theme-dark {
  .button {
    border-color: #64748b;

    @at-root .button--contrast {
      color: #fff;
      background-color: #172033;
    }
  }
}
~~~

### CSS output

~~~css
.theme-dark .button {
  border-color: #64748b;
}

.button--contrast {
  color: #fff;
  background-color: #172033;
}
~~~

The selector written after `@at-root` is emitted without `.theme-dark .button` being added. This is only correct if a global `.button--contrast` is intended; it no longer depends on the dark theme ancestor.

## 5. Keep a media context while leaving selector nesting

### SCSS

~~~scss
@media print {
  .invoice {
    color: #222;

    .invoice__actions {
      display: none;
    }

    @at-root .print-watermark {
      display: block;
      opacity: 0.15;
    }
  }
}
~~~

### CSS output

~~~css
@media print {
  .invoice {
    color: #222;
  }

  .invoice .invoice__actions {
    display: none;
  }

  .print-watermark {
    display: block;
    opacity: 0.15;
  }
}
~~~

Plain `@at-root` removes style-rule nesting. The enclosing `@media print` remains, so the watermark is still print-only. The nested actions selector retains its ordinary relationship to `.invoice`.

## 6. Leave the media context with a query

### SCSS

~~~scss
@media print {
  .invoice {
    @at-root (without: media) {
      .print-watermark {
        position: fixed;
        inset-block-start: 1rem;
        opacity: 0.15;
      }
    }
  }
}
~~~

### CSS output

~~~css
.print-watermark {
  position: fixed;
  inset-block-start: 1rem;
  opacity: 0.15;
}
~~~

The query removes the style-rule context and the enclosing media context. Use this only when the watermark should appear outside print styles too.

## 7. Keep a style rule and discard at-rule context

### SCSS

~~~scss
@media print {
  .report {
    @at-root (with: rule) {
      .report__title {
        font-weight: 700;
      }
    }
  }
}
~~~

### CSS output

~~~css
.report__title {
  font-weight: 700;
}
~~~

`with: rule` keeps style rules and excludes at-rules. The generated title rule therefore leaves the surrounding print query and `.report` selector.

## 8. Generate related element selectors with selector.unify()

### SCSS

~~~scss
@use "sass:selector";

@mixin style-form-control($element) {
  @at-root #{selector.unify(&, $element)} {
    @content;
  }
}

.form-control {
  @include style-form-control("input") {
    border: 1px solid #64748b;
    padding: 0.625rem 0.75rem;
  }

  @include style-form-control("select") {
    border: 1px solid #64748b;
    padding: 0.625rem 0.75rem;
  }
}
~~~

### CSS output

~~~css
input.form-control {
  border: 1px solid #64748b;
  padding: 0.625rem 0.75rem;
}

select.form-control {
  border: 1px solid #64748b;
  padding: 0.625rem 0.75rem;
}
~~~

The selector function combines the current class with each element type. `@at-root` prevents Sass from adding `.form-control` again as an ancestor. The `@content` block supplies the declarations.

## 9. Build a responsive utility with a nested media rule

### SCSS

~~~scss
@mixin hide-below($breakpoint) {
  @media (width < $breakpoint) {
    @content;
  }
}

.sidebar {
  width: 18rem;

  @include hide-below(48rem) {
    display: none;
  }
}
~~~

### CSS output

~~~css
.sidebar {
  width: 18rem;
}

@media (width < 48rem) {
  .sidebar {
    display: none;
  }
}
~~~

A mixin can package a repeated conditional wrapper. The emitted media syntax still depends on browser support. For projects supporting browsers without range syntax, use a compatible min/max form.

## 10. Keep keyframe children valid

### SCSS

~~~scss
@keyframes slide-in {
  from {
    opacity: 0;
    transform: translateX(1rem);
  }

  70% {
    opacity: 1;
    transform: translateX(-0.125rem);
  }

  to {
    transform: translateX(0);
  }
}

.toast {
  animation: slide-in 240ms ease-out both;
}
~~~

### CSS output

~~~css
@keyframes slide-in {
  from {
    opacity: 0;
    transform: translateX(1rem);
  }

  70% {
    opacity: 1;
    transform: translateX(-0.125rem);
  }

  to {
    transform: translateX(0);
  }
}

.toast {
  animation: slide-in 240ms ease-out both;
}
~~~

Keyframes use valid keyframe selectors such as `from`, percentages, and `to`; they are not ordinary nested component selectors.

## 11. A small comparison

### SCSS

~~~scss
@media (min-width: 50rem) {
  .page {
    @at-root {
      .page__toolbar {
        display: flex;
      }
    }

    @at-root (without: media) {
      .page__print-note {
        display: block;
      }
    }
  }
}
~~~

### CSS output

~~~css
@media (min-width: 50rem) {
  .page__toolbar {
    display: flex;
  }
}

.page__print-note {
  display: block;
}
~~~

The first `@at-root` leaves the style-rule nesting but keeps the media condition. The second also removes the media condition.

## 12. Debug the output deliberately

When a nested result surprises you:

1. Find the nearest enclosing style rule and at-rule for the source block.
2. Write down the selector that should remain.
3. Decide whether each media or supports condition should remain.
4. Use plain `@at-root` or a query only for the context that must change.
5. Compile and compare the actual selector and at-rule nesting with the expected CSS.

## Further reading

- [Sass: @at-root](https://sass-lang.com/documentation/at-rules/at-root/)
- [Sass: CSS At-Rules](https://sass-lang.com/documentation/at-rules/css/)
- [Sass: Parent Selector](https://sass-lang.com/documentation/style-rules/parent-selector/)
- [Sass: sass:selector module](https://sass-lang.com/documentation/modules/selector/)
