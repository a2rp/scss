# Framework integrations: Angular, React, and Vite

Sass is a source language that must be compiled to CSS before a browser can use it. In an application, a framework's CLI or build tool usually owns that compilation. Angular CLI can compile Sass styles as part of its build. React itself does not prescribe a styling pipeline, so a bundler such as Vite must process imported Sass files. Vite supports Sass when the Sass implementation is installed in the project.

The key is to choose one build pipeline for each stylesheet, understand which styles are global or component-scoped, and make the same compilation work in development, tests, and production.

## Learning goals

After this lesson, you should be able to:

- Explain which tool compiles Sass in Angular, React, and Vite projects.
- Set up and locate global and component-level SCSS.
- Configure a Sass include path in Angular or preprocessor options in Vite.
- Share Sass modules using `@use` without emitting duplicate CSS.
- Distinguish Sass modules, CSS Modules, and framework style scoping.
- Diagnose common integration and production-build problems.

## 1. The integration model

A browser does not execute `.scss` or `.sass` files. A typical application pipeline looks like this:

~~~text
SCSS source
    -> framework CLI or bundler
    -> compiled CSS and assets
    -> browser
~~~

Sass handles Sass-language features such as variables, mixins, modules, and nesting. The framework or bundler decides how source files are discovered, how CSS is attached or emitted, whether styles are scoped, and how production assets are built.

Do not run a second independent Sass compilation for a file already owned by the framework pipeline. Two pipelines can create duplicate CSS, stale output, or inconsistent compiler versions.

## 2. Angular: choose SCSS for a new project

When creating an Angular application, select SCSS as its stylesheet language:

~~~sh
ng new storefront --style=scss
~~~

The Angular CLI generates SCSS files for the application styles and components. The option affects generated files; it does not turn Sass into a browser feature. The CLI still compiles those files during development and builds.

For an existing project, inspect its generated files and workspace configuration before changing extensions. Convert styles deliberately and ensure the relevant Angular build target still recognizes them. Do not rename files while leaving old stylesheet references in components or configuration.

## 3. Angular global and component styles

Global styles affect the application as a whole. A common entry file is `src/styles.scss`; the Angular workspace's build configuration lists it under the application's `styles` option. Global resets, document defaults, typography, and site-wide design tokens commonly belong in this global style graph.

A component stylesheet is referenced by that component's metadata. For example:

~~~ts
@Component({
  selector: 'app-product-card',
  templateUrl: './product-card.html',
  styleUrl: './product-card.scss',
})
export class ProductCard {}
~~~

The exact generated metadata can vary with Angular version and project setup; follow the component convention already used in the project. Angular's view-encapsulation behavior determines how component CSS is applied. Sass itself does not scope selectors to a component.

Keep component-specific selectors local to the component stylesheet. Keep global selectors and shared base styles in the global stylesheet. Avoid placing an entire site's global rules in every component file.

## 4. Angular Sass include paths

Angular CLI accepts Sass preprocessor options in the workspace configuration. An include path can make a shared directory available to Sass module URLs:

~~~json
{
  "projects": {
    "storefront": {
      "architect": {
        "build": {
          "options": {
            "stylePreprocessorOptions": {
              "includePaths": ["src/styles"]
            }
          }
        }
      }
    }
  }
}
~~~

With `src/styles/_tokens.scss`, a style file can then write:

~~~scss
@use "tokens" as tokens;

.product-card {
  color: tokens.$text-color;
}
~~~

The `includePaths` setting changes where Sass looks for modules. It does not automatically include a stylesheet in the browser. A file must still be loaded by an entry point or a component stylesheet.

If the project has multiple build targets, check which targets need the same setting. Tests and other builders may have their own style configuration.

## 5. React: React does not compile Sass

React provides component and rendering APIs; React itself does not define a Sass compiler or decide how CSS files are bundled. The project setup determines whether an SCSS import is supported.

In a Vite-based React project, install a Sass implementation and import an SCSS file from the application entry or a component:

~~~sh
npm install --save-dev sass-embedded
~~~

If the platform or project requires the alternative package, use `sass` instead. Keep the chosen package in the project manifest and commit the lockfile so local development and CI use a reproducible compiler dependency.

A global entry can import a shared stylesheet:

~~~jsx
import './styles/main.scss';

createRoot(document.getElementById('root')).render(<App />);
~~~

A component can import its stylesheet from its module:

~~~jsx
import './ProductCard.scss';

export function ProductCard({ name }) {
  return <article className="product-card">{name}</article>;
}
~~~

A plain Sass import does not automatically scope its selectors to the React component. `.product-card` is still a CSS class selector in the compiled stylesheet. Use naming conventions or a CSS Modules feature provided by the build tool if local class-name scoping is desired.

Other React setups may need a loader, plugin, or framework-specific configuration. Follow the toolchain's Sass documentation rather than assuming that every React project accepts `.scss` imports.

## 6. Vite: install the preprocessor, not a Sass plugin

Vite recognizes `.scss` and `.sass` files and sends them to the corresponding preprocessor. Install the preprocessor in the project:

~~~sh
npm install --save-dev sass-embedded
~~~

Or install `sass` where that is the project's appropriate implementation. Vite's built-in support does not mean the Sass package is included automatically.

Import styles from JavaScript or TypeScript:

~~~js
import './styles/main.scss';
~~~

For a React component, importing an SCSS file works the same way when the Vite project has Sass installed. Vite handles style transformation, development updates, and CSS assets in the production build.

## 7. Configure Vite preprocessor options

Most projects can use Sass with no Vite configuration. If you need Sass-specific options, pass them through `css.preprocessorOptions`:

~~~js
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        // Add Sass-specific compiler options here.
      },
    },
  },
});
~~~

Option names and support belong to the Sass implementation and Vite version in use. Consult the installed versions' documentation before adding configuration.

Vite also offers an `additionalData` option that prepends Sass source to each processed file. It can be useful for declarations that produce no CSS, such as variables and mixins:

~~~js
export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: '@use "src/styles/tools" as *;',
      },
    },
  },
});
~~~

Use this carefully. If the injected source emits CSS, that CSS may be repeated for every processed stylesheet. Explicit `@use` statements make dependencies visible and are often easier to maintain.

## 8. Sass modules and CSS Modules are different

Sass modules organize compile-time Sass members. A file such as `_tokens.scss` can expose variables, functions, or mixins through `@use`:

~~~scss
@use "../styles/tokens" as tokens;

.button {
  background: tokens.$brand;
}
~~~

CSS Modules are a build-tool feature that transforms CSS class names and returns a mapping to JavaScript or JSX. Vite recognizes the `.module.scss` naming pattern when Sass is configured:

~~~jsx
import styles from './Button.module.scss';

export function Button() {
  return <button className={styles.button}>Save</button>;
}
~~~

~~~scss
.button {
  padding: 0.6rem 1rem;
}
~~~

The Sass module system does not rename `.button`; the CSS Modules pipeline does. Likewise, CSS Modules do not replace Sass's `@use` system. They solve different problems and can be used together.

## 9. Share tokens without duplicating CSS

Put reusable Sass-only definitions in partials or modules:

~~~scss
// _tokens.scss
$brand: #2457d6;
$surface: #f4f6fb;
$text: #202534;
~~~

Load only the members a stylesheet needs:

~~~scss
@use "tokens" as tokens;

.notice {
  color: tokens.$text;
  background: tokens.$surface;
}
~~~

A Sass module that contains only variables, functions, and mixins usually produces no CSS by itself. A module that contains selectors or other CSS-producing rules emits CSS when loaded. If many component styles each load a CSS-producing module, repeated output can result. Keep foundational modules CSS-free where practical, and load global base styles once from the global entry point.

## 10. Asset paths and imports

An SCSS file can refer to assets, but the final URL must still resolve after the build tool processes and emits files. Prefer paths that are valid from the source file and verify them in the generated application. Bundlers may rewrite asset URLs; a standalone Sass CLI build may have different asset-copying responsibilities.

