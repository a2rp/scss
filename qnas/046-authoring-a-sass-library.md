# Authoring a Sass library — Q&A

This Q&A is about building reusable Sass APIs with the module system and, where relevant, publishing them as Node.js packages. Sass features and package-import behavior have version requirements, so verify the minimum version stated by the library and used by the consuming build.

## 1. What is a Sass library?

A Sass library is a set of reusable Sass modules distributed for other stylesheets to use. It may provide variables, functions, mixins, CSS output, or a combination of these through documented entry points.

## 2. When should I make a library instead of keeping Sass files in an app?

Make a library when multiple projects need the same stable API or when you intend to distribute reusable Sass. For styles used by only one app, ordinary project modules may be easier to change and maintain.

## 3. What should a library's public API include?

Only the members consumers are expected to use: documented variables, functions, mixins, and supported stylesheet entry points. Keep helpers and file layout details private so you can change the implementation without breaking downstream stylesheets.

## 4. What does @forward do?

<code>@forward</code> loads a Sass module and makes its public variables, functions, and mixins available to users of the forwarding stylesheet. It lets a package organize code across files while presenting a single entry point.

## 5. Does @forward make members available inside the forwarding file?

No. A forwarding file must also use <code>@use</code> if its own Sass code needs those members. When both rules load a configurable module, put <code>@forward</code> first so configuration can take effect before the local <code>@use</code>.

## 6. How do I create a single library entry point?

Create a facade such as <code>_index.scss</code> and forward the supported modules:

~~~scss
@forward "tokens" show $brand, $radius;
@forward "mixins" show focus-ring;
~~~

Consumers can then load one stable URL instead of depending on internal files.

## 7. How do consumers refer to members from a library?

Use the namespace created by <code>@use</code>:

~~~scss
@use "acme-ui" as ui;

.card {
  color: ui.$brand;
  @include ui.focus-ring;
}
~~~

Namespaces help make member origins clear and reduce naming collisions.

## 8. How do I expose only some members of an internal module?

Use <code>show</code> to forward an allowlist or <code>hide</code> to exclude selected members. A deliberate allowlist helps prevent newly added internal helpers from becoming public by accident.

## 9. How can I make a Sass member private?

Start its name with <code>-</code> or <code>_</code>. Sass treats those members as private to the module. For package-level privacy, also avoid forwarding internal modules from public entry points.

## 10. What does the @forward prefix option do?

<code>@forward "mixins" as ui-*</code> adds <code>ui-</code> to the names forwarded from that module. It can create a consistent public naming scheme, though a long prefix may be redundant when consumers already use a package namespace.

## 11. Should a library recommend @use ... as *?

Usually no. The unnamespaced form can create collisions with members from other modules. Document a clear namespace for consumers and use explicit namespaces inside the library as well.

## 12. Does @forward emit CSS?

It can. Forwarding a module loads it like <code>@use</code>; any CSS produced by that module is included in the compiled output. <code>@forward</code> is an API re-export rule, not a way to suppress CSS.

## 13. How do I make a tools-only entry point emit no CSS?

Keep its dependency graph free of top-level selectors and CSS-producing rules. Variables, functions, and mixin definitions generally do not emit CSS until a mixin is included.

## 14. Why separate a Sass-only entry from a stylesheet entry?

Consumers may want tokens or mixins without receiving a full set of global selectors. Separate entry points or opt-in mixins make CSS inclusion intentional and help prevent duplicate or unwanted output.

## 15. How do I let consumers configure library variables?

Define configurable values with <code>!default</code> and let consumers supply values on the first <code>@use ... with</code>:

~~~scss
@use "acme-ui" with (
  $brand: #125b45
);
~~~

Document which variables are configurable and their defaults.

## 16. Can consumers configure a module after it has already loaded?

No. Configuration must be supplied the first time that module is loaded during a compilation. If another module loaded it earlier, later configuration will not replace that module instance's configuration.

## 17. Why use @forward with !default configuration?

A facade can set its own defaults while still allowing consumers to override them. This is useful when the internal module defaults should differ from the library's public defaults. Document the resulting values and configuration path.

## 18. Should every internal variable use !default?

No. Every configurable variable increases the public surface and compatibility burden. Mark only intentional customization points as configurable.

## 19. Can a Sass library use a map for configuration?

Yes. A map can group related settings such as colors, spacing, or breakpoints. Document valid keys, defaults, and what happens when a caller asks for an unknown key. Use Sass's built-in map module for map operations.

## 20. How should a public Sass function handle invalid input?

Return a documented value for valid input and fail clearly for invalid input when continuing would create misleading CSS. Sass's <code>@error</code> can report a useful message; avoid silently returning <code>null</code> unless that behavior is intentional and documented.

## 21. What is a good Sass library directory structure?

Keep public entry points and internal modules separate. A common structure has <code>sass/_index.scss</code>, Sass-only token and mixin files, optional component styles, and an internal directory for private helpers. The consumer should not need to depend on that internal structure.

## 22. Does a Sass URL have to include the partial underscore and extension?

No. Sass can resolve a partial such as <code>_tokens.scss</code> from a module URL such as <code>tokens</code>. Use forward slashes in Sass URLs and match filename casing exactly.

## 23. What is the Node.js pkg: importer?

