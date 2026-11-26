# Capstone: build a reusable SCSS design system — Q&A

These answers review the architecture and implementation choices in the capstone. They distinguish Sass's build-time module system from CSS custom properties and runtime browser behavior.

## 1. What is a design system?

A design system is a maintained set of shared design decisions, implementation patterns, components, and usage guidance. In this capstone, it includes tokens, Sass modules, CSS theme variables, accessible components, a compiled entry point, and documentation. It is more than a color palette or a folder of mixins.

## 2. How large should the first version be?

Small enough to be understood and adopted. Start with repeated tokens and components that a product actually needs. Add abstractions when they remove real duplication or make the API clearer; do not build a large hypothetical component catalog before consumers need it.

## 3. What belongs in a design-system README?

Explain how to include the CSS, how to load the public Sass API, which tokens and components are supported, how to select or override a theme, how to compile source, the intended support policy, and how contributors can propose changes. Document whether API changes are compatible or breaking.

## 4. What is the difference between a Sass variable and a CSS custom property?

A Sass variable is evaluated during compilation. It is useful for maps, functions, mixins, and build-time values such as breakpoints. A CSS custom property is resolved in the browser, participates in the cascade, and can inherit or change by theme or subtree.

## 5. Which values should be Sass variables?

Use Sass values for decisions made while generating CSS: spacing and radius maps, breakpoint thresholds, helper functions, mixins, and compile-time configuration. A Sass map can provide a checked API so consumers do not copy raw token values everywhere.

## 6. Which values should be CSS custom properties?

Use CSS custom properties for values that should change at runtime or inherit through the DOM, such as theme colors or component-specific overrides. They let the browser choose a value based on the active theme selector or a nearer scope without recompiling Sass.

## 7. Can CSS custom properties be used in media queries?

Ordinary <code>var()</code> substitution is for property values, not the condition in a media or container query. Keep query thresholds in Sass or another build-time configuration. CSS custom properties can still be used for the declarations inside a query.

## 8. Does a <code>var()</code> fallback support browsers without custom properties?

No. In <code>color: var(--text, black)</code>, the fallback helps when custom properties are understood but the named value is unavailable or invalid. A browser that does not understand the entire <code>var()</code> declaration ignores it. For older browsers, put a regular declaration first:

