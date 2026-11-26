# Component styles and architecture — Q&A

This Q&A covers Sass module boundaries, application entry points, component selectors, and style-reuse choices. Sass documentation is linked at the end for syntax and compiler behavior.

## 1. What does stylesheet architecture mean?

It is the way a project organizes Sass source, shares values and helpers, chooses which CSS gets emitted, and controls the cascade. A clear architecture makes dependencies visible and keeps changes local to the part of the interface they affect.

It is a project convention, not a Sass feature with one required folder layout.

## 2. What is a good starting folder structure?

Start with an application entry point and a few responsibility-based groups, for example:

~~~text
styles/
  app.scss
  settings/
  foundations/
  components/
  utilities/
~~~

Add more directories only when they make real responsibilities easier to find. A small project can use fewer files.

## 3. What is a Sass partial?

A partial is a Sass file commonly named with a leading underscore, such as §_button.scss§. It is normally loaded from another Sass file; its underscore is omitted from the module URL, so §@use "components/button"§ can load it.

The underscore is a naming convention for partial files. The Sass module system is provided by §@use§ and §@forward§.

## 4. Which Sass file should I compile?

Compile the designated application entry point, such as §app.scss§. That file loads the foundations, components, and utilities needed for that output bundle.

Compiling every partial independently can create multiple CSS files with repeated dependencies and makes the final composition less obvious.

## 5. What is the difference between @use and @import?

§@use§ loads a module with a namespace and makes its members available only to the stylesheet that loads it. It also includes that module’s CSS in the compilation once.

The legacy §@import§ model shares members globally and can execute the same stylesheet repeatedly. Sass’s module system is the preferred way to structure new code and migrate old code.

## 6. Does @use emit a module’s CSS?

If the loaded module contains CSS rules, those rules are included in the compiled output. If it only defines Sass variables, functions, and mixins, there may be no CSS to emit until those members are used.

A module is loaded once per compilation, even if multiple modules use it. Separate application entry points are separate compilations and can each include that CSS.

## 7. Why use namespaces?

Namespaces make a dependency explicit and reduce name collisions:

~~~scss
@use "../settings/tokens" as tokens;

.button {
  color: tokens.$color-text;
}
~~~

Reading §tokens.$color-text§ shows where the value came from. Default namespaces use the final module URL component; an §as§ clause can choose a different name.

## 8. Should I use @use as *?

§@use "module" as *§ removes the namespace for that module’s members. This can shorten references, but it makes their origin less visible and may introduce collisions. Prefer namespaced members, especially for dependencies written by other people or shared across a codebase.

## 9. Where must @use appear?

Place §@use§ at the top level before rules other than §@forward§. Sass permits variable declarations before §@use§ when those variables are used to configure the module. Do not put §@use§ inside a selector or conditional block.

## 10. What is @forward for?

§@forward§ re-exports public variables, functions, and mixins from another Sass module so consumers can load a single public entry point.

~~~scss
// settings/_index.scss
@forward "tokens";
@forward "breakpoints" show $breakpoint-table;
~~~

A forwarding file can use §show§ or §hide§ to control its public API. Forward only members consumers should depend on.

## 11. Is a barrel file always a good idea?

No. A barrel or index file is useful when it presents a small, stable public API. It becomes unhelpful when it forwards every implementation detail, hides dependencies, or forces consumers to load unrelated CSS.

Use a barrel when it improves the interface for real consumers; otherwise direct module paths may be clearer.

## 12. Should tokens and component CSS live in the same file?

They can, but separating value-only settings from CSS-emitting component modules often makes dependencies easier to understand. A token file can define values and accessors; a button module can use those values and emit button selectors.

The important thing is that a module’s role is clear. Loading a token just to get an unrelated global reset is an unnecessarily broad dependency.

## 13. Should every component be a Sass mixin?

No. If loading a component module should always include its CSS, define the component rule directly in that module. A mixin is useful when its output is configurable or should be emitted at a caller-selected location.

Wrapping every selector in a mixin adds an include API without necessarily improving reuse.

## 14. When should I use a mixin?

Use a mixin when a reusable style pattern accepts arguments, emits several related declarations, or needs to place a content block in a controlled context.

~~~scss
@mixin focus-ring($color) {
  &:focus-visible {
    outline: 3px solid $color;
    outline-offset: 2px;
  }
}
~~~

A mixin copies its generated styles into each include location. Use it where that behavior is part of the design.

## 15. When should I use a function?

Use a function when the result should be a Sass value, such as a calculated size or a validated token:

~~~scss
@function spacing($step) {
  @return $step * 0.25rem;
}
~~~

Use a mixin when the reusable operation should emit CSS rules or declarations.

## 16. Can mixins accept content blocks?

Yes. A mixin can contain §@content§, and the code in the §@include§ block is inserted at that point. This is useful for wrapping caller-provided styles in a media query or selector context.

Content blocks are lexically scoped: they see variables where the mixin is included, not local variables created inside the mixin.

## 17. When is @extend appropriate?

§@extend§ expresses a selector relationship: elements matching the extender should also match the extended selector. A placeholder selector such as §%message-base§ can hold shared styles without appearing in CSS until extended.

Use it when that relationship is intentional and the resulting selector list remains understandable.

## 18. Why can @extend be surprising?

§@extend§ updates rules that contain the extended selector, including rules beyond the line where the extension appears. It can produce larger or more complex selector lists than expected.

It also cannot extend a selector outside the current media context from inside a media query. For configurable or purely mechanical reuse, a mixin may make the output and intent easier to follow.

## 19. Is @extend always smaller than a mixin?

