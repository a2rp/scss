# 030 — Module Configuration and !default

## Learning goals

After this lesson, you should be able to:

- Explain how `!default` assigns a fallback value.
- Identify which module variables can be configured by a consumer.
- Configure a module with the `@use ... with (...)` syntax.
- Arrange configuration so it is applied before the module is first loaded.
- Share one module configuration across dependent modules in a compilation.
- Build separate themes with separate entrypoints.
- Decide when a configuration mixin or parameterized mixin is a better fit.

## 1. Why libraries provide defaults

A Sass library needs defaults so it works when a consumer does nothing. At the same time, an application may need to change its brand color, spacing, border radius, or other design choices.

The `!default` flag lets a library provide fallback values that can be configured when the module is loaded.

~~~scss
// _theme.scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;
$border-radius: 0.5rem !default;
~~~

The library can use those values to generate CSS:

~~~scss
// _theme.scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;
$border-radius: 0.5rem !default;

.card {
  color: #263238;
  background-color: $surface-color;
  border: 2px solid $brand-color;
  border-radius: $border-radius;
}
~~~

Without any consumer configuration, Sass uses the defaults.

## 2. What !default means

A `!default` declaration assigns its value only when the variable has not already been given a non-null value. This is different from ordinary assignment, which replaces the variable's previous value.

~~~scss
$brand-color: #1769aa !default;
$brand-color: #4b2e83 !default;
~~~

The first value remains because the variable is already set. In library code, `!default` marks values that consumers are allowed to configure. Use it for intentional settings, not every internal variable.

A variable with a value of `null` is treated as unset for `!default` assignment. This allows libraries to use a null default when a setting is optional.

## 3. Configure a module with @use ... with (...)

A consumer overrides the module's `!default` variables in a `with (...)` clause:

~~~scss
// app.scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa,
  $border-radius: 0.25rem
);
~~~

The module's CSS uses the configured values:

~~~css
.card {
  color: #263238;
  background-color: #f4f1fa;
  border: 2px solid #4b2e83;
  border-radius: 0.25rem;
}
~~~

The configuration names include the leading dollar sign. Values are SassScript expressions, so they can be variables or computed Sass values as well as literals.

## 4. Combine a custom namespace and configuration

If a custom namespace is needed, write the `as` clause before `with`:

~~~scss
@use "theme" as site-theme with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa
);

.page {
  color: site-theme.$text-color;
}
~~~

This configures the `theme` module and gives the consumer the `site-theme` namespace. The order matters: `as` comes before `with`.

## 5. Only top-level !default variables can be configured

A variable must be declared at the top level of the module with `!default` to be configurable through `@use ... with`.

~~~scss
// _theme.scss
$brand-color: #1769aa !default;
$-internal-cache: () !default;

@mixin button {
  color: $brand-color;
}
~~~

The consumer can configure the public `$brand-color`, but a private variable such as `$-internal-cache` is not part of the consumer API. Local variables inside a mixin or style rule are not module configuration settings.

Keep the configurable variables public and document what changing them affects.

## 6. Configure before the module is first loaded

A module keeps one configuration per Sass compilation. Configuration must be passed when it is first loaded.

~~~scss
// app.scss
@use "theme" with (
  $brand-color: #4b2e83
);

@use "components/card";
~~~

The component can load the same theme module and sees the configured value:

~~~scss
// components/_card.scss
@use "../theme";

.card {
  border-color: theme.$brand-color;
}
~~~

If another dependency loads `theme` first without configuration, the entrypoint cannot configure it afterward. Put configuration in the entrypoint and ensure it runs before dependencies load the module.

## 7. One configuration is shared throughout a compilation

If several modules load the same theme module, they share the same loaded module and configuration.

~~~text
app.scss
  ├── configures theme
  ├── loads components/card
  └── loads components/button

components/_card.scss
  └── loads theme

components/_button.scss
  └── loads theme
~~~

Both components see the configuration supplied by `app.scss`. A single compilation cannot load the same module twice with different configurations. This makes entrypoint configuration useful for setting a consistent theme across a component library.

## 8. Build different themes with separate entrypoints

Create one Sass entrypoint for each theme:

~~~scss
// light.scss
@use "theme" with (
  $brand-color: #1769aa,
  $surface-color: #ffffff
);

