# 022 — Built-in Sass Modules: Examples

These examples use the modern Dart Sass module system. Each snippet can be saved and compiled on its own. Keep every **@use** statement at the top of its stylesheet.

## 1. Math: divide Sass values

Load **sass:math** to perform explicit Sass arithmetic.

~~~scss
@use "sass:math";

$columns: math.div(12px, 4px);

.grid {
  width: math.div(100%, 2);
  gap: math.round(2.4px);
}
~~~

The Sass calculation **math.div(12px, 4px)** returns the unitless number **3**. The generated declarations are:

~~~css
.grid {
  width: 50%;
  gap: 2px;
}
~~~

Use **math.div()** when Sass should calculate a value during compilation. Use CSS **calc()** when the browser needs to combine values later.

## 2. Color: derive a related color

The color module includes functions for adjusting, scaling, changing, and mixing colors.

~~~scss
@use "sass:color";

$brand: #2457c5;

.button {
  background-color: $brand;
  border-color: color.scale($brand, $lightness: 15%);
}
~~~

The function returns a color derived from **$brand**. The exact serialized color can depend on Sass's color representation, so inspect the CSS produced by the Sass version used in your project.

For new Sass, prefer namespaced module functions over legacy global helpers such as **lighten()**.

## 3. List: select a value by index

List indexes start at **1**.

~~~scss
@use "sass:list";

$spacing-scale: 0.25rem, 0.5rem, 1rem, 2rem;

.card {
  padding: list.nth($spacing-scale, 3);
}
~~~

The third item is **1rem**, so the generated CSS is:

~~~css
.card {
  padding: 1rem;
}
~~~

Other useful list operations include **list.length()** and **list.append()**.

## 4. Map: look up theme tokens

Maps associate keys with values. Use **map.get()** to retrieve a value by key.

~~~scss
@use "sass:map";

$theme: (
  "primary": #2457c5,
  "surface": #f5f7fa,
  "text": #172033
);

.button {
  background-color: map.get($theme, "primary");
  color: map.get($theme, "surface");
}
~~~

Generated CSS:

~~~css
.button {
  background-color: #2457c5;
  color: #f5f7fa;
}
~~~

If the key might not exist, check it with **map.has-key()** before relying on the result.

## 5. String: transform a label

Use **sass:string** for string operations. Interpolation inserts a Sass result into CSS syntax.

~~~scss
@use "sass:string";

$label: "primary button";

.button {
  --button-label: #{string.to-upper-case($label)};
}
~~~

The custom property value becomes **PRIMARY BUTTON**. Other string functions can count characters, find a substring, or return part of a string.

## 6. Meta: inspect a Sass value

Meta functions are useful for inspecting values while developing Sass utilities.

~~~scss
@use "sass:meta";

$tokens: ("small": 0.5rem, "large": 2rem);

@debug meta.type-of($tokens);
@debug meta.inspect($tokens);
~~~

The first debug message is **map**. The second prints a readable representation of the map in the compiler output. Debug messages are not included as CSS declarations.

## 7. Selector: build and inspect selectors

The selector module can manipulate selector values. Its functions do not emit CSS by themselves; use the returned selector in a rule if you want a rule in the output.

~~~scss
@use "sass:selector";

$title-selector: selector.append(".card", "__title");

#{$title-selector} {
  font-weight: 700;
}

@debug selector.is-superselector("a", "a.active");
~~~

The generated selector is **.card__title**, and the debug statement reports **true** because **a** matches all links matched by **a.active**.

For ordinary styles, write the selector directly. Selector functions are useful when building advanced utilities that need to inspect or construct selectors.

## 8. Use a custom namespace alias

The default namespace is the last part of the module URL. Give it another name with **as** if that makes the local stylesheet clearer.

~~~scss
@use "sass:math" as m;

$half: m.div(1, 2);

.split {
  width: m.div(100%, 2);
}
~~~

The module's functions are now accessed through **m** in this stylesheet.

## 9. Combine modules in a component stylesheet

A stylesheet can load several built-in modules. Each **@use** remains explicit and appears before the other Sass rules.

~~~scss
@use "sass:list";
@use "sass:map";
@use "sass:math";

$theme: (
  "primary": #2457c5,
  "surface": #f5f7fa
);
$spacing-scale: 0.25rem, 0.5rem, 1rem, 2rem;

.card {
  background-color: map.get($theme, "surface");
  border-color: map.get($theme, "primary");
  padding: list.nth($spacing-scale, 3);
  width: math.div(100%, 2);
}
~~~

Generated CSS:

~~~css
.card {
  background-color: #f5f7fa;
  border-color: #2457c5;
  padding: 1rem;
  width: 50%;
}
~~~

The namespaces make it clear which module supplies each function.

## 10. Spot the legacy global calls

Older Sass code may use global functions:

~~~scss
$half: 1 / 2;
$second: nth($spacing-scale, 2);
$primary: map-get($theme, "primary");
~~~

Prefer the explicit module forms:

~~~scss
@use "sass:list";
@use "sass:map";
@use "sass:math";

$half: math.div(1, 2);
$second: list.nth($spacing-scale, 2);
$primary: map.get($theme, "primary");
~~~

The module version documents where each operation comes from. Check the official Sass docs when migrating older code because the replacement depends on the function and Sass version.

## Compile a snippet

Save an example as **modules.scss** and compile it with Dart Sass:

~~~sh
sass modules.scss modules.css
~~~

Read the compiler's debug output separately from the generated CSS. The **@debug** statements in the meta and selector examples report information to the compiler; they do not create declarations.

## Key takeaways

- Load built-in modules with **@use "sass:module-name"**.
- Use the module namespace when calling its functions.
- Use **math**, **color**, **list**, **map**, **string**, **meta**, and **selector** for related Sass values.
- List indexes begin at **1**; a missing map key returns **null**.
- Prefer namespaced functions instead of legacy global aliases.
- Compare the expected CSS with output from the Dart Sass version used in your project.
