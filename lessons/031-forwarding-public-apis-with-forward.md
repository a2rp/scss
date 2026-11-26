# 031 — Forwarding Public APIs with @forward

## Learning goals

After this lesson, you should be able to:

- Explain how `@forward` exposes members from one module through another.
- Build a public entrypoint that hides internal file organization.
- Control forwarded members with `show` and `hide`.
- Add prefixes to members forwarded from different modules.
- Use `@use` as well when the forwarding stylesheet needs a member locally.
- Forward configurable defaults while still allowing downstream configuration.
- Keep a library's public Sass API stable and intentional.

## 1. What @forward is for

A library may organize its Sass source across many files:

~~~text
sass/
  _buttons.scss
  _forms.scss
  _tokens.scss
  _index.scss
~~~

Consumers should not have to know every internal path. The `@forward` rule lets an entrypoint expose selected members from those files:

~~~scss
// sass/_index.scss
@forward "buttons";
@forward "forms";
@forward "tokens";
~~~

A consumer can load the entrypoint:

~~~scss
@use "sass" as ui;

.button {
  @include ui.button-reset;
  color: ui.$brand-color;
}
~~~

The consumer uses the entrypoint's namespace. The internal modules remain free to move as long as the public entrypoint preserves its contract.

## 2. @forward exposes members to downstream users

`@forward` loads a module and makes its public variables, functions, and mixins available to users of the forwarding module.

~~~scss
// sass/_buttons.scss
@mixin reset {
  border: 0;
  font: inherit;
}

// sass/_index.scss
@forward "buttons";

// app.scss
@use "sass";

.button {
  @include sass.reset;
}
~~~

The `reset` mixin is available to `app.scss` through the `sass` module.

Forwarded members are not automatically available inside `_index.scss` itself. If the entrypoint needs a member locally, add an `@use` rule too.

## 3. Build a single public entrypoint

A central `_index.scss` can forward the modules consumers are meant to use:

~~~scss
// sass/_index.scss
@forward "tokens";
@forward "buttons";
@forward "forms";
~~~

~~~scss
// app.scss
@use "sass" as ui;

.form-field {
  color: ui.$text-color;
  @include ui.input-reset;
}
~~~

The entrypoint acts as a boundary between implementation and public API. Consumers can use `sass` instead of importing internal files like `sass/forms` or `sass/tokens`.

Sass can resolve a directory URL through its `_index.scss` file. For example, `@use "sass"` can load `sass/_index.scss`.

## 4. Forward only selected members with show

Use `show` when the entrypoint should expose a small, explicit list:

~~~scss
// sass/_buttons.scss
@mixin reset {
  border: 0;
  font: inherit;
}

@mixin button-reset {
  @include reset;
  cursor: pointer;
}

@mixin accessible-focus {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}

// sass/_index.scss
@forward "buttons" show button-reset, accessible-focus;
~~~

Downstream users can access `button-reset` and `accessible-focus`, but not the unlisted `reset` mixin.

Member names are listed without `@` or parentheses. Include the `$` when listing a variable.

## 5. Hide selected members with hide

Use `hide` when most members should be forwarded except a few:

~~~scss
// sass/_tokens.scss
$brand-color: #1769aa;
$text-color: #263238;
$legacy-brand: #0b4f7a;

// sass/_index.scss
@forward "tokens" hide $legacy-brand;
~~~

Consumers can use `$brand-color` and `$text-color` through the entrypoint, but `$legacy-brand` is not part of that API.

Private members, which start with `-` or `_`, are not public members to forward. Use `show` or `hide` to control which public members cross the entrypoint.

## 6. Add prefixes when forwarding modules

Two modules may define members with the same short name. A prefix can make their public names distinct:

~~~scss
// sass/_buttons.scss
@mixin reset {
  border: 0;
  font: inherit;
}

// sass/_forms.scss
@mixin reset {
  padding: 0;
  border: 1px solid #8a94a6;
}