~~~scss
color: #172033;
color: var(--color-text, #172033);
~~~

## 9. What does <code>@use</code> do?

It loads another Sass module and provides its public variables, functions, and mixins under a namespace. Each Sass file must load the modules it directly uses. A module's CSS is included only once per compilation, which helps prevent duplicate output.

## 10. Why use namespaces?

Namespaces show where a function or mixin comes from and reduce name collisions. For example, <code>tokens.space(3)</code> is clearer than an unqualified <code>space(3)</code> when a project has multiple modules.

## 11. What does <code>@forward</code> do?

It loads a module and exposes selected public members to consumers of the forwarding module. This lets a package offer one stable entry point while keeping its internal file structure private.

## 12. Should the public Sass API forward every module?

Usually not. Forward only members that consumers are meant to rely on. A narrow API is easier to document and change. If consumers depend on every internal map, helper, and partial, refactoring becomes a breaking change.

## 13. Does <code>@forward</code> include a module's CSS too?

Yes. Forwarding a stylesheet also makes its CSS part of the generated output when the forwarder is loaded. Keep API-only token modules free of component CSS if consumers should be able to load functions and mixins without emitting unrelated styles.

## 14. Why use a separate CSS entry point and Sass API file?

They serve different needs. The CSS entry point loads the theme, base rules, and components required by the application. The Sass API exposes tools that a consumer may call from its own Sass. Keeping them separate avoids forcing every API consumer to include all component CSS.

## 15. Why are some Sass map names prefixed with a dash?

Sass names beginning with <code>-</code> or <code>_</code> are private to their module. The public functions can validate and retrieve values without exposing the map's structure. Consumers depend on <code>space()</code>, for example, rather than the internal token map.

## 16. Why should token lookup functions validate keys?

A typo should produce a clear compiler error, not an empty value that creates confusing CSS. A checked function documents the supported token names and catches mistakes early.

## 17. When should a function become a mixin?

A function should return a value that can be used inside an expression or declaration. A mixin emits a block of declarations or rules, often with nested content or media queries. Use the simplest abstraction that matches what the API needs to produce.

## 18. When should module configuration use <code>!default</code>?

Use <code>!default</code> when consumers genuinely need to configure a Sass variable at load time. Sass module configuration is fixed the first time the module is loaded in a compilation. If configuration becomes a complicated collection of values or needs runtime switching, consider a map or mixin API, or CSS custom properties for runtime values.

## 19. Why use role-based color tokens?

A component can ask for a role such as <code>--color-surface</code> or <code>--color-action</code> instead of knowing a specific theme palette. A theme can then change those role values while component structure remains the same. Role names are more useful to consumers than names like <code>--purple-600</code> when the value's purpose is what matters.

## 20. How does a subtree theme work?

Custom properties inherit. Applying <code>data-theme="dark"</code> to a container defines the dark role values for that subtree and its descendants. A nested theme can override the inherited values again. Test text, borders, controls, and focus indicators in every supported theme.

## 21. Should every component use a custom property for every style?

No. Use runtime custom properties where consumers or themes need a supported override. Keep internal spacing and structure simple when they are not part of the public customization contract. Too many public variables make the system harder to document and maintain.

## 22. How should a button component handle semantics?

Use a <code>&lt;button&gt;</code> for actions and an <code>&lt;a&gt;</code> for navigation. Preserve an accessible name, keyboard operation, visible focus, sufficient contrast, and understandable disabled or busy state. A class name should not be a substitute for choosing the correct HTML element.

## 23. How should design-system components handle focus?

Provide a visible keyboard focus indicator. If using a newer selector such as <code>:focus-visible</code>, keep a useful fallback for the project's support target and test it with keyboard navigation. Do not remove outlines globally without supplying an equivalent visible treatment.

## 24. Should every design token be public?

No. Public tokens are a compatibility contract. Expose values only when a consumer has a good reason to use or override them. Private implementation tokens can change without forcing every consuming project to update.

## 25. Is every repeated value automatically a token?

No. Promote a value when it represents a meaningful design decision that should stay consistent. If two unrelated details happen to use the same number today, they may not represent the same token. A token should communicate intent and have a defined use.

## 26. How can I avoid duplicate component CSS?

Compile one intended application entry point. Sass's module system loads a given module once in a compilation, but independently compiling many partials and including each output file can still duplicate or conflict with styles. Review the actual CSS files your application loads.

## 27. What does the Sass public API commit the project to?

Once documented and used by consumers, names and behavior become contracts. Removing or renaming a public function, mixin, CSS custom property, or component class can break adopters. Version significant changes, communicate migration steps, and avoid publishing internal implementation details as public API.

## 28. How should the system evolve?

Use consumer feedback and repeated product needs. Review whether tokens are used, whether components still match product patterns, and whether browser and accessibility expectations changed. Remove obsolete tokens deliberately and document breaking changes rather than silently changing behavior.

## References

- [Sass: <code>@use</code> modules](https://sass-lang.com/documentation/at-rules/use/)
- [Sass: <code>@forward</code> modules](https://sass-lang.com/documentation/at-rules/forward/)
- [Sass: built-in modules](https://sass-lang.com/documentation/modules/)
- [Sass: <code>@import</code> and module-system changes](https://sass-lang.com/documentation/breaking-changes/import/)
- [MDN: using CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascading_variables/Using_custom_properties)
- [MDN: <code>var()</code> function](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference/Values/var)
- [MDN: inheritance](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Cascade/Inheritance)
- [MDN: registering CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/Guides/Properties_and_values_API/Registering_properties)
- [Next: SCSS quick reference and glossary](../lessons/054-scss-quick-reference-and-glossary.md)
