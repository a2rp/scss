# 018 — Mixin Basics: Examples

These examples use argument-free Sass mixins and show the CSS emitted at each include.

## 1. Define and include a reset mixin

Source SCSS:

~~~scss
@mixin list-reset {
  margin: 0;
  padding: 0;
  list-style: none;
}

.navigation ul {
  @include list-reset;
}
~~~

Generated CSS:

~~~css
.navigation ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
~~~

The mixin definition emits no CSS on its own. The include inserts its declarations into the .navigation ul rule.

## 2. Reuse a group of form-control declarations

Source SCSS:

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

Each include expands into its selector. The extra declarations remain local to the component rule.

## 3. Include one mixin inside another

Source SCSS:

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

.breadcrumbs ul {
  @include horizontal-list;
}
~~~

Generated CSS:

~~~css
.breadcrumbs ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
.breadcrumbs ul li {
  display: inline-block;
}
~~~

The horizontal-list mixin reuses list-reset and adds a nested rule. Its output is resolved at the include site.

## 4. Reuse a focus treatment in separate selectors

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

The selectors stay distinct while reusing the declaration group.

## 5. Include a mixin with a nested rule

Source SCSS:

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

The nested parent selector resolves in the .search-field context where the mixin is included.

## 6. Package a fixed media query

Source SCSS:

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

The mixin groups the normal transition with its reduced-motion override. Include it only where this same behavior belongs.

## 7. Include a mixin at the top level

A mixin may emit a standalone style rule when it is included at the top level.

Source SCSS:

~~~scss
@mixin skip-link {
  .skip-link {
    position: absolute;
    inset-block-start: 0;
    inset-inline-start: -100vw;
  }
}

@include skip-link;
~~~

Generated CSS:

~~~css
.skip-link {
  position: absolute;
  inset-block-start: 0;
  inset-inline-start: -100vw;
}
~~~

The mixin definition is not itself a CSS selector. The top-level include emits the rule.

## 8. Inspect repeated output

Source SCSS:

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

The declaration appears once at each include location. Mixins make Sass source reusable; they do not create a shared CSS reference.

## 9. Understand declaration order

Source SCSS:

~~~scss
@mixin surface {
  background-color: white;
  border: 1px solid #d0d5dd;
}

.panel {
  padding: 1rem;
  @include surface;
  color: #202124;
}
~~~

Generated CSS:

~~~css
.panel {
  padding: 1rem;
  background-color: white;
  border: 1px solid #d0d5dd;
  color: #202124;
}
~~~

The mixin declarations appear where the include occurs. Source order can matter when declarations set the same property or when nested rules are involved.

## 10. A component using several focused mixins

Source SCSS:

~~~scss
@mixin control-base {
  box-sizing: border-box;
  min-height: 2.75rem;
  padding: 0.625rem 0.875rem;
  border: 1px solid #98a2b3;
  border-radius: 0.5rem;
  font: inherit;
}

@mixin keyboard-focus {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}

.search-input {
  @include control-base;
  width: 100%;

  &:focus-visible {
    @include keyboard-focus;
  }
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
.search-input:focus-visible {
  outline: 3px solid #84caff;
  outline-offset: 3px;
}
~~~

The base declarations and focus treatment have separate names and are included in their relevant contexts.

## 11. Compile a practice stylesheet

Save the source in src/mixins.scss, then compile it with Dart Sass:

~~~sh
sass src/mixins.scss dist/mixins.css
~~~

Open the generated CSS and locate every @include from the source. The Sass at-rules will be gone; their declarations and rules will appear at the include sites.

## 12. Practice tasks

1. Define a list-reset mixin and use it on a navigation list.
2. Create a control-base mixin and include it in an input and a select.
3. Create a keyboard-focus mixin and include it under two focus-visible selectors.
4. Make a horizontal-list mixin that includes list-reset and emits a nested li rule.
5. Put a nested pseudo-element rule in a mixin and include it inside a component selector.
6. Include one mixin twice and identify each copy in the generated CSS.
7. Include a standalone utility rule mixin at the top level and inspect where it is emitted.
8. Decide whether a shared style should be an HTML class or a Sass mixin, then explain why.

## Output review checklist

- Is the mixin definition separate from the @include that emits its styles?
- Does each include produce the expected declarations and nested rules?
- Did nested selectors resolve in the include context?
- Are repeated output blocks intentional?
- Is the mixin focused on one reusable purpose?
