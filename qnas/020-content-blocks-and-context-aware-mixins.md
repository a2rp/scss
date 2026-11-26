# 020 - Content Blocks and Context-Aware Mixins: Q&A

## Content block basics

### 1. What is a Sass content block?

It is a block of Sass statements supplied at an @include site and inserted by a mixin where it uses @content.

### 2. How does a mixin declare an insertion point?

Put the @content at-rule inside the mixin body wherever the caller's block should be inserted.

### 3. How does an @include supply a content block?

Write the include followed by a block in braces containing the caller's Sass statements.

### 4. What kinds of statements can a content block contain?

It can contain declarations, nested style rules, CSS at-rules, and other valid Sass statements.

### 5. Does the browser receive @content?

No. @content is Sass syntax. Sass inserts the supplied block during compilation and emits CSS.

### 6. What happens to the block if a mixin uses @content twice?

The block is inserted at both locations, so the CSS can be generated more than once.

### 7. Is a content block the same as a mixin argument?

No. An ordinary argument passes a value. A content block passes a block of Sass statements.

## Context-aware mixins

### 8. What makes a mixin context-aware?

The mixin chooses a context, such as a selector state or media query, and inserts caller-provided styles inside it.

### 9. How can a mixin wrap styles in a hover state?

Use a nested selector with the parent selector, such as &:hover, and put @content inside that rule.

### 10. How can a mixin wrap styles in a media query?

Put @content inside an @media rule in the mixin body. The caller's declarations are emitted inside that query.

### 11. Can a responsive mixin also accept a breakpoint argument?

Yes. It can use a normal mixin parameter for the breakpoint and a content block for the styles that belong inside the media query.

### 12. Where do nested selectors in a content block appear?

They are inserted at the @content location, inside whatever selector or at-rule context surrounds that location in the mixin.

### 13. What does &:focus-visible mean inside a context-aware mixin?

It uses the selector context at the include site and adds the focus-visible pseudo-class to that parent selector.

## Scope

### 14. What does it mean that a content block is lexically scoped?

The block can access variables from the scope where the @include is written. Its variable visibility is based on the caller's source location.

### 15. Can a content block read a variable declared locally inside the mixin?

No. A mixin-local variable is not in the lexical scope where the caller wrote the content block.

### 16. How can a mixin pass its local value into a content block?

Call @content(value) in the mixin, and declare a parameter for that value in the include's using (...) clause.

### 17. Will a content block automatically receive the name of a mixin parameter?

No. The mixin must explicitly pass the value with @content(...), and the caller must accept it with using (...).

## Passing arguments to content

### 18. What is the purpose of @content(value)?

It passes a value from the mixin to the caller-provided content block.

### 19. What is the purpose of using (value) after an include?

It declares the local parameter name that receives an argument passed by @content(...).

### 20. Does the number of using parameters need to match the values passed by @content?

Yes. The content block must declare the arguments it accepts. Keep the two sides in sync.

### 21. Can the content block use the same variable name as the mixin parameter?

It can choose any clear local parameter name in using (...). The important part is that the mixin passes the value and the content block declares a receiving parameter.

### 22. Should content arguments be passed by name or position?

Prefer positional arguments. The content block must explicitly declare what it receives, and changing the argument list can require changes at every include site.

### 23. Which Dart Sass version supports passing arguments to content blocks?

Dart Sass added support in version 1.15.0. LibSass does not support this feature. Check the compiler used by a project before relying on it.

## Design and debugging

### 24. When should I use a content block instead of a regular argument?

Use a content block when the caller needs to provide a group of Sass statements for a context controlled by the mixin. Use a regular argument when the caller only needs to provide a value.

### 25. Can a content block contain more than property declarations?

Yes. It can include nested selectors and other valid Sass statements, which lets the caller customize a whole section of styles inside the mixin's context.

### 26. Does every mixin need @content?

No. A mixin that always emits a fixed group of styles may need only its definition and optional arguments. Add a content block only when callers need to supply their own Sass statements.

### 27. What is a risk of using multiple @content statements?

Each one inserts the supplied block, which can duplicate declarations or entire nested rules. Check the generated output to confirm that repeated insertion is intentional.

### 28. What if the include supplies a block but the mixin has no @content statement?

The supplied statements have no insertion point in the mixin, so they are not used. Put @content in the mixin at the intended location.

### 29. What should I check when a content block cannot find a variable?

Check whether the variable is declared at the include site's scope. A variable local to the mixin is not automatically visible in the content block; pass it explicitly if needed.

### 30. How do I debug a context-aware mixin?

Inspect the generated selector or at-rule and verify that the content block was inserted in the intended location. Then compare the output with the markup and responsive conditions.

### 31. Can I use both ordinary arguments and a content block in one mixin?

Yes. The arguments can configure the context, while the content block supplies the statements that go inside it.

### 32. Are content blocks used by the browser at runtime?

No. Sass expands them at compile time. The browser receives the resulting CSS rules.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="019-mixin-arguments-defaults-and-keyword-args.md">019 - Mixin Arguments, Defaults, and Keyword Arguments: Q&amp;A</a></td><td><a href="../lessons/020-content-blocks-and-context-aware-mixins.md">Lesson</a></td><td><a href="../examples/020-content-blocks-and-context-aware-mixins.md">Example</a></td><td><a href="021-custom-functions-and-return-values.md">021 - Custom Functions and Return Values: Q&amp;A</a></td></tr>
  </tbody>
</table>
