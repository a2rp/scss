# Authoring a Sass library

A Sass library packages reusable variables, functions, mixins, and sometimes CSS so that other stylesheets can use them through a clear public API. The module system makes it possible to organize implementation across files while exposing a smaller entry point with <code>@forward</code>.

A good library makes its supported members easy to find, keeps internal details private, avoids unexpected CSS output, and documents how consumers load and configure it. Treat published Sass names and output behavior as an API that other projects may rely on.

## Learning goals

After this lesson, you should be able to:

- Design a small, namespaced public Sass API.
- Build a package entry point with <code>@forward</code>.
- Keep implementation members private or hidden.
- Separate Sass-only modules from modules that emit CSS.
- Offer configurable defaults without making configuration unpredictable.
- Expose Sass files through a package's <code>exports</code> map.
- Load a published package with the Dart Sass Node.js package importer.
- Validate API compatibility and generated CSS.

## 1. Decide whether a Sass library is needed

A set of styles used only inside one application may be simpler as ordinary project modules. A library is useful when styles need to be shared across applications or published as a reusable package.

Before creating one, define:

- Who will use it: one internal application, several company projects, or external users.
- What it provides: design tokens, mixins, functions, CSS components, or a combination.
- Which names consumers are allowed to depend on.
- Whether loading the package should emit CSS automatically.
- Which Sass implementation and minimum version the package supports.

Make those decisions before exposing every source file as a public entry point.

## 2. Separate public API from implementation

A library can contain internal modules and one stable entry point:

~~~text
sass/
  index.scss
  tokens.scss
  mixins.scss
  components.scss
  _tokens.scss
  _mixins.scss
  _functions.scss
  components/
    _button.scss
    _field.scss
  internal/
    _math.scss
    _selectors.scss
~~~

Consumers load the public entry point. They should not need to know which internal file defines each member. This leaves room to reorganize implementation without changing documented import paths.

Use partial filenames for Sass files intended to be loaded by other stylesheets rather than compiled as standalone outputs.

## 3. Create a namespaced public API with @forward

A library entry point can forward selected members from internal modules:

~~~scss
// sass/index.scss
@forward "tokens" show $brand, $surface, $radius;
@forward "mixins" show focus-ring, visually-hidden;
@forward "functions" show fluid-size;
~~~

A consumer loads the entry point with a namespace:

~~~scss
@use "acme-ui" as ui;

.card {
  color: ui.$brand;
  border-radius: ui.$radius;
}

.skip-link {
  @include ui.visually-hidden;
}
~~~

The namespace helps prevent collisions with members from other libraries. Keep member names simple within their modules and let the consumer's namespace distinguish the package.

## 4. A forwarded member is not automatically local to the facade

<code>@forward</code> exposes members to stylesheets that load the facade. It does not make those members directly available for use inside the facade's own Sass code.

If the facade also defines styles or members that use a forwarded module, load it with <code>@use</code> as well:

~~~scss
// sass/index.scss
@forward "tokens" show $brand, $surface;
@use "tokens" as tokens;

:root {
  --acme-brand: #{tokens.$brand};
}
~~~

When forwarding and using the same configurable module, put <code>@forward</code> first so configuration can be applied before the facade loads it with <code>@use</code>.

## 5. Expose only intentional members

Use <code>show</code> to publish an allowlist:

~~~scss
@forward "tokens" show $brand, $surface;
~~~

Use <code>hide</code> when most members are public but a few should remain internal:

~~~scss
@forward "tokens" hide $debug-palette, _internal-scale;
~~~

A member beginning with <code>-</code> or <code>_</code> is private to its defining module and cannot be loaded by consumers through the module system. Do not forward internal helper modules from the package's public entry points.

An allowlist is usually easier to review when a library has many implementation members: adding a new internal helper will not silently expand the public API.

## 6. Add a prefix when forwarding public names

A facade may add a consistent prefix to forwarded members:

~~~scss
@forward "mixins" as ui-*;
~~~

If the source module defines <code>focus-ring</code>, consumers refer to it as <code>ui-focus-ring</code> under the facade's namespace:

~~~scss
@use "acme-ui" as acme;

.button:focus-visible {
  @include acme.ui-focus-ring;
}
~~~

