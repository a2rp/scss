# Accessible motion and responsive patterns - Q&A

These answers explain how to organize responsive Sass and respect motion preferences. Browser support changes over time; consult the linked CSS and Sass references when choosing a production support baseline.

## 1. What does responsive design mean?

Responsive design adapts content and layout to the space and environment in which a page is used. It includes flexible layout, readable text, usable controls, and behavior that remains understandable at narrow and wide sizes. It is not a collection of fixed layouts for specific phone and tablet models.

## 2. What is mobile-first CSS?

Mobile-first CSS starts with the narrow layout as the default. Media queries then add or adjust layout when more space is available. It often works well because a simple single-column flow is a useful baseline and wider layouts become enhancements.

## 3. Should I use a media query for every screen size?

No. Grid, Flexbox, wrapping, and intrinsic sizing can adapt continuously. Add a media query when the layout needs a meaningful change, such as a sidebar becoming a second column. Choose the breakpoint by testing where the content needs more space, not by targeting a device name.

## 4. What does the viewport meta element do?

The element <code>&lt;meta name="viewport" content="width=device-width, initial-scale=1"&gt;</code> tells mobile browsers to use the device width as the layout viewport and start at the expected scale. This lets viewport-based responsive CSS behave as intended. It belongs in the HTML document head.

## 5. What is the difference between a media query and a container query?

A media query tests the viewport or another user-agent environment condition, such as reduced-motion preference. A container query tests the size or other supported characteristics of a containing element. Use media queries for page-level changes and container queries when a reusable component should respond to its own allocated space.

## 6. Does a container query style the container itself?

A size container query styles matching descendants based on an eligible container. It does not style the queried container itself according to its own size. A common pattern is to set <code>container-type: inline-size</code> on a card and use <code>@container</code> to rearrange an inner layout.

## 7. Why set <code>container-type: inline-size</code>?

It establishes inline-size containment so the browser can evaluate inline-size queries for descendants. Use it on the component region whose available width should control the internal layout. Check intrinsic sizing after adding containment, especially when that element is itself a grid or flex item.

## 8. What is a good fallback for container queries?

Start with a complete base layout that works without the container query, such as a single-column card. Use the query to enhance the layout when more component space is available. Do not hide essential content or controls in the enhancement. If an older browser is in the support target, test the baseline there.

## 9. How can I centralize breakpoints in Sass?

Put a small map of meaningful breakpoints in a Sass module and expose a mixin that looks up a name and emits a media query. This keeps repeated values consistent. Include an <code>@error</code> for unknown names so a typo is caught during compilation instead of silently producing a missing rule.

~~~scss
@use "sass:map";

