# Authoring a Sass library — Examples

These examples build a small Sass library from internal modules to a documented package entry point. Package names and paths are illustrative; replace them with the package name and versions used by your project.

## 1. Start with a library source layout

~~~text
acme-ui-sass/
  package.json
  README.md
  sass/
    index.scss
    tokens.scss
    mixins.scss
    components.scss
    _tokens.scss
    _mixins.scss
    _functions.scss
    _internal/
      _scale.scss
~~~

Consumers will use the package entry point. Internal modules can be reorganized as long as the public API remains stable.

## 2. Define Sass-only tokens

~~~scss
// sass/_tokens.scss
$brand: #2457d6 !default;
$text: #202534 !default;
$surface: #f4f6fb !default;
$radius: 0.5rem !default;
$focus: #8a4fff !default;
~~~

These variables define defaults but emit no selectors by themselves. Provide a stable tokens subpath through a small facade:

~~~scss
// sass/tokens.scss
@forward "_tokens";
~~~

This gives consumers a documented token entry without exposing the internal file layout.

## 3. Define a reusable mixin with explicit dependencies

~~~scss
// sass/_mixins.scss
@use "tokens" as tokens;

@mixin focus-ring {
  outline: 3px solid tokens.$focus;
  outline-offset: 2px;
}

@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
  white-space: nowrap;
}
~~~

A public subpath can re-export the implementation module:

~~~scss
// sass/mixins.scss
@forward "_mixins";
~~~

The module loads the tokens it needs. A consumer should not need to load tokens first for this module to work.

## 4. Add a function using a Sass built-in module

~~~scss
// sass/_functions.scss
@use "sass:map";

$spacing: (
  small: 0.5rem,
  medium: 1rem,
  large: 1.5rem,
);

@function space($name) {
  @if not map.has-key($spacing, $name) {
    @error "Unknown spacing token: #{$name}.";
  }

  @return map.get($spacing, $name);
}
~~~

A clear error makes invalid input easier to diagnose than silently generating a missing CSS value.

## 5. Create a public facade with allowlisted members

~~~scss
// sass/index.scss
@forward "tokens" show $brand, $text, $surface, $radius, $focus;
@forward "mixins" show focus-ring, visually-hidden;
@forward "functions" show space;
~~~

The facade publishes selected members while keeping the internal file organization out of consumer stylesheets.

## 6. Use the library through its facade

~~~scss
// app.scss
@use "acme-ui" as ui;

.card {
  color: ui.$text;
  background: ui.$surface;
  border-radius: ui.$radius;
  padding: ui.space(medium);
}

.card:focus-visible {
  @include ui.focus-ring;
}
~~~

The namespace makes each member's source clear.

## 7. Use @use inside the facade when it needs a forwarded value

~~~scss
// sass/index.scss
@forward "tokens" show $brand;
@use "tokens" as tokens;

:root {
  --acme-brand: #{tokens.$brand};
}
~~~

The forward makes <code>$brand</code> available to consumers. The local use lets the facade refer to that member in its own code.

## 8. Forward only what consumers should see

~~~scss
// sass/_helpers.scss
$gap: 1rem;
$debug-grid: false;

@mixin stack {
  display: flex;
  flex-direction: column;
  gap: $gap;
}
~~~

~~~scss
// sass/index.scss
@forward "helpers" show $gap, stack;
~~~

The internal debug value is not included in the facade's public API.

## 9. Hide an internal member while forwarding the rest

~~~scss
@forward "helpers" hide $debug-grid;
~~~

This is useful when most members of a module are intentionally public but one or more must remain implementation details.

## 10. Prefix names forwarded from a module

~~~scss
// sass/index.scss
@forward "mixins" as ui-*;
~~~

If <code>_mixins.scss</code> defines <code>focus-ring</code>, consumers write:

~~~scss
@use "acme-ui" as acme;

.button:focus-visible {
  @include acme.ui-focus-ring;
}
~~~

A namespace plus a prefix can be redundant; choose one consistent public naming style.

