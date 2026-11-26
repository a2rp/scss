# Project structure and naming conventions — Q&A

This Q&A covers Sass file layout, module naming, entry points, module APIs, and cross-platform paths. Architectural choices are conventions; the Sass module-loading behavior described here follows the official Sass documentation linked at the end.

## 1. What does Sass project structure mean?

It describes where Sass source files live, how they depend on each other, which files are compiled into CSS, and how their output is composed. A useful structure helps developers find a rule and understand where its values come from.

Sass does not require a particular directory tree.

## 2. What is a good structure for a small project?

Start with a single entry point and a few files with clear responsibilities:

~~~text
styles/
  app.scss
  _tokens.scss
  _base.scss
  _components.scss
~~~

Split files or introduce folders when they become difficult to navigate or have independent roles.

## 3. When should I add folders?

Add folders when grouping by responsibility or ownership makes files easier to find. Common groups include settings, foundations, components, utilities, and features.

Do not add layers of directories just because a large framework uses them. Use the smallest structure that fits the current project.

## 4. What is a Sass partial?

A partial is a Sass file conventionally prefixed with an underscore, such as <code>_button.scss</code>, that is intended to be loaded by another stylesheet. The underscore is omitted from its module URL, so <code>@use "components/button";</code> can load it.

Sass tooling treats partials as files that should not be compiled on their own by default.

## 5. Is every Sass file starting with an underscore a module?

Any Sass stylesheet can be loaded as a module with <code>@use</code>. The underscore convention indicates the file is a partial intended to be loaded by another stylesheet, rather than an entry point compiled on its own.

The module system is the behavior of <code>@use</code> and <code>@forward</code>, not the underscore itself.

## 6. Which files should I compile?

Compile the application entry points, such as <code>app.scss</code> or <code>admin.scss</code>. Those files compose the partials and modules needed for each CSS output.

Do not compile every partial along with the entry points unless that is an intentional build requirement.

## 7. Can a project have multiple entry points?

Yes. Separate apps or bundles may need different CSS outputs. Each entry point should list the modules it needs and be documented in the build configuration.

Each entry point is its own compilation, so shared module CSS may appear in each separate output.

## 8. Will Sass emit a module’s CSS when I use it?

If a loaded module contains CSS rules, its CSS is included in the output. If it contains only variables, functions, or mixins that are not included, it may emit no CSS.

The module system loads a module only once per compilation, even if multiple files use it.

## 9. What should go in an entry point?

The entry point should compose modules for one output bundle in a readable order. It should not become a place for all component implementation details.

~~~scss
@use "foundations/reset";
@use "foundations/typography";
@use "components/button";
@use "utilities/visually-hidden";
~~~

## 10. How should I name Sass files?

Use descriptive, consistent names such as <code>_form-field.scss</code>, <code>_breakpoints.scss</code>, and <code>_visually-hidden.scss</code>. Kebab-case is a common convention for multiword filenames.

Sass does not enforce a naming convention, so consistency within the project matters more than the specific style chosen.

## 11. Does the Sass filename have to match its CSS selector?

No. A component stylesheet named <code>_card.scss</code> may define selectors such as <code>.card</code>, <code>.card__title</code>, and <code>.card--featured</code>. A meaningful relationship helps developers find the source, but Sass does not require identical names.

## 12. What should I name tokens?

Choose stable names that communicate purpose. Primitive names may describe palette values, while semantic names describe how components use them, such as <code>$color-text-primary</code> or <code>$color-action-primary</code>.

Use consistent prefixes and separators. Avoid maintaining several naming styles for the same token category.

## 13. Should files be organized by type or by feature?

Either can work. Type-based organization keeps all components together; feature-based organization puts related styles near the owning feature. Use the approach that matches how the project is maintained.

A hybrid is common: shared tokens and foundations in one location, feature-specific styles beside feature code.

## 14. What is an index Sass file?

A folder can contain <code>_index.scss</code>. Sass loads that index when a module URL names the folder, so consumers can use a folder-level path such as <code>@use "settings";</code>.

An index can forward a public API, but it is optional. Use one only when the shorter or more stable path helps real consumers.

## 15. What is the difference between @use and @forward?

<code>@use</code> loads members into the current stylesheet under a namespace and loads the module’s CSS. <code>@forward</code> exposes members from another module to consumers of the forwarding module.

A file that needs both to use and re-export a module’s members may need both rules. The <code>@forward</code> should come first when both reference the same module so downstream configuration reaches it before it is loaded by <code>@use</code>.

## 16. Can @forward control which names are public?

Yes. Use <code>show</code> to expose only selected members or <code>hide</code> to omit selected members. It can also add a prefix to forwarded names.

~~~scss
@forward "tokens" show $color-brand, $color-text;
@forward "spacing" as spacing-*;
~~~

Use these controls to keep a public API small and stable.

## 17. Does @forward make the members available inside the forwarding file?

No. It makes members available to users of the forwarding module. If the forwarding file itself needs those members, load the module with <code>@use</code> as well.

Sass loads the module once, so forwarding it and using it does not duplicate the module’s CSS within that compilation.

## 18. How do I keep implementation details private?

Prefix a Sass variable, function, or mixin with an underscore or hyphen to make it private to its module. A package can also keep an entire module private by not forwarding it from its public entry point.