$breakpoints: (
  content-wide: 48rem
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

The consuming stylesheet can load this module with <code>@use</code> and call the mixin. The browser evaluates the compiled media query; Sass does not detect the viewport at compile time.

## 10. Should breakpoint names describe device sizes?

Usually not. Names such as <code>content-wide</code> or <code>navigation-expanded</code> describe what the design needs. Names like <code>tablet</code> can become misleading when the component is used in a different context or devices change size.

## 11. Can CSS custom properties be used as media-query breakpoints?

Sass variables are substituted during compilation and can be used to produce media-query conditions. CSS custom properties are evaluated at runtime in declarations, but are not generally usable as values in ordinary media-query conditions. For shared breakpoints, Sass maps or another build-time token system are a suitable option.

## 12. What is <code>prefers-reduced-motion</code>?

It is a CSS media feature that detects whether the user has indicated a preference to minimize non-essential movement. Its values include <code>reduce</code> and <code>no-preference</code>. It expresses a preference, not a screen size or device category.

## 13. Does reduced motion mean every animation must disappear?

No. The preference should lead you to remove, reduce, or replace non-essential motion. Some animation may convey essential information or be the functionality itself. Review each effect and make sure the information remains available without motion.

## 14. Why is a complete static state important?

If content is invisible until an entrance animation finishes, disabling that animation can leave the content hidden. Make the normal state readable and usable, and layer decorative movement on top. This also makes the page more resilient if animation support or script timing differs.

## 15. Is it better to add motion by default or only for <code>no-preference</code>?

For optional motion, an opt-in rule such as <code>@media (prefers-reduced-motion: no-preference)</code> keeps the base style free of the effect. Another valid design is to define motion by default and explicitly reduce it in a <code>reduce</code> query. The right choice depends on the component, but the non-motion state must remain complete.

## 16. Is <code>animation: none</code> always the best reduced-motion rule?

No. It is clear for a purely decorative animation, but a blanket rule can remove useful feedback or break code that expects animation events. Some effects should be replaced with a static state or a less motion-heavy change. Prefer component-specific rules and test the result.

## 17. Can I globally set <code>* { animation: none !important; }</code>?

It is not a good default. It can disable essential or user-authored animation, override component decisions, and leave controls or scripts in inconsistent states. Use targeted reduced-motion styles for non-essential effects and give users a clear control when the experience has site-level animation settings.

## 18. What is the difference between WCAG 2.3.3 and 2.2.2?

WCAG 2.3.3, Animation from Interactions, concerns non-essential motion animation that a user action triggers; it is a Level AAA criterion and permits an exception for animation essential to functionality or information. WCAG 2.2.2, Pause, Stop, Hide, addresses certain automatically started moving, blinking, or scrolling content. The start condition matters, and a reduced-motion query alone does not address every automatic-motion case.

## 19. Is <code>prefers-reduced-motion</code> enough to claim WCAG conformance?

No. It is one technique for honoring motion preferences. Conformance depends on the whole interface, applicable success criteria, content, and interaction behavior. Check automatic movement separately, preserve status information, and test the actual user experience.

## 20. How should I handle a loading indicator?

Keep the status understandable without relying only on movement. Provide text such as “Loading results” where appropriate, and expose the status semantically in the interface. If the spinner is decorative, it can be removed or made static under reduced motion while the status text remains.

## 21. How should a carousel behave for reduced motion?

Avoid automatic movement unless it is appropriate and users can control it. Provide clear previous and next controls, preserve keyboard access, and keep the current item state available without animated sliding. A reduced-motion preference should remove or replace the slide animation. Automatic movement can also require a pause or stop mechanism depending on its behavior.

## 22. Should I use <code>transition-duration: 0.01ms</code> as a universal fix?

Not as a universal fix. Near-zero durations can still trigger transition events and may conflict with JavaScript that waits for those events. Prefer explicitly turning off decorative animation, using a non-motion state, or adjusting the specific transition after testing its component behavior.

## 23. How do I avoid horizontal overflow on narrow screens?

Use flexible tracks such as <code>minmax(0, 1fr)</code>, allow controls and text to wrap, and check children with long or unbroken content. A grid or flex item may need <code>min-width: 0</code> or the logical equivalent <code>min-inline-size: 0</code>. Avoid fixed widths and unnecessary <code>100vw</code> sizing inside a page that already has padding.

## 24. How do I make responsive text without extreme sizes?

Use a bounded fluid value when appropriate, for example <code>font-size: clamp(1rem, 0.9rem + 1vw, 1.5rem)</code>. Keep a readable minimum and maximum, and verify zoom and text enlargement. Fluid scaling should not prevent users from reading or resizing text.

## 25. What should I test for a component that uses both breakpoints and motion?

Test the component in a wide page, a narrow container, and a narrow viewport; then test with reduced motion enabled and disabled. Verify keyboard operation, visible focus, readable status, wrapping, and overflow. Also inspect the compiled CSS and confirm the no-motion state still exposes the same content and controls.

## 26. How do I test the reduced-motion path?

Enable the reduced-animation or reduced-motion preference in the operating system or use a browser's media-feature emulation when available. Reload or re-evaluate the page, then verify that non-essential motion is removed or replaced and that content remains visible. Also test the normal preference so the optional motion path is reviewed.

## 27. How much Sass abstraction should I use?

Enough to keep repeated design decisions consistent, but not so much that the emitted CSS becomes hard to trace. A small breakpoint map and a few purposeful mixins are often sufficient. Avoid a mixin for every selector and every device width; inspect the compiled output when abstractions emit many nested rules.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="050-css-output-performance-and-maintenance.md">050 - CSS output, performance, and maintenance - Q&amp;A</a></td><td><a href="../lessons/051-accessible-motion-and-responsive-patterns.md">Lesson</a></td><td><a href="../examples/051-accessible-motion-and-responsive-patterns.md">Example</a></td><td><a href="052-browser-support-and-progressive-enhancement.md">052 - Browser support and progressive enhancement - Q&amp;A</a></td></tr>
  </tbody>
</table>
