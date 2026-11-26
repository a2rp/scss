# 022 — Built-in Sass Modules

## Learning goals

After this lesson, you should be able to:

- Explain why Sass organizes built-in functions into modules.
- Load a built-in module with **@use** and call its members through a namespace.
- Recognize the purpose of the math, color, list, map, string, meta, and selector modules.
- Use common module functions in SCSS.
- Prefer namespaced module APIs over legacy global function names.
- Explain the difference between Sass calculations and CSS functions such as **calc()**.

## 1. What are built-in modules?

Sass includes functions for working with numbers, colors, lists, maps, strings, selectors, and Sass itself. Modern Sass groups these APIs into built-in modules.

A built-in module is loaded with **@use** and has a URL beginning with **sass:**. For example:

~~~scss
@use "sass:math";

$third: math.div(1, 3);

.column {
  width: math.div(100%, 3);
}
~~~

The namespace is **math**, taken from the last part of the module URL. Calling **math.div()** makes it clear which module provides the operation.

The Sass module system with **@use** is supported by Dart Sass. Use a current Dart Sass version for the namespaced examples in this course.

## 2. How @use works

Write an **@use** rule near the top of the stylesheet, before style rules and most other statements. Then access the module's public members through its namespace.

~~~scss
@use "sass:math";
@use "sass:color";

$gap: math.div(24px, 2);
$brand: #2457c5;

.button {
  gap: $gap;
  border-color: color.scale($brand, $lightness: 15%);
}
~~~

A module member is accessed as **namespace.member**:

- A variable: **math.$pi**
- A function: **math.div(12px, 4px)**
- A mixin, when a module provides one: **@include namespace.mixin-name**

Members loaded with **@use** are visible only in the stylesheet that loads them. Another Sass file must write its own **@use** rule if it needs that module.

## 3. Namespaces and aliases

The default namespace comes from the module URL. You can choose a shorter or more descriptive namespace with **as**:

~~~scss
@use "sass:math" as m;

$half: m.div(1, 2);

.box {
  width: m.div(100%, 2);
}
~~~

The namespace keeps function calls clear and helps avoid collisions. Avoid loading every member into the global scope with **as *** in normal project code; unqualified calls are harder to trace and can conflict with other functions.

Each stylesheet that uses a module should load it explicitly. This makes the source of a function visible where it is called.

## 4. The built-in modules

Sass provides these main built-in modules:

| Module | What it helps you do | Common examples |
| --- | --- | --- |
| **sass:math** | Work with numbers, units, rounding, and calculations. | **math.div()**, **math.round()**, **math.max()** |
| **sass:color** | Inspect, adjust, and scale colors. | **color.adjust()**, **color.scale()**, **color.change()** |
| **sass:list** | Read and build Sass lists. | **list.nth()**, **list.length()**, **list.append()** |
| **sass:map** | Read and update key-value maps. | **map.get()**, **map.has-key()**, **map.merge()** |
| **sass:string** | Inspect and transform strings. | **string.length()**, **string.to-upper-case()**, **string.slice()** |
| **sass:meta** | Inspect Sass values and access Sass functions or mixins dynamically. | **meta.type-of()**, **meta.inspect()**, **meta.get-function()** |
| **sass:selector** | Inspect and manipulate selector values. | **selector.nest()**, **selector.append()**, **selector.is-superselector()** |

The module reference documents all members and their supported versions. This lesson introduces frequently useful functions; it is not an exhaustive function-by-function reference.

## 5. sass:math

Load **sass:math** for Sass calculations involving numbers.

~~~scss
@use "sass:math";

$ratio: math.div(12px, 4px);
$third: math.div(1, 3);

.grid {
  width: math.div(100%, 3);
  gap: math.round(2.4px);
}
~~~

Sass can simplify the results:

- **math.div(12px, 4px)** returns the unitless number **3**.
- **math.div(1, 3)** returns a decimal approximation.
- **math.div(100%, 3)** returns approximately **33.3333333333%**.
- **math.round(2.4px)** returns **2px**.

Use **math.div()** when you mean Sass division. A slash can also be meaningful CSS syntax, so explicit division makes the intent clear and avoids relying on the older slash-division behavior.

CSS has its own runtime math functions, including **calc()**, **min()**, **max()**, and **clamp()**. Sass's **math** functions perform Sass calculations during compilation; CSS functions can remain in the stylesheet for the browser to evaluate. Choose based on whether Sass already has the values or the browser must combine them at runtime.

## 6. sass:color

Load **sass:color** to work with colors.

~~~scss
@use "sass:color";

$brand: #2457c5;

.button {
  background-color: $brand;
  border-color: color.scale($brand, $lightness: 15%);
}
~~~

**color.scale()** adjusts a color proportionally toward a channel's minimum or maximum. It is useful for deriving a related color while keeping the adjustment within the channel's range. Its exact serialized CSS color can depend on the color representation Sass chooses.

Other commonly used APIs include:

- **color.adjust()** for adding or subtracting channel amounts.
- **color.change()** for setting specific channel values.
- **color.mix()** for mixing colors.

Prefer the namespaced color module functions. Older global color helpers such as **lighten()** and **darken()** have legacy behavior and are deprecated; use the modern color module APIs for new Sass.

## 7. sass:list

Sass lists hold ordered values. Load **sass:list** when you want to access, count, or modify their entries.

~~~scss
@use "sass:list";

$spacing-scale: 0.25rem, 0.5rem, 1rem, 2rem;

.card {
  padding: list.nth($spacing-scale, 3);
}
~~~

