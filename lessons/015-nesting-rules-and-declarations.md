# 015 - Nesting Rules and Declarations

## Learning goals

After this lesson, you should be able to:

- Explain how a nested style rule is combined with its parent selector.
- Read the CSS Sass generates from nested SCSS.
- Nest selector lists and understand their Cartesian-product output.
- Use child, adjacent-sibling, and general-sibling combinators in nested rules.
- Nest declarations for properties with shared prefixes, such as font and transition.
- Predict the output when declarations appear before and after nested rules.
- Keep nesting shallow and avoid accidental selector complexity.
- Distinguish Sass nesting from native CSS nesting.

## 1. Why nesting exists

A stylesheet often repeats the same selector for related states or descendants. SCSS lets you place a related rule inside its parent rule. Sass combines the selectors while compiling the file.

Source SCSS:

~~~scss
.card {
  padding: 1rem;
  background-color: white;

  h2 {
    margin-block: 0;
  }

  p {
    color: #454545;
  }
}
~~~

Generated CSS:

~~~css
.card {
  padding: 1rem;
  background-color: white;
}
.card h2 {
  margin-block: 0;
}
.card p {
  color: #454545;
}
~~~

The nested h2 rule means a descendant h2 inside .card. It does not mean every h2 in the document.

Nesting is a source-organization feature. Sass emits ordinary CSS selectors, so the browser receives the compiled selector rules.

## 2. The default descendant relationship

When a nested selector has no explicit combinator, Sass places a space between the outer and inner selectors. This is the descendant combinator.

~~~scss
.article {
  .title {
    font-size: 1.5rem;
  }
}
~~~

The generated selector is .article .title. It matches an element with class title anywhere inside an element with class article, including at multiple descendant levels.

It does not match an element that merely has both classes. That relationship is written differently and will be covered in the next lesson about the parent selector.

## 3. Nest by component relationship

Nesting is easiest to understand when the indentation reflects a real relationship in the markup.

~~~scss
.site-header {
  .navigation {
    display: flex;
  }

  .brand {
    font-weight: 700;
  }
}
~~~

This generates two selectors:

~~~css
.site-header .navigation {
  display: flex;
}
.site-header .brand {
  font-weight: 700;
}
~~~

The nested blocks do not create new HTML elements. They only construct selectors for elements that already exist in the document.

A nested selector can also be an element, class, ID, attribute selector, pseudo-class, or another valid CSS selector:

~~~scss
.form {
  input {
    font: inherit;
  }

  [aria-invalid="true"] {
    border-color: #b42318;
  }

  :focus-visible {
    outline: 3px solid #175cd3;
  }
}
~~~

The plain nested pseudo-class above targets a descendant matching :focus-visible. To attach a pseudo-class to the parent itself, Sass's parent selector is used; that syntax is the subject of lesson 016.

## 4. Nesting selector lists

A parent rule and a nested rule can each contain comma-separated selector lists. Sass combines every outer selector with every inner selector.

~~~scss
.notice,
.alert {
  p,
  ul {
    margin-block: 0.75rem;
  }
}
~~~

Generated CSS:

~~~css
.notice p,
.notice ul,
.alert p,
.alert ul {
  margin-block: 0.75rem;
}
~~~

Two outer selectors multiplied by two inner selectors create four selector combinations. A longer list at both levels can grow quickly.

Before nesting two lists, count the possible combinations and check whether all of them are intentional. If the result is difficult to predict, write the selectors separately.

## 5. Use combinators to express the exact relationship

A space creates a descendant relationship. Other combinators can make the intended relationship explicit.

### Direct children

~~~scss
.menu {
  > li {
    list-style: none;
  }
}
~~~

Generated CSS:

~~~css
.menu > li {
  list-style: none;
}
~~~

The child combinator > matches only li elements whose parent is the element matching .menu.

### Adjacent siblings

~~~scss
.heading {
  + p {
    margin-block-start: 0;
  }
}
~~~

Generated CSS:

~~~css
.heading + p {
  margin-block-start: 0;
}
~~~

The adjacent-sibling combinator + matches the next p element when it has the same parent as the .heading element.

### General siblings

~~~scss
.tab {
  ~ .tab-panel {
    border-block-start: 1px solid #d0d5dd;
  }
}
~~~

