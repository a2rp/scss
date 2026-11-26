# Capstone: build a reusable SCSS design system

A design system turns repeated design decisions into a small, dependable set of tokens, components, and usage rules. In this capstone, you will build a compact Sass system with a documented Sass API, runtime color themes, responsive tokens, accessible components, and a compiled CSS entry point.

The important architectural choice is to keep compile-time Sass tools distinct from runtime CSS values. Sass maps and functions help authors write consistent source; CSS custom properties let the browser resolve values that can change with a theme or component context.

## Learning goals

After this capstone, you should be able to:

- Organize a small design system into Sass modules with clear responsibilities.
- Separate public Sass API from private implementation details.
- Use <code>@use</code> and <code>@forward</code> without global namespace collisions.
- Decide which values belong in Sass maps and which belong in CSS custom properties.
- Build themeable tokens and reusable components with accessible states.
- Compile a single stylesheet entry point and inspect its generated CSS.
- Document adoption, customization, and maintenance rules for the system.

## 1. Define the system's scope

A design system does not need to include every component a product may ever use. Start with the repeated decisions that are already present or likely to be shared:

- Color roles such as page, surface, text, border, action, and focus.
- Spacing and corner-radius scales.
- A small number of layout breakpoints.
- Typography defaults and content widths.
- A few reusable components, such as buttons and cards.
- Interaction states for hover, keyboard focus, disabled controls, and reduced motion.
- A documented way to load and customize the system.

Before writing Sass, define a short set of principles. For example: content remains readable without decorative effects; colors are referenced by role; components do not own page layout; responsive rules follow available space; and unsupported enhancements keep a usable fallback.

Limit the first version to a coherent set of tokens and components. A small API that people can learn is easier to maintain than a large collection of values nobody uses.

## 2. Plan the module structure

Separate source by responsibility. One possible structure is:

~~~text
styles/
  _index.scss                 public Sass API
  app.scss                    CSS output entry point
  foundation/
    _tokens.scss              Sass maps, functions, and mixins
    _theme.scss               runtime CSS custom properties
    _base.scss                document defaults
  components/
    _button.scss
    _card.scss
~~~

The public API file forwards only the Sass tools consumers are meant to use. The output entry point loads the theme, base styles, and components. Consumers can load the Sass API without accidentally emitting all system CSS.

A module should have one clear job. Avoid a single enormous file, but do not split every declaration into a separate partial either. Split when a file has a distinct responsibility or public contract.

## 3. Separate compile-time and runtime tokens

Sass values are resolved during compilation. They are useful for calculations, maps, breakpoint mixins, and values that should not change after the CSS is built.

CSS custom properties are resolved by the browser. They participate in the cascade and can inherit, so they work well for themes and local overrides.

A practical division is:

| Value type | Good fit | Example |
| --- | --- | --- |
| Build-time Sass value | Breakpoint used to emit a media query | <code>content-wide: 48rem</code> |
| Build-time Sass value | Spacing scale used by a function | <code>space(3)</code> |
| Runtime CSS custom property | Theme color that changes with a theme selector | <code>--color-surface</code> |
| Runtime CSS custom property | A value a component or consumer can override | <code>--button-radius</code> |

CSS custom properties cannot be substituted into ordinary media or container query conditions. Keep those query thresholds in Sass or another build-time configuration. Conversely, do not compile a theme color into a Sass variable if the product needs to change it at runtime without recompiling.

## 4. Create a small Sass token API

A token module can keep maps private while exposing checked functions and a purposeful responsive mixin:

~~~scss
@use "sass:map";

$-spaces: (
  1: 0.25rem,
  2: 0.5rem,
  3: 0.75rem,
  4: 1rem,
  6: 1.5rem
);

$-radii: (
  small: 0.25rem,
  medium: 0.5rem,
  large: 0.875rem
);

$-breakpoints: (
  content-wide: 48rem
);

@function space($step) {
  $value: map.get($-spaces, $step);

  @if $value == null {
    @error "Unknown spacing step: #{$step}";
  }

  @return $value;
}

