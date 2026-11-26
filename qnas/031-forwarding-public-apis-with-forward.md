# 031 — Forwarding Public APIs with @forward: Q&A

## Core behavior

### 1. What does @forward do?

It loads a Sass module and makes its public variables, functions, and mixins available to users of the forwarding module.

### 2. Why use @forward?

It lets a Sass library expose one documented entrypoint while keeping implementation code split across internal files.

### 3. How does a consumer load the forwarded API?

The consumer uses the forwarding stylesheet with `@use`:

~~~scss
@use "design-system" as ui;

.button {
  @include ui.button-reset;
}
~~~

The namespace comes from the consumer's `@use` rule, and the member name comes from the forwarding API.

### 4. Does @forward make members available inside the forwarding stylesheet?

No. It exposes them to downstream users. If the forwarding stylesheet itself needs a member, it must also load the module with `@use`.

### 5. Does @forward require a module namespace?

No. `@forward` does not define a namespace for use inside its own stylesheet. A downstream stylesheet chooses a namespace when it writes `@use`.

### 6. Can a consumer load a directory entrypoint?

Yes. Sass can resolve a directory URL through its `_index.scss` file. For example, `@use "design-system"` can load `design-system/_index.scss`.

## Controlling visibility

### 7. What is the default visibility of @forward?

By default, it forwards all public variables, functions, and mixins from the loaded module. Private members beginning with `-` or `_` are not public API members.

### 8. What does show do?

A `show` clause forwards only the listed members:

~~~scss
@forward "buttons" show reset, accessible-focus;
~~~

### 9. What does hide do?

A `hide` clause forwards all public members except the listed ones:

~~~scss
@forward "tokens" hide $internal-scale;
~~~

### 10. Which members should I list with a dollar sign?

List variables with `$`, such as `$brand-color`. List mixins and functions by member name, without `@` or function-call parentheses.

### 11. Can I use both show and hide in one forward?

Use one visibility strategy for each `@forward` rule: `show` for an allow-list or `hide` for exclusions. Split forwarding across rules or modules when the API needs a different selection.

### 12. Do show and hide change the original module?

No. They change which members are exposed through that forwarding rule. The source module's members and names are unchanged.

### 13. Can @forward expose a private member?

No. Private Sass members are not public members. Keep implementation details private or do not forward their module through a public entrypoint.

### 14. Does the underscore in a partial filename make all its members private?

No. An underscore in a filename marks a partial. Member privacy is determined by the member name starting with a hyphen or underscore.

## Prefixes and name conflicts

### 15. How does a prefix work?

Write `as <prefix>-*`:

~~~scss
@forward "buttons" as button-*;
~~~

If the source module has a public mixin named `reset`, downstream users refer to it as `button-reset`.

### 16. Does a forwarding prefix rename the source member?

No. It changes the name exposed through that forward, not the member's name in the defining module.

### 17. When are prefixes useful?

Use them when combining modules with overlapping member names or when the entrypoint should make a member's role clear.

### 18. What if two forwarded modules have the same public name?

The combined API may have conflicting members. Filter members with `show` or `hide`, or add distinct prefixes to the forwarded modules.

### 19. Can consumers choose the namespace and the library choose a forwarded prefix?

Yes. The library controls the member names exposed by `@forward`; the consumer chooses the namespace for the whole entrypoint with its own `@use ... as ...`.

## Using and configuring forwarded modules

### 20. What is the difference between @use and @forward?

`@use` makes members available in the current stylesheet through a namespace. `@forward` makes public members available to downstream users of the current stylesheet.

### 21. What order should I use when both @forward and @use load the same module?

Write `@forward` first. That lets downstream configuration be applied before the local `@use` loads the module without configuration.

### 22. Can @forward configure the module it loads?

Yes. A `@forward ... with (...)` clause can set defaults for the upstream module's configurable variables.

### 23. Why put !default in an @forward configuration?

It allows a forwarding module to set an opinionated default while still allowing downstream users to override that value when they configure the forwarding module.

~~~scss
// _library.scss
$brand-color: #1769aa !default;

// _design-system.scss
@forward "library" with (
  $brand-color: #4b2e83 !default
);

// app.scss
@use "design-system" with (
  $brand-color: #8b3a3a
);
~~~

The final consumer value is used because the forwarding default remains configurable.

### 24. Do consumers configure the internal module URL or the public entrypoint?

They should configure the public entrypoint they load. The entrypoint forwards the configuration to its internal module.

### 25. Does @forward include CSS from the forwarded module?

Yes. Top-level CSS from the forwarded module is included in the output when the forwarding module is loaded.

### 26. Is forwarded CSS emitted every time a module is encountered?

No. Sass loads a module once per compilation. Its CSS is included once even if it appears through multiple dependencies in the module graph.

## API design and troubleshooting

### 27. Should a library forward every internal file?

Usually not. Forward the supported API and keep implementation modules internal. This lets the library reorganize its files without forcing consumers to change imports.

### 28. Is renaming a forwarded member a breaking change?

It can be. Consumers may depend on the public name. Treat public names and supported entrypoints as part of the library's compatibility contract.

### 29. Why can't app.scss access a forwarded mixin?

Check that it loads the correct entrypoint with `@use`, that the member is public, and that it is not excluded or renamed by a `show`, `hide`, or prefix clause.

### 30. Why can the forwarding file not use a member it forwards?

Add an `@use` rule in that file. Forwarding exposes members downstream; it does not create a local namespace.

### 31. Why did a top-level reset appear after adding an entrypoint?

A forwarded module can emit CSS. Inspect the forwarded files and remove or intentionally manage top-level rules if loading the entrypoint should not add them.

### 32. Why did a downstream configuration fail after I added @use?

The local `@use` may have loaded the module before the `@forward` could pass along downstream configuration. Put `@forward` first.

### 33. Should I use @forward for every Sass dependency?

No. Use it to define a module or package's public API. A stylesheet that simply needs another module's members should use `@use`.

## Quick reference

| Goal | Syntax |
| --- | --- |
| Forward a module's public members | `@forward "tokens";` |
| Forward only selected members | `@forward "buttons" show reset, accessible-focus;` |
| Exclude selected members | `@forward "tokens" hide $internal-scale;` |
| Prefix forwarded names | `@forward "buttons" as button-*;` |
| Set a consumer-overridable forward default | `@forward "theme" with ($brand: purple !default);` |
| Use a forwarded member locally too | `@forward "buttons"; @use "buttons";` |

## Further reading

- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
