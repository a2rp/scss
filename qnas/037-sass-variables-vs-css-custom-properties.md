# 037 — Sass Variables versus CSS Custom Properties: Q&A

## Core differences

### 1. What is the main difference between a Sass variable and a CSS custom property?

A Sass variable is evaluated by Sass during compilation. A CSS custom property is emitted as CSS and resolved by the browser through the cascade and inheritance.

### 2. Does a Sass variable appear in the compiled CSS?

The Sass variable itself does not. Sass replaces references with the value it computed. A custom property declaration does appear in the output.

### 3. What does a CSS custom property look like?

It is a CSS declaration whose name starts with two hyphens, such as `--brand: #1769aa;`. Use `var(--brand)` to reference its value in another CSS property.

### 4. Can Sass use a CSS custom property as though it were a Sass variable?

No. Sass does not know the browser's runtime computed value. It cannot use `var(--brand)` as a Sass map key or use its browser-resolved value in a Sass calculation.

### 5. Can a browser read Sass variables?

No. Sass variables are compiled away. Use a CSS custom property when the browser or JavaScript needs to read, inherit, or change a token.

### 6. Are CSS custom properties always global?

No. Their values are determined by CSS rules, the cascade, and inheritance. A custom property can be declared on the root, a theme container, or an individual component.

## Sass compilation and interpolation

### 7. Why must I interpolate a Sass value in a custom property declaration?

Sass treats custom property values as largely arbitrary CSS token streams. It passes Sass-looking expressions through unless interpolation asks it to evaluate and insert a Sass value.

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}
~~~

### 8. What happens if I write `--brand: $brand;` without interpolation?

Sass generally emits the literal text `$brand` in that custom property. It does not substitute the Sass variable there.

### 9. Do I need interpolation in every custom property value?

No. Use interpolation when inserting a Sass expression. Plain CSS values such as `--brand: #1769aa;` or `--shadow: 0 1px 2px black;` can be written directly.

### 10. How do I put a Sass calculation in a custom property?

Interpolate the calculation so Sass resolves it at build time:

~~~scss
$unit: 0.25rem;
--space: #{$unit * 4};
~~~

The emitted custom property contains a concrete value such as `1rem`.

### 11. Does interpolation preserve quotes around a Sass string?

No. Interpolation removes quotes from strings. Use `meta.inspect()` from `sass:meta` when the quoted representation must be preserved in the custom property value.

### 12. Can a Sass map generate CSS custom properties?

Yes. Use Sass control flow to iterate the map and interpolation to create each dashed property name and value. The map and loop are evaluated by Sass; the resulting custom properties are CSS.

### 13. Can a Sass variable change earlier CSS after reassignment?

No. Sass variables are imperative. Earlier declarations keep the value they used when Sass evaluated them; reassignment affects later Sass statements.

### 14. Can a Sass variable be inherited by a nested Sass file?

Sass variables follow Sass scope rules, not CSS inheritance. In module-based code, a file must load the Sass module whose members it uses. CSS custom properties follow CSS cascade and inheritance rules in the browser.

## CSS cascade, inheritance, and runtime

### 15. Do unregistered custom properties inherit by default?

Yes. An ordinary custom property inherits from its parent unless a value is set on the current element. A property registered with `@property` can specify different inheritance behavior.

### 16. Can two elements use different values for the same custom property?

Yes. A rule can set a different value on a component, theme wrapper, or other element. Descendants inherit the value in their CSS context unless they override it.

### 17. Can I switch a theme without recompiling Sass?

Yes. Define different custom property values under theme selectors, such as a class or data attribute. Changing the active selector changes which CSS values the browser applies.

### 18. Can I change a custom property with JavaScript?

Yes. JavaScript can update an element's inline custom property or use the CSS Object Model. Any CSS declarations that reference it can then use the updated value after style recalculation.

### 19. Can custom properties be set on a single component?

Yes. Define the property on the component or modifier class. This creates a local override that can be inherited by its descendants.

### 20. Does CSS inheritance mean every CSS property inherits?

No. Inheritance depends on the property. Ordinary unregistered custom properties inherit by default, while many normal CSS properties do not. A registered custom property can declare whether it inherits.

### 21. Are Sass variables and custom properties resolved at the same time?

No. Sass variables are resolved during compilation. Custom property values are resolved by CSS processing in the browser and can participate in runtime cascade, inheritance, and changes.

## var() and fallback behavior

### 22. What does the second argument to var() do?

It is a fallback used when the referenced custom property is missing or has its guaranteed-invalid value:

