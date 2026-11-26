# 022 - Built-in Sass Modules: Q&A

## Module system and @use

### 1. What is a built-in Sass module?

A built-in module is a group of Sass functions, variables, or mixins for a related purpose. Examples include **sass:math** for numbers and **sass:map** for maps.

### 2. How do you load a built-in module?

Use **@use** with the module's **sass:** URL.

~~~scss
@use "sass:math";
~~~

The module's members then become available through the **math** namespace.

### 3. What does a namespace do?

A namespace identifies which module provides a member. For example, **math.div()** is the division function from **sass:math**. Namespaces reduce naming collisions and make code easier to trace.

### 4. Where should @use appear?

Put **@use** near the top of the stylesheet, before style rules and other ordinary statements. A stylesheet may declare configuration variables before a **@use** in supported configuration patterns, but regular rules should come after its **@use** statements.

### 5. Are members loaded with @use global?

No. A module's variables, functions, and mixins are visible in the stylesheet that loads the module. Another stylesheet must load the module itself if it needs those members.

### 6. Can you choose a different namespace?

Yes. Use **as** to provide an alias.

~~~scss
@use "sass:math" as m;

$half: m.div(1, 2);
~~~

Here **m** is the namespace used for members from **sass:math**.

### 7. What does @use "sass:math" as * do?

It makes the module's members available without a namespace. This can make names harder to trace and can create collisions, so prefer the normal namespace unless there is a clear reason to do otherwise.

### 8. Does @use load a module more than once?

A module is loaded once in a Sass compilation, even if it is used from multiple places. This helps prevent duplicated evaluation and CSS output.

### 9. Which Sass implementation supports built-in modules with @use?

Dart Sass supports the module system. Older Sass implementations such as LibSass do not support **@use**. Check the compiler in the project before using modules in a legacy setup.

## Built-in module purposes

### 10. What does sass:math provide?

It provides functions for Sass number operations, such as division, rounding, and comparing values.

~~~scss
@use "sass:math";

$ratio: math.div(12px, 4px);
~~~

The result is the unitless number **3**.

### 11. When should I use math.div()?

Use **math.div()** when you want Sass to divide values during compilation.

~~~scss
@use "sass:math";

.half {
  width: math.div(100%, 2);
}
~~~

The result is **50%**. Use CSS **calc()** when the browser needs to combine values at runtime, such as a percentage and a custom property.

### 12. What does sass:color provide?

It provides functions for inspecting, adjusting, scaling, changing, and mixing colors. Use the namespaced APIs such as **color.scale()** and **color.adjust()**.

### 13. Why prefer color.scale() over old global lighten() or darken()?

The module API is explicit and uses modern Sass color operations. Legacy global color helpers have older behavior and are deprecated for new code. Use **sass:color** for new color transformations.

### 14. What does sass:list provide?

It provides functions to inspect and modify Sass lists. Common examples are **list.nth()**, **list.length()**, and **list.append()**.

### 15. Do Sass list indexes start at zero or one?

They start at **1**.

~~~scss
@use "sass:list";

$values: 0.5rem, 1rem, 2rem;
$second: list.nth($values, 2);
~~~

The value of **$second** is **1rem**.

### 16. What does sass:map provide?

It provides functions to look up, check, merge, remove, and otherwise work with Sass maps.

~~~scss
@use "sass:map";

$theme: ("primary": #2457c5);
$primary: map.get($theme, "primary");
~~~

### 17. What does map.get() return when a key is missing?

It returns **null**. If a missing value would cause confusing output, check the map with **map.has-key()** before using the value.

### 18. What does sass:string provide?

It provides string operations such as **string.length()**, **string.index()**, **string.slice()**, and **string.to-upper-case()**. String indexes start at **1**.

### 19. What does sass:meta provide?

It exposes information about Sass values and features. For example, **meta.type-of()** reports a value's type, and **meta.inspect()** returns a readable representation that is useful for debugging.

~~~scss
@use "sass:meta";

@debug meta.type-of(("primary": #2457c5));
~~~

The debug output reports **map**.

### 20. What does sass:selector provide?

It provides functions that inspect or manipulate selector values, such as **selector.append()**, **selector.nest()**, and **selector.is-superselector()**. These are most useful for advanced Sass utilities; ordinary selector nesting is simpler for everyday styles.

### 21. Do selector module functions write CSS rules?

No. They return or inspect selector values. You still need to use a selector in a stylesheet rule to emit CSS.

## Errors and migration

### 22. What happens if a namespaced function is called without its namespace?

Sass cannot find it as a member of that module. If the module was loaded as **math**, call **math.div()** rather than **div()**.

### 23. What if Sass says it cannot find a module?

Check that the module URL is spelled correctly, that the compiler supports **@use**, and that the code uses the correct **sass:** module name.

### 24. Why does Sass warn about an old global function?

Many built-in functions were available globally before modules were introduced. Sass keeps some global aliases for compatibility but discourages them and is deprecating legacy APIs. Prefer the namespaced equivalent documented for the module.

### 25. Is every CSS function a Sass module function?

No. CSS functions such as **calc()**, **var()**, **min()**, **max()**, and **clamp()** are CSS functions. Sass also has **sass:math** functions that operate on Sass values during compilation. Some names overlap, so choose based on whether the value should be calculated by Sass or by the browser.

### 26. Is Sass @import the same as a browser CSS @import?

No. Sass's **@import** historically loaded Sass files at compile time and exposed their members globally. A plain CSS import remains a browser instruction. Sass **@import** is deprecated in Dart Sass 1.80.0; use **@use** for Sass modules.

## Quick review

- Load built-in modules using **@use "sass:module-name"**.
- Call module members through their namespace, such as **map.get()**.
- Use the module that matches the value you are working with.
- Sass math functions run during compilation; CSS calculations may run in the browser.
- Prefer namespaced APIs over legacy global aliases.
- Consult the official documentation for every function's exact behavior and compatibility.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="021-custom-functions-and-return-values.md">021 - Custom Functions and Return Values: Q&amp;A</a></td><td><a href="../lessons/022-built-in-sass-modules.md">Lesson</a></td><td><a href="../examples/022-built-in-sass-modules.md">Example</a></td><td><a href="023-conditionals-and-boolean-logic.md">023 - Conditionals and Boolean Logic: Q&amp;A</a></td></tr>
  </tbody>
</table>