Generated CSS:

~~~css
.tab ~ .tab-panel {
  border-block-start: 1px solid #d0d5dd;
}
~~~

The general-sibling combinator ~ matches later .tab-panel siblings that share a parent with .tab.

Sass permits a combinator at the start of a nested selector. Use this only when the relationship is clear to someone reading the parent rule.

## 6. Nested property declarations

Sass also allows a property name to contain a nested block. Sass joins the outer and inner property names with a hyphen.

~~~scss
.button {
  font: {
    family: Arial, sans-serif;
    size: 1rem;
    weight: 600;
  }
}
~~~

Generated CSS:

~~~css
.button {
  font-family: Arial, sans-serif;
  font-size: 1rem;
  font-weight: 600;
}
~~~

This is property nesting. It is different from selector nesting: the block after font: contains property suffixes, not selectors.

Common uses include font, text-decoration, transition, animation, and list-style properties. Use it only when the shared prefix makes the source easier to scan.

~~~scss
.panel {
  transition: {
    property: opacity, transform;
    duration: 180ms;
    timing-function: ease-out;
  }
}
~~~

Generated CSS:

~~~css
.panel {
  transition-property: opacity, transform;
  transition-duration: 180ms;
  transition-timing-function: ease-out;
}
~~~

The generated CSS still contains separate longhand declarations. Sass does not automatically combine them into a shorthand declaration.

## 7. Combining a shorthand with nested longhands

For a property that has a shorthand form, the shorthand value can be followed by nested longhand declarations.

~~~scss
.page {
  margin: 1rem {
    top: 0;
    inline: auto;
  }
}
~~~

Generated CSS:

~~~css
.page {
  margin: 1rem;
  margin-top: 0;
  margin-inline: auto;
}
~~~

Source order matters because CSS declarations can override earlier declarations. Here the later margin-top and margin-inline declarations override the corresponding sides of the shorthand where those properties apply.

Use this form only when the CSS cascade and override are intentional. If a reader has to reconstruct several shorthand overrides, ordinary declarations may be clearer.

## 8. Declarations around nested rules

A style-rule block can contain both declarations and nested rules. Dart Sass preserves their written order.

~~~scss
.card {
  color: #202124;

  .badge {
    color: white;
  }

  border: 1px solid #d0d5dd;
}
~~~

With Dart Sass 1.92.0 or later, the generated CSS is:

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

Sass emits the .card rule twice so the border declaration stays after the nested badge rule in CSS source order. The duplicate selector is intentional; it preserves the behavior of the source.

Older Sass behavior moved declarations to the beginning of the parent rule. Dart Sass deprecated that behavior in 1.77.7 and changed to source-order behavior in 1.92.0. When working with an older compiler, check its version and warnings before relying on the ordering shown here.

### Make grouping explicit when helpful

If declarations should belong together before the nested rule, place them there:

~~~scss
.card {
  color: #202124;
  border: 1px solid #d0d5dd;

  .badge {
    color: white;
  }
}
~~~

If a declaration must appear after a nested rule, its position is meaningful. Do not move it merely to avoid seeing a repeated selector in generated CSS.

Clear grouping helps both code review and cascade debugging.

## 9. Nesting plain CSS at-rules

Some CSS at-rules can be nested inside a style rule. Sass carries the selector into the at-rule context.

~~~scss
.notice {
  color: #344054;

  @media (min-width: 48rem) {
    max-width: 42rem;
  }
}
~~~

Generated CSS:

~~~css
.notice {
  color: #344054;
}
@media (min-width: 48rem) {
  .notice {
    max-width: 42rem;
  }
}
~~~

The media query does not become part of the selector. It wraps the matching CSS so that the declaration applies only when the media condition is true.

At-rules have different rules about what may appear inside them. Consult the at-rule documentation when nesting something other than a common conditional rule such as @media or @supports.

## 10. Sass nesting and native CSS nesting are not identical

Sass processes its nesting at compile time and emits selectors in CSS. Native CSS nesting is parsed by the browser. The two systems have overlapping syntax, but they are not interchangeable in every case.

For example, Sass permits selector suffixes through the parent selector, and Sass selector-list expansion can differ from the browser's native nesting behavior. Lesson 016 covers Sass's parent selector syntax. Do not assume that copying a Sass nested block directly into a CSS file has the same meaning.