Use Sass `@use` for Sass modules. Use JavaScript imports or the framework's documented global-style configuration to connect a stylesheet to the application. A Sass module being resolvable does not automatically make it a global browser stylesheet.

## 11. Development, tests, and production

Use the framework's normal development command so that Sass compilation is part of the application pipeline. Use the project build command for production output. Do not deploy the `.scss` source expecting browsers to compile it.

For Angular, check the relevant build target's `styles`, component-style settings, and preprocessor options. For Vite, check the installed Sass package, stylesheet imports, and Vite configuration. For any toolchain, confirm the production build includes the emitted CSS and referenced assets.

Tests can use a different builder or environment from the application. If a test target needs global styles, configure that target too. A page that works in the dev server but lacks styles in tests or production often indicates that different targets are using different style inputs or options.

## 12. Common mistakes

### Installing Sass globally but not in the project

A developer's global executable may hide a missing project dependency. Add the compiler to the project and verify builds from a clean install.

### Expecting React to understand SCSS by itself

React does not own Sass preprocessing. Confirm that the project's framework or bundler processes the imported file.

### Treating a component SCSS file as automatically scoped

Sass compiles syntax; it does not know React component boundaries. Angular scoping is handled by Angular's component style system, and CSS Modules are handled by a build pipeline.

### Adding a Vite-specific Sass plugin unnecessarily

Vite has built-in preprocessor integration. Install Sass itself first; add plugins only for a requirement that built-in support does not meet.

### Importing CSS-producing Sass modules everywhere

A shared module with selectors can emit repeated CSS at each use site. Keep tokens and mixins separate from global base styles.

### Configuring only the development target

A dev server can succeed while a test or production target uses different options. Keep the relevant build configurations aligned.

### Running the Sass CLI and bundler on the same entry

Two compilers can produce conflicting or duplicated assets. Let one pipeline own each entry point.

### Using an untracked compiler version

Declare the Sass package in the manifest and commit the lockfile. Avoid relying on an unpinned global Sass executable in CI.

## 13. Integration checklist

Before considering the integration complete, confirm:

1. The project has a Sass implementation installed and recorded in its manifest.
2. Global styles are connected through the application entry or framework configuration.
3. Component styles follow the framework's established file and metadata convention.
4. Shared Sass modules resolve through explicit relative paths or a documented include/load path.
5. Sass-only modules do not unexpectedly emit repeated global CSS.
6. Development, test, and production targets include the intended styles.
7. Asset URLs work in the built application.
8. Exactly one pipeline owns each Sass entry point.
9. The production output contains compiled CSS; no browser is expected to compile SCSS.

## Quick reference

| Need | Angular | React with Vite |
| --- | --- | --- |
| Choose SCSS on project creation | `ng new app --style=scss` | Install Sass in the Vite project |
| Global stylesheet | Workspace `styles` entry, commonly `src/styles.scss` | Import from the application entry |
| Component stylesheet | Component metadata references the SCSS file | Import from the component module |
| Sass include/config | `stylePreprocessorOptions` and `includePaths` | `css.preprocessorOptions.scss` when needed |
| Component class isolation | Angular view encapsulation setting | CSS Modules via `.module.scss` when supported |
| Build output | Angular CLI build | Vite production build |

## Official references

- [Angular workspace configuration: styles and Sass preprocessor options](https://angular.dev/reference/configs/workspace-config)
- [Angular application generation options](https://angular.dev/cli/generate/application)
- [Vite CSS features and preprocessors](https://vite.dev/guide/features)
- [Vite shared configuration options](https://vite.dev/config/shared-options)
- [React common elements: applying CSS styles](https://react.dev/reference/react-dom/components/common)
- [React: add React to an existing project](https://react.dev/learn/add-react-to-an-existing-project)
- [Install Sass](https://sass-lang.com/install/)

## Next lesson

Continue with [Authoring a Sass library](046-authoring-a-sass-library.md).
