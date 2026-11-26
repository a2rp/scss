# 035 - Migrating to Sass Modules: Q&A

## Migration planning

### 1. What does a successful Sass module migration accomplish?

It makes Sass dependencies explicit, defines a controlled public API where needed, and preserves the intended compiled CSS and consumer behavior. Replacing import syntax is only one part of the work.

### 2. Is replacing every `@import` with `@use` enough?

No. Each file must add its own module dependencies and qualify references with namespaces. Nested imports, public library entrypoints, CSS imports, and configuration require separate decisions.

### 3. What should I inventory before migrating?

List application entrypoints, internal partials, public Sass entrypoints, load paths, custom importers, third-party files, warning sources, and files that use nested Sass imports. Record which files the project owns and which it does not.

### 4. What is the difference between an entrypoint and a partial?

An entrypoint is compiled directly to a CSS bundle by the build. A partial is generally an implementation file loaded by other Sass files, often with a leading underscore. The same file can technically be compiled directly, but the project should intentionally choose its entrypoints.

### 5. Why does a partial need its own `@use` rule?

Sass module members are scoped to the stylesheet that loads them. If `_button.scss` refers to `tokens.$brand`, it must load the tokens module itself, even if the app entrypoint also loads tokens.

### 6. Does a module namespace affect the generated CSS selector names?

No. A namespace qualifies Sass variables, functions, and mixins in Sass source. It does not scope or rename CSS selectors emitted by the module.

### 7. Should I use `@use "..." as *` everywhere to avoid long namespaces?

No. `as *` removes the namespace signal and can cause name collisions. Prefer a clear module namespace, a short alias, or a well-designed facade. Use unnamespaced imports only for a small, controlled case where the names are known not to conflict.

### 8. Does `@use` load a module's CSS?

Yes. Any top-level CSS emitted by the module is included in the compilation. Sass loads a module once per compilation, but it does not automatically merge every duplicate selector emitted by different sources.

### 9. Is a top-level style rule in a module automatically scoped to that module?

No. Sass modules scope Sass members, not CSS selectors. A rule such as `.button { ... }` emits a normal global CSS selector unless the application uses a separate scoping mechanism.

## Public APIs and facades

### 10. What does `@forward` do?

`@forward` loads another Sass stylesheet and re-exports its public variables, functions, and mixins so a stylesheet that uses the forwarding module can access them.

### 11. What is the practical difference between `@use` and `@forward`?

`@use` makes another module's members available inside the current stylesheet under a namespace. `@forward` exposes another module's public members through the current stylesheet to its consumers. A file can use both when it needs to consume a module and re-export it.

### 12. Why create a package facade?

A facade gives consumers one stable Sass URL while allowing the library to reorganize internal files. It also makes the supported public API easier to document and review.

### 13. How can I expose only selected members?

Use `show` to forward an allowlist or `hide` to exclude selected members:

~~~scss
@forward "tokens" show $brand, $text;
@forward "mixins" hide debug-outline;
~~~

The names in these lists are Sass member names, including the dollar sign for variables.

### 14. How do I add a prefix to forwarded names?

Use a wildcard prefix:

~~~scss
@forward "components/button" as button-*;
~~~

A forwarded mixin named `reset` is then exposed as `button-reset`. The prefix becomes part of the public API.

### 15. How do I keep implementation helpers private?

Start a member's name with `-` or `_` to make it private to its Sass module, and avoid forwarding internal modules or members from the public facade.

### 16. If a facade both forwards and uses the same module, which rule should come first?

Usually put `@forward` before `@use` when the facade should pass configuration through to the forwarded module. The `@use` rule can then make that module's members available locally as well.

### 17. Can I change a facade's public names later?

You can, but consumers may depend on those names. Treat documented Sass members, facade paths, aliases, and prefixes as compatibility commitments and plan breaking changes deliberately.

## Module configuration

### 18. What does `!default` mean for migration?

A variable declared with `!default` supplies a default value that a consumer can override when the module is first loaded with `@use ... with`.

### 19. Can any Sass variable be configured through `with`?

No. The target module must declare the variable with `!default`. The configured variable name must also match a public configurable variable in that module.

### 20. What happens if another dependency loads the module before the entrypoint configures it?

The module has already been loaded with its original configuration. A later `@use ... with` cannot reconfigure the same module in that compilation. Configure shared modules early in the entrypoint before loading dependents.

### 21. Can two entrypoints use different configurations of the same module?

Yes, if they are compiled separately. Each Sass compilation has its own module loading and configuration. For example, compile `light.scss` and `dark.scss` as independent entrypoints.

### 22. Should a library use `@use ... with` for every kind of customization?

Not necessarily. It is convenient for a limited set of default variables. For dynamic or complex customization, a documented mixin-based configuration API may be more suitable.

## Sass Migrator

### 23. What does the module migrator automate?

It can convert many legacy Sass import relationships to `@use` and update Sass member references with namespaces. It may also update configuration patterns. Review its output because it cannot decide the project's intended API or CSS behavior.

### 24. How do I preview changes without applying them?

Use `--dry-run` and `--verbose`:

~~~bash
npx sass-migrator module --dry-run --verbose src/scss/app.scss
~~~

Add `--migrate-deps` when you want the migrator to propose edits to the loaded Sass dependencies too.

