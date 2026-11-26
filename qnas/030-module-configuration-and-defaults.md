# 030 - Module Configuration and !default: Q&A

## Defaults

### 1. What does the !default flag do?

It assigns a variable's value only when the variable is not already set to a non-null value. Libraries use it to provide fallback settings that consumers can override.

### 2. What happens when a !default variable already has a value?

The existing value remains. A later assignment with `!default` does not replace it.

### 3. What happens if a !default variable is null?

Sass treats a null value as unset for `!default`, so the fallback value is assigned.

### 4. Should every variable in a module use !default?

No. Use it for deliberate public configuration options. Internal implementation variables should usually use ordinary assignments and may be private.

### 5. Is a Sass !default variable the same as a CSS custom property?

No. A Sass variable is evaluated during compilation and is compiled away. A CSS custom property is emitted as CSS and can participate in the browser's cascade and runtime theme changes.

## Configuring with @use

### 6. How do I configure a module while loading it?

Pass values in a `with (...)` clause:

~~~scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa
);
~~~

The module must declare those top-level variables with `!default`.

### 7. Does a configuration variable include its dollar sign?

Yes. Write `$brand-color: value` in the `with (...)` clause.

### 8. Can I configure only one setting?

Yes. Any omitted configurable variables keep the module's defaults.

### 9. Can I use a custom namespace and configuration together?

Yes. Put the `as` clause before the `with` clause:

~~~scss
@use "theme" as site-theme with (
  $brand-color: #4b2e83
);
~~~

### 10. Which variables can @use ... with configure?

Only top-level variables in the module that are declared with `!default` and exposed as public members can be configured by consumers.

### 11. Can I configure a variable declared inside a mixin or selector?

No. Configuration is for top-level module variables, not local variables inside a mixin or style rule.

### 12. Can a consumer configure a private variable?

No. A private member is not part of the module's public configuration API. Expose a deliberate public setting if consumers need to control that behavior.

### 13. Can the configured value be another Sass variable or expression?

Yes. The values in the `with (...)` clause are SassScript expressions.

~~~scss
$project-brand: #4b2e83;

@use "theme" with (
  $brand-color: $project-brand
);
~~~

### 14. Can variables be declared before @use?

Yes. Sass permits variable declarations before `@use` rules, which is useful when those variables provide values to module configuration. Other style rules must still come after the `@use` rules.

## Load order and module identity

### 15. When must configuration be applied?

When the module is first loaded in the current Sass compilation. If another dependency loads it first, you cannot apply `@use ... with` afterward.

### 16. What happens if the same module is loaded again?

The same loaded module and its configuration are reused. Other files that load it see that module state.

### 17. Can two components use the same theme configuration?

Yes. Configure the theme in the entrypoint before loading the components. Each component's own `@use` of that theme sees the shared configuration.

### 18. Can the same module have two configurations in one compilation?

No. A module has one configuration per compilation.

### 19. How can I produce light and dark CSS builds with different configurations?

Create separate Sass entrypoints and compile them separately. Each compilation can load the module with its own configuration.

### 20. How can I emit multiple theme variants in a single compiled stylesheet?

A single module configuration is not designed for that. Use a parameterized mixin or a theme API that receives values for each emitted variant.

### 21. Does a module's CSS use the configured values?

Yes. Top-level CSS generated when the module loads uses the configured settings. Modules that load the same configured module can also use those shared values.

## Reassignment versus configuration

### 22. Is changing a module variable after loading the same as configuring it with with?

No. Sass permits reassignment of an existing public module variable through its namespace, but that is a separate mechanism from `@use ... with (...)`.

### 23. Why prefer with over post-load reassignment?

Load-time configuration is explicit and happens before dependent modules use the module. Post-load reassignment changes shared module state and can make output depend on evaluation order.

### 24. Can built-in module variables be reassigned?

No. Sass built-in module variables, such as `math.$pi`, cannot be reassigned.

### 25. Does changing a module variable retroactively change CSS already emitted?

No. Sass evaluates stylesheet code as it loads and compiles it. Reassignment affects later uses of that module variable; it does not rewrite CSS that has already been generated.

## Choosing an API

### 26. When is @use ... with a good fit?

Use it for a small set of build-wide defaults that should be selected once for an entrypoint, such as a library's brand color or base radius.

### 27. When is a mixin API a better fit?

A mixin with parameters can be better when styles must be emitted multiple times with different values in one compilation, or when a library has many interdependent configuration settings.

### 28. When should I use CSS custom properties instead?

Use CSS custom properties when the browser must change values at runtime, such as responding to a theme attribute or user interaction. Sass configuration chooses values while compiling.

### 29. How should a library document configurable variables?

List each supported variable, its default, the effect of changing it, and whether it affects only generated CSS or other module behavior. Treat public settings as part of the library API.

## Troubleshooting

### 30. Why does Sass say a configured variable does not exist?

Check the spelling and dollar sign, then confirm that the module declares the variable at the top level with `!default` and that it is public.

### 31. Why did my configured value have no effect?

Confirm that the intended module URL was loaded, the `with (...)` clause is on the first load, and downstream files load the same module.

### 32. Why does a component still use the default value?

A dependency may have loaded the module before the entrypoint configured it. Move the configuration earlier in the entrypoint's dependency order.

### 33. Why does Sass reject my namespace and with order?

Write the namespace before the configuration:

~~~scss
@use "theme" as site-theme with (
  $brand-color: #4b2e83
);
~~~

### 34. Why does changing a Sass setting not change the page without a rebuild?

Sass settings are compile-time values. Recompile the stylesheet to change their output, or use CSS custom properties for runtime changes.

### 35. Why do different files see the same configured value?

They load the same Sass module within the same compilation. Sass reuses that module and its configuration.

## Quick reference

| Requirement | Approach |
| --- | --- |
| Provide a default setting | `$setting: value !default;` |
| Configure on first load | `@use "module" with ($setting: value);` |
| Alias and configure | `@use "module" as m with ($setting: value);` |
| Configure variables by entrypoint | Separate and separately compile entrypoints |
| Emit multiple values in one stylesheet | Use a parameterized mixin |
| Change a value in the browser | Use a CSS custom property |
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="029-module-namespaces-and-private-members.md">029 - Module Namespaces and Private Members: Q&amp;A</a></td><td><a href="../lessons/030-module-configuration-and-defaults.md">Lesson</a></td><td><a href="../examples/030-module-configuration-and-defaults.md">Example</a></td><td><a href="031-forwarding-public-apis-with-forward.md">031 - Forwarding Public APIs with @forward: Q&amp;A</a></td></tr>
  </tbody>
</table>
