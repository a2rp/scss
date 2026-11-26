# 035 - Migrating to Sass Modules

## Learning goals

After this lesson, you should be able to:

- Plan a Sass module migration in small, reviewable steps.
- Decide which files are entrypoints, implementation modules, and public APIs.
- Replace implicit global dependencies with explicit `@use` rules and namespaces.
- Build a stable package facade with `@forward`.
- Configure modules safely with `!default` and `@use ... with`.
- Use Sass Migrator to automate mechanical work, then review the result.
- Preserve intended CSS output and public Sass APIs while the migration is in progress.

## 1. Migration is a source and API change

Moving from Sass imports to modules is more than replacing the word `@import` with `@use`. A migration changes how a stylesheet finds its variables, functions, and mixins:

- With legacy Sass imports, members become available in a shared global scope.
- With `@use`, each file declares its own dependencies and refers to their members through a namespace.
- With `@forward`, a package can deliberately expose a public API through a facade.
- Module configuration becomes explicit and is applied when the module is first loaded.

The resulting CSS may also change if the old project emitted styles through repeated imports, depended on import order, or had selectors that relied on generated global state.

Treat the migration as a behavior-preserving refactor. First capture how the project works, then change its source structure, then compare the compiled output and public API.

## 2. Inventory the project before editing

Start by listing the Sass files and finding all import sites. Identify:

1. **Application entrypoints** - files compiled into CSS bundles, such as `app.scss` and `admin.scss`.
2. **Implementation partials** - internal files such as `_tokens.scss`, `_buttons.scss`, and `_layout.scss`.
3. **Public library entrypoints** - files consumers are expected to load.
4. **Third-party dependencies** - stylesheets installed from packages or supplied through load paths.
5. **Special cases** - nested imports, CSS imports, import-only compatibility files, and files with global configuration.

Useful searches:

~~~bash
rg -n "@import|map-get\(|nth\(|unit\(" src --glob "*.scss" --glob "*.sass"
find src -type f \( -name "*.scss" -o -name "*.sass" \)
~~~

Adapt these commands to the tools installed on your machine. On Windows, a file search in your editor or PowerShell is also fine.

Make a migration table before changing code:

| File or area | Current role | Loaded by | Owned here? | Migration concern |
| --- | --- | --- | --- | --- |
| `src/scss/app.scss` | App entrypoint | Build command | Yes | Main dependency graph |
| `src/scss/_tokens.scss` | Internal module | App and components | Yes | Variables need namespaces |
| `src/scss/vendor/_legacy.scss` | Dependency | App | No | Check dependency support |
| `src/scss/_print.scss` | Style partial | App, inside media | Yes | Nested import needs a deliberate replacement |

This inventory prevents a broad automated change from silently rewriting code you do not maintain.

## 3. Record the existing behavior

Before refactoring, record the project's current state:

- The exact entrypoints and build commands.
- Sass compiler and migrator versions.
- Load paths, importers, and package resolution settings.
- Current deprecation warnings.
- Generated CSS for representative entrypoints.
- Public variables, functions, and mixins used by consumers.
- Any expected visual or runtime behavior that depends on the compiled CSS.

Keep generated CSS snapshots outside source folders if the project does not normally commit build output. The purpose is comparison, not to add build artifacts accidentally.

If a component relies on a global variable, write down where that value currently comes from. Its apparent source may be an indirect import several files away.

## 4. Give each file an explicit module boundary

A Sass module should declare the dependencies whose members it uses. Do not assume an entrypoint's `@use` statements are visible inside a partial.

### Before: hidden global dependency

~~~scss
// _tokens.scss
$brand: #1769aa;

// _button.scss
.button {
  color: $brand;
}

// app.scss
@import "tokens";
@import "button";
~~~

The button partial works only because the entrypoint imported tokens first. Compiling or understanding the partial by itself is difficult.

### After: dependency declared by the file that uses it

~~~scss
// _tokens.scss
$brand: #1769aa;

// _button.scss
@use "tokens";

.button {
  color: tokens.$brand;
}

// app.scss
@use "button";
~~~

Now `_button.scss` documents its own dependency. The app entrypoint loads the button module, and the button module loads tokens.

A file should not add a redundant `@use` merely because another file uses the same dependency. It should add the rule when that file itself refers to the dependency's Sass members.

## 5. Migrate one dependency path at a time

For a large codebase, avoid a single unreviewed rewrite of every Sass file. Choose a small dependency path, for example:

1. A leaf module containing values or helpers.
2. The component or feature that consumes it.
3. The entrypoint that loads the feature.
4. Any public facade that exposes those members.

Convert that path, compile it, and review the diff. Then move to the next path.

A useful order is:

