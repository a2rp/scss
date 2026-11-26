# 030 — Module Configuration and !default: Examples

These examples show module defaults, load-time configuration, and other ways to handle style variations. Multi-file examples include their paths in comments.

## 1. Use module defaults without configuration

### styles/_theme.scss

~~~scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;
$text-color: #263238 !default;
$border-radius: 0.5rem !default;

@mixin card {
  .card {
    color: $text-color;
    background-color: $surface-color;
    border: 2px solid $brand-color;
    border-radius: $border-radius;
  }
}
~~~

### styles/default.scss

~~~scss
@use "theme";

@include theme.card;
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #ffffff;
  border: 2px solid #1769aa;
  border-radius: 0.5rem;
}
~~~

Because the entrypoint provides no configuration, the module's defaults are used.

## 2. Override several defaults during the first load

### styles/purple.scss

~~~scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa,
  $text-color: #241b35,
  $border-radius: 0.25rem
);

@include theme.card;
~~~

### CSS output

~~~css
.card {
  color: #241b35;
  background-color: #f4f1fa;
  border: 2px solid #4b2e83;
  border-radius: 0.25rem;
}
~~~

Only the entrypoint's values differ. The module still owns the style generation.

## 3. Override one default and keep the rest

### styles/blue.scss

~~~scss
@use "theme" with (
  $brand-color: #135cb5
);

@include theme.card;
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #ffffff;
  border: 2px solid #135cb5;
  border-radius: 0.5rem;
}
~~~

Unspecified configurable values keep their defaults.

## 4. Configure a module and choose its namespace

### styles/site.scss

~~~scss
@use "theme" as site-theme with (
  $brand-color: #8b3a3a,
  $surface-color: #fff8f0
);

@include site-theme.card;
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #fff8f0;
  border: 2px solid #8b3a3a;
  border-radius: 0.5rem;
}
~~~

The namespace clause comes before the configuration clause.

## 5. Share one configured theme across component modules

### File structure

~~~text
styles/
  _theme.scss
  components/
    _button.scss
    _card.scss
  app.scss
~~~

### styles/_theme.scss

~~~scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;
$text-color: #263238 !default;
~~~

### styles/components/_button.scss

~~~scss
@use "../theme";

.button {
  color: #ffffff;
  background-color: theme.$brand-color;
}
~~~

### styles/components/_card.scss

~~~scss
@use "../theme";

.card {
  color: theme.$text-color;
  background-color: theme.$surface-color;
  border: 1px solid theme.$brand-color;
}
~~~

### styles/app.scss

~~~scss
@use "theme" with (
  $brand-color: #4b2e83,
  $surface-color: #f4f1fa,
  $text-color: #241b35
);

@use "components/button";
@use "components/card";
~~~

### CSS output

~~~css
.button {
  color: #ffffff;
  background-color: #4b2e83;
}

.card {
  color: #241b35;
  background-color: #f4f1fa;
  border: 1px solid #4b2e83;
}
~~~

The entrypoint configures the theme before loading either component. Both components load the same module and see the same configured values.

## 6. Supply a Sass variable as a configuration value

### styles/_project-settings.scss

~~~scss
$project-brand: #146c5b;
$project-surface: #f2faf7;
~~~

### styles/app.scss

~~~scss
@use "project-settings";

@use "theme" with (
  $brand-color: project-settings.$project-brand,
  $surface-color: project-settings.$project-surface
);

@include theme.card;
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #f2faf7;
  border: 2px solid #146c5b;
  border-radius: 0.5rem;
}
~~~

Sass permits variables before `@use` rules when they provide module configuration. The `@use` rules themselves must still appear before style rules.

## 7. Build light and dark CSS separately

Each entrypoint configures the theme once and is compiled separately.

### styles/light.scss

~~~scss
@use "theme" with (
  $brand-color: #1769aa,
  $surface-color: #ffffff,
  $text-color: #263238
);

@include theme.card;
~~~

### styles/dark.scss

~~~scss
@use "theme" with (
  $brand-color: #a8c7fa,
  $surface-color: #172033,
  $text-color: #f4f7fb
);

@include theme.card;
~~~

### CSS output from light.scss

