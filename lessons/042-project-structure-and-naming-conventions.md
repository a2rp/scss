# Project structure and naming conventions

A Sass project structure should make three things easy to answer: which file is compiled, where a rule belongs, and which modules it depends on. Naming conventions make those answers visible in paths and selectors.

There is no universally correct directory tree. Choose the smallest structure that supports the project’s entry points and team workflow, then expand it when a real boundary appears.

## Learning goals

After this lesson, you should be able to:

- Choose a Sass structure for a small app, a larger app, or a reusable package.
- Name partials, modules, components, tokens, and entry points consistently.
- Keep dependency direction and CSS output intentional.
- Use module index files and <code>@forward</code> only when they provide a useful API.
- Avoid duplicate entry points, hidden globals, vague filenames, and over-fragmentation.
- Review module paths in a way that works across operating systems.

## 1. Start from the build entry points

First identify the stylesheets your application or build system compiles. Each compiled entry point creates an output stylesheet or bundle. A project might have one global entry point, or separate entry points for an admin app, a public site, and a widget.

~~~text
src/
  styles/
    app.scss
    components/
      _button.scss
      _card.scss
~~~

Here, <code>app.scss</code> is compiled and the partials are loaded by that entry point. Make the build boundary explicit in the project documentation or build scripts.

Avoid compiling an entry point and all its partials through a broad file glob. That can produce extra CSS outputs that the application does not need.

## 2. Pick a structure that matches project size

### Small application

~~~text
styles/
  app.scss
  _tokens.scss
  _base.scss
  _buttons.scss
  _forms.scss
~~~

This is enough when the files remain easy to scan and have clear roles.

### Growing application

~~~text
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
~~~

### Feature-oriented application

~~~text
src/
  styles/
    app.scss
    settings/
    foundations/
    utilities/
  features/
    billing/
      _invoice-table.scss
      _payment-form.scss
    profile/
      _profile-card.scss
      _avatar.scss
~~~

Feature-oriented files can sit near feature code when the application already uses feature ownership. Do not split source between distant folders just to follow a fashionable tree.

## 3. Use partial filenames intentionally

A leading underscore is Sass’s convention for a file intended to be loaded as a module rather than compiled on its own:

~~~text
_button.scss
_tokens.scss
_index.scss
~~~

The underscore is omitted from the module URL:

~~~scss
@use "components/button";
@use "settings/tokens";
~~~

The extension is usually omitted too. Keep the on-disk filename and module URL aligned, and use the same casing in both places.

A non-partial file such as <code>app.scss</code> is commonly an entry point, although the build configuration—not the filename alone—determines what gets compiled.

## 4. Name files for their responsibility

Prefer descriptive, stable names:

~~~text
_button.scss
_form-field.scss
_breakpoints.scss
_visually-hidden.scss
~~~

Use kebab-case for multiword filenames and keep names consistent. Avoid names that reveal little:

~~~text
_misc.scss
_stuff.scss
_helpers2.scss
_new.scss
~~~

If <code>_helpers.scss</code> contains token accessors, breakpoints, resets, and component variants, split it by responsibility or rename it to describe its actual role.

## 5. Use one clear entry point per output

An entry point composes the modules needed by one output bundle:

~~~scss
// app.scss
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "components/card";
@use "utilities/visually-hidden";
~~~

A separate application can have its own entry point:

~~~scss
// admin.scss
@use "foundations/reset";
@use "components/button";
@use "admin/dashboard";
@use "admin/navigation";
~~~

Shared modules can be loaded by both entry points. Each entry point is compiled independently, so its output may contain its own copy of shared CSS.

Do not create multiple entry points unless the build or product needs separate CSS outputs.

## 6. Keep source files and generated output separate

A common boundary is:

~~~text
src/
  styles/
    app.scss
    components/
dist/
  app.css
  app.css.map
~~~

Treat the output directory as generated build output. Do not put hand-maintained Sass next to generated CSS unless that is an intentional project convention. Configure the build system to compile the entry point into the chosen output location.

Whether generated files are committed depends on the deployment workflow. Make that decision once and document it.

## 7. Follow one dependency direction

A simple dependency graph moves from shared values toward application composition:

~~~text
tokens and settings
        ↓