Not necessarily in a way that should drive architecture. A mixin can repeat declarations in generated CSS; an extend can combine selectors and affect selector relationships. Choose based on semantics, predictability, and maintainability rather than trying to minimize uncompressed output text.

## 20. Should I use BEM or another naming convention?

Choose and document a convention that suits the project. BEM-style block, element, and modifier names are one option:

~~~scss
.card { }
.card__title { }
.card--featured { }
~~~

The convention should give components clear roots and avoid accidental styling across unrelated parts of the page. Sass does not require a particular class naming system.

## 21. How much selector nesting is too much?

There is no Sass-enforced numeric limit. Nest only when it makes a component relationship clearer, and keep the final selector short enough to inspect. Deep nesting often ties styles to exact markup and makes overrides difficult.

Use §&§ for concise pseudo-classes or a deliberate component naming pattern, not to mirror every HTML ancestor.

## 22. Should components style raw HTML elements?

A component may need scoped element styles, but broad rules such as §h2 { ... }§ in a shared component stylesheet can affect unrelated content. Scope such rules under a component root when that is the intended contract:

~~~scss
.card {
  h2 {
    margin-block: 0 0.5rem;
  }
}
~~~

Prefer explicit component classes when the style should be reusable or applied selectively.

## 23. How do I keep component CSS from becoming global?

Give each component a recognizable root class, scope descendant rules to that root, and avoid generic global selectors in its module. Use a naming convention and review compiled selectors when a module uses lots of nesting.

Sass itself does not create browser-level style encapsulation; it compiles selectors into CSS.

## 24. How should I control CSS order?

Use one entry point that loads foundations, components, and utilities in an intentional order. Keep this order visible instead of scattering global imports across feature files.

If the application uses CSS cascade layers, declare their order centrally and assign styles consistently. Sass module boundaries organize source; the browser still resolves the final CSS cascade.

## 25. Can I load a component module from multiple other modules?

Yes. Within one compilation, §@use§ loads a given module once, so its CSS is not duplicated just because several modules depend on it. The dependency should still be explicit in each file that uses its Sass members.

If you build separate CSS entry points, each is compiled separately and may include the same module’s CSS in its own output.

## 26. How should modules depend on each other?

Prefer a simple dependency direction: settings and tokens at the base, then foundations and reusable helpers, then components, then the app entry point. Avoid cycles where two modules require each other to initialize or define their public API.

A one-directional graph is easier to load and reason about.

## 27. What is a private Sass member?

A variable, function, or mixin whose name begins with §-§ or §_§ is private to its defining module and is not available to modules that load it with §@use§.

Use private members for implementation details. Do not make a member public unless another stylesheet should intentionally depend on it.

## 28. How do configurable Sass modules work?

A module can define variables with §!default§, and a consumer can configure them the first time it loads that module:

~~~scss
@use "theme" with (
  $brand: #2747b8
);
~~~

This configuration is compile-time and module-level. It is not a way to switch themes after the browser loads the page.

## 29. How do I support runtime themes?

Emit CSS custom properties and override them under theme selectors or other runtime conditions:

~~~scss
:root {
  --surface: #fff;
  --text: #182033;
}

[data-theme="dark"] {
  --surface: #151a24;
  --text: #f3f5fa;
}
~~~

Sass maps can help generate each set of declarations. The browser applies the selected CSS custom properties at runtime.

## 30. How do I migrate a legacy Sass codebase?

First identify which files are compiled as entries and where global variables come from. Convert shared values and helpers into modules, add explicit §@use§ statements in each consumer, and give references namespaces. Then review the generated CSS and remove obsolete imports.

Do not replace every §@import§ mechanically: legacy code may depend on global scope, execution order, or nested imports.

## 31. Should each component get its own file?

Create a separate file when the component has enough styles, logic, or independent ownership to benefit from it. Small related components can share a module. Avoid both extremes: one enormous file and one tiny file for every selector.

The structure should make related changes easy to find.

## 32. How should utilities be organized?

Keep utilities small, intentionally reusable, and named for the behavior they provide. Load only the utilities the application needs. A utility module can emit a defined collection of classes, while a mixin can provide styles for callers that need a custom selector.

Do not make every component depend on a large utility bundle solely to access one helper.

## 33. How do I know whether a module should emit CSS?

Ask whether loading it should visibly add styles to the application. A token module usually should not emit unrelated selectors. A component module often should. A mixin library may emit no CSS until a mixin is included.

Make that behavior clear in the module’s name, documentation, and entry-point imports.

## 34. What are common architecture mistakes?

- Depending on global members that were loaded elsewhere.
- Compiling partials as well as the application entry point without intent.
- Forwarding every internal member through a public index.
- Hiding normal component rules inside unnecessary mixins.
- Using §@extend§ just to deduplicate declarations.
- Nesting selectors to mirror deep HTML structure.
- Mixing tokens and unrelated CSS output in one broad module.
- Relying on a long chain of later overrides to repair earlier styles.
- Building circular or unclear module dependencies.

## Official Sass references

- [Sass modules and §@use§](https://sass-lang.com/documentation/at-rules/use/)
- [The §@forward§ rule](https://sass-lang.com/documentation/at-rules/forward/)
- [Mixins and §@include§](https://sass-lang.com/documentation/at-rules/mixin/)
- [The §@extend§ rule](https://sass-lang.com/documentation/at-rules/extend/)
- [Placeholder selectors](https://sass-lang.com/documentation/style-rules/placeholder-selectors/)
- [Sass module system](https://sass-lang.com/blog/the-module-system-is-launched/)