// sass/_index.scss
@forward "buttons" as button-*;
@forward "forms" as form-*;
~~~

~~~scss
// app.scss
@use "sass" as ui;

.button {
  @include ui.button-reset;
}

input {
  @include ui.form-reset;
}
~~~

The prefixes apply to names exposed through the forwarding module. The original mixin names inside their defining files do not change.

## 7. Use @forward and @use together

Sometimes an entrypoint needs to expose a module to consumers and use that module in its own styles.

~~~scss
// sass/_buttons.scss
@mixin accessible-focus {
  outline: 3px solid #135cb5;
  outline-offset: 2px;
}

// sass/_index.scss
@forward "buttons";
@use "buttons";

.button:focus-visible {
  @include buttons.accessible-focus;
}
~~~

Write `@forward` before `@use` when both rules load the same module. This ensures downstream configuration can be applied before the local `@use` loads the module without configuration.

The `@forward` makes the mixin available to users of `sass`; the `@use` provides the `buttons` namespace inside `_index.scss`.

## 8. Forward a default configuration to consumers

A forwarding module can set a default configuration for a module while allowing downstream users to override it.

~~~scss
// sass/_theme.scss
$brand-color: #1769aa !default;
$surface-color: #ffffff !default;

// sass/_index.scss
@forward "theme" with (
  $brand-color: #4b2e83 !default
);

// app.scss
@use "sass" with (
  $brand-color: #8b3a3a
);
~~~

The forwarding module's brand default is purple, but the consumer's configuration can choose another brand. The forwarded module's `!default` in the forward configuration allows the downstream value to take precedence.

Use this when a library wants an opinionated default while preserving consumer control. Configuration still applies when the module is first loaded in a compilation.

## 9. Forwarded CSS is still included

`@forward` loads a module like `@use`, so top-level CSS from a forwarded module is included in the compiled output.

~~~scss
// sass/_reset.scss
* {
  box-sizing: border-box;
}

// sass/_index.scss
@forward "reset";

// app.scss
@use "sass";
~~~

The compiled CSS contains:

~~~css
* {
  box-sizing: border-box;
}
~~~

A module's CSS is included once per compilation even if it is also loaded through another path in the module graph. Think about this side effect when forwarding modules that contain top-level rules.

## 10. Keep a public API stable

A public mixin, function, variable, and its forwarded name become part of the library's API. Consumers may depend on them.

Good API practices:

- Forward only members that consumers are meant to use.
- Keep internal helpers private or omit their modules from public entrypoints.
- Use prefixes when combining modules with overlapping names.
- Document the entrypoint and its supported members.
- Keep internal filenames out of consumer examples unless direct access is intentional.
- Treat removing or renaming a forwarded member as a breaking change.

## 11. Common mistakes

### Expecting forwarded members to be local

`@forward` exposes members downstream. Add `@use` when the forwarding stylesheet itself needs to reference them.

### Writing @use before @forward for the same module

Put `@forward` first so downstream configuration is applied before the local `@use`.

### Forwarding too much

A plain `@forward` exposes all public members in that module. Use `show` or `hide` when only part of the module belongs in the public API.

### Forgetting the dollar sign on a variable

Variables in a `show` or `hide` list include `$`, such as `$brand-color`. Mixin and function names do not.

### Assuming a prefix changes the source module

A prefix changes the names exposed through the forward. The source file and its internal member names remain the same.

### Forgetting that forwarded modules can emit CSS

A forwarded module may contain top-level CSS. Loading the public entrypoint can therefore add CSS to the output.

## 12. Summary

- `@forward` exposes another module's public members through a library entrypoint.
- Forwarded members are not automatically available inside the forwarding stylesheet; use `@use` for local access.
- `show` and `hide` control which public members are exposed.
- `as <prefix>-*` changes the names downstream users see.
- Forward before `@use` when both load the same module.
- Forwarded modules may emit CSS.
- A considered public API lets libraries reorganize internal files without breaking consumers.

## Further reading

- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