Keep only intentional public members available to consumers.

## 19. What should a tokens folder contain?

It can contain design values such as colors, spacing, typography scales, radii, and breakpoints. Keep the folder focused on values and related accessors.

Avoid including unrelated component selectors in a module consumers load just to access a token.

## 20. Should a tokens module emit CSS?

Usually a token module can be value-only, but it may intentionally generate CSS custom properties. Make the behavior explicit. If it emits CSS, loading it through a component can contribute those declarations to the output.

Separate values and CSS generation when that makes their ownership or output easier to reason about.

## 21. How do I name the CSS output directory?

Choose a directory that matches the build, such as <code>dist/</code>, <code>public/css/</code>, or a framework-specific assets folder. Keep generated output separate from hand-maintained Sass when possible.

The specific name is a project convention. Document whether generated files are committed or recreated during build/deploy.

## 22. Why do Sass URLs use forward slashes on Windows?

Sass module URLs use forward slashes across platforms:

~~~scss
@use "settings/tokens";
~~~

Do not write a Windows path with backslashes inside a Sass URL. Relative module URLs are resolved by Sass from the current stylesheet first.

## 23. Does module URL casing matter?

Yes. Use the same casing in the URL and filename. A path that works on a case-insensitive filesystem may fail on a case-sensitive one, and inconsistent casing can make modules appear to be different paths.

Consistent casing makes projects more portable.

## 24. Can Sass find a partial when the underscore is omitted?

Yes. The partial’s leading underscore and file extension are normally omitted from its module URL. For example, <code>_tokens.scss</code> can be loaded with <code>@use "tokens";</code>.

Use the project’s established path convention consistently.

## 25. Should I compile partials directly?

Usually no. Compile the intended entry point, which loads its partials. Compiling both every partial and the entry point can create extra outputs or duplicate CSS across files.

Set the build tool’s Sass inputs to the actual entry points rather than relying on a broad source glob.

## 26. How do I avoid hidden dependencies?

Add a <code>@use</code> statement to every file that uses another module’s members, and refer to those members through its namespace. Do not depend on a different file having loaded global variables first.

Explicit dependencies reduce order-sensitive behavior and simplify refactoring.

## 27. How should module dependencies flow?

A practical direction is from settings and tokens to foundations and helpers, then to components, and finally to application entry points. Shared low-level modules should not depend on specific components.

This is a design guideline, not a compiler rule. Its goal is to keep module relationships easy to follow.

## 28. Can two modules use each other?

Avoid circular dependencies. A cycle makes it harder to understand initialization and ownership, and Sass can report an error when a module is loaded recursively.

Move shared values or behavior into a lower-level module that both consumers can use.

## 29. Should every component have a separate file?

Not necessarily. A distinct file helps when a component has enough styles, logic, or independent ownership to benefit from it. Closely related small rules can share a module.

Avoid both a huge catch-all file and a file for every tiny selector.

## 30. How do I keep component selectors scoped?

Use a recognizable component root and keep related selectors under it when appropriate:

~~~scss
.card {
  &__title { }
  &--featured { }
}
~~~

Sass nesting produces CSS selectors; it does not provide browser style encapsulation. Review the actual selector output and avoid global element rules in component modules unless that is intentional.

## 31. Is a folder called helpers a good idea?

It can be, if the folder contains a clear collection of reusable helpers. If it becomes a mix of tokens, resets, mixins, and component-specific logic, split it into modules named for their responsibilities.

Good names reduce the need to inspect unrelated code.

## 32. How do I migrate from legacy global @import?

Inventory entry points and shared members first. Convert shared values and helpers to modules, add explicit <code>@use</code> dependencies in consumers, introduce namespaces, and review the resulting CSS.

Do not replace every import mechanically. <code>@use</code> has different visibility, loading, and placement behavior. Sass provides a migrator for supported migration steps.

## 33. What is over-engineering in a Sass structure?

A structure is over-engineered when folders, wrappers, or APIs add navigation and indirection without clarifying a current responsibility. Examples include many empty category folders, one-line files that only forward private code, or multiple entry points that produce identical CSS.

Start simple and add boundaries when they make a real change easier to locate or control.

## 34. How can I review whether the structure works?

Ask someone to locate a token, a component rule, and the compiled entry point without first reading every file. Check whether the module dependencies are explicit, CSS output is intentional, and names describe actual responsibilities.

If those tasks are difficult, improve names and boundaries before adding more layers.

## 35. What are common project-structure mistakes?

- Compiling partials as standalone outputs.
- Depending on global variables and import order.
- Using vague names such as <code>_misc.scss</code>.
- Forwarding every internal Sass member.
- Mixing generated CSS with authored source without a clear rule.
- Using backslashes or inconsistent casing in module URLs.
- Creating too many tiny files or one unmaintainable catch-all.
- Building folder depth before there is a real need.

## Official Sass references

- [Sass modules and @use](https://sass-lang.com/documentation/at-rules/use/)
- [The @forward rule](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass stylesheet structure](https://sass-lang.com/documentation/syntax/structure/)
- [Sass module system](https://sass-lang.com/blog/the-module-system-is-launched/)
- [Sass migration guide for @import](https://sass-lang.com/documentation/breaking-changes/import/)
- [Sass migrator](https://sass-lang.com/documentation/cli/migrator/)