foundations and helpers
        ↓
components and utilities
        ↓
application entry point
~~~

A button can use color tokens and focus helpers. The token module should not need to load the button module. Avoid cycles where modules require each other’s initialization or CSS.

When a shared value seems to require importing a component, reconsider which layer owns that value.

## 8. Make dependencies explicit

Every file should load the members it uses with <code>@use</code>:

~~~scss
// components/_button.scss
@use "../settings/tokens" as tokens;

.button {
  color: tokens.$color-text;
  background: tokens.$color-brand;
}
~~~

Do not rely on a different file having loaded variables into a global scope. Explicit dependencies make a file easier to move, review, and reuse.

Sass module URLs use forward slashes, even on Windows. Match path casing exactly so the same source also works on case-sensitive filesystems.

## 9. Use index files for a purposeful public interface

An <code>_index.scss</code> file is automatically used when the module URL names its containing folder. It can forward a small public API:

~~~scss
// settings/_index.scss
@forward "tokens" show $color-brand, $color-text;
@forward "breakpoints" show $breakpoint-table;
~~~

A consumer can load the folder:

~~~scss
@use "settings" as settings;

.title {
  color: settings.$color-text;
}
~~~

An index file is useful when it gives consumers a stable path or hides internal organization. If it merely forwards everything without improving the interface, direct module paths may be clearer.

Members forwarded from a module are exposed to consumers of the forwarder, but the forwarding file does not automatically make them available to its own code. Add <code>@use</code> too if the forwarder itself needs those members.

## 10. Separate public API from private implementation

Sass variables, functions, and mixins whose names begin with <code>-</code> or <code>_</code> are private to their defining module. A package can also keep a module internal by not forwarding it from its public entry point.

~~~scss
// settings/_spacing.scss
$_unit: 0.25rem;

@function _from-step($step) {
  @return $_unit * $step;
}

$space-small: _from-step(2);
$space-large: _from-step(6);
~~~

Consumers can use the public variables but cannot access the private helper through <code>@use</code>. Keep APIs small so later refactors do not break unrelated consumers.

## 11. Make each file’s CSS behavior clear

A module can define values only, emit CSS, or do both. Make the intended behavior apparent:

- <code>_tokens.scss</code>: values and accessors, normally no selectors.
- <code>_button.scss</code>: button selectors and declarations.
- <code>_focus-ring.scss</code>: a reusable mixin, normally no CSS until included.
- <code>app.scss</code>: selects and orders modules for the application output.

This is a convention, not a Sass requirement. The key is to know whether loading a module will add CSS.

## 12. Name tokens by role

A token can describe a raw palette entry or a semantic role:

~~~scss
$blue-700: #2747b8;
$color-action-primary: $blue-700;
$color-text-primary: #182033;
$space-control-inline: 1rem;
~~~

Choose names that explain the token’s meaning. Avoid encoding a temporary design decision or one-off page name into a global token unless that name is truly part of the design system.

Use consistent prefixes and separators. Do not mix <code>$spaceSmall</code>, <code>$spacing-small</code>, and <code>$gap_sm</code> for the same category.

## 13. Match component names to selector contracts

Choose a class convention and use it consistently:

~~~scss
.card { }
.card__title { }
.card--featured { }
~~~

The Sass filename can be <code>_card.scss</code> because the component’s root selector is <code>.card</code>. Keep file and selector names related enough that developers can find a component, without requiring them to be identical.

A component module should scope descendant rules to a recognizable root when appropriate. Sass nesting is a source convenience; it does not encapsulate selectors in the browser.

## 14. Keep nesting readable

Use nesting for clear relationships, such as a component modifier or pseudo-class:

~~~scss
.button {
  &:focus-visible {
    outline: 3px solid currentColor;
  }

  &--primary {
    font-weight: 700;
  }
}
~~~

Avoid mirroring a long HTML hierarchy in Sass. The compiled selector is what participates in the cascade, so review the selector Sass will generate.

## 15. Choose files and mixins based on use

A component partial can emit its normal component selectors when loaded. A mixin is useful when callers need configurable output or must choose where the CSS is emitted.

