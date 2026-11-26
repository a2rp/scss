# 029 — Module Namespaces and Private Members: Q&A

## Namespaces and scope

### 1. What is a Sass module namespace?

It is the name used to access a module's public variables, functions, and mixins after loading it with `@use`.

### 2. How is the default namespace chosen?

Sass uses the last part of the module URL without its extension. For example, `@use "design-system/colors"` gives the default namespace `colors`.

### 3. How do I choose a different namespace?

Use an `as` clause:

~~~scss
@use "design-system/colors" as palette;

.card {
  color: palette.$text;
}
~~~

### 4. Is a namespace global across the project?

No. An alias belongs to the stylesheet that declares the `@use`. Another stylesheet can load the same module with a different alias.

### 5. Why would two modules need custom namespaces?

They may have the same default namespace, or the default name may be too vague or long. Explicit aliases keep the origin of each member clear.

### 6. What does @use "module" as * do?

It makes the module's members available without a namespace in that stylesheet. This can make references shorter, but can also cause name collisions and hide where a member came from.

### 7. When is as * reasonable?

It can be reasonable for a small module you own when flattening its names is intentional and conflicts are controlled. It is risky for third-party modules or unstable APIs.

### 8. Does @use in an entrypoint make its namespace available to every partial?

No. Each stylesheet that directly uses a module member needs its own `@use`.

### 9. Do namespaces appear in compiled CSS?

No. Namespaces organize Sass members in source code. Only the resulting CSS declarations and rules appear in the CSS output.

## Public and private members

### 10. Are Sass members public by default?

Yes. A variable, function, or mixin that does not start with `-` or `_` is public to stylesheets that load the module.

### 11. How do I make a variable private?

Start its name with a hyphen or underscore:

~~~scss
$-internal-gap: 0.75rem;
$_fallback-color: #222;
~~~

These members can be used inside the defining module but are not part of its public API.

### 12. Can mixins and functions be private too?

Yes. Sass applies the same leading hyphen or underscore convention to variables, mixins, and functions.

### 13. Can another stylesheet access a private member through a namespace?

No. A consumer cannot access the defining module's private member. It should call a public function or mixin that implements the supported behavior.

### 14. Does a partial filename make its members private?

No. The leading underscore in a filename such as `_tokens.scss` is a file-loading convention. Member privacy is controlled by the member name itself.

### 15. Does a private member prevent the module's CSS from being emitted?

No. Private status controls access to Sass members. It does not automatically suppress CSS defined at the top level of a module.

### 16. Should every helper be private?

Not necessarily. Make a member private when consumers should not depend on it. Public mixins and functions can call private helpers internally.

## @forward and library APIs

### 17. What does @forward do?

It loads another module and makes its public members available to users of the forwarding module. It is commonly used to build a package entrypoint that forwards members from internal files.

### 18. Does @forward make forwarded members available inside the forwarding stylesheet?

No. If that stylesheet also needs to use those members itself, it needs an `@use` rule too.

### 19. Should @forward or @use come first when both load the same module?

Write `@forward` first. If a downstream user configures the forwarded module, forwarding it first lets that configuration be applied before the local `@use` loads it without configuration.

### 20. What does @forward "tokens" hide ... do?

It excludes the listed members from the forwarding module's public API. Other public members remain forwarded.

### 21. What does @forward "tokens" show ... do?

It forwards only the listed members. This is useful when the entrypoint should expose a small, explicit API.

### 22. How do I list a variable in show or hide?

Include the dollar sign, such as `$brand-color`. Mixins and functions are listed by their member names without `@` or parentheses.

### 23. Does @forward automatically expose private members?

No. Members with a leading hyphen or underscore are private and are not exposed as public members through forwarding.

### 24. How can @forward add a prefix?

Use the `as <prefix>-*` syntax:

~~~scss
@forward "buttons" as button-*;
~~~

A public mixin named `reset` is then available downstream as `button-reset`.

### 25. Does an @forward prefix rename the original member inside its module?

No. It changes the names exposed through that forwarding rule. The defining module keeps its original member names.

### 26. Why should a library use a single entrypoint?

A stable entrypoint gives consumers a documented place to load public members. It lets the library change internal file organization without requiring consumers to load each implementation partial directly.

### 27. Does @forward include CSS from the forwarded module?

Yes. CSS from the forwarded module is included in the compiled CSS when the forwarding module is loaded.

### 28. Does forwarding every module automatically make a good API?

No. It may expose implementation details that consumers should not rely on. Use `show`, `hide`, and prefixes to keep the public API intentional.

## Common mistakes and troubleshooting

### 29. Sass says a namespace is undefined. What should I check?

Check that the current stylesheet has the correct `@use`, that the module URL resolves, and that you are using the namespace from that exact `@use` rule.

### 30. Sass says a member is undefined even though the module loaded. Why?

Check the member spelling and visibility. It may be private, misspelled, or omitted by an `@forward show` rule.

### 31. Why can't a partial use the alias declared in app.scss?

Aliases are file-local. Add an `@use` to the partial and use the namespace chosen in that partial.

### 32. Why does Sass report a conflict after using as *?

Two modules may expose the same unqualified member name. Restore namespaces or choose explicit aliases to distinguish them.

### 33. Why does a downstream user not see a member from an internal module?

The entrypoint may not forward that module, or a `show`/ `hide` rule may exclude the member. Check the full forwarding chain and public member name.

### 34. Why can a forwarding stylesheet not use the member it forwards?

Forwarding publishes the member downstream; it does not create a local namespace in the current stylesheet. Add `@use` if the current file also needs the member.

### 35. Should consumers load underscore-prefixed files directly?

Usually a library should document its public entrypoint. Partials are implementation files, and exposing them as consumer dependencies can make future refactors breaking changes.

### 36. What should I do if a helper must stay internal but consumers need one operation?

Keep the helper private and provide a public function or mixin that performs the supported operation. This limits what consumers can accidentally depend on.

## Quick reference

| Goal | Syntax |
| --- | --- |
| Use default namespace | `@use "colors";` then `colors.$brand` |
| Choose an alias | `@use "colors" as palette;` |
| Flatten names in one file | `@use "tokens" as *;` |
| Make a member private | `$-internal-value`, `@mixin -helper`, or `@function -helper()` |
| Forward all public members | `@forward "buttons";` |
| Exclude members | `@forward "tokens" hide $internal-value;` |
| Forward selected members | `@forward "buttons" show reset, accessible-focus;` |
| Prefix forwarded members | `@forward "buttons" as button-*;` |

## Further reading

- [Sass: @use](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: @forward](https://sass-lang.com/documentation/at-rules/forward/)
