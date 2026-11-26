# 015 — Nesting Rules and Declarations: Q&A

## Understanding nested selectors

### 1. What does Sass do with a nested style rule?

Sass combines the nested selector with its parent selector and emits a CSS rule. For example, a p nested inside .card becomes .card p in the generated CSS.

### 2. Does a nested selector without a combinator mean a direct child?

No. A plain nested selector uses a descendant relationship, represented by a space. It can match a descendant at any depth.

### 3. How do I target only a direct child?

Use the child combinator > in the nested selector. A direct-child selector matches only elements whose immediate parent matches the outer selector.

### 4. What is the difference between descendant and child selectors?

A descendant selector such as .card p can match a p anywhere inside .card. A child selector such as .card > p matches only a p directly inside .card.

### 5. How are comma-separated selector lists combined?

Sass combines every outer selector with every inner selector. Two outer selectors and two inner selectors produce four combinations.

### 6. Why can nesting two selector lists generate a lot of CSS?

The number of combinations is the product of the number of outer and inner selectors. Larger lists can create many selectors and make it harder to check whether every combination is intended.

### 7. Does selector nesting change selector specificity?

Sass emits ordinary CSS selectors. The specificity of each generated selector follows the normal CSS rules for that selector. Nesting itself is not a new specificity adjustment.

### 8. Can I nest sibling combinators?

Yes. The adjacent-sibling combinator + and general-sibling combinator ~ can be used to describe relationships between elements that share a parent.

### 9. Does indentation in SCSS create HTML elements?

No. Indentation and nesting organize the stylesheet source. They do not change the DOM or add elements to the markup.

### 10. What happens if a nested selector begins with a combinator?

Sass uses it as the relationship between the outer and inner selectors. Use this syntax only when the intended child or sibling relationship is clear.

## Nested property declarations

### 11. What is property nesting?

Property nesting is a Sass syntax that lets you place declarations under a property prefix. Sass joins the outer property name and inner property names with a hyphen.

### 12. What does a nested font block generate?

A font block containing family, size, and weight generates font-family, font-size, and font-weight declarations.

### 13. Is a nested property block the same as a nested selector?

No. In property nesting, the inner names complete property names. In selector nesting, the inner block creates a selector that Sass combines with the parent selector.

### 14. Does Sass combine nested longhand properties into a shorthand?

No. Sass emits the expanded longhand declarations. It does not automatically combine them into a shorthand property.

### 15. Can I write a shorthand and then nested longhand declarations?

Yes. Sass emits the shorthand first and the nested longhand declarations after it. The later declarations can override parts of the shorthand according to the CSS cascade.

### 16. When should I avoid property nesting?

Avoid it when the shared prefix does not improve readability or when a reader has to mentally reconstruct the final property names. Ordinary declarations are always a valid alternative.

## Declaration order and generated CSS

### 17. What happens when a declaration appears after a nested rule?

In modern Dart Sass, the declaration remains in that source order. Sass may emit the parent selector more than once to preserve the order in the generated CSS.

### 18. Why might the same selector appear twice in generated CSS?

The repeated selector preserves source order when declarations and nested rules are interleaved. It is valid CSS and is not automatically a bug.

### 19. Which Dart Sass version introduced the current mixed-declaration behavior?

Dart Sass 1.92.0 and later preserve declaration order around nested rules in the same way as CSS. Check the compiler version used by the project.

### 20. Why do some older examples show declarations moved to the beginning?

Older Sass behavior moved declarations before nested rules. Dart Sass deprecated that behavior in 1.77.7, then changed to source-order output in 1.92.0.

### 21. Should I move a declaration before a nested rule just to avoid duplicate selectors?

Only if moving it does not change the intended cascade. Declaration order can affect which value wins, so do not reorganize it solely to make the generated CSS look shorter.

### 22. How can I make the intended grouping easier to read?

Keep declarations that conceptually belong together in one group before nested rules. If a declaration must follow a nested rule, keep it in that position and understand the generated output.

### 23. How do I verify the exact output for my project?

Compile the SCSS with the same Dart Sass version and options used by the project, then inspect the generated CSS. Compiler versions can differ in behavior.

## At-rules, maintainability, and CSS nesting

### 24. What happens to a media query nested inside a selector?

Sass places the generated selector rule inside the media query. The selector still targets the same element, but the declarations apply only when the media condition matches.

### 25. Does Sass nesting require browser support for native CSS nesting?

No. Sass resolves SCSS nesting during compilation and outputs CSS selectors. Native CSS nesting is interpreted by the browser and has some different semantics.

### 26. Can I copy Sass nested selectors directly into a CSS file?

Do not assume the meaning will be identical. Sass and native CSS nesting overlap, but have differences, including selector-list behavior and Sass-specific selector suffix patterns.

### 27. Why should I keep nesting shallow?

Deep nesting can generate long selectors, increase specificity, and tie a component's styles to an unnecessary chain of ancestors. Shallow selectors are easier to find and change.

### 28. Should every level of the HTML tree be mirrored in SCSS?

No. Nest only when the relationship is useful for organizing styles and accurately expresses the selector requirement. SCSS should not mechanically mirror every markup level.

### 29. What does a plain nested pseudo-class select?

Without Sass's parent selector, a nested pseudo-class is a descendant selector. For example, a plain :hover nested under .button targets a hovered descendant, not the .button itself. The parent selector is covered in lesson 016.

### 30. What is a reliable way to review a nested rule?

Read the generated selector and say which elements it matches. Check whether each ancestor and combinator is necessary, then verify the compiled CSS if the output is not obvious.

### 31. If Sass compiles successfully, does that guarantee the selector matches my markup?

No. Successful compilation confirms the Sass syntax is accepted. It does not verify that the selector describes the intended elements in the actual document.

### 32. What should I check when a declaration seems to be overridden?

Inspect the generated CSS and compare selector matching, specificity, source order, media conditions, and later declarations. With mixed rules, confirm the Sass version and output order as well.

## Quick review

### 33. What does a plain nested selector use as its relationship?

A descendant relationship, represented by a space.

### 34. How does property nesting form a final property name?

It joins the outer and inner property names with a hyphen.

### 35. What is the key behavior change to remember for mixed declarations?

Dart Sass 1.92.0 and later preserves the original source order around nested rules, even if the generated CSS repeats a parent selector.

## Official references

- [Sass style rules and selector nesting](https://sass-lang.com/documentation/style-rules/)
- [Sass property declarations and nested properties](https://sass-lang.com/documentation/style-rules/declarations/)
- [Sass mixed declarations change](https://sass-lang.com/documentation/breaking-changes/mixed-decls/)
- [Sass and native CSS nesting](https://sass-lang.com/blog/sass-and-native-nesting/)
- [Sass parent selector](https://sass-lang.com/documentation/style-rules/parent-selector/)