@use "components";
~~~

~~~scss
// dark.scss
@use "theme" with (
  $brand-color: #a8c7fa,
  $surface-color: #172033
);

@use "components";
~~~

Compile `light.scss` and `dark.scss` separately. Each compilation gets its own theme configuration and can produce a different CSS file.

Do not try to configure the same module twice in one entrypoint to create two simultaneously active themes. For runtime theme switching, CSS custom properties or separately scoped theme mixins are usually a better tool.

## 9. Sass configuration is compile-time

A configured Sass variable is evaluated while Sass compiles. It does not remain as a variable in the CSS and cannot change in a browser at runtime.

~~~scss
$surface-color: #ffffff !default;

.card {
  background-color: $surface-color;
}
~~~

The compiled output contains the configured color value. If the browser needs to switch themes dynamically, define CSS custom properties instead:

~~~css
:root {
  --surface-color: #ffffff;
}

[data-theme="dark"] {
  --surface-color: #172033;
}

.card {
  background-color: var(--surface-color);
}
~~~

Use Sass module configuration for build-time variants. Use CSS custom properties when the value needs to change at runtime through the cascade, DOM, or user interaction.

## 10. Use a mixin for more flexible configuration

The `@use ... with (...)` mechanism is intentionally simple. It configures a module once, before it loads. If a library needs multiple style sets in one compilation or has many interdependent options, a mixin that receives parameters can be more flexible.

~~~scss
// _theme-styles.scss
@mixin card($brand-color, $surface-color) {
  .card {
    color: $brand-color;
    background-color: $surface-color;
  }
}
~~~

~~~scss
// app.scss
@use "theme-styles";

@include theme-styles.card(
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa
);

[data-theme="dark"] {
  @include theme-styles.card(
    $brand-color: #a8c7fa,
    $surface-color: #172033
  );
}
~~~

This mixin can emit multiple theme contexts in one compilation. Choose the API carefully: configuration variables are convenient for one build-wide setting, while parameters make each style emission explicit.

For a library with many options, a configuration mixin or a map of settings may also be easier to maintain than an increasingly long `with (...)` list.

## 11. Keep defaults and overrides understandable

A well-designed configurable module should make these points clear:

- Which variables are public configuration options?
- What is each default value?
- Which rules or mixins use each option?
- Must the module be configured before another dependency loads it?
- Does the setting apply to one build or should users be able to emit multiple variations?
- Is a Sass setting appropriate, or does the value need to change in the browser?

Avoid making internal implementation variables configurable. Every public option becomes part of the library's API and may need to be supported over time.

## 12. Common mistakes

### Configuring a variable that lacks !default

Only top-level variables declared with `!default` can be configured through `@use ... with`. Mark intentional public settings in the module.

### Configuring a module after another file loaded it

A module is configured only on its first load in the compilation. Move configuration into the entrypoint before dependencies load the module.

### Writing the clauses in the wrong order

When an `@use` has both a namespace and configuration, write `as` before `with`:

~~~scss
@use "theme" as site-theme with (
  $brand-color: #4b2e83
);
~~~

### Expecting two configurations from one module in one build

A module cannot hold separate configurations for different callers in one compilation. Use separate entrypoints or parameterized mixins when multiple variations must be emitted together.

### Expecting Sass variables to change in the browser

Sass values are compiled away. Use CSS custom properties for runtime theme switching.

### Adding too many public settings

Configuration options create API commitments. Expose the values consumers truly need, and keep the rest private or derive them from a smaller set of settings.

## 13. Summary

- `!default` assigns a value only when no non-null value is already set.
- `@use ... with (...)` configures a module's top-level `!default` variables.
- Configuration is applied the first time a module loads and remains shared for that module during the compilation.
- Put configuration in the entrypoint before dependencies load the module.
- Separate Sass entrypoints can build separate compile-time themes.
- Use mixin parameters for multiple style variations in one compilation.
- Use CSS custom properties when theme values must change at runtime.

## Further reading

- [Sass: @use — Configuration](https://sass-lang.com/documentation/at-rules/use/#configuration)
- [Sass: Variables — Default Values](https://sass-lang.com/documentation/variables/#default-values)
- [Sass: Variables — Configuring Modules](https://sass-lang.com/documentation/variables/#configuring-modules)