## 11. Mark internal members private

~~~scss
// sass/_internal/_scale.scss
$-factor: 1.25;

@function -step($value) {
  @return $value * $-factor;
}
~~~

Members starting with a hyphen or underscore are private to their Sass module and are not available to consumers through <code>@use</code>.

## 12. Configure tokens from a consumer stylesheet

~~~scss
@use "acme-ui" as ui with (
  $brand: #125b45,
  $radius: 0.25rem
);

.button {
  color: white;
  background: ui.$brand;
  border-radius: ui.$radius;
}
~~~

The library's corresponding defaults must be declared with <code>!default</code>. Configuration needs to be applied the first time the module is loaded during this compilation.

## 13. Configure defaults through a facade

~~~scss
// sass/index.scss
@forward "tokens" with (
  $brand: #2457d6 !default,
  $radius: 0.5rem !default
);
~~~

Consumers can override the facade's defaults:

~~~scss
@use "acme-ui" with (
  $brand: #8a3ffc
);
~~~

This creates a stable place to set package defaults without exposing the implementation module path.

## 14. Keep the tools entry point free of emitted CSS

~~~scss
// sass/index.scss
@forward "tokens";
@forward "mixins";
@forward "functions";
~~~

If the forwarded modules contain only variable, function, and mixin definitions, loading the facade does not emit selectors. Verify the output when changing the facade's dependencies.

## 15. Offer an opt-in mixin for component CSS

~~~scss
// sass/_button.scss
@mixin button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border: 0;
  border-radius: 0.5rem;
  padding: 0.65rem 1rem;
}
~~~

~~~scss
// sass/index.scss
@forward "button" show button;
~~~

~~~scss
// consumer.scss
@use "acme-ui" as ui;

.save-button {
  @include ui.button;
}
~~~

Only the consumer's <code>.save-button</code> selector is emitted from the mixin. The library does not add button CSS merely because the API is loaded.

## 16. Publish a deliberate CSS entry point

~~~scss
// sass/components.scss
@use "button";

.button {
  @include button.button;
}
~~~

A package can expose this CSS-producing entry separately from its Sass tools. Document that loading it emits a global <code>.button</code> selector and should be done once.

## 17. Map package root and subpaths in package.json

~~~json
{
  "name": "@acme/ui-sass",
  "version": "1.0.0",
  "exports": {
    ".": {
      "sass": "./sass/index.scss"
    },
    "./tokens.scss": {
      "sass": "./sass/tokens.scss"
    },
    "./mixins.scss": {
      "sass": "./sass/mixins.scss"
    },
    "./components.scss": {
      "sass": "./sass/components.scss"
    }
  }
}
~~~

Each exported file must be present in the package and intentionally supported. The root entry is the main public API; subpaths should be added only for useful, documented use cases.

## 18. Add a fallback JavaScript export when one package provides both

~~~json
{
  "name": "@acme/ui",
  "exports": {
    ".": {
      "sass": "./sass/index.scss",
      "import": "./dist/index.js",
      "require": "./dist/index.cjs",
      "default": "./dist/index.js"
    }
  }
}
~~~

The Sass condition is for Sass-aware importers. JavaScript conditions must point to valid JavaScript files. Do not point the JavaScript default condition at an SCSS file.

## 19. Load the package root with the Sass CLI

Install the package in a consumer project and enable the Node package importer:

~~~sh
npm install @acme/ui-sass
sass --pkg-importer=node src/app.scss dist/app.css
~~~

Consumer stylesheet:

~~~scss
@use "pkg:@acme/ui-sass" as ui;

.notice {
  color: ui.$brand;
}
~~~

The package importer requires a compatible Dart Sass version. A framework or bundler may have its own package resolution behavior.

## 20. Load an exported subpath

~~~scss
@use "pkg:@acme/ui-sass/tokens.scss" as tokens;

.notice {
  color: tokens.$text;
}
~~~

The subpath must be listed in the package's <code>exports</code> map and point to a file included in the published package.