@function radius($size) {
  $value: map.get($-radii, $size);

  @if $value == null {
    @error "Unknown radius: #{$size}";
  }

  @return $value;
}

@mixin above($name) {
  $min-width: map.get($-breakpoints, $name);

  @if $min-width == null {
    @error "Unknown breakpoint: #{$name}";
  }

  @media (min-width: $min-width) {
    @content;
  }
}
~~~

The Sass module system treats members beginning with <code>-</code> or <code>_</code> as private to the module. Consumers use the functions and mixin rather than depending on the underlying maps. That lets the system change its internal structure while keeping a stable public API.

Use checked lookups so a misspelled token fails compilation with a useful message instead of returning an unexpected value.

## 5. Publish the Sass API with <code>@forward</code>

Create <code>styles/_index.scss</code>:

~~~scss
@forward "foundation/tokens" show space, radius, above;
~~~

A consumer can now write:

~~~scss
@use "styles" as ds;

.page-shell {
  padding-inline: ds.space(4);
  border-radius: ds.radius(medium);
}

@include ds.above(content-wide) {
  .page-shell {
    padding-inline: ds.space(6);
  }
}
~~~

The namespace makes each token's origin clear. Use <code>@forward</code> to create a supported public entry point, and omit implementation members from that entry point. Do not forward every internal file simply because it exists.

If a module contains styles as well as API members, forwarding it also makes its CSS part of the output when loaded. Keep a token/API-only module free of component CSS so consumers can use its functions without emitting unrelated styles.

## 6. Define runtime theme tokens

Create <code>styles/foundation/_theme.scss</code>:

~~~scss
:root,
[data-theme="light"] {
  color-scheme: light;
  --color-page: #f8fafc;
  --color-surface: #fff;
  --color-text: #172033;
  --color-muted: #475569;
  --color-border: #cbd5e1;
  --color-action: #4f46e5;
  --color-action-hover: #3730a3;
  --color-on-action: #fff;
  --color-focus: #7c3aed;
}

[data-theme="dark"] {
  color-scheme: dark;
  --color-page: #0f172a;
  --color-surface: #1e293b;
  --color-text: #f8fafc;
  --color-muted: #cbd5e1;
  --color-border: #475569;
  --color-action: #a78bfa;
  --color-action-hover: #c4b5fd;
  --color-on-action: #1e1b4b;
  --color-focus: #c4b5fd;
}
~~~

Components refer to color roles instead of hard-coding a separate palette for every component. A page can select the dark theme with <code>data-theme="dark"</code> on the root element or on a subtree. A theme switch control would need to update that attribute; Sass does not implement the runtime interaction.

Provide declaration fallbacks when the product supports browsers that do not understand CSS custom properties:

