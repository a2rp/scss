# 002. How Sass Fits CSS and a Web Build

## Learning goals

After this lesson, you should be able to:

- Explain the different jobs of HTML, CSS, JavaScript, Sass, a compiler, a bundler, and a development server.
- Trace an SCSS source file through compilation until the browser receives CSS.
- Compile Sass directly with the Sass command-line interface.
- Describe what a build tool can add around Sass compilation.
- Recognize why a successful development build and a production build may produce different files.
- Find the likely cause when styles are missing from the page.

## The browser-facing technologies

A browser works with web platform resources. In a typical page:

| Technology | Main responsibility |
| --- | --- |
| HTML | Describes the document structure and meaning of its content. |
| CSS | Describes how that content should be presented. |
| JavaScript | Adds behavior and can update the document or its styles. |
| Sass / SCSS | A stylesheet authoring language that adds features such as variables, nesting, and reusable logic, then compiles to CSS. |

The browser does not need SCSS to render the page. Sass turns the source into CSS first. The browser then parses the resulting CSS and applies it to matching HTML elements.

A useful mental model is:

~~~text
HTML + CSS + JavaScript
          |
          v
        Browser

SCSS source
    |
    v
Sass compiler
    |
    v
CSS output
    |
    v
Browser
~~~

Sass is therefore part of the stylesheet authoring and build process. It does not replace CSS, HTML, JavaScript, or the browser.

## Follow one style from source to page

Suppose the project contains this SCSS:

~~~scss
$brand-color: #1769aa;

.card {
  border: 1px solid $brand-color;
  padding: 1rem;

  &:hover {
    opacity: 0.9;
  }
}
~~~

The Sass compiler evaluates Sass-specific syntax and emits ordinary CSS. Conceptually, the output looks like this:

~~~css
.card {
  border: 1px solid #1769aa;
  padding: 1rem;
}

.card:hover {
  opacity: 0.9;
}
~~~

The important transformation is that the variable and parent selector syntax have been resolved. The browser receives CSS selectors and declarations; it does not receive the original Sass variable or nesting instructions.

For new Sass work, prefer the modern color modules such as sass:color over legacy global color functions. A later lesson covers Sass modules and function APIs in detail. The simplified output above is only illustrating the compilation boundary.

## Compile Sass directly

The Sass command-line interface can compile one source file into one CSS file:

~~~sh
npx sass src/styles.scss public/styles.css
~~~

Here:

- src/styles.scss is the source that you edit.
- public/styles.css is the generated CSS output.
- npx runs the project-installed Sass executable when Sass is installed in the project.

The HTML page must load the generated CSS file, not the SCSS source:

~~~html
<link rel="stylesheet" href="./styles.css">
~~~

During development, the Sass CLI can watch a source file and recompile it when it changes:

~~~sh
npx sass --watch src/styles.scss:public/styles.css
~~~

The command-line compiler is useful for learning the Sass-to-CSS transformation and for small projects. A real project may have several source files, assets, JavaScript modules, and output settings, so its build process often coordinates more than one command.

## What a build tool adds

A build tool coordinates project tasks. Depending on the tool and its configuration, it may:

1. Start a local development server.
2. Resolve imports between project files.
3. Invoke Sass when it encounters an SCSS file.
4. Rebuild affected files after edits.
5. Provide fast browser updates during development.
6. Create optimized output for deployment.
7. Generate source maps that help connect output back to source code.

These responsibilities are related but not identical:

| Term | Meaning |
| --- | --- |
| Sass compiler | Transforms Sass source into CSS. |
| Bundler | Follows and combines project dependencies into browser-ready assets. |
| Development server | Serves the project locally and may coordinate rebuilds or live updates. |
| Build tool | Provides commands and configuration to coordinate development and production work. |
| Browser | Loads the produced HTML, CSS, JavaScript, and other assets and renders the page. |

One tool may provide several of these capabilities. The terms describe jobs, not a rule that every project must install a different program for each job.

## Sass in a Vite project

Vite is one example of a build tool. Its CSS feature documentation describes importing CSS from JavaScript and using CSS preprocessors. To use SCSS in a Vite project, install a Sass implementation supported by the current Vite version, then import the stylesheet from the application entry point:

~~~sh
npm install -D sass
~~~

~~~js
import "./styles.scss";
~~~

Vite can then process that SCSS import as part of its development and production workflows. Follow the current Vite documentation for its exact supported behavior and configuration; other build tools can use different setup steps.

A direct Sass CLI project and a Vite project can both use Sass. The difference is how the compiled CSS is connected to the rest of the project:

- With the CLI, you choose the input and output paths in the Sass command.
- With Vite, the application imports the stylesheet and Vite manages it as part of the project dependency graph.

## Development output and production output

A development workflow is optimized for editing and feedback. It may use a local server, rebuild changed files, and provide readable source maps.

A production workflow prepares assets for deployment. Depending on the configured tool, it can minify CSS, process imports, add content hashes to filenames, and write files to a dedicated output directory.

Do not assume that a development URL or generated filename is also the production filename. Use the project’s documented build command and deployment output directory.

## Source files and generated files

Keep a clear distinction between files you author and files tools generate:

~~~text
src/
  styles.scss       source you edit

public/ or dist/
  styles.css        generated output
~~~

The exact directory names depend on the project. Some projects commit generated CSS because it is deployed or consumed directly. Other projects generate it during deployment and exclude it from version control. Follow the project’s existing convention; avoid editing generated output when the source file is meant to be authoritative.

## Debugging missing styles

When the page does not show an expected style, check the pipeline in order:

1. **Is the source file correct?** Confirm the selector matches the HTML and the declaration is valid.
2. **Did Sass compile?** Look for compiler errors in the terminal. A syntax error can prevent updated CSS from being written.
3. **Is the output current?** Check that the compiler or build tool watched the right source file and produced the expected CSS.
4. **Is the CSS loaded?** Inspect the page’s network requests or the stylesheet link/import path.
5. **Did the browser apply the rule?** Inspect the element’s matched rules and computed styles.
6. **Is another rule taking precedence?** A loaded declaration may lose because of the cascade, specificity, importance, or source order.

This sequence separates common causes: a Sass compilation problem, a wrong path, a stale output file, or a CSS cascade conflict.

## Key takeaways

- Sass is compiled into CSS before the browser applies the styles.
- The browser-facing output is CSS, even when the authoring file is SCSS.
- The Sass CLI can compile and watch files directly.
- A build tool can coordinate Sass with imports, JavaScript, a development server, and production output.
- Build-tool behavior depends on the selected tool and its configuration.
- Debug the complete path from SCSS source to generated CSS to the browser’s applied styles.

## Practice

1. Draw the path from an SCSS file to the browser and label each stage.
2. Compile a small SCSS file with the Sass CLI. Identify the source and output paths.
3. Open the generated CSS and find one Sass feature that has been transformed.
4. Change the source, recompile it, and confirm that the browser receives the updated stylesheet.
5. In a Vite project, identify where the SCSS file is imported and which command starts development.

## Further reading

- [Sass documentation](https://sass-lang.com/documentation/)
- [Sass command-line interface](https://sass-lang.com/documentation/cli/)
- [Sass syntax overview](https://sass-lang.com/documentation/syntax/)
- [Vite CSS features](https://vite.dev/guide/features#css)