For this lesson, the key distinction is:

- SCSS is compiled; its nested rules become ordinary selectors in generated CSS.
- Native CSS nesting remains nested in the CSS file for the browser to interpret.

## 11. Keep nesting shallow

A few levels can make related component rules easy to find. Deep nesting can make the generated selector long, increase specificity, and hide which markup the rule matches.

Hard to scan:

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

This generates a selector tied to the entire ancestor chain:

~~~css
.application .content .sidebar .navigation a {
  color: #175cd3;
}
~~~

Prefer selectors that express the component or state directly, with only the relationships the design requires. A useful review habit is to read the generated selector aloud and ask whether every ancestor is necessary.

Nesting should organize source code, not encourage a one-to-one copy of every HTML indentation level.

## 12. Common mistakes

### Mistake: assuming a nested selector targets the parent

~~~scss
.button {
  :hover {
    color: red;
  }
}
~~~

This targets a hovered descendant of .button. It does not target .button when it is hovered. Lesson 016 explains the parent selector syntax for that case.

### Mistake: assuming indentation changes HTML

Nested rules do not create DOM nodes or change markup. They only compile to selectors.

### Mistake: overlooking selector-list expansion

Nesting two comma-separated lists can multiply the number of selectors. Inspect the generated CSS when the combinations matter.

### Mistake: nesting every declaration group

Property nesting is optional syntax, not a requirement. Do not use it if it makes a declaration harder to recognize.

### Mistake: changing declaration order to hide duplicate output

With modern Dart Sass, declarations after a nested rule remain after it in the generated CSS. Move a declaration only when changing the cascade is acceptable.

### Mistake: copying SCSS nesting into a CSS file

Sass selector nesting and native CSS nesting have differences. Compile SCSS with Sass, or deliberately author CSS according to browser-supported CSS nesting semantics.

## 13. A practical component example

Source SCSS:

~~~scss
.product-card {
  padding: 1rem;
  border: 1px solid #d0d5dd;
  border-radius: 0.75rem;

  .product-card__title {
    margin-block: 0;
    font-size: 1.125rem;
  }

  .product-card__description {
    color: #475467;
    line-height: 1.5;
  }

  .product-card__meta {
    font: {
      size: 0.875rem;
      weight: 600;
    }
  }

  @media (min-width: 48rem) {
    max-width: 30rem;
  }
}
~~~

The selectors remain scoped to .product-card, and the font block expands into font-size and font-weight. The media rule places the max-width declaration under the matching condition.

For a real project, compare this source with its generated CSS and check whether each selector and nesting level is still easy to maintain.

## 14. Exercises

1. Nest a ul rule inside .breadcrumbs and reset its list-style and margin.
2. Add a direct-child rule for li elements inside .breadcrumbs.
3. Nest a paragraph and a list under both .notice and .warning. Write down the expected selector combinations before compiling.
4. Use property nesting to write font-family, font-size, and font-weight inside a .label rule.
5. Put a nested .badge rule between two .card declarations. Compile with a current Dart Sass version and inspect the emitted order.
6. Move the declarations in exercise 5 before the nested rule. Compare the output and explain why the second .card rule disappeared.
7. Nest a max-width declaration under a media query inside .layout, then verify the generated CSS.
8. Simplify a deeply nested selector by removing one ancestor relationship that is not required by the markup.

## Summary

- A nested style rule is combined with its parent selector; a plain nested selector uses a descendant relationship.
- Selector lists at both levels expand into all outer-and-inner combinations.
- Explicit combinators describe child and sibling relationships.
- Nested property blocks join their property names with hyphens.
- Modern Dart Sass preserves declaration order around nested rules, even when it emits the parent selector more than once.
- Sass nesting compiles to CSS; it is not exactly the same as native CSS nesting.
- Keep nesting shallow and verify the generated selectors.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="014-comments-and-css-output.md">014 - Comments and Generated CSS Output</a></td><td><a href="../qnas/015-nesting-rules-and-declarations.md">Q&amp;A</a></td><td><a href="../examples/015-nesting-rules-and-declarations.md">Example</a></td><td><a href="016-parent-selector.md">016 - The Parent Selector</a></td></tr>
  </tbody>
</table>