~~~css
color: var(--text-color, #263238);
~~~

The fallback is CSS text; it does not define the custom property.

### 23. Does a var() fallback validate the custom property's type?

No. If a custom property exists and substitutes successfully but makes the consuming property invalid, the browser does not retry the fallback. A fallback is not a type-checking system.

### 24. Can the fallback itself be another var() reference?

Yes. Fallbacks can contain other CSS values, including another `var()` call:

~~~css
color: var(--local-text, var(--theme-text, #263238));
~~~

The browser can use the next fallback when the earlier referenced custom property is missing or invalid.

### 25. Can Sass replace a var() fallback at build time?

Not unless you explicitly interpolate a Sass expression into the declaration. Otherwise, `var()` and its fallback are CSS for the browser to evaluate.

### 26. Can a CSS custom property be used as a media-query breakpoint?

No. CSS custom properties are not available as media feature values. A Sass variable can generate a literal breakpoint during compilation; a browser can instead use CSS media query conditions directly.

### 27. Can I use a custom property inside calc()?

Yes. Custom properties can hold values used in CSS calculations, as long as the substituted tokens form a valid expression:

~~~css
:root {
  --space-unit: 0.25rem;
}

.card {
  padding: calc(var(--space-unit) * 4);
}
~~~

The browser resolves the expression.

### 28. What happens if custom properties refer to each other in a cycle?

A cyclic dependency on the same element makes the involved custom properties invalid at computed-value time. Avoid self-references and circular token chains.

## Registration with @property

### 29. What does @property add?

It can declare a custom property's syntax, whether it inherits, and its initial value. Browser support determines whether the registration behavior is applied.

### 30. Does @property make a custom property a Sass variable?

No. It remains a CSS custom property. The browser can validate and process it according to the registration.

### 31. What does the inherits descriptor control?

It controls whether a registered custom property inherits its value from its parent. This lets a CSS author choose inheritance behavior for a registered token.

### 32. What is the syntax descriptor for?

It describes the allowed value type or grammar, such as `"<length>"` or `"<percentage>"`. The browser can reject values that do not match the declared syntax.

### 33. When is an initial-value required in @property?

It is required when the syntax is not the universal syntax, `*`. The initial value must match the declared syntax and be computationally independent. If syntax is `*`, initial-value may be omitted. The syntax and inherits descriptors are required for a valid registration in either case.

## Choosing and combining the two

### 34. When should I use a Sass variable?

Use it for compile-time calculations, maps, mixins, functions, selector generation, module configuration, and build-time conditions.

### 35. When should I use a CSS custom property?

Use it for values that should respond to the CSS cascade, inherit through the document, vary by theme or component, be changed at runtime, or be accessed by JavaScript.

### 36. Can I use both for one token?

Yes. Sass can hold the build-time default and emit it as a custom property. The browser can then override that CSS token at runtime.

### 37. Is @use with !default a replacement for a runtime theme?

No. Sass module configuration sets values when Sass compiles a stylesheet. Custom properties support browser-time changes without recompiling.

### 38. Should I define the same design token in both Sass and CSS?

Avoid maintaining duplicate values independently. If both are needed, choose a clear source of truth, such as a Sass map that generates custom property defaults, and document where runtime overrides are allowed.

### 39. Do CSS custom properties always improve performance?

No universal rule follows from using them. Choose based on behavior, cascade needs, maintainability, and actual application performance measurements. Do not choose a token mechanism from an unsupported performance assumption.

### 40. How do I debug a custom property value?

Inspect the element in browser developer tools. Check which rule wins in the cascade, whether the property is inherited or overridden, and the computed value used by the declaration referencing it.

### 41. What is a quick decision rule?

If Sass must compute the value before CSS is sent to the browser, start with a Sass variable. If the browser must choose or change the value based on element context or runtime state, start with a CSS custom property.

## Quick reference

~~~scss
@use "sass:meta";

$brand: #1769aa;
$font-stack: ("Field Sans", sans-serif);

:root {
  --brand: #{$brand};
  --font-stack: #{meta.inspect($font-stack)};
}

.button {
  color: var(--button-text, #263238);
  background: var(--brand);
}
~~~

- Sass variables are build-time values and compile away.
- CSS custom properties are emitted and participate in the browser's cascade.
- Interpolate Sass values inside custom property declarations.
- Use `var()` to reference a CSS custom property.
- Use `@property` for optional browser-side syntax and inheritance registration.

## Related material

- [Lesson: Sass variables versus CSS custom properties](../lessons/037-sass-variables-vs-css-custom-properties.md)
- [Examples: Sass variables versus CSS custom properties](../examples/037-sass-variables-vs-css-custom-properties.md)
- [Sass: Variables](https://sass-lang.com/documentation/variables/)
- [Sass: Custom property declarations](https://sass-lang.com/documentation/style-rules/declarations/#custom-properties)
- [Sass: Interpolation](https://sass-lang.com/documentation/interpolation/)
- [CSS Custom Properties for Cascading Variables](https://www.w3.org/TR/css-variables/)
- [CSS Properties and Values API: `@property`](https://www.w3.org/TR/css-properties-values-api-1/)