~~~css
.card {
  color: #263238;
  background-color: #ffffff;
  border: 2px solid #1769aa;
  border-radius: 0.5rem;
}
~~~

### CSS output from dark.scss

~~~css
.card {
  color: #f4f7fb;
  background-color: #172033;
  border: 2px solid #a8c7fa;
  border-radius: 0.5rem;
}
~~~

These are separate Sass compilations. Do not place both configurations in the same entrypoint.

## 8. Emit multiple theme variants in one stylesheet with a mixin

### styles/_theme-styles.scss

~~~scss
@mixin card($brand-color, $surface-color, $text-color) {
  .card {
    color: $text-color;
    background-color: $surface-color;
    border: 2px solid $brand-color;
  }
}
~~~

### styles/app.scss

~~~scss
@use "theme-styles";

@include theme-styles.card(
  $brand-color: #1769aa,
  $surface-color: #ffffff,
  $text-color: #263238
);

[data-theme="dark"] {
  @include theme-styles.card(
    $brand-color: #a8c7fa,
    $surface-color: #172033,
    $text-color: #f4f7fb
  );
}
~~~

### CSS output

~~~css
.card {
  color: #263238;
  background-color: #ffffff;
  border: 2px solid #1769aa;
}

[data-theme=dark] .card {
  color: #f4f7fb;
  background-color: #172033;
  border: 2px solid #a8c7fa;
}
~~~

The mixin receives values at each include site, so one compilation can emit more than one variation. The nested selector scopes the dark card styles under the theme attribute.

## 9. Compare compile-time settings with runtime CSS custom properties

### Sass module configuration

~~~scss
// _theme.scss
$surface-color: #ffffff !default;

@mixin card {
  .card {
    background-color: $surface-color;
  }
}

// app.scss
@use "theme" with (
  $surface-color: #172033
);

@include theme.card;
~~~

This compiles to a fixed color:

~~~css
.card {
  background-color: #172033;
}
~~~

### Runtime CSS custom property

~~~scss
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

Here the browser can choose the custom property value from the active theme selector without recompiling Sass.

## 10. See why !default matters

### styles/_settings.scss

~~~scss
$brand-color: #1769aa;
~~~

### styles/app.scss

~~~scss
@use "settings" with (
  $brand-color: #4b2e83
);
~~~

This configuration fails because `$brand-color` was not declared with `!default` in the module.

The configurable version is:

~~~scss
// styles/_settings.scss
$brand-color: #1769aa !default;
~~~

Now the entrypoint can override the fallback:

~~~scss
// styles/app.scss
@use "settings" with (
  $brand-color: #4b2e83
);
~~~

## 11. Distinguish load-time configuration from reassignment

### styles/_theme-with-css.scss

~~~scss
$brand-color: #1769aa;

.card {
  border-color: $brand-color;
}
~~~

### styles/reassign.scss

~~~scss
@use "theme-with-css";

theme-with-css.$brand-color: #4b2e83;
~~~

The module's top-level CSS was evaluated when it loaded, so changing the Sass variable afterward does not rewrite that already-generated rule:

~~~css
.card {
  border-color: #1769aa;
}
~~~

For a configurable module, use `!default` and `@use ... with (...)` so the value is set before the module loads. Reassignment is a separate Sass feature and can make behavior depend on evaluation order.

## 12. A practical choice guide

| Need | Example approach |
| --- | --- |
| One build-wide brand setting | Configure `!default` variables with `@use ... with` |
| Separate light and dark CSS files | Compile separate entrypoints |
| Multiple variants in the same stylesheet | Use a parameterized mixin |
| Theme changes in the browser | Use CSS custom properties |
| Internal implementation value | Keep it private and do not expose it as configuration |

## Further reading

- [Sass: @use — Configuration](https://sass-lang.com/documentation/at-rules/use/#configuration)
- [Sass: @use — With Mixins](https://sass-lang.com/documentation/at-rules/use/#with-mixins)
- [Sass: @use — Reassigning Variables](https://sass-lang.com/documentation/at-rules/use/#reassigning-variables)
- [Sass: Variables — Default Values](https://sass-lang.com/documentation/variables/#default-values)
