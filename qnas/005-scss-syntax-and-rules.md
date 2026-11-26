# 005. SCSS Syntax and Rules - Q&A

## Syntax and stylesheet structure

### 1. What does SCSS stand for in a Sass project?

SCSS is the CSS-like syntax of Sass. It uses the .scss file extension, braces for blocks, and semicolons to separate statements.

### 2. Is SCSS valid CSS?

Most ordinary CSS can be written unchanged in SCSS, but SCSS also allows Sass-specific syntax. There are a few parsing differences and special cases, so do not treat every possible CSS text fragment as automatically valid SCSS.

### 3. What is the other Sass syntax?

The indented syntax, commonly called Sass syntax, uses the .sass extension. It uses indentation instead of braces and newlines instead of semicolons to separate statements.

### 4. Can .scss and .sass files use different Sass features?

They support the same Sass language features, but their syntax differs. A project should use the syntax indicated by each file’s extension and remain consistent in its own source files.

### 5. What is a Sass statement?

A statement is a syntactic instruction in a Sass stylesheet. Style rules, variable declarations, at-rules, and mixin or function definitions are examples of statements.

### 6. What is an expression?

An expression is a value-producing part of Sass syntax. It can appear on the right side of a variable declaration or a CSS property declaration. A plain CSS value and a Sass expression can both be used there.

### 7. What is a style rule?

A style rule consists of a selector and a block. The selector identifies the elements to style, and the block contains declarations or nested rules.

### 8. What is a declaration?

A declaration pairs a CSS property with a value:

~~~scss
color: navy;
~~~

Declarations are generally written inside a style rule or a supported CSS at-rule.

## Braces, semicolons, and values

### 9. What do braces do in SCSS?

Opening and closing braces define blocks. Style rules and many at-rules use blocks to contain declarations or other statements.

### 10. Are semicolons required in SCSS?

Semicolons separate statements within a block. The semicolon after the final statement just before a closing brace is technically optional, but keeping it consistently is a useful convention.

### 11. Can one SCSS declaration span multiple lines?

Yes, when the value syntax allows it, such as a long list or function expression. Keep the declaration’s semicolon at the end of the complete value rather than breaking it into multiple declarations.

### 12. Do spaces inside a CSS value separate declarations?

No. Spaces and commas inside a value are part of that value. The semicolon marks the end of the declaration.

### 13. Can property values use Sass variables or function calls?

Yes. A property value can contain a Sass expression. Sass evaluates it during compilation and emits the resulting CSS value.

### 14. Are CSS custom properties the same as Sass variables?

No. Sass variables begin with a dollar sign and are evaluated during compilation. CSS custom properties begin with two hyphens and remain in the CSS output for the browser to use at runtime.

### 15. Why does a Sass variable inside a CSS custom property need interpolation?

Sass preserves custom property values as CSS text. To insert a SassScript value into that text, use interpolation:

~~~scss
$accent: #1769aa;

:root {
  --accent: #{$accent};
}
~~~

The generated CSS contains the resolved color.

### 16. What does nesting do in SCSS?

A nested style rule is written inside another style rule’s block. Sass combines the selectors and emits ordinary CSS. Keep nesting shallow so the output selectors remain easy to read.

## At-rules and comments

### 17. What is an at-rule?

An at-rule starts with the @ symbol. CSS at-rules include @media and @font-face. Sass also has at-rules such as @use, @mixin, and @include.

### 18. Are all at-rules Sass-specific?

No. Some are part of CSS and are passed through or transformed as CSS. Others are Sass-specific and tell the compiler to load modules, define reusable content, or perform other Sass work.

### 19. What is a silent comment?

A silent comment starts with // and is not emitted into CSS:

~~~scss
// Source-only note
.card {
  padding: 1rem;
}
~~~

### 20. What is a block or loud comment?

A block comment starts with /* and ends with */. It is normally emitted as a CSS comment in expanded output, but Sass strips it from compressed output unless it begins with /*!.

### 21. What is a documentation comment?

A documentation comment starts with ///. SassDoc can read these comments to generate documentation for Sass libraries.

### 22. Why does a comment show in generated CSS?

If the source uses a block comment, Sass normally preserves it in expanded output. Use // for a source-only note, or remember that compressed output strips ordinary block comments.

## Errors and good practice

### 23. What happens when SCSS has invalid syntax?

Sass reports a parse error and usually includes the file, line, and nearby source. It generally fails compilation rather than silently recovering as a browser may do with invalid CSS.

### 24. Why can an error be reported on a line after the actual mistake?

A missing brace, quote, or comment terminator earlier in the file can cause later text to be parsed in the wrong context. Inspect the surrounding lines before the reported location.

### 25. Which encoding should Sass source files use?

Use UTF-8 for Sass source files. Dart Sass currently documents UTF-8 as its supported encoding.

### 26. Why keep the last semicolon even though it can be omitted?

Consistent semicolons make it easier to add another declaration and reduce mistakes during edits and merges.

### 27. Should I use deep nesting to mirror every element in the HTML?

No. Deep nesting can create long selectors and make generated CSS harder to predict. Nest only when it improves organization, and inspect the compiled selector.

### 28. What is a useful first check when the compiler reports a syntax error?

Look at the first error, the named file, and several nearby lines. Check braces, semicolons, quotes, comment delimiters, and whether a declaration is inside a valid block.

## Quick review

- **SCSS:** CSS-shaped Sass syntax using braces and semicolons.
- **Statement:** A Sass instruction such as a style rule, variable, or at-rule.
- **Expression:** A value-producing part of a declaration or Sass statement.
- **Declaration:** A property and value separated by a colon.
- **Silent comment:** // comment that is not emitted as CSS.
- **Block comment:** /* comment */ normally emitted except in compressed output.
- **Sass variable:** A compile-time value beginning with $.
- **CSS custom property:** A runtime CSS value beginning with --.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="004-compiling-scss-to-css.md">004 - Compiling SCSS to CSS - Q&amp;A</a></td><td><a href="../lessons/005-scss-syntax-and-rules.md">Lesson</a></td><td><a href="../examples/005-scss-syntax-and-rules.md">Example</a></td><td><a href="006-sass-variables-and-scope.md">006 - Sass Variables and Scope - Q&amp;A</a></td></tr>
  </tbody>
</table>
