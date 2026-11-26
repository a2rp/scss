# 002. How Sass Fits CSS and a Web Build — Q&A

## Concepts

### 1. What does Sass do in a web project?

Sass is a stylesheet authoring language and compiler. You write Sass or SCSS source, and the compiler transforms it into CSS that browsers can use.

### 2. Can a browser render an SCSS file directly?

No. Browsers understand CSS, not Sass syntax. Compile SCSS to CSS and make the resulting CSS available to the page.

### 3. Does Sass replace CSS?

No. Sass adds authoring features, but its output is CSS. Understanding CSS remains essential because the browser applies the compiled CSS rules.

### 4. What are the roles of HTML, CSS, and JavaScript?

HTML provides document structure and meaning. CSS controls presentation. JavaScript adds behavior and can interact with the document and styles.

### 5. What is the basic Sass-to-browser pipeline?

The SCSS source is read by the Sass compiler, which emits CSS. The page loads that CSS, and the browser applies its rules to matching elements.

~~~text
SCSS source -> Sass compiler -> CSS output -> Browser
~~~

### 6. What happens to a Sass variable during compilation?

The compiler substitutes its value where the variable is used. The generated CSS contains the resulting CSS value rather than the Sass variable declaration.

### 7. What happens to nested selectors during compilation?

Sass expands the nesting into regular CSS selectors. For example, a nested hover rule inside .card becomes a separate .card:hover rule.

### 8. What is a Sass compiler?

It is the program that transforms Sass or SCSS source into CSS. The Sass command-line interface is one way to run that compiler.

### 9. What is a bundler?

A bundler follows dependencies between project files and prepares the assets needed by the browser. Depending on the project, it may combine or transform assets as part of a build.

### 10. What is a development server?

It serves a project on the developer’s machine so the application can be opened in a browser. Some development servers also coordinate rebuilds and fast updates.

### 11. Are a compiler, bundler, development server, and build tool always separate programs?

No. A build tool may combine several capabilities. The names describe responsibilities; a project’s tool choice determines which program performs each responsibility.

## Commands and project setup

### 12. How can Sass compile one file from the terminal?

Install Sass in the project, then run a command such as:

~~~sh
npx sass src/styles.scss public/styles.css
~~~

The first path is the source file. The second path is the generated CSS file.

### 13. How can Sass automatically rebuild a file after edits?

Use the Sass CLI watch option:

~~~sh
npx sass --watch src/styles.scss:public/styles.css
~~~

The process stays active and recompiles when the watched source changes.

### 14. Which file should an HTML page load: the SCSS source or the generated CSS?

Load the generated CSS. For example:

~~~html
<link rel="stylesheet" href="./styles.css">
~~~

The browser cannot interpret the original SCSS syntax.

### 15. How does a Vite project use an SCSS file?

Install a supported Sass implementation, then import the SCSS file from the application entry point:

~~~sh
npm install -D sass
~~~

~~~js
import "./styles.scss";
~~~

Vite processes that import as part of its project workflow. Check the current Vite documentation for version-specific details.

### 16. Is Vite required to use Sass?

No. Sass can be compiled directly with its CLI or integrated with another tool. Vite is one possible project setup.

### 17. Why does the Sass CLI workflow look different from the Vite workflow?

With the CLI, the command explicitly names the input and output paths. In Vite, the application imports the SCSS file and the tool manages it as part of the dependency graph.

## Development, production, and debugging

### 18. How can development output differ from production output?

Development favors fast feedback and may include readable source maps. Production output may be minified, fingerprinted, or written to a deployment directory. Exact behavior depends on the project’s build tool and configuration.

### 19. What is a source map useful for?

A source map helps browser developer tools connect generated output back to its original source. It can make debugging compiled or minified assets easier.

### 20. Should generated CSS always be committed to version control?

No universal rule applies. Some projects commit generated CSS because it is consumed or deployed directly. Others create it during deployment and exclude it. Follow the project’s established workflow.

### 21. What should you check first if a Sass edit has no visible effect?

Check that the source selector and declaration are correct, then look for compiler errors. A failed compilation can leave old output in place.

### 22. Sass compiled successfully, but the page is still unstyled. What next?

Check that the page loads the correct generated CSS file. Inspect the browser’s network requests and confirm the stylesheet path is valid.

### 23. The stylesheet loads, but a rule is not applied. What should you inspect?

Inspect the element’s matched rules and computed styles. The selector may not match, or another rule may win because of cascade order, specificity, or importance.

### 24. Why is it useful to debug the pipeline one stage at a time?

It helps distinguish a source syntax error from stale output, an incorrect file path, a loading failure, or a CSS cascade conflict. Each stage has different evidence to inspect.

### 25. How should a team decide whether to edit SCSS or generated CSS?

Use the file designated as the source of truth by the project. If CSS is generated from SCSS, update the SCSS and rebuild; manual edits to generated output may be overwritten.

## Quick review

- **Authoring source:** Sass or SCSS.
- **Compilation result:** CSS.
- **Browser input:** HTML, CSS, JavaScript, and other supported assets.
- **Direct compiler workflow:** Choose source and output paths in the Sass CLI.
- **Integrated workflow:** A build tool coordinates Sass with the rest of the project.
- **Debugging order:** Source, compilation, output, loading, selector match, cascade.
