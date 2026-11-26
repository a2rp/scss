# 034 — Legacy @import and Global Built-In Deprecations: Q&A

## Deprecation basics

### 1. Which Sass features does this chapter cover?

It covers the deprecated Sass @import rule and global calls to Sass built-in functions that are available in sass: modules.

### 2. When were these patterns deprecated?

Dart Sass began deprecating Sass @import and global built-in calls in version 1.80.0. They emit deprecation warnings during the transition period.

### 3. Does a deprecation warning mean the code immediately fails?

No. A deprecation warning is a signal to migrate before the feature is removed in a future major release. The warning is not the same as a compilation error.

### 4. Are the @import and global built-in deprecations one warning type?

No. They are separate deprecations with separate identifiers: import and global-builtin. Fixing one does not fix the other.

### 5. Is every @import rule in an SCSS file a deprecated Sass import?

No. A plain CSS @import is still valid CSS intended for the browser. Identify whether the rule loads a Sass stylesheet or is a CSS import before changing it.

### 6. Which are plain CSS imports?

These examples are CSS imports:

~~~scss
@import url("print.css");
@import url("https://example.test/fonts.css");
@import "theme.css";
~~~

They are preserved for the browser. Do not replace them with @use.

## Moving from @import to the module system

### 7. Why is Sass moving away from global @import?

It puts members in a shared global scope, making it harder to tell where a variable or mixin came from. Repeated imports can also repeat work and CSS.

### 8. What replaces a Sass @import?

Use @use when a stylesheet needs a module's members. Use @forward when a stylesheet needs to expose members through a public entrypoint.

### 9. How does @use make a dependency clearer?

It gives members a namespace based on the module URL unless you choose another alias:

~~~scss
@use "tokens";

.card {
  color: tokens.$text-color;
}
~~~

The namespace shows where the member is defined.

### 10. How do I migrate a mixin reference?

Load the module and include the mixin through its namespace:

~~~scss
@use "button-tools";

.button {
  @include button-tools.base;
}
~~~

### 11. What if two modules have the same default namespace?

Choose explicit aliases:

~~~scss
@use "components/button/mixins" as button-mixins;
@use "components/form/mixins" as form-mixins;
~~~

### 12. Does @use change CSS output order?

It can. Module loading and source order affect where CSS is emitted. Review cascade behavior after migrating, especially if the old import order was intentional.

### 13. Does @use duplicate CSS when the same module is loaded repeatedly?

The Sass module system loads a module once per compilation. This can remove duplicate CSS that repeated Sass @import rules previously emitted.

### 14. Can I configure a theme by setting global variables before @use?

Use the module configuration syntax with !default variables:

~~~scss
@use "theme" with (
  $text-color: white,
  $surface-color: #20242a
);
~~~

Configuration must be applied when the module is first loaded in that compilation.

## Replacing global built-ins

### 15. What is a global built-in function call?

It is a call to a Sass built-in function without the module namespace, such as map-get() or nth().

### 16. How do I migrate map-get()?

Load sass:map and call map.get():

~~~scss
@use "sass:map";

$brand: map.get($theme, "brand");
~~~

### 17. How do I migrate nth()?

Load sass:list and call list.nth():

~~~scss
@use "sass:list";

$second-item: list.nth($items, 2);
~~~

### 18. How do I migrate unit()?

Load sass:math and call math.unit():

~~~scss
@use "sass:math";

$unit-name: math.unit(12px);
~~~

### 19. Are all CSS functions part of the global built-in deprecation?

No. A CSS function written as CSS is not automatically a Sass global built-in. Check the warning and function documentation before changing a call that may belong to CSS.

### 20. What if my project defines its own function with the same name as a Sass built-in?

The global-builtin deprecation is about Sass's built-in functions. Still, namespaced project modules help make custom and built-in functions easier to distinguish.

### 21. Can a similarly named module function have different behavior?

It can. Consult the function's module documentation, especially for legacy color helpers or functions that received newer signatures. Do not blindly add a namespace to every name.

## Nested imports and special cases

### 22. Can I put @use inside a media query?

No. @use must be at the top level of a stylesheet. Load the module at the top and include a mixin inside the media query.

### 23. What is the preferred replacement for a nested Sass import?

