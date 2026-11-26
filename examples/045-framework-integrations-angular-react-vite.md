# Framework integrations: Angular, React, and Vite — Examples

These examples show how a framework or bundler connects Sass source to browser-ready CSS. They are reference snippets, not a complete application repository. Adjust project names and paths to match the toolchain and version already in use.

## 1. Create a new Angular application with SCSS

~~~sh
ng new storefront --style=scss
cd storefront
ng serve
~~~

The Angular CLI creates SCSS files and compiles them through the application build. The browser receives CSS, not the original SCSS.

## 2. Keep Angular global styles in one entry file

Example files:

~~~text
src/
  styles.scss
  app/
    app.component.ts
    app.component.scss
~~~

Add global rules to <code>src/styles.scss</code>:

~~~scss
html {
  font-family: system-ui, sans-serif;
}

body {
  margin: 0;
  color: #202534;
}
~~~

Confirm the global entry appears in the Angular build target's <code>styles</code> option. Angular CLI commonly adds it when the project is generated.

## 3. Add a component stylesheet

Component metadata can refer to a separate SCSS file:

~~~ts
@Component({
  selector: 'app-alert',
  template: '<p class="alert">Saved successfully.</p>',
  styleUrl: './alert.component.scss',
})
export class AlertComponent {}
~~~

~~~scss
.alert {
  border-inline-start: 0.25rem solid #2457d6;
  padding: 1rem;
}
~~~

Angular's component style system controls how component styles are applied. The SCSS compiler only transforms Sass syntax.

## 4. Configure an Angular Sass include path

In the relevant project's build options in <code>angular.json</code>, add:

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

Retain the project's existing properties; this is a focused fragment, not a replacement for the full workspace file. Depending on project configuration, the current builder's workspace structure may differ.

## 5. Load shared Angular tokens through the include path

Source:

~~~scss
// src/styles/_tokens.scss
$brand: #2457d6;
$text: #202534;
~~~

Component style:

~~~scss
@use "tokens" as tokens;

.alert {
  border-color: tokens.$brand;
  color: tokens.$text;
}
~~~

The include path helps Sass resolve the module. It does not load <code>_tokens.scss</code> as a global stylesheet by itself.

## 6. Use a relative Sass module path in Angular

Without an include path, use a path relative to the importing style:

~~~scss
@use "../styles/tokens" as tokens;

.page-title {
  color: tokens.$text;
}
~~~

Relative module URLs avoid hidden search-path assumptions, especially in small projects.

## 7. Choose SCSS when generating a React project with Vite

Create a React project using the Vite starter and install a Sass implementation:

~~~sh
npm create vite@latest storefront -- --template react
cd storefront
npm install
npm install --save-dev sass-embedded
npm run dev
~~~

If the chosen platform does not support <code>sass-embedded</code>, use the Sass package supported by the project's Vite version instead. Keep the selected dependency in the manifest and lockfile.

## 8. Import a global SCSS file in a Vite React entry

~~~jsx
// src/main.jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.jsx';
import './styles/main.scss';

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
~~~

The import puts the global stylesheet into Vite's CSS pipeline. Selectors in this file are global CSS selectors.

## 9. Add a global Sass reset and base style

~~~scss
// src/styles/main.scss
*,
*::before,
*::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: system-ui, sans-serif;
}
~~~

Import this entry once. Avoid importing a CSS-producing global reset from every component.

## 10. Import a normal SCSS file from a React component

~~~jsx
// src/components/ProductCard.jsx
import './ProductCard.scss';

export function ProductCard({ name, price }) {
  return (
    <article className="product-card">
      <h2>{name}</h2>
      <p>{price}</p>
    </article>
  );
}
~~~

~~~scss
// src/components/ProductCard.scss
.product-card {
  border: 1px solid #d9deea;
  border-radius: 0.75rem;
  padding: 1rem;
}
~~~

This is ordinary CSS after compilation. The class is not local just because the file is next to a React component.

## 11. Use Sass tokens from a React component stylesheet

~~~scss
// src/styles/_tokens.scss
$brand: #2457d6;
$surface: #f4f6fb;
$text: #202534;
~~~