Use prefixing only when it improves the public naming scheme. Combining a package namespace and a long repeated prefix can make every call unnecessarily verbose.

## 7. Keep implementation modules explicit

Each Sass file has its own module scope. An implementation module should load the modules it uses:

~~~scss
// sass/_mixins.scss
@use "tokens" as tokens;

@mixin focus-ring {
  outline: 3px solid tokens.$focus;
  outline-offset: 2px;
}
~~~

A public subpath can re-export the implementation module:

~~~scss
// sass/mixins.scss
@forward "_mixins";
~~~

Do not rely on a consumer loading some other file first. A module should compile when loaded through its documented entry point without depending on accidental global members.

## 8. Separate Sass-only APIs from CSS output

A Sass-only module can define variables, functions, and mixins without emitting selectors:

~~~scss
// sass/_tokens.scss
$brand: #2457d6 !default;
$surface: #f4f6fb !default;
$radius: 0.5rem !default;
$focus: #8a4fff !default;
~~~

The package can provide a stable tokens subpath through a small facade:

~~~scss
// sass/tokens.scss
@forward "_tokens";
~~~

A separate stylesheet or mixin can emit CSS when the consumer chooses it:

~~~scss
// sass/_base.scss
@use "tokens" as tokens;

@mixin reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  body {
    margin: 0;
    color: tokens.$text;
  }
}
~~~

This lets consumers use the tokens without automatically receiving every CSS rule. A forwarded module can emit CSS if it contains CSS-producing rules; <code>@forward</code> does not suppress output.

## 9. Decide how component styles are included

There are several valid library designs:

1. **Sass tools only:** publish tokens, functions, and mixins; consumers write the selectors.
2. **Opt-in CSS mixins:** publish mixins that emit CSS only when included.
3. **Explicit stylesheet entry points:** publish a separate file that emits a deliberate set of CSS.
4. **Automatic CSS on load:** include CSS-producing rules in the module users load.

Prefer intentional, opt-in output when consumers need control over which styles are included. If loading the main entry automatically emits CSS, document that behavior clearly and avoid requiring consumers to load it from many entry files.

## 10. Use configurable defaults carefully

Variables marked <code>!default</code> can be configured by consumers when they first load a module:

~~~scss
// sass/_tokens.scss
$brand: #2457d6 !default;
$radius: 0.5rem !default;

.button {
  color: white;
  background: $brand;
  border-radius: $radius;
}
~~~

~~~scss
@use "acme-ui" as ui with (
  $brand: #125b45,
  $radius: 0.25rem
);
~~~

Configuration applies to the module instance for the compilation. It must be provided the first time that module is loaded; loading it earlier through another path can prevent later configuration from taking effect. Document the configurable names and their defaults.

Avoid making every implementation variable configurable. A public configuration variable becomes part of the library's compatibility commitment.

## 11. Configure an upstream module through a facade

A facade can set package defaults while still allowing downstream consumers to override them:

~~~scss
// sass/index.scss
@forward "tokens" with (
  $brand: #2457d6 !default,
  $radius: 0.5rem !default
);
~~~

The <code>!default</code> marker in the forwarding configuration means the facade's values act as defaults. A consumer can configure the forwarded values through the facade:

~~~scss
@use "acme-ui" with (
  $brand: #8a3ffc
);
~~~

Use this pattern when a package needs sensible defaults and a supported customization point. Keep the configured variable names stable once published.

## 12. Use maps for related configuration

A library with many related options may accept a map instead of exposing many independent values:

~~~scss
@use "sass:map";

$theme: (
  brand: #2457d6,
  surface: #f4f6fb,
  radius: 0.5rem,
);

@function setting($name) {
  @return map.get($theme, $name);
}
~~~

If consumers need to configure the map, define the map with <code>!default</code> and document its keys. Validate required keys and provide useful errors when an unsupported key is requested.

## 13. Make functions predictable

Public functions should have focused inputs, predictable return values, and clear documentation:

~~~scss
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

A function that accepts arbitrary strings or silently returns <code>null</code> can make consumer mistakes hard to locate. Use <code>@error</code> for invalid inputs when continuing would produce misleading CSS.

## 14. Avoid broad global namespace imports

A consumer can write <code>@use "library" as *</code> to avoid a namespace, but it increases the chance of collisions. Recommend a namespace in the library documentation:

~~~scss
@use "acme-ui" as ui;

.toolbar {
  gap: ui.space(medium);
}
~~~

The library's own implementation should also prefer explicit namespaces. This makes the origin of every member visible.

## 15. Package entry points with package.json exports

For a Node.js package, Sass can use a conditional <code>sass</code> export:

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
    }
  }
}
~~~

This declares a Sass entry point for the package root and an intentional tokens subpath. Include only entry points that consumers are meant to use. If the same npm package also provides JavaScript, add the appropriate JavaScript conditions and files to each export according to Node.js package rules.

The Sass <code>sass</code> export condition is interpreted by Sass's Node.js package importer. It does not itself make a stylesheet a JavaScript module.

## 16. Load a package through the Node.js pkg: importer

Dart Sass's Node.js package importer supports <code>pkg:</code> URLs. The importer was added in Dart Sass 1.71.0. With the CLI, enable it explicitly:

~~~sh
sass --pkg-importer=node src/app.scss dist/app.css
~~~

Then a stylesheet can load the package entry:

~~~scss
@use "pkg:@acme/ui-sass" as ui;

.heading {
  color: ui.$brand;
}
~~~

The package importer uses the package's <code>exports</code> Sass condition where available. Check the Sass version used by the actual build tool before relying on this behavior.

## 17. Configure the importer through the JavaScript API

When compiling through the Dart Sass JavaScript API, add a <code>NodePackageImporter</code> to the importer list:

~~~js
import * as sass from 'sass';

const result = sass.compile('src/app.scss', {
  importers: [new sass.NodePackageImporter()],
});

console.log(result.css);
~~~

This is useful for custom build scripts. A framework or bundler may configure its own importers and resolution rules, so check its documentation before adding another importer.

## 18. Publish stable subpaths, not source internals

A package can expose a root entry and selected subpaths:

~~~json
{
  "exports": {
    ".": {
      "sass": "./sass/index.scss"
    },
    "./mixins.scss": {
      "sass": "./sass/mixins.scss"
    },
    "./tokens.scss": {
      "sass": "./sass/tokens.scss"
    }
  }
}
~~~

Consumers can depend on these named entry points. If a file is not listed in <code>exports</code>, do not document it as a supported package import path. Avoid making internal file layout part of the public contract.

## 19. Choose a stable directory layout

~~~text
package/
  package.json
  README.md
  LICENSE
  sass/
    index.scss
    tokens.scss
    mixins.scss
    components.scss
    _tokens.scss
    _mixins.scss
    _functions.scss
    _components.scss
    internal/
      _math.scss
~~~

Keep documentation and package metadata near the source. If publishing compiled CSS as well as Sass, define distinct, documented outputs and package export paths for each format.

## 20. Document the public API

For each public entry point, document:

- The exact Sass URL consumers should load.
- Public variables, functions, and mixins.
- Default values and valid configuration.
- Whether loading the entry point emits CSS.
- Supported Sass implementation and minimum version.
- Any required importer or build-tool configuration.
- Compatibility and deprecation policy.

Provide examples that compile with the documented package version. Avoid telling users to import internal files that may move.

## 21. Keep package output intentional

A Sass library can emit CSS through top-level selectors, forwarded modules, or mixins included by the consumer. Test each public entry point and inspect the output:

~~~sh
sass --pkg-importer=node test/fixtures/entry.scss test/output.css
~~~

If an entry point is intended to expose tools only, its output should not unexpectedly contain global component styles. If it intentionally emits a stylesheet, document the emitted selectors and expected placement.

## 22. Test the public API from a consumer fixture

Create a fixture that uses only the public package URL:

~~~scss
@use "pkg:@acme/ui-sass" as ui;

.demo {
  color: ui.$brand;
  @include ui.focus-ring;
}
~~~

Compile it using the same Dart Sass version and importer mode that consumers are expected to use. This catches accidental reliance on private paths or local load-path settings.

## 23. Test each documented subpath

If the package documents a tokens entry point and a mixins entry point, write a small fixture for each:

~~~scss
@use "pkg:@acme/ui-sass/tokens.scss" as tokens;
@use "pkg:@acme/ui-sass/mixins.scss" as mixins;

.panel {
  color: tokens.$text;
  @include mixins.focus-ring;
}
~~~