It is a Dart Sass importer that resolves <code>pkg:</code> URLs using Node.js package resolution. The built-in Node.js package importer is available starting in Dart Sass 1.71.0 and must be enabled in the compilation environment.

## 24. How do I enable the Node package importer on the Sass CLI?

Pass the CLI option:

~~~sh
sass --pkg-importer=node src/app.scss dist/app.css
~~~

The consuming build must use a Dart Sass version that supports the option. A bundler may have its own importer configuration.

## 25. How can Sass find a package entry point?

A package can define a <code>sass</code> condition in its <code>package.json</code> <code>exports</code> map. The Node package importer checks this conditional export when resolving a <code>pkg:</code> URL.

## 26. What does a Sass conditional export look like?

~~~json
{
  "name": "@acme/ui-sass",
  "exports": {
    ".": {
      "sass": "./sass/index.scss"
    }
  }
}
~~~

The path must exist in the published package. If the package also supports JavaScript imports, add the relevant JavaScript conditions and files rather than treating the Sass path as JavaScript.

## 27. Can a package expose multiple Sass subpaths?

Yes. Add explicit exported subpaths such as <code>./tokens.scss</code> or <code>./mixins.scss</code> and point each Sass condition to the intended file. Only document paths that are deliberately part of the package API.

## 28. Does package.json "exports" make SCSS a JavaScript module?

No. The <code>sass</code> condition tells Sass's importer where to find Sass source. It does not make the SCSS file executable as JavaScript or browser CSS.

## 29. Do I need to pass the Sass importer through the JavaScript API?

If your script needs to resolve <code>pkg:</code> URLs, add a <code>NodePackageImporter</code> instance to the Sass compile options. Frameworks and bundlers may already configure importers, so check their Sass integration before adding another one.

## 30. What should a library README explain?

Document the supported Sass version, exact package URLs, public members, configuration defaults, emitted CSS behavior, required importer options, and compatibility policy. Provide a minimal consumer example that uses only public entry points.

## 31. How can I validate the library from a consumer's perspective?

Create a small fixture that loads the published package URL, uses documented members, and compiles with the supported importer and Sass version. This catches accidental dependence on local load paths or private implementation files.

## 32. Should I test the package root and each subpath?

Yes, if you document them as public entry points. Confirm that each package export exists, resolves with the expected importer, exposes only supported members, and emits the intended CSS.

## 33. Can changes to generated CSS break users even if Sass members stay the same?

Yes. Consumers may rely on emitted selectors, cascade order, or global CSS behavior. Treat meaningful output changes as compatibility changes and describe them in release notes.

## 34. How should I deprecate a public mixin?

Keep a compatibility wrapper for a documented transition period, emit a useful <code>@warn</code>, and direct users to the replacement. State when removal is planned. Do not emit vague warnings that provide no next step.

## 35. What changes may require a major version?

Removing or renaming documented variables, functions, mixins, package subpaths, configuration options, or expected CSS output can break consumers. Follow the package's versioning policy and include migration guidance for breaking changes.

## 36. Can I change internal files without a major release?

Usually, if the files are not public and the documented API and output contract remain compatible. Avoid encouraging consumers to import internal paths, or those paths may become de facto public dependencies.

## 37. Should the library support LibSass or other old implementations?

Only if the public syntax and modules actually work there and you test that support. Features such as <code>@use</code>, <code>@forward</code>, and the Node.js package importer require Dart Sass. Be explicit about the implementation and version you support.

## 38. How do I avoid a duplicated CSS bundle?

Do not load a CSS-emitting entry from every component or application entry. Keep tokens and mixins Sass-only where possible, load global styles once, and include component CSS only through an intentional entry or mixin.

## 39. What are common mistakes when authoring a Sass package?

- Forwarding all implementation modules without reviewing the public API.
- Assuming a facade can use forwarded names without <code>@use</code>.
- Treating <code>@forward</code> as CSS-free.
- Publishing a package path that is absent from the tarball.
- Relying on a local load path that consumers do not have.
- Using <code>pkg:</code> without documenting the Dart Sass importer requirement.
- Exposing configurable variables without defining their contract.
- Removing documented names or changing CSS output without a migration plan.
- Testing only internal relative imports rather than public package imports.

## 40. What is the short library-author checklist?

- Define a small public API and stable entry points.
- Use <code>@forward</code> deliberately and <code>@use</code> for local dependencies.
- Keep internal helpers private and out of public exports.
- Separate Sass-only tools from CSS-emitting entry points.
- Document and test configurable defaults.
- Publish valid Sass conditional exports for Node package consumers.
- State the Sass implementation, minimum version, and importer setup.
- Test public entry points and the CSS they generate.
- Version API and output changes responsibly.

## Official references

- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: NodePackageImporter](https://sass-lang.com/documentation/js-api/classes/nodepackageimporter/)
- [Dart Sass CLI and the Node package importer](https://sass-lang.com/documentation/cli/dart-sass/)
- [Sass: variables and configurable defaults](https://sass-lang.com/documentation/variables/)
- [Sass: @import migration considerations](https://sass-lang.com/documentation/at-rules/import/)
- [npm package.json exports](https://docs.npmjs.com/cli/v11/configuring-npm/package-json#exports)