~~~scss
// src/components/ProductCard.scss
@use "../styles/tokens" as tokens;

.product-card {
  color: tokens.$text;
  background: tokens.$surface;
  border-color: tokens.$brand;
}
~~~

Only Sass members are shared here. The token module emits no selectors, so using it does not create a repeated global block.

## 12. Use CSS Modules with SCSS in Vite

Name the stylesheet with the <code>.module.scss</code> suffix:

~~~jsx
import styles from './Button.module.scss';

export function Button({ children }) {
  return <button className={styles.button}>{children}</button>;
}
~~~

~~~scss
.button {
  border: 0;
  border-radius: 0.5rem;
  padding: 0.65rem 1rem;
  background: #2457d6;
  color: white;
}
~~~

Vite's CSS Modules handling transforms class names and exposes a mapping. Sass variables and <code>@use</code> remain compile-time Sass features.

## 13. Combine CSS Modules and Sass modules

~~~scss
// Button.module.scss
@use "../styles/tokens" as tokens;

.button {
  background: tokens.$brand;
  color: tokens.$on-brand;
}
~~~

The build tool handles the module class mapping; Sass resolves the token module. Each tool has a separate role.

## 14. Add a small Vite Sass preprocessor configuration

Most projects do not need extra configuration. If Sass-specific options are needed, use the preprocessor options field:

~~~js
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        // Add options supported by the installed Sass implementation.
      },
    },
  },
});
~~~

Do not copy an option from another compiler or Vite version without checking the matching documentation.

## 15. Make Sass tools available through Vite additionalData

~~~js
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  css: {
    preprocessorOptions: {
      scss: {
        additionalData: '$space-unit: 0.25rem;',
      },
    },
  },
});
~~~

Example injected module:

~~~scss
$space-unit: 0.25rem;
~~~

Only inject declarations that do not emit CSS. If injected source contains selectors, those rules may be repeated in every processed stylesheet.

## 16. Compare explicit @use and automatic injection

Explicit dependency:

~~~scss
@use "../styles/tools" as tools;

.save-button {
  padding: tools.$space-unit * 4;
  @include tools.focus-ring;
}
~~~

Automatic injection removes the repeated <code>@use</code> line but hides the file's dependency. Prefer the explicit version when discoverability and portability matter more than reducing a small amount of repetition.

## 17. Keep global styles and component styles separate

~~~text
src/
  styles/
    _tokens.scss
    main.scss
  components/
    Header.jsx
    Header.module.scss
~~~

~~~scss
// styles/main.scss
@use "tokens" as tokens;

:root {
  --brand: #{tokens.$brand};
}

body {
  margin: 0;
}
~~~

~~~scss
// components/Header.module.scss
@use "../styles/tokens" as tokens;

.header {
  color: tokens.$text;
}
~~~

The global entry is imported once. The component stylesheet contains component-specific rules and is imported with its module mapping.

## 18. Add a Sass function without emitting global CSS

~~~scss
// src/styles/_colors.scss
@use "sass:map";

$colors: (
  brand: #2457d6,
  text: #202534,
);

@function token-color($name) {
  @return map.get($colors, $name);
}
~~~
~~~scss
// src/components/Badge.scss
@use "../styles/colors" as colors;

.badge {
  background: colors.token-color(brand);
  color: colors.token-color(text);
}
~~~

This illustrates a Sass-only module dependency. For production code, use supported Sass modules and functions appropriate to the project's Sass version.

## 19. Use CSS custom properties for values that change at runtime

~~~scss
:root {
  --surface: #ffffff;
  --text: #202534;
}

[data-theme="dark"] {
  --surface: #171b24;
  --text: #f4f6fb;
}

.card {
  color: var(--text);
  background: var(--surface);
}
~~~

Sass compiles the stylesheet, but CSS custom properties remain in the output so the browser can respond to runtime theme changes.

## 20. Reference an image from SCSS and verify the build output

~~~scss
.hero {
  background-image: url("../assets/hero.webp");
  background-position: center;
  background-size: cover;
}
~~~