1. Replace global built-in calls with namespaced built-in modules.
2. Convert low-level Sass imports to explicit module use.
3. Update the Sass files that consume those members.
4. Migrate the application entrypoints.
5. Build or repair the public library facade.
6. Handle nested imports and compatibility requirements.
7. Remove temporary migration scaffolding only after consumers are updated.

Global built-ins and Sass imports are separate deprecations. Completing one part does not automatically complete the other. See the previous chapter for those warning categories.

## 6. Choose namespaces that make ownership clear

By default, a module's namespace is based on its file name. Use that namespace when it is clear and unique:

~~~scss
@use "tokens";
@use "breakpoints";

.card {
  color: tokens.$text;
  @media (min-width: breakpoints.$wide) {
    max-width: 70rem;
  }
}
~~~

Use an alias when the default name is unclear or collides:

~~~scss
@use "components/card" as card-component;
@use "admin/components/card" as admin-card;

.page {
  @include card-component.surface;
  @include admin-card.surface;
}
~~~

Avoid `@use "..." as *` for application and shared library code. It removes the namespace signal and can recreate the same naming collisions the module system is intended to prevent. An unnamespaced load can be reasonable for a tightly controlled local module, but make that decision deliberately.

If a namespace becomes long in many places, consider a facade that presents a better public API instead of weakening every import.

## 7. Separate implementation modules from the public API

A folder can contain many internal files while exposing only a small supported surface to its users. An `_index.scss` file can forward chosen modules:

~~~scss
// styles/_tokens.scss
$brand: #1769aa !default;
$text: #263238 !default;

// styles/_mixins.scss
@mixin visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip-path: inset(50%);
}

// styles/_index.scss
@forward "tokens" show $brand, $text;
@forward "mixins" show visually-hidden;
~~~

A consumer can then load the package facade:

~~~scss
@use "styles" as ui;

.banner {
  color: ui.$text;
  background: ui.$brand;
  @include ui.visually-hidden;
}
~~~

The facade provides one stable module URL even if internal files are later moved or reorganized.

### @use and @forward serve different purposes

- `@use` makes a module's public members available inside the current stylesheet under a namespace.
- `@forward` re-exports public members to stylesheets that load the current stylesheet.

A facade that needs to use a forwarded member internally may need both rules:

~~~scss
@forward "tokens" show $brand;
@use "tokens";

.preview {
  color: tokens.$brand;
}
~~~

If a stylesheet both forwards and uses the same module, put `@forward` before its `@use` when the facade should pass configuration through to the forwarded module.

## 8. Make the public API intentional

A library's Sass API is a compatibility surface. Before changing it, check:

- Which Sass files consumers currently load.
- Which variables, functions, and mixins they use.
- Whether names include legacy prefixes.
- Whether public members are documented or represented in examples.
- Whether consumers load individual files or one central entrypoint.

Use `show` and `hide` to control which members are re-exported. Use a prefix when forwarded names need context:

~~~scss
// components/_button.scss
@mixin reset {
  border: 0;
  font: inherit;
}

// _index.scss
@forward "components/button" as button-*;
~~~

Consumers of the facade can call `@include ui.button-reset;`. A prefix is part of the public name; changing it later can break consumers.

Members beginning with `-` or `_` are private to the module and are not available to other modules. Do not expose implementation-only helpers just because they happen to be defined in a file that is forwarded wholesale.

## 9. Plan module configuration

A module can define defaults that an entrypoint overrides when it first loads that module:

~~~scss
// _theme.scss
$brand: #1769aa !default;
$surface: white !default;

// _button.scss
@use "theme";

.button {
  color: theme.$surface;
  background: theme.$brand;
}

// dark.scss
@use "theme" with (
  $brand: #8ab4f8,
  $surface: #20242a
);

@use "button";
~~~

The values in `dark.scss` configure the shared theme module. `_button.scss` loads the same module and sees that configuration.

Configuration is applied once per module per Sass compilation. Make the intended configuration happen before another dependency loads that module. Only variables declared with `!default` are configurable through `with`.

For many configuration values, a map or a dedicated mixin-based API may be easier to maintain than a long list of configured variables. Choose the API based on how consumers need to customize the library.

## 10. Use Sass Migrator as a starting point

The module migrator can rewrite many import relationships and member references. Install or invoke the version appropriate for the project. For an npm-based project:

~~~bash
npm install --save-dev sass-migrator
~~~

Preview the change to an entrypoint and the dependencies it loads:

~~~bash
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

Review the proposed diff before applying it:

~~~bash
npx sass-migrator module --migrate-deps src/scss/app.scss
~~~

The migrator may add `@use` rules, add member namespaces, and adjust legacy configuration. It can also change files reached through dependencies. Check every changed path before accepting the migration.

When Sass files are found through a load path, pass the same load path to the migrator:

~~~bash
npx sass-migrator --load-path=src/scss module --dry-run --verbose --migrate-deps src/scss/app.scss
~~~

If you need to update global built-ins separately, `--built-in-only` can migrate those calls while leaving Sass `@import` rules for a later step.

The migrator cannot decide whether a public API should keep an old name, whether repeated CSS was intentional, or whether an output difference is acceptable. Those are project decisions.

## 11. Handle files the migrator cannot safely decide

### Nested imports

`@use` must be at the top level. Convert nested Sass imports to a mixin that emits styles where requested, or use `sass:meta` and `meta.load-css()` where it is appropriate. The previous chapter explains these patterns.

### Third-party modules

Check whether the library has a module-based release or official migration instructions. Avoid editing package files that are installed during dependency setup; such edits are temporary and likely to be overwritten.

If an upstream dependency still uses imports, decide whether to upgrade it, replace it, patch it through a maintained mechanism, or temporarily tolerate its warnings.

### CSS imports

Preserve browser-managed CSS imports such as `@import url("...")` and imports of `.css` files. They are not Sass module imports.

### Custom importers and build integrations

If the project uses a custom importer, framework plugin, or unusual load path, test the migration through the actual build integration. A successful command-line compile alone may not represent how production resolves modules.

## 12. Verify generated CSS and behavior

After each migration slice, compile the same entrypoint with the same relevant options used before the change. Compare the generated CSS and investigate differences such as:

- Selectors or declarations appearing more than once.
- Rules emitted in a different order.
- Missing styles because a mixin is no longer included.
- A variable receiving a different value because configuration was loaded too late.
- Different CSS output from a changed built-in function.
- Imports that have become module loads or remained browser-level CSS imports.

Review the source diff as well as the CSS diff. Output equivalence is important, but it does not prove the public Sass API is still compatible.

For a library, compile a small consumer fixture that loads the documented facade and uses its documented members. If compatibility is promised, also compile a fixture for the legacy entrypoint while it remains supported.

## 13. A practical migration checklist

### Before the migration

- [ ] List entrypoints, partials, load paths, and Sass dependencies.
- [ ] Identify files owned by the project versus third-party packages.
- [ ] Record compiler commands, warnings, and representative CSS output.
- [ ] Identify the public Sass API and known consumers.

### During the migration

- [ ] Convert a small dependency path at a time.
- [ ] Add each dependency to the file that uses its members.
- [ ] Use clear namespaces; avoid accidental global exposure.
- [ ] Add a facade with `@forward` when the package needs a stable public API.
- [ ] Configure `!default` variables before the module is loaded.
- [ ] Treat nested imports, CSS imports, and third-party imports as separate cases.
- [ ] Preview migrator changes before applying them.

### After each slice

- [ ] Compile through the project's real build command.
- [ ] Review Sass source and generated CSS differences.
- [ ] Verify documented public members with a consumer fixture.
- [ ] Check warnings and confirm the change did not only hide them.
- [ ] Commit a focused migration with an informative message.

## 14. Common migration mistakes

### Assuming @use is a drop-in spelling change

It is not. Add namespaces and make each file's own dependencies explicit.

### Assuming an entrypoint's members are visible everywhere

Each Sass file has its own module scope. Add `@use` where members are actually referenced.

### Forwarding every internal member

A facade is a public API. Use `show`, `hide`, prefixes, and private member names to limit accidental commitments.

### Configuring a module after it was already loaded

Configuration only applies the first time a module is loaded in a compilation. Set it in the entrypoint before loading dependents.

### Treating migrator output as a finished design

The migrator is a useful mechanical assistant. Review naming, CSS output, API compatibility, and ownership of every changed dependency.

### Rewriting third-party packages in place

Dependency installation may erase local edits. Prefer supported upstream updates or a maintained patch process.

### Silencing warnings as proof of completion

Silencing changes diagnostic output, not Sass source or compatibility. Track owned and third-party warnings separately.

## 15. Summary

A successful Sass module migration makes dependencies explicit while preserving the behavior and APIs the project intends to support. Inventory the dependency graph, convert a small path at a time, use `@use` for local dependencies and `@forward` for public facades, and establish module configuration before dependents load it. Use Sass Migrator to reduce mechanical work, then verify source, generated CSS, build integration, and consumer-facing APIs.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="034-legacy-import-and-global-built-in-deprecations.md">034 - Legacy @import and Global Built-In Deprecations</a></td><td><a href="../qnas/035-migrating-to-sass-modules.md">Q&amp;A</a></td><td><a href="../examples/035-migrating-to-sass-modules.md">Example</a></td><td><a href="036-sass-and-css-at-rules.md">036 - Sass with CSS At-Rules</a></td></tr>
  </tbody>
</table>
