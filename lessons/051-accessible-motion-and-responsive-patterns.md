# Accessible motion and responsive patterns

A Sass design system must respond to both the available space and the person using the interface. Viewport size is only one input: components can appear in narrow sidebars, wide content regions, or modal dialogs, and some users ask their operating system to reduce non-essential movement.

Sass helps organize these rules with maps, mixins, modules, and shared tokens. The browser still evaluates the resulting CSS media and container queries, so keep the generated behavior understandable and test the interface at different widths and motion preferences.

## Learning goals

After this lesson, you should be able to:

- Explain the difference between viewport media queries and container queries.
- Build mobile-first layouts and choose breakpoints from content needs.
- Organize Sass breakpoints without creating a large device-specific ruleset.
- Respect the <code>prefers-reduced-motion</code> setting.
- Preserve meaningful state and feedback when an animation is reduced or removed.
- Review responsive and motion behavior with keyboard, zoom, narrow viewports, and assistive technology.

## 1. Design for content and available space

Responsive design adapts a page as its usable space and environment change. It does not mean making one fixed layout for each device brand or model.

Begin with semantic HTML and a layout that works in normal document flow. Then use flexible CSS tools:

- Grid and Flexbox for layouts that can reflow.
- Percentages, <code>min()</code>, <code>max()</code>, and <code>clamp()</code> for fluid values with sensible bounds.
- <code>max-width</code> to keep long text lines or page shells readable.
- Media queries when the design genuinely needs a change at a viewport condition.
- Container queries when a component's layout depends on its own available space.

A viewport breakpoint should follow the content. If a navigation row wraps awkwardly at a particular width, that is evidence for a breakpoint. Picking a breakpoint just because a device is called a tablet is not.

A mobile-first stylesheet sets the narrow layout as the default and adds enhancements as space becomes available. This keeps the base rules simple and makes narrow screens a first-class layout.

For mobile browsers, include the viewport declaration in the HTML document:

~~~html
<meta name="viewport" content="width=device-width, initial-scale=1">
~~~

Without an appropriate viewport, mobile browsers may use a wider virtual layout viewport and scale the page down, so the design may not behave as expected.

## 2. Use viewport media queries for page-level layout

A media query can test viewport conditions and user preferences. Sass variables can make a small set of project breakpoints reusable:

~~~scss
$breakpoints: (
  content-wide: 48rem,
  page-wide: 72rem
);

@mixin above($name) {
  $min-width: map.get($breakpoints, $name);

  @if $min-width == null {
    @error "Unknown breakpoint: #{$name}";
  }

  @media (min-width: $min-width) {
    @content;
  }
}
~~~

Because this mixin uses <code>map.get()</code>, load the built-in Sass map module first:

~~~scss
@use "sass:map";
~~~

A mobile-first page layout can then use the same token:

~~~scss
@use "sass:map";
@use "breakpoints" as bp;

.page-shell {
  width: min(100% - 2rem, 72rem);
  margin-inline: auto;
}

.article-layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr);
  gap: 2rem;
}

@include bp.above(content-wide) {
  .article-layout {
    grid-template-columns: minmax(0, 2fr) minmax(14rem, 1fr);
  }
}
~~~

In this example, the default is a single column. At a wider viewport, the article and complementary content can sit side by side. <code>minmax(0, 1fr)</code> allows grid tracks to shrink below their content's automatic minimum; this can help prevent long content from forcing horizontal overflow.

Only add a breakpoint when the layout needs it. Grid and Flexbox can often wrap naturally without any media query:

~~~scss
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(100%, 17rem), 1fr));
  gap: 1rem;
}
~~~

This lets the browser fit as many cards as the available space allows while keeping a card from becoming wider than its container on a very narrow screen.

## 3. Use container queries for reusable components

A viewport query asks how large the page viewport is. A container query asks how much space a named or nearest eligible container provides. This is useful when the same card appears in different layouts: a wide page may contain a narrow sidebar card, so the viewport alone cannot tell the card how much room it has.

A size-query container needs containment. Use <code>container-type: inline-size</code> when the component responds to its available inline width:

~~~scss
.product-card {
  container-type: inline-size;
}

.product-card__layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr);
  gap: 1rem;
}

@container (min-width: 34rem) {
  .product-card__layout {
    grid-template-columns: minmax(0, 1fr) 12rem;
    align-items: start;
  }
}
~~~

The base one-column style remains usable when container queries are unavailable. The enhancement rearranges the card only when the card's own container is wide enough. Container-query rules style descendants based on the container; they do not select the container itself based on its own query.

Keep the fallback valuable. Do not make essential content or controls appear only inside a container query. Check the component in both a full-width region and a constrained region, such as a sidebar or dialog.

## 4. Respect reduced-motion preferences

Some people experience dizziness, nausea, headaches, or distraction from motion. Operating systems and browsers expose a user preference through the <code>prefers-reduced-motion</code> media feature:

- <code>reduce</code> means the user requested less motion.
- <code>no-preference</code> means the user has not expressed a reduced-motion preference.

The preference is not a request to hide information. Start with a complete, understandable non-animated state. Add optional movement only when the user has not asked for reduced motion:

~~~scss
.notice {
  opacity: 1;
  transform: none;
}

@media (prefers-reduced-motion: no-preference) {
  .notice[data-entering="true"] {
    animation: notice-enter 180ms ease-out both;
  }
}