~~~scss
.page {
  background-color: #f8fafc;
  background-color: var(--color-page, #f8fafc);
  color: #172033;
  color: var(--color-text, #172033);
}
~~~

The earlier declaration is a fallback if the later declaration is rejected. The second argument in <code>var()</code> is useful when custom properties are supported but the named property is missing or invalid. It does not make a browser that lacks custom-property support understand the entire declaration.

## 7. Build base styles and accessible components

Base styles should set sensible document defaults, not impose component-specific layout on every page. Components should define their own internal structure and interactive states.

A button component might use Sass functions for fixed design-system spacing and CSS custom properties for runtime colors:

~~~scss
@use "../foundation/tokens" as ds;

.button {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-block-size: 2.75rem;
  padding: ds.space(2) ds.space(4);
  border: 1px solid transparent;
  border-radius: ds.radius(medium);
  background-color: #4f46e5;
  background-color: var(--color-action, #4f46e5);
  color: #fff;
  color: var(--color-on-action, #fff);
  font: inherit;
  font-weight: 600;
  text-align: center;
  text-decoration: none;
  cursor: pointer;
}

.button:hover {
  background-color: #3730a3;
  background-color: var(--color-action-hover, #3730a3);
}

.button:focus-visible {
  outline: 3px solid #7c3aed;
  outline: 3px solid var(--color-focus, #7c3aed);
  outline-offset: 3px;
}

.button:disabled,
.button[aria-disabled="true"] {
  cursor: not-allowed;
  opacity: 0.65;
}
~~~

Do not communicate state through color alone. Keep text labels and focus indicators visible, preserve sufficient contrast for each theme, and make disabled or busy states understandable. Use semantic HTML: a button performs an action, while an anchor navigates to a destination.

## 8. Compose one CSS entry point

The application stylesheet can load the system modules in an intentional order:

~~~scss
@use "foundation/theme";
@use "foundation/base";
@use "components/button";
@use "components/card";
~~~

The entry point defines which CSS the application includes. A consumer that only needs Sass tokens can instead load the public API:

~~~scss
@use "styles" as ds;
~~~

Sass's module system loads a module once in a compilation, even if it is referenced through multiple module paths. The namespace rules also prevent token and mixin names from silently becoming global.

Keep CSS output intentional. Compile the application entry point, not every partial independently. Partials that begin with an underscore are source modules and are not separate application entry points.

## 9. Document the design-system contract

A reusable system needs guidance as well as code. Document:

- How to install or include the stylesheet.
- Which stylesheet is the application entry point.
- How to load the public Sass API.
- Which Sass functions and mixins are supported.
- Which CSS custom properties are public theme tokens.
- How to select a theme or override a token.
- How to add a component without changing existing behavior.
- What browser support policy the consuming project must choose.
- How contributors compile and review generated CSS.
- How and when a token or component API may change.

Treat public Sass members, CSS custom properties, and component classes as APIs. Use consistent names and explain whether consumers may override them. Avoid exposing Sass maps as public API unless consumers truly need to depend on their structure.

## 10. Review accessibility, responsiveness, and maintenance

For each component, review its normal, hover, focus, active, disabled, and reduced-motion behavior where applicable. Test the light and dark theme, keyboard use, zoom, narrow layouts, and long content. Check that a theme switch does not leave text or borders with poor contrast.

Compile the Sass and inspect the CSS the browser receives. Confirm that:

- The intended entry point includes each component once.
- Public Sass APIs have clear namespaces.
- Theme colors use role-based custom properties.
- Fallback declarations appear before runtime-variable declarations.
- Breakpoints remain in Sass rather than CSS custom properties.
- Components do not depend on one specific page structure.
- The output has no accidental duplicate or obsolete rules.

A design system is maintained over time. Review unused tokens and APIs, remove obsolete decisions deliberately, and communicate breaking changes to consumers.

## 11. Capstone deliverables

Complete the project when you have:

1. A documented source tree with a single CSS output entry point.
2. A small Sass token module with checked functions or mixins.
3. A public API module that forwards only supported members.
4. Runtime theme custom properties for at least two themes.
5. At least two reusable components with accessible states.
6. A compiled stylesheet produced from the application entry point.
7. A README that explains installation, use, theming, and contribution.
8. A support and review checklist that the consuming project can adopt.

The final system should be small enough that a new contributor can find a token, understand its intended use, and see how it reaches the compiled stylesheet.

## Summary

A reusable SCSS design system combines a small public Sass API with CSS output and documented usage rules. Use Sass maps and functions for build-time tokens and mixins; use CSS custom properties when the value needs to respond to themes or runtime context. Keep the public API narrow, components semantic and accessible, and the compiled output easy to inspect.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="052-browser-support-and-progressive-enhancement.md">052 - Browser support and progressive enhancement</a></td><td><a href="../qnas/053-capstone-build-a-reusable-scss-design-system.md">Q&amp;A</a></td><td><a href="../examples/053-capstone-build-a-reusable-scss-design-system.md">Example</a></td><td><a href="054-scss-quick-reference-and-glossary.md">054 - SCSS quick reference and glossary</a></td></tr>
  </tbody>
</table>
