# Project structure and naming conventions — Examples

The examples use Sass’s module system. File and folder names are illustrative; adapt them to the application’s build setup. Compile the application entry point with Dart Sass, for example:

~~~sh
sass src/styles/app.scss public/css/app.css
~~~

## 1. A small project layout

~~~text
src/
  styles/
    app.scss
    _tokens.scss
    _base.scss
    _components.scss
public/
  css/
    app.css
~~~

Use this when a few Sass modules are still easy to understand. The output CSS is separate from its authored Sass source.

## 2. A larger responsibility-based layout

~~~text
src/
  styles/
    app.scss
    settings/
      _tokens.scss
      _breakpoints.scss
      _index.scss
    foundations/
      _reset.scss
      _typography.scss
    components/
      _button.scss
      _card.scss
      _form-field.scss
    utilities/
      _visually-hidden.scss
public/
  css/
    app.css
~~~

Use folders when the files have real categories. Avoid creating empty categories before they are needed.

## 3. Compose an application entry point

~~~scss
// src/styles/app.scss
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "components/card";
@use "components/form-field";
@use "utilities/visually-hidden";
~~~

This is the Sass file the build should compile for the main application stylesheet.

## 4. Compile separate application bundles

~~~scss
// src/styles/public.scss
@use "foundations/reset";
@use "components/button";
@use "components/card";
~~~

~~~scss
// src/styles/admin.scss
@use "foundations/reset";
@use "components/button";
@use "admin/navigation";
@use "admin/dashboard";
~~~

Each entry point describes one bundle. Compile each intended entry separately and avoid also compiling all partials into standalone CSS.

## 5. Load a partial by its module URL

File path:

~~~text
src/styles/components/_button.scss
~~~

Module URL from <code>src/styles/app.scss</code>:

~~~scss
@use "components/button";
~~~

Sass omits the partial underscore and file extension from the URL.

## 6. Use descriptive, consistent names

~~~text
settings/
  _breakpoints.scss
  _tokens.scss
components/
  _form-field.scss
  _navigation-menu.scss
utilities/
  _visually-hidden.scss
~~~

These names reveal each file’s job. If a module’s responsibility changes, rename it to match its new role rather than leaving it under a vague label.

## 7. Keep module paths portable

~~~scss
@use "settings/tokens";
@use "components/form-field";
~~~

Use forward slashes and match the actual path casing, including on Windows. Avoid backslash paths such as <code>settings\tokens</code> in Sass module URLs.

## 8. Make token dependencies explicit

~~~scss
// settings/_tokens.scss
$color-brand: #3157d5;
$color-text: #182033;
$space-medium: 1rem;
~~~

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  padding: tokens.$space-medium;
  color: #fff;
  background: tokens.$color-brand;
}
~~~

The button module states its dependency and references members through the namespace.

## 9. Create a folder-level settings API

~~~scss
// settings/_index.scss
@forward "tokens" show $color-brand, $color-text;
~~~

~~~scss
// components/_card.scss
@use "../settings" as settings;

.card {
  color: settings.$color-text;
  border-color: settings.$color-brand;
}
~~~

The Sass URL names the folder and resolves its <code>_index.scss</code>. Only the listed token variables are forwarded.

## 10. Use a public API and a private helper

~~~scss
// settings/_spacing.scss
$_unit: 0.25rem;

@function _step($count) {
  @return $_unit * $count;
}

$space-small: _step(2);
$space-large: _step(6);
~~~

~~~scss
// components/_panel.scss
@use "../settings/spacing";

.panel {
  padding: spacing.$space-large;
}
~~~

The public spacing variable is usable by the component; the private helper and unit are implementation details.

## 11. Separate value-only modules from component CSS

~~~scss
// settings/_colors.scss
$color-brand: #3157d5;
$color-text: #182033;
~~~

~~~scss
// components/_link.scss
@use "../settings/colors" as colors;

.link {
  color: colors.$color-brand;
}

.link:focus-visible {
  outline: 2px solid colors.$color-text;
}
~~~

The settings module defines values. The component module emits selectors. Loading the settings module alone does not add those component rules.

## 12. Group feature-owned styles by feature

~~~text
src/
  styles/
    app.scss
    settings/
      _tokens.scss
    foundations/
      _reset.scss
  features/
    billing/
      _invoice-table.scss
      _payment-form.scss
    profile/
      _profile-card.scss
      _avatar.scss
~~~

An entry point can load a feature’s Sass module with a relative module URL or configured load path. Keep this approach consistent with how the application locates feature code.

## 13. Keep shared styles in a shared module

~~~scss
// styles/foundations/_typography.scss
body {
  font-family: system-ui, sans-serif;
  line-height: 1.5;
}

h1,
h2,
h3 {
  line-height: 1.2;
}
~~~