Ensure each export path points to an existing Sass file and forwards only the documented API.

## 24. Use warnings and errors as part of the API

A library can warn about a deprecated member or error on invalid input:

~~~scss
@use "buttons" as buttons;

@mixin old-button {
  @warn "old-button is deprecated; use button instead.";
  @include buttons.button;
}
~~~

A deprecation warning should give consumers a replacement and, when possible, a removal timeline. Avoid warning during ordinary compilation without a clear action for the user.

## 25. Plan deprecations before removing members

When replacing a public member, a compatibility period can keep a wrapper that delegates to the new implementation:

~~~scss
@use "mixins" as mixins;

@mixin legacy-focus {
  @warn "legacy-focus is deprecated; use focus-ring instead.";
  @include mixins.focus-ring;
}
~~~

Document the replacement and planned removal. Removing or renaming a public variable, function, mixin, or package subpath can break downstream builds even if the library's own examples still compile.

## 26. Version the public Sass API

Use the package's versioning policy to communicate changes. A change to an internal helper that is not public may be safe; removing a documented public member or changing emitted CSS can be breaking for consumers.

Treat output changes as API changes when users rely on generated selectors, source order, or configuration behavior. Include migration notes for deliberate breaking changes.

## 27. Avoid a package entry that emits everything

A broad entry point that immediately emits every component's CSS can make it difficult for consumers to include only what they need. Consider separate Sass-only and CSS-emitting entry points, or opt-in mixins that emit selected components.

If a full stylesheet is useful, publish it as an explicit entry such as <code>pkg:@acme/ui-sass/full</code> and explain its output.

## 28. Keep compatibility claims specific

Document the Sass implementation and minimum version you actually support. Features such as <code>@use</code>, <code>@forward</code>, and the Node.js <code>pkg:</code> importer have version requirements. Do not claim support for legacy implementations that cannot compile the public API.

Pin or lock the compiler in the library's own validation environment, and test the lowest supported version when practical.

## 29. Check file names and import URL casing

Sass resolves module URLs and package paths. Use forward slashes in URLs, and preserve exact filename casing:

~~~scss
@use "pkg:@acme/ui-sass/tokens.scss";
~~~

A case mismatch can appear to work on a case-insensitive filesystem and fail in a different environment. Test on the platforms used by consumers or CI.

## 30. Common library-authoring mistakes

- Forwarding every implementation file and making internals public by accident.
- Assuming <code>@forward</code> makes names available inside the facade.
- Mixing global CSS emission into a tools-only entry point.
- Allowing undocumented configuration variables to become accidental API.
- Using <code>as *</code> and creating collisions.
- Publishing an <code>exports</code> path that points to a missing file.
- Requiring <code>pkg:</code> without documenting the Dart Sass version and importer setup.
- Testing only with relative local imports instead of the published package URL.
- Removing a public member without a deprecation or migration path.
- Depending on another module being loaded earlier by the consumer.

## Quick reference

| Need | Sass feature |
| --- | --- |
| Load members for this module | <code>@use</code> |
| Re-export public members | <code>@forward</code> |
| Publish only selected members | <code>show</code> or <code>hide</code> |
| Keep a member private to its module | Prefix its name with <code>-</code> or <code>_</code> |
| Add a forwarded naming prefix | <code>@forward "module" as prefix-*</code> |
| Offer configurable defaults | <code>!default</code> and <code>@use ... with</code> |
| Publish a Node Sass entry | The <code>sass</code> condition in <code>package.json</code> exports |
| Load a package by Node package resolution | <code>pkg:</code> with the Node package importer |
| Avoid automatic CSS output | Keep the tools entry point free of top-level CSS |

## Official references

- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: NodePackageImporter](https://sass-lang.com/documentation/js-api/classes/nodepackageimporter/)
- [Dart Sass CLI and the Node package importer](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: writing Sass packages](https://sass-lang.com/documentation/js-api/classes/nodepackageimporter/)
- [Sass: variables and configurable defaults](https://sass-lang.com/documentation/variables/)
- [Sass: @import and migration considerations](https://sass-lang.com/documentation/at-rules/import/)
- [npm package.json exports](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#exports)

## Next lesson

Continue with [Warnings, errors, and debugging](047-warnings-errors-and-debugging.md).
