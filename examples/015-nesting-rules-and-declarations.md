# 015 — Nesting Rules and Declarations: Examples

Examples below show SCSS source beside the CSS produced by Dart Sass. Mixed declaration output assumes Dart Sass 1.92.0 or later.

## 1. Descendant nesting

~~~scss
.article {
  max-width: 68ch;

  h2 {
    margin-block: 2rem 0.75rem;
  }

  p {
    line-height: 1.65;
  }
}
~~~

~~~css
.article {
  max-width: 68ch;
}
.article h2 {
  margin-block: 2rem 0.75rem;
}
.article p {
  line-height: 1.65;
}
~~~

A plain nested selector becomes a descendant selector with a space.

## 2. Selector-list expansion

~~~scss
.notice,
.warning {
  p,
  ul {
    margin-block: 0.75rem;
  }
}
~~~

~~~css
.notice p,
.notice ul,
.warning p,
.warning ul {
  margin-block: 0.75rem;
}
~~~

Two outer selectors times two inner selectors produce four combinations.

## 3. Combinators

~~~scss
.menu {
  > li {
    list-style: none;
  }
}

.heading {
  + p {
    margin-block-start: 0;
  }
}

.tab {
  ~ .tab-panel {
    border-block-start: 1px solid #d0d5dd;
  }
}
~~~

~~~css
.menu > li {
  list-style: none;
}
.heading + p {
  margin-block-start: 0;
}
.tab ~ .tab-panel {
  border-block-start: 1px solid #d0d5dd;
}
~~~

The child combinator > selects direct children. The adjacent-sibling combinator + selects the next sibling. The general-sibling combinator ~ selects later siblings with the same parent.

## 4. Nested properties

~~~scss
.label {
  font: {
    family: Arial, sans-serif;
    size: 0.875rem;
    weight: 600;
  }

  transition: {
    property: opacity, transform;
    duration: 180ms;
    timing-function: ease-out;
  }
}
~~~

~~~css
.label {
  font-family: Arial, sans-serif;
  font-size: 0.875rem;
  font-weight: 600;
  transition-property: opacity, transform;
  transition-duration: 180ms;
  transition-timing-function: ease-out;
}
~~~

Sass joins each nested property name to its prefix using a hyphen. The output is still separate CSS longhand properties.

## 5. Shorthand with nested longhands

~~~scss
.page {
  margin: 1rem {
    top: 0;
    inline: auto;
  }
}
~~~

~~~css
.page {
  margin: 1rem;
  margin-top: 0;
  margin-inline: auto;
}
~~~

The shorthand is emitted first. Later longhands can override parts of it according to the CSS cascade.

## 6. Declarations interleaved with a nested rule

~~~scss
.card {
  color: #202124;

  .badge {
    color: white;
  }

  border: 1px solid #d0d5dd;
}
~~~

With Dart Sass 1.92.0 or later:

~~~css
.card {
  color: #202124;
}
.card .badge {
  color: white;
}
.card {
  border: 1px solid #d0d5dd;
}
~~~

The repeated .card selector preserves source order. Older Sass behavior moved declarations before nested rules; use the version configured by the project when checking output.

## 7. Keep parent declarations together

~~~scss
.card {
  color: #202124;
  border: 1px solid #d0d5dd;

  .badge {
    color: white;
  }
}
~~~

~~~css
.card {
  color: #202124;
  border: 1px solid #d0d5dd;
}
.card .badge {
  color: white;
}
~~~

Grouping declarations before the nested block keeps the generated parent rule together when that order matches the intended cascade.

## 8. A nested media query

~~~scss
.layout {
  padding-inline: 1rem;

  @media (min-width: 48rem) {
    max-width: 72rem;
    margin-inline: auto;
  }
}
~~~

~~~css
.layout {
  padding-inline: 1rem;
}
@media (min-width: 48rem) {
  .layout {
    max-width: 72rem;
    margin-inline: auto;
  }
}
~~~

The nested selector is emitted inside the media condition.

## 9. Review selector depth

~~~scss
.application {
  .content {
    .sidebar {
      .navigation {
        a {
          color: #175cd3;
        }
      }
    }
  }
}
~~~

~~~css
.application .content .sidebar .navigation a {
  color: #175cd3;
}
~~~

If every ancestor is not required for the intended match, a shorter source selector may be clearer:

~~~scss
.navigation a {
  color: #175cd3;
}
~~~

~~~css
.navigation a {
  color: #175cd3;
}
~~~

Use the shorter selector only if it matches the intended links without affecting unrelated content.

## 10. Compile and practice

Save the source in src/nesting.scss and compile it:

~~~sh
sass src/nesting.scss dist/nesting.css
~~~

Open the generated file and compare the selectors and declaration order.

Practice tasks:

1. Nest a ul rule inside .breadcrumbs and set list-style and margin.
2. Add a direct-child rule for li elements inside .breadcrumbs.
3. Nest p and ul inside both .notice and .warning, then count the output selector combinations.
4. Nest font-family, font-size, and font-weight inside .caption.
5. Put a nested .badge rule between two .card declarations and inspect output with Dart Sass 1.92.0 or later.
6. Move both .card declarations before the nested rule and compare.
7. Add a max-width declaration under a media query nested inside .layout.
8. Simplify a deep selector after checking which ancestor relationships the markup actually requires.

When reviewing output, check that descendants, direct children, and siblings are not confused; that nested property names are joined as expected; and that mixed declarations retain the intended cascade order.