List indexes start at **1**, so the third item is **1rem**. Common functions include:

- **list.nth($list, $index)** to get an item.
- **list.length($list)** to count items.
- **list.append($list, $value)** to return a list with a value added.

A Sass list can use commas or spaces as separators. Preserve the separator that fits the value's meaning.

## 8. sass:map

A map stores key-value pairs. Load **sass:map** to look up values and work with map data.

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

Common map functions include:

- **map.get($map, $key)** to retrieve a value.
- **map.has-key($map, $key)** to check whether a key exists.
- **map.merge($map1, $map2)** to return a merged map.
- **map.remove($map, $key)** to return a map without a key.

A missing key returns **null** from **map.get()**. Check that the key exists when a missing value would make the resulting styles confusing or invalid.

## 9. sass:string

Load **sass:string** for operations on Sass strings.

~~~scss
@use "sass:string";

$label: "primary button";

.button {
  --button-label: #{string.to-upper-case($label)};
}
~~~

The interpolation places the result into the CSS custom property value. The value is **PRIMARY BUTTON**.

Other string functions include:

- **string.length($string)** to count characters.
- **string.index($string, $substring)** to find a substring, or return **null** if it is absent.
- **string.slice($string, $start, $end)** to return part of a string.
- **string.quote($string)** to return a quoted string.

Sass string indexes start at **1**. Use string functions when Sass needs to build or inspect a value; use interpolation when inserting a Sass value into CSS syntax.

## 10. sass:meta

Load **sass:meta** when a stylesheet needs to inspect Sass values or work with Sass constructs dynamically.

~~~scss
@use "sass:meta";

$tokens: ("small": 0.5rem, "large": 2rem);

@debug meta.type-of($tokens);
@debug meta.inspect($tokens);
~~~

The first debug statement reports **map**. The second prints a human-readable representation of the value. **@debug** messages appear in compiler output and do not become CSS declarations.

The meta module has more advanced APIs for retrieving and applying function or mixin values. Those are useful for libraries and dynamic Sass utilities; ordinary stylesheets can often use direct function calls instead.

## 11. sass:selector

The selector module can inspect and manipulate selector values. This is useful for advanced Sass APIs that generate selectors.

~~~scss
@use "sass:selector";

@debug selector.append(".card", "__title");
@debug selector.is-superselector("a", "a.active");
~~~

The first debug message is **.card__title**. The second is **true**, because every element matched by **a.active** is also matched by **a**.

Selector functions return selector values; they do not by themselves write CSS. To put a computed selector into a style rule, use interpolation carefully. For most everyday styles, ordinary nested selectors are simpler than constructing selectors through functions.

## 12. Modules versus legacy global functions

Older Sass code often calls functions without a namespace, such as **nth()**, **map-get()**, or **lighten()**. Sass historically made many built-in functions globally available.

For new code, prefer explicit module calls such as **list.nth()**, **map.get()**, and **color.scale()**. Namespaces show where each operation comes from, avoid name collisions, and align with the modern Sass module system.

Sass **@import** is deprecated in Dart Sass 1.80.0. For Sass modules, prefer **@use**. A plain CSS import is different: CSS imports that should remain browser requests are still CSS.

## 13. Common mistakes

### Forgetting to load a module

A namespaced member is available only after the module is loaded.

~~~scss
@use "sass:math";

.box {
  width: math.div(100%, 2);
}
~~~

### Calling a member without its namespace

If the module was loaded as **math**, call **math.div()**, not **div()**.

### Putting @use after a style rule

Place **@use** statements before style rules and other ordinary statements.

### Confusing Sass and CSS math

Use **math.div()** for a Sass calculation. Keep **calc()** when the browser needs to combine values at runtime.

### Using deprecated global helpers in new code

Prefer the current namespaced module API instead of old global aliases.

### Assuming every Sass implementation supports @use

The built-in module system requires Dart Sass. Check the compiler used by the project before adopting module syntax in legacy environments.

## 14. Practice

1. Load **sass:math** and calculate half of **48px**.
2. Create a comma-separated spacing list and use **list.nth()** to select its second value.
3. Create a theme map and retrieve its **primary** color with **map.get()**.
4. Use **string.to-upper-case()** to transform a label.
5. Explain when Sass's **math.div()** is appropriate and when a CSS **calc()** should remain in the output.
6. Explain why **@use "sass:color"** is clearer than calling a global color helper.

## Key takeaways

- Built-in Sass modules group related functions behind explicit namespaces.
- Load modules with **@use "sass:module-name"** near the top of the stylesheet.
- Use **math**, **color**, **list**, **map**, **string**, **meta**, and **selector** for their respective value types.
- Prefer modern namespaced functions over legacy global aliases.
- Sass module functions calculate or inspect values; CSS functions may remain for the browser.
- Check the official module reference for a function's full behavior and version compatibility.

## Further reading

- [Sass built-in modules](https://sass-lang.com/documentation/modules/)
- [Sass @use](https://sass-lang.com/documentation/at-rules/use/)
- [sass:math](https://sass-lang.com/documentation/modules/math/)
- [sass:color](https://sass-lang.com/documentation/modules/color/)
- [sass:list](https://sass-lang.com/documentation/modules/list/)
- [sass:map](https://sass-lang.com/documentation/modules/map/)
- [sass:string](https://sass-lang.com/documentation/modules/string/)
- [sass:meta](https://sass-lang.com/documentation/modules/meta/)
- [sass:selector](https://sass-lang.com/documentation/modules/selector/)