Wrap the imported stylesheet's component CSS in a mixin and include it in the desired context:

~~~scss
@use "print";

@media print {
  @include print.styles;
}
~~~

### 24. What if I cannot edit the stylesheet to add a mixin?

Use the sass:meta load-css() mixin where needed:

~~~scss
@use "sass:meta";

@media print {
  @include meta.load-css("print");
}
~~~

Be aware that the loaded stylesheet is compiled before its CSS is nested; parent selectors in the caller do not affect its rules.

### 25. Does the module migrator handle nested imports?

The module migrator can convert nested imports to use meta.load-css(). Review this output and decide whether a dedicated mixin would make the behavior clearer.

### 26. Can CSS imports be moved to @use?

No. @use loads Sass modules. Keep browser-managed CSS imports as CSS imports.

## Sass Migrator

### 27. What does the module migration do?

It converts Sass imports toward @use and updates member references and namespaces. It may also add module loads, convert configured defaults, and handle nested imports. It is more than replacing the text @import.

### 28. How can I preview a module migration?

Use --dry-run and --verbose:

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Review the proposed changes before applying them.

### 29. What does --migrate-deps do?

It tells the migrator to update stylesheet dependencies of the entrypoint as well as the files explicitly passed. Without it, the migrator changes only the listed files, though it still needs to read dependencies to resolve them.

### 30. How do I apply the migration after reviewing it?

Run the module migrator without --dry-run:

~~~bash
npx sass-migrator module --migrate-deps src/scss/app.scss
~~~

Review the changed source files and compile the project through its normal build process.

### 31. What does --built-in-only do?

It migrates global built-in function calls to module-based calls while leaving @import rules in place. This can be used as an intermediate migration step; the Sass @import deprecation remains.

~~~bash
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

### 32. What if my project uses a Sass load path?

Pass matching --load-path options to the migrator so it can resolve the same dependencies as the build. The migrator treats files resolved through load paths as dependencies and does not migrate them as application-owned files.

### 33. Does the migrator support every custom importer?

No. The Sass migrator has limitations around custom importers. Check its documentation if the project uses a custom module-resolution setup.

### 34. Should I run a migrator across node_modules?

Usually not. Migrate code that your project owns. For dependency warnings, look for an updated dependency release or report the issue to its maintainer.

## Warnings and compatibility

### 35. Should I silence the warnings?

Prefer fixing Sass code you own. Silencing a deprecation warning does not update the code or prepare it for a future major release.

### 36. Can I silence only dependency warnings?

Dart Sass's --quiet-deps option can quiet warnings from dependencies, commonly those loaded through load paths. It does not migrate the dependency or suppress warnings from the entrypoint's own relative Sass files.

### 37. How do I temporarily silence one deprecation?

Dart Sass supports specific deprecation identifiers such as import and global-builtin. Silencing the two requires both identifiers. Use this only as a temporary, targeted compatibility measure.

### 38. What should I review after running the migrator?

Review namespaces, module configuration, entrypoint order, generated CSS, public library APIs, and any remaining CSS imports. Confirm that the application's normal build still produces the intended styling.

### 39. Should a library keep old @import support for existing users?

If the library has existing @import consumers, plan a compatibility strategy instead of silently breaking them. Sass supports import-only files with the .import.scss naming convention, and the Sass Migrator has options for forwarding members during a library migration.

## Quick reference

~~~scss
// Sass module:
@use "tokens";

.card {
  color: tokens.$brand-color;
}

// Built-in modules:
@use "sass:map";
@use "sass:list";
@use "sass:math";

$brand: map.get($theme, "brand");
$second: list.nth($items, 2);
$unit-name: math.unit(12px);

// Browser-managed CSS imports remain CSS:
@import url("fonts.css");
~~~

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
npx sass-migrator module --migrate-deps src/scss/app.scss
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

## Related material

- [Lesson: Legacy @import and global built-in deprecations](../lessons/034-legacy-import-and-global-built-in-deprecations.md)
- [Example: Legacy @import and global built-in deprecations](../examples/034-legacy-import-and-global-built-in-deprecations.md)
- [Breaking change: @import and global built-ins](https://sass-lang.com/documentation/breaking-changes/import/)
- [Sass Migrator](https://sass-lang.com/documentation/cli/migrator/)