## 21. Enable NodePackageImporter in a Sass JavaScript script

~~~js
import * as sass from 'sass';

const result = sass.compile('src/app.scss', {
  importers: [new sass.NodePackageImporter()],
});

console.log(result.css);
~~~

This programmatic approach is useful when a custom build script owns compilation. A framework or bundler may already configure Sass importers.

## 22. Create a consumer fixture for the public API

~~~scss
// test/fixtures/public-api.scss
@use "pkg:@acme/ui-sass" as ui;

.sample {
  color: ui.$brand;
  border-radius: ui.$radius;
  @include ui.focus-ring;
}
~~~

The fixture checks the documented package URL and members rather than internal relative paths.

## 23. Compile the fixture in the expected importer mode

~~~sh
sass --pkg-importer=node test/fixtures/public-api.scss test/output/public-api.css
~~~

Run this with the Sass version the package claims to support. Inspect the generated CSS and make sure expected declarations appear without unintended global styles.

## 24. Check a Sass-only entry emits no CSS

Fixture:

~~~scss
@use "pkg:@acme/ui-sass" as ui;

.probe {
  color: ui.$brand;
}
~~~

Expected output should contain the <code>.probe</code> rule but no unrelated component selectors. Keep fixtures for tools-only and CSS-emitting entry points separate.

## 25. Keep a public API manifest in the README

~~~md
## Sass API

Load the tools API with @use "pkg:@acme/ui-sass" as ui.

- Variables: $brand, $surface, $radius
- Mixins: focus-ring, visually-hidden
- Functions: space($name)

The tools entry emits no global component CSS.
~~~

Update this list when adding or deprecating members. It gives users a concise contract and helps reviewers notice accidental API changes.

## 26. Deprecate an old mixin with a wrapper

~~~scss
// sass/_legacy.scss
@use "mixins" as current;

@mixin old-focus {
  @warn "old-focus is deprecated; use focus-ring instead.";
  @include current.focus-ring;
}
~~~

Forward the wrapper during the announced migration period:

~~~scss
@forward "legacy" show old-focus;
~~~

The warning names a replacement. Document the planned removal instead of leaving the warning timeline ambiguous.

## 27. Check paths that are listed in exports

~~~text
package.json
sass/
  index.scss
  tokens.scss
  mixins.scss
  components.scss
  _tokens.scss
  _mixins.scss
~~~

An export such as <code>./sass/components.scss</code> must match an actual package file exactly. Check both spelling and letter casing, especially when publishing from Windows to environments with case-sensitive filesystems.

## 28. Verify files included in the npm package

~~~sh
npm pack --dry-run
~~~

Review the file list before publishing. Confirm that every Sass entry referenced by <code>exports</code> and every internal module it loads will be included in the package.

## 29. Compare the public API between releases

~~~text
Release 1:
  ui.$brand
  ui.space()
  ui.focus-ring

Release 2:
  ui.$brand
  ui.space()
  ui.focus-ring
  ui.visually-hidden
~~~

Adding a documented member is usually compatible. Removing or changing an existing member or altering emitted CSS may break consumers and should follow the project's versioning and migration policy.

## 30. Review a compact publishing checklist

~~~text
[ ] Public Sass entry point is documented
[ ] Internal modules are not accidentally exported
[ ] Configurable defaults and their values are documented
[ ] CSS output behavior is explicit
[ ] package.json Sass exports point to real files
[ ] Sass package files are included by npm pack
[ ] pkg: importer and minimum Dart Sass version are documented
[ ] Consumer fixtures compile through the public package URL
[ ] Breaking changes have release notes and migration guidance
~~~

## Official references

- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: NodePackageImporter](https://sass-lang.com/documentation/js-api/classes/nodepackageimporter/)
- [Dart Sass CLI and the Node package importer](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: variables and configurable defaults](https://sass-lang.com/documentation/variables/)
- [Sass: @import migration considerations](https://sass-lang.com/documentation/at-rules/import/)
- [npm package.json exports](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#exports)