Do not create a mixin for every component just to avoid writing a normal selector. Likewise, do not put reusable value calculations in a CSS-emitting component file if a function or settings module is clearer.

## 16. Avoid over-fragmentation

A file per selector can make navigation harder. Group rules that change together, and split a file when it has multiple independent responsibilities or is difficult to review.

Ask: “Would a developer looking for this behavior know which file to open?” If not, improve names or boundaries before adding more folders.

## 17. Use namespaces instead of global imports

With <code>@use</code>, members are visible only in the stylesheet that loads the module and are namespaced by default:

~~~scss
@use "settings/tokens" as tokens;

.card {
  color: tokens.$color-text;
}
~~~

This makes short member names safe and their source easy to locate. Avoid <code>as *</code> for shared or third-party modules unless removing the namespace has a clear benefit and cannot create conflicts.

## 18. Distinguish third-party code from project code

Keep vendor styles in an identifiable dependency location or build step. Do not edit generated vendor CSS directly if a package manages its source. Add a narrow override in your own styles when needed, and document why it exists.

The exact directory depends on the package manager and build system; the goal is to distinguish maintained project code from external or generated files.

## 19. Migrate an existing project incrementally

For a project based on global <code>@import</code>:

1. List the actual compiled entry points.
2. Identify files that define shared variables and mixins.
3. Convert those files into Sass modules.
4. Add explicit <code>@use</code> statements in each consumer.
5. Add namespaces and resolve references.
6. Add <code>@forward</code> only for intentional public APIs.
7. Compare the built CSS and remove obsolete imports.

A mechanical replacement from <code>@import</code> to <code>@use</code> may fail because the rules have different visibility and loading behavior.

## 20. Project structure review checklist

Before treating a Sass structure as settled, check that:

- Each CSS output has a named, documented entry point.
- Build tooling compiles entry points rather than every partial.
- Partial names and module URLs match and use consistent casing.
- Dependencies are explicit and point in a clear direction.
- Index files expose a useful, limited API.
- Token names describe stable roles.
- CSS-emitting modules are easy to identify.
- Components have recognizable selector roots.
- Generated CSS has an intentional order.
- The structure is not more complex than the project needs.

## Common mistakes

### Using vague filenames

A file named <code>_misc.scss</code> forces readers to inspect it to understand its role. Name a module after its responsibility.

### Treating an index file as mandatory

A folder does not need an index file unless a stable folder-level API improves its use.

### Compiling every Sass source file

Compiling partials and entry points together can emit unwanted standalone CSS. Configure the build around the intended entry files.

### Using Windows backslashes in Sass URLs

Write <code>@use "settings/tokens";</code> with forward slashes. Sass resolves module URLs, not Windows-style file paths.

### Depending on import order for variables

A module should load its own dependencies. Do not rely on a variable being globally available because a different stylesheet happened to load first.

### Making every name public

Forwarding all internals makes them hard to change. Expose only the names consumers should use.

### Building a structure before the project needs it

Avoid empty folders and wrapper files that do not clarify a current responsibility or entry point.

## Quick reference

| Concern | Convention |
| --- | --- |
| Files loaded by other modules | Prefix Sass partial filenames with <code>_</code> |
| Module URLs | Omit <code>_</code> and <code>.scss</code>; use forward slashes |
| Entry point | Use one explicit entry point per CSS output |
| Shared dependencies | Load with namespaced <code>@use</code> in each consumer |
| Folder-level public API | Add <code>_index.scss</code> and <code>@forward</code> when useful |
| Internal Sass members | Prefix private names with <code>_</code> or <code>-</code> |
| Component Sass file | Use a descriptive name such as <code>_button.scss</code> |
| Component selectors | Use a consistent, scoped naming convention |
| Generated CSS | Keep it in a documented output location |

## Further reading

- [Sass modules and @use](https://sass-lang.com/documentation/at-rules/use/)
- [The @forward rule](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass stylesheet structure](https://sass-lang.com/documentation/syntax/structure/)
- [Sass module system](https://sass-lang.com/blog/the-module-system-is-launched/)
- [Sass migration guide for @import](https://sass-lang.com/documentation/breaking-changes/import/)

## Next lesson

Continue with [npm scripts and build integration](043-npm-scripts-and-build-integration.md).