Place the image where the selected bundler expects it and inspect the generated CSS and asset path after production build. Standalone Sass CLI does not perform every bundler asset-copy or URL-rewriting task.

## 21. Add a second Angular global stylesheet intentionally

An Angular workspace can include more than one global stylesheet through the build target's <code>styles</code> array:

~~~json
{
  "projects": {
    "storefront": {
      "architect": {
        "build": {
          "options": {
            "styles": [
              "src/styles.scss",
              "src/print.scss"
            ]
          }
        }
      }
    }
  }
}
~~~

Keep the application's existing styles and add entries only when they represent intentional global bundles. Component styles should continue to be referenced by their components.

## 22. Keep style settings aligned across Angular targets

A build target and a test target may have separate configuration. If a Sass include path or global style is needed in both, add the corresponding setting to each relevant target instead of assuming build settings are inherited everywhere.

Use the actual workspace builder structure in the project. Angular CLI configuration evolves, so validate placement against the official workspace configuration documentation for the installed CLI generation.

## 23. Run the same project's development and production commands

~~~sh
npm run dev
npm run build
~~~

For Angular, the equivalent project commands commonly include:

~~~sh
ng serve
ng build
~~~

Use the scripts and CLI commands actually present in the repository. A working dev server is not proof that the production target emits the right styles and assets.

## 24. Inspect compiled output

After a production build, inspect the generated output directory for CSS files and check that the expected selectors and asset references are present. The output directory varies by framework and build configuration.

Do not expect a browser to fetch an <code>.scss</code> file and compile it itself.

## 25. Check the project's installed Sass version

~~~sh
npm ls sass sass-embedded
~~~

This shows which Sass packages are installed in the project dependency tree. If the result differs between local development and CI, check the lockfile and installation command.

## 26. Avoid compiling an entry twice

A bundler-owned entry should usually be compiled by that bundler alone. If a standalone Sass script is required for another artifact, give it a distinct input/output role and prevent both pipelines from emitting the same stylesheet.

## 27. Find a broken Sass module path

Given this structure:

~~~text
src/
  styles/
    _tokens.scss
  components/
    Card.scss
~~~

A relative import from <code>Card.scss</code> is:

~~~scss
@use "../styles/tokens" as tokens;
~~~

If the project instead configures <code>src/styles</code> as an include path, a module URL such as <code>@use "tokens"</code> can resolve there. Use one documented convention consistently.

## 28. Check an Angular component stylesheet reference

~~~ts
@Component({
  selector: 'app-card',
  templateUrl: './card.html',
  styleUrl: './card.scss',
})
export class CardComponent {}
~~~

Confirm that the referenced file exists, the extension is correct, and the project's Angular version supports the metadata convention in use. Generated projects may use a different supported form.

## 29. Check a Vite CSS Module import

~~~text
src/
  components/
    Button.jsx
    Button.module.scss
~~~

~~~jsx
import styles from './Button.module.scss';

<button className={styles.button}>Continue</button>
~~~

The filename suffix and matching relative path matter. A file named <code>Button.scss</code> is ordinarily imported as side-effect CSS, not as a CSS Module mapping.

## 30. A compact reusable integration checklist

~~~text
[ ] Sass implementation is declared in package.json and the lockfile
[ ] Global stylesheet is connected once
[ ] Component stylesheet follows framework conventions
[ ] Shared Sass modules resolve consistently
[ ] Sass modules are not confused with CSS Modules
[ ] CSS-producing globals are not injected repeatedly
[ ] Test and production targets include the intended styles
[ ] Asset URLs work in the built output
[ ] One pipeline owns each Sass entry point
~~~

## Official references

- [Angular workspace configuration: Sass and styles](https://angular.dev/reference/configs/workspace-config)
- [Angular application generation options](https://angular.dev/cli/generate/application)
- [Vite CSS features and preprocessors](https://vite.dev/guide/features)
- [Vite shared configuration options](https://vite.dev/config/shared-options)
- [React common elements: applying CSS styles](https://react.dev/reference/react-dom/components/common)
- [Install Sass](https://sass-lang.com/install/)
