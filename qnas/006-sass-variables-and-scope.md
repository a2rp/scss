# 006. Sass Variables and Scope - Q&A

## Sass variable basics

### 1. How do I declare a Sass variable?

Start the name with a dollar sign, follow it with a colon and value, and end the statement with a semicolon:

~~~scss
$brand: #1769aa;
~~~

### 2. How do I use a Sass variable?

Reference its name in a Sass expression, such as a property value:

~~~scss
$brand: #1769aa;

.button {
  background-color: $brand;
}
~~~

Sass substitutes the value when it compiles the stylesheet.

### 3. Does a Sass variable appear in the generated CSS?

No. Sass variables are compile-time values. The generated CSS contains the resolved value, not the Sass variable declaration.

### 4. What kinds of values can Sass variables hold?

They can hold Sass values such as numbers, strings, colors, lists, maps, booleans, and null. The value type determines which operations are appropriate.

### 5. Should variable names describe the current value or the purpose?

Prefer the purpose. For example, $brand-color is usually more useful than $blue because the brand color may change later.

### 6. Are hyphens and underscores different in Sass variable names?

No. Sass identifiers treat hyphens and underscores as equivalent. $font-size and $font_size refer to the same variable, so do not use them as if they were separate settings.

## Reassignment and timing

### 7. What happens if I assign a new value to an existing Sass variable?

A normal assignment replaces its value for subsequent Sass evaluation. CSS already emitted for earlier uses is not retroactively changed.

### 8. Why is Sass variable behavior called imperative?

The value used depends on the order of the source. A use sees the current value at that point, and a later reassignment affects later uses.

### 9. Can a Sass variable change in the browser at runtime?

No. Sass variables are removed during compilation. Use a CSS custom property when a value must remain available to the browser or JavaScript.

### 10. Can Sass and CSS variables work together?

Yes. A Sass variable can provide a compiled value for a CSS custom property:

~~~scss
$brand: #1769aa;

:root {
  --brand: #{$brand};
}
~~~

The CSS custom property remains in the output; the Sass variable is substituted during compilation.

### 11. Why does a Sass variable inside a custom property need interpolation?

Sass passes custom-property values through as CSS text. Interpolation marks the Sass expression that should be evaluated and inserted into that text.

## Scope and shadowing

### 12. What is a top-level Sass variable?

It is a variable declared outside a nested rule or block. It is available later in the same stylesheet module.

### 13. What is a local Sass variable?

It is declared inside a block and is usually available only inside that block and its nested blocks.

### 14. What happens if a local variable has the same name as a top-level variable?

The local variable shadows the outer one within its block. The outer variable keeps its original value for uses outside that block.

### 15. Does a module loaded with @use place all its variables in the caller’s global scope?

No. A module has its own scope. Its public members are accessed through the module namespace, such as settings.$brand.

### 16. What is the purpose of module namespaces for variables?

They make it clear which module provides a variable and avoid accidental name collisions between modules.

### 17. Can I use a variable after its local block ends?

Not if the variable was declared only inside that block. Move the declaration to a broader scope or pass the value through an explicit module, mixin, or function design.

### 18. How do variables behave in flow-control rules?

Assignments to variables that already exist in the surrounding scope can update those variables. A variable created only inside a flow-control block does not become available outside it.

### 19. Should I rely on variables created inside an @if or loop being visible afterward?

No. Declare the variable in the scope where you need it. Flow-control scoping has special rules, so explicit initialization and use within a clear scope is easier to maintain.

## Defaults and globals

### 20. What does !default do?

It assigns a fallback value only when the variable is not set or its value is null. If a non-null value already exists, !default preserves it.

### 21. Does !default preserve false or zero?

Yes. It preserves any existing non-null value, including false and zero. The fallback is used when the variable is undefined or null.

### 22. When should I use !default?

Use it for settings that should have a default but may be supplied by a caller, commonly in a Sass library or configurable module.

### 23. How can a caller configure a module’s default variable?

The module declares a top-level variable with !default. The caller supplies an override when it first loads the module:

~~~scss
@use "theme" with (
  $brand: #7c3aed
);
~~~

Only top-level variables marked !default are configurable this way.

### 24. Can I configure the same module multiple times with different values in one compilation?

No. A Sass module is loaded once and keeps its configuration for that compilation. Provide the configuration when the module is first loaded.

### 25. What does !global do?

It assigns a variable in the top-level scope of the current stylesheet rather than the current local scope. Use it sparingly because shared mutable state makes code harder to follow.

### 26. Can !global create a new global variable from anywhere?

Do not use it that way. The supported pattern targets a variable already declared at the top level of the current stylesheet. Prefer explicit declarations and module configuration instead.

## Common mistakes

### 27. Why is a Sass variable printed literally in the CSS custom property?

The custom-property value is treated as CSS text. Write interpolation around the Sass expression, such as --brand: #{$brand};.

### 28. Why did a later reassignment not change an earlier rule?

Sass variable uses are evaluated in source order. The earlier rule was already compiled using the earlier value.

### 29. Why is a variable unavailable in a different stylesheet?

A variable declared in one module is not automatically global in another module. Load the module with @use and access its public variable through the namespace.

### 30. Why might two differently spelled variable names refer to the same value?

Hyphens and underscores are equivalent in Sass identifiers. For example, $font-size and $font_size are the same Sass variable name.

## Quick review

- **Sass variable:** A compile-time value whose name begins with $.
- **CSS custom property:** A runtime CSS value whose name begins with --.
- **Top-level variable:** Available within its Sass module after declaration.
- **Local variable:** Usually limited to its declaring block and nested blocks.
- **Shadowing:** A local variable hides an outer variable with the same name inside its scope.
- **!default:** Assigns a fallback only when the variable is undefined or null.
- **!global:** Reassigns an existing top-level variable; use sparingly.
- **Custom-property interpolation:** Inserts a Sass expression into CSS custom-property text.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="005-scss-syntax-and-rules.md">005 - SCSS Syntax and Rules - Q&amp;A</a></td><td><a href="../lessons/006-sass-variables-and-scope.md">Lesson</a></td><td><a href="../examples/006-sass-variables-and-scope.md">Example</a></td><td><a href="007-sass-value-types-and-null.md">007 - Sass Value Types and Null: Q&amp;A</a></td></tr>
  </tbody>
</table>