~~~scss
// styles/app.scss
@use "foundations/typography";
@use "../features/profile/profile-card";
@use "../features/billing/invoice-table";
~~~

Only put rules here when they are truly shared foundation styles. Keep feature-specific rules with their owning feature when that makes maintenance clearer.

## 14. Keep generated CSS separate from source

~~~text
project/
  src/
    styles/
      app.scss
  public/
    assets/
      app.css
      app.css.map
~~~

Configure the build command to write CSS and source maps to the chosen output folder. Do not edit generated CSS by hand; change its Sass source and rebuild it.

Whether the generated output is committed should follow the deployment workflow for the project.

## 15. Avoid an implicit global dependency

Problem: the component references a variable without loading its module.

~~~scss
// components/_button.scss
.button {
  color: $color-text;
}
~~~

Correction: make the dependency explicit.

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  color: tokens.$color-text;
}
~~~

The corrected file no longer depends on another stylesheet being loaded first.

## 16. Use an index only when it improves the interface

Useful index:

~~~scss
// settings/_index.scss
@forward "colors";
@forward "spacing";
~~~

Consumer:

~~~scss
@use "settings" as settings;

.panel {
  color: settings.$color-text;
  padding: settings.$space-large;
}
~~~

If consumers need one stable settings API, this can be convenient. If the index only forwards every internal file and hides where values live, direct module paths may be easier to maintain.

## 17. Keep a component’s naming contract together

~~~scss
// components/_alert.scss
.alert {
  padding: 1rem;
}

.alert__title {
  margin-block: 0 0.5rem;
}

.alert--warning {
  border-inline-start: 0.25rem solid #a65d00;
}
~~~

~~~html
<aside class="alert alert--warning">
  <h2 class="alert__title">Review required</h2>
  <p>Check the submitted details.</p>
</aside>
~~~

The file and class names make the relationship easy to find. The example uses one naming convention consistently.

## 18. Make an entry point easy to audit

~~~scss
// app.scss
// 1. Global foundations
@use "foundations/reset";
@use "foundations/typography";

// 2. Application components
@use "components/button";
@use "components/card";
@use "components/form-field";

// 3. Utilities
@use "utilities/visually-hidden";
~~~

The comments explain the order by responsibility. Use grouping comments when they help the team; avoid redundant comments on every line.

## 19. Migrate a small global-import project

Before:

~~~scss
// app.scss
@import "tokens";
@import "base";
@import "button";
~~~

After converting the modules:

~~~scss
// app.scss
@use "tokens";
@use "base";
@use "button";
~~~

~~~scss
// _button.scss
@use "../tokens" as tokens;

.button {
  color: tokens.$color-text;
}
~~~

The consumer now loads its dependency explicitly. In a real migration, check every file and compare the generated CSS; global import behavior may have been relied on elsewhere.

## 20. Avoid compiling every source file

Unfocused build input:

~~~text
src/styles/**/*.scss
~~~

If this pattern includes both entry points and partials, the build may create unintended CSS for partials. Prefer a configured list of application entries, such as:

~~~text
src/styles/app.scss
src/styles/admin.scss
~~~

The exact build syntax depends on the project’s tool. The important point is to define the intended outputs explicitly.

## 21. A naming audit worksheet

For each Sass file, record:

| File | Responsibility | Emits CSS? | Public members? | Loaded by |
| --- | --- | --- | --- | --- |
| <code>settings/_tokens.scss</code> | Design values | No | Yes | Components |
| <code>foundations/_reset.scss</code> | Document defaults | Yes | No | App entries |
| <code>components/_button.scss</code> | Button styles | Yes | No | App entries |
| <code>utilities/_visually-hidden.scss</code> | Reusable utility class | Yes | No | Selected entries |

Replace the sample rows with the files in your project. This quickly reveals vague modules, unused files, and entry points that load more CSS than needed.

## 22. Practice tasks

1. Choose one real CSS output and identify its Sass entry point.
2. Rename one vague Sass file to describe its responsibility.
3. Add an explicit <code>@use</code> dependency where a file relies on a global token.
4. Decide whether a folder-level <code>_index.scss</code> would improve the settings API.
5. Check Sass module URLs for backslashes and casing mismatches.
6. Separate generated CSS from authored Sass if they currently share an unclear location.
7. List which modules emit CSS and who loads them.
8. Remove one unnecessary wrapper or folder if it adds no useful boundary.

## Reference links

- [Sass modules and @use](https://sass-lang.com/documentation/at-rules/use/)
- [The @forward rule](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass stylesheet structure](https://sass-lang.com/documentation/syntax/structure/)
- [Sass module system](https://sass-lang.com/blog/the-module-system-is-launched/)
- [Sass migrator](https://sass-lang.com/documentation/cli/migrator/)