### 25. What does `--migrate-deps` include?

It asks the migrator to update Sass stylesheets loaded through the dependency graph. Inspect the changed files and confirm that each dependency is owned by the project or intentionally maintained there.

### 26. Does `--migrate-deps` mean third-party packages are safe to edit?

No. It may reach files that the project does not own, depending on the dependency graph and resolution configuration. Avoid committing edits to installed packages unless the project has a maintained patch strategy.

### 27. What if the project uses Sass load paths?

Pass the relevant load paths to the migrator so it can resolve dependencies consistently with the build. Review whether files reached through those paths are project-owned or external.

### 28. What is `--built-in-only` for?

It migrates global built-in function calls to namespaced Sass built-in module calls while leaving Sass `@import` rules in place. It is useful when the work is split into stages; it does not finish the import migration.

### 29. Can the migrator handle every build system or custom importer?

Do not assume so. If the project uses a framework plugin, custom importer, or package-specific resolution, run the migration and compile through the real build integration. The CLI may not reproduce every environment-specific behavior.

### 30. Should I run the migrator on the whole repository in one command?

For a small repository, a broad run may be manageable. For a large or published codebase, prefer a preview and focused slices so each diff is reviewable and any CSS or API change is easier to trace.

## Special cases and compatibility

### 31. How should I migrate a nested Sass import?

`@use` cannot be nested. Prefer exposing a mixin from the imported stylesheet and including it inside the desired at-rule. `sass:meta` and `meta.load-css()` can be an alternative when changing the loaded file is not practical.

### 32. Should a CSS import be replaced by `@use`?

No. Preserve browser-managed imports such as `@import url("...")` or imports of a `.css` file. Sass imports and CSS imports have different behavior.

### 33. What if a third-party library still uses Sass imports?

Check for an updated module-based release or official guidance. If none exists, decide whether to upgrade, replace, patch through a maintained mechanism, or temporarily tolerate its warnings. Do not rewrite installed package files casually.

### 34. Can `@use` load a module that still contains legacy imports?

Yes. Sass can load a stylesheet that itself uses `@import`, which can help migrate the application incrementally. The dependency may still produce deprecation warnings and its API may change when the dependency migrates.

### 35. Do import-only files eliminate the deprecation for legacy consumers?

No. An import-only file can provide a compatibility API for old Sass consumers while a module API is offered separately. Consumers using the import-only file still use the deprecated Sass `@import` rule.

### 36. Are paths in Sass module URLs Windows paths?

No. Sass module URLs use forward slashes, including on Windows. Match the file name's letter case so the code works on case-sensitive filesystems and does not accidentally load what Sass considers separate URLs.

## Verification and delivery

### 37. What should I compare after a migration?

Compile the same entrypoints with the same relevant options, then review CSS rules, declaration values, rule ordering, repeated output, and missing output. Also review Sass source and the public API; CSS comparison alone is not enough.

### 38. Does the module system deduplicate identical CSS selectors?

No. It prevents the same module from being loaded repeatedly in one compilation. It does not merge identical selectors emitted by different modules or repeated mixin includes.

### 39. How can I test a library's Sass API?

Create a small consumer fixture that loads the documented facade and references the documented variables, functions, and mixins. Compile it with the supported Sass toolchain. If the library promises an old API during transition, keep a separate fixture for that API.

### 40. Does silencing deprecation warnings mean the migration is complete?

No. Silencing changes warning output; it does not change source code, CSS behavior, or API compatibility. Track warnings from owned code separately from warnings in dependencies.

### 41. How should I split the migration into commits?

Use focused commits such as built-in namespace updates, one group of module boundaries, public facade changes, and compatibility cleanup. Each commit should compile and have a message that describes the migration slice.

### 42. What is a concise migration checklist?

Inventory dependencies and APIs, preview the migrator, convert small paths, define a facade if needed, configure modules before dependents load them, preserve CSS imports, review generated CSS, compile a consumer fixture, and inspect warnings and changed files.

## Quick reference

~~~scss
// Consume another module.
@use "tokens" as theme;

// Re-export selected public members.
@forward "tokens" show $brand, $text;

// Configure a default before dependents load it.
@use "theme" with (
  $brand: #8ab4f8
);
~~~

~~~bash
npx sass-migrator module --dry-run --verbose src/scss/app.scss
npx sass-migrator module --dry-run --verbose --migrate-deps src/scss/app.scss
npx sass-migrator module --migrate-deps src/scss/app.scss
npx sass-migrator module --built-in-only --migrate-deps src/scss/app.scss
~~~

## Related material

- [Lesson: Migrating to Sass modules](../lessons/035-migrating-to-sass-modules.md)
- [Examples: Migrating to Sass modules](../examples/035-migrating-to-sass-modules.md)
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="034-legacy-import-and-global-built-in-deprecations.md">034 - Legacy @import and Global Built-In Deprecations: Q&amp;A</a></td><td><a href="../lessons/035-migrating-to-sass-modules.md">Lesson</a></td><td><a href="../examples/035-migrating-to-sass-modules.md">Example</a></td><td><a href="036-sass-and-css-at-rules.md">036 - Sass with CSS At-Rules: Q&amp;A</a></td></tr>
  </tbody>
</table>