@keyframes notice-enter {
  from {
    opacity: 0;
    transform: translateY(0.5rem);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
~~~

The notice is visible in its normal state. The optional entrance effect does not carry the notice's message or determine whether it can be read. If reduced motion is requested, the animation rule does not match and the notice still appears.

Another approach is to define an animation normally and override it in a <code>reduce</code> query:

~~~scss
@media (prefers-reduced-motion: reduce) {
  .carousel,
  .dialog,
  .status-message {
    scroll-behavior: auto;
    animation-duration: 0.01ms;
    animation-iteration-count: 1;
    transition-duration: 0.01ms;
  }
}
~~~

Use this pattern carefully. A broad override can accidentally remove a necessary status or progress cue, and near-zero durations can interact poorly with JavaScript that listens for animation events. Prefer explicit component rules when different effects need different reduced-motion behavior. If an effect is entirely decorative and non-essential, disabling it is often clearer:

~~~scss
@media (prefers-reduced-motion: reduce) {
  .decorative-parallax {
    transform: none;
  }

  .decorative-spinner {
    animation: none;
  }
}
~~~

Do not hide content until an animation finishes. Do not rely only on movement, position, or color to communicate a state. Keep text, labels, focus, and status information available even when the animation is absent.

## 5. Distinguish interaction-triggered and automatic motion

WCAG treats motion cases according to how they start. Success Criterion 2.3.3, Animation from Interactions, is a Level AAA criterion about non-essential animation triggered by user interaction; such animation can be disabled unless it is essential to the information or functionality. The guidance also distinguishes automatically started animation, which is addressed by a different criterion, 2.2.2, Pause, Stop, Hide.

This distinction matters in practice:

- A card that dramatically flies across the page after a click is interaction-triggered movement.
- A carousel that starts moving by itself is automatic movement.
- An essential animation that communicates progress or is itself the user's task may need a different treatment from a decorative flourish.

The <code>prefers-reduced-motion</code> preference is a useful way to honor a user's system setting. It is not a substitute for checking whether automatic movement needs a pause, stop, or hide control, nor is one media query proof that an entire interface conforms to WCAG.

## 6. Keep Sass responsive rules maintainable

Sass can help share responsive tokens, but a mixin for every selector and every possible device quickly obscures the CSS. Prefer a small set of named content breakpoints and use them only where a layout needs to change.

Good practices include:

- Keep breakpoint values in one Sass module and load it with <code>@use</code>.
- Name breakpoints by purpose, such as <code>content-wide</code>, instead of device marketing labels.
- Keep responsive rules near the component they affect, unless the project has an established responsive organization.
- Use container queries for components that are reused in differently sized regions.
- Inspect compiled CSS when a mixin emits media queries in many places.
- Avoid deeply nested media queries that make the output and cascade hard to trace.
- Prefer logical properties such as <code>margin-inline</code> and <code>padding-block</code> where they suit the component's writing direction.

Sass creates CSS at build time; it does not decide the user's actual viewport or preference. Keep the final media and container queries readable so they can be inspected in browser developer tools.

## 7. Review the experience at different sizes and preferences

A responsive review should cover more than one desktop and one phone preset. Resize the viewport gradually and check the points where content starts to wrap, collide, or become difficult to use. Also test:

1. A narrow viewport and a wide viewport.
2. Browser zoom and larger text settings.
3. A long translated string or unusually long unbroken content.
4. Keyboard navigation, visible focus, and controls that remain reachable.
5. The same component in a wide page, narrow sidebar, and dialog.
6. Reduced motion enabled, then disabled.
7. Information and status feedback with animations absent.
8. Scrolling, sticky regions, and dialogs when the layout reflows.

Fix the underlying layout before adding another breakpoint. For example, a child with <code>min-width: 0</code>, a flexible grid track, or a wrapping control row may solve overflow more cleanly than a special rule for one device width.

## 8. Practice task

Create a responsive product listing with:

- A centered page shell with a readable maximum width.
- A card grid that fits available space without hard-coding a column count for every device.
- A product card that changes its internal layout with a container query.
- One optional entrance effect that only runs when the user has no reduced-motion preference.
- A visible status message that remains readable without animation.
- Sass breakpoint tokens for one page-level layout change.

Compile the Sass, then inspect the generated CSS. Resize the page, constrain the product card independently of the viewport, and enable reduced motion. Confirm that the content and controls work in each state.

## Summary

Responsive styles should follow content and available space. Use fluid layout first, viewport media queries for page-level changes, and container queries when a reusable component responds to its own space. Sass maps and mixins can organize shared breakpoints, but should not hide the resulting CSS.

Treat reduced motion as a user preference. Make the static state complete, keep essential information available, and classify motion by whether it is interaction-triggered or automatic before deciding how to control it.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous lesson</th><th>Chapter Q&amp;A</th><th>Chapter example</th><th>Next lesson</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="050-css-output-performance-and-maintenance.md">050 - CSS output, performance, and maintenance</a></td><td><a href="../qnas/051-accessible-motion-and-responsive-patterns.md">Q&amp;A</a></td><td><a href="../examples/051-accessible-motion-and-responsive-patterns.md">Example</a></td><td><a href="052-browser-support-and-progressive-enhancement.md">052 - Browser support and progressive enhancement</a></td></tr>
  </tbody>
</table>
