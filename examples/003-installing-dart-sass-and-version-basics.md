# 003. Installing Dart Sass and Version Basics — Examples

This guide creates a small npm project that installs Dart Sass locally, verifies the executable and package version, compiles one stylesheet, and repeats the install from the lockfile.

## Before you begin

Install Node.js with npm. From a terminal, check that both commands are available:

~~~sh
node --version
npm --version
~~~

Use a project folder that is not inside another unrelated npm project.

## Example 1: Install Sass in the project

Create a new folder and initialize npm:

~~~sh
mkdir sass-version-demo
cd sass-version-demo
npm init -y
~~~

Install Sass as a development dependency:

~~~sh
npm install -D sass
~~~

This creates or updates package.json and package-lock.json. Sass is a build-time tool for this example, so it belongs in devDependencies.

Check the local compiler and dependency tree:

~~~sh
npx sass --version
npm ls sass
~~~

Both commands should report the Sass version installed in this project. If a standalone sass command on your computer reports a different version, that does not change which local package npx uses from this project.

## Example 2: Compile an SCSS file

Create these folders and files in the project:

~~~text
sass-version-demo/
├── package.json
├── package-lock.json
├── index.html
├── src/
│   └── styles.scss
└── dist/
    └── styles.css
~~~

Create src/styles.scss:

~~~scss
$brand: #1769aa;

.notice {
  padding: 1rem;
  color: #17212b;
  background: #f4f7fb;
  border-left: 0.25rem solid $brand;

  strong {
    color: $brand;
  }
}
~~~

Create index.html:

~~~html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Dart Sass install example</title>
    <link rel="stylesheet" href="./dist/styles.css">
  </head>
  <body>
    <main class="notice">
      <strong>Dart Sass is installed.</strong>
      <p>The browser loads the generated CSS file.</p>
    </main>
  </body>
</html>
~~~

Compile the SCSS source:

~~~sh
npx sass src/styles.scss dist/styles.css
~~~

The compiler reads src/styles.scss and writes dist/styles.css. The browser gets CSS, not the SCSS source. The result will contain rules similar to:

~~~css
.notice {
  padding: 1rem;
  color: #17212b;
  background: #f4f7fb;
  border-left: 0.25rem solid #1769aa;
}
.notice strong {
  color: #1769aa;
}
~~~

Open index.html in a browser. If your browser or local environment does not load the file as expected, serve the folder with a static development server. The stylesheet link must still point to dist/styles.css.

## Example 3: Save repeatable project commands

Add scripts to the scripts object in package.json:

~~~json
{
  "scripts": {
    "build:css": "sass src/styles.scss dist/styles.css",
    "watch:css": "sass --watch src/styles.scss:dist/styles.css"
  }
}
~~~

If package.json already has a scripts object, add these entries without removing existing scripts. Now compile with:

~~~sh
npm run build:css
~~~

Keep a watcher active while editing with:

~~~sh
npm run watch:css
~~~

The command uses the project-local Sass executable installed in node_modules. Team members do not need a separate global Sass version to run the project script.

## Example 4: Recreate the install from the lockfile

Keep package.json and package-lock.json in version control. Do not commit node_modules.

To simulate a clean clone or build environment, remove node_modules and reinstall from the lockfile:

~~~sh
npm ci
npx sass --version
npm run build:css
~~~

npm ci restores the resolved package tree recorded by package-lock.json. The version printed by the local Sass command should match the Sass package resolved for this project.

If package-lock.json does not exist yet, create it with npm install. Once it is committed, use npm ci for clean lockfile-based installs.

## Example 5: Read a package version range

A package.json created by npm install -D sass may contain a version range similar to this:

~~~json
{
  "devDependencies": {
    "sass": "^1.108.2"
  }
}
~~~

This is an example range, not a requirement to use that particular release. With a stable 1.x version, the caret allows compatible releases below 2.0.0. package-lock.json records the exact version resolved by npm.

To inspect the current project’s declared range and installed version, run:

~~~sh
npm pkg get devDependencies.sass
npm ls sass
~~~

The first command reads the selector from package.json. The second reports the package currently installed in the project.

## Example 6: Request and verify a particular release

To intentionally request a particular Sass release, use its version in the install command:

~~~sh
npm install -D sass@1.108.2
~~~

Then review the dependency changes and check the executable:

~~~sh
git diff -- package.json package-lock.json
npx sass --version
npm run build:css
~~~

The number is an example from the version available while this guide was prepared. Choose a version that fits your project and check the current official release information. Commit package.json and package-lock.json together when the update is intentional.

## Example 7: Compare the package and standalone commands

| Install type | Example verification | When it is useful |
| --- | --- | --- |
| Project-local npm package | npx sass --version | A project that records its Sass dependency and should use that version consistently. |
| Standalone Sass executable | sass --version | A system-wide command-line compiler that does not depend on an npm project. |
| Embedded npm package | npm ls sass-embedded | A supported build integration that needs or benefits from the embedded implementation. |

When working inside an npm repository, run the project’s scripts or npx sass so you know which compiler is being used.

## Troubleshooting

### npx sass --version fails

Confirm that the terminal is in the project root and that npm install -D sass completed. Then inspect the installation with npm ls sass.

### sass works globally, but not in the project

A global executable does not install the local project dependency. Run npm install -D sass from the project root.

### The browser page has no styles

Check that the compile command created dist/styles.css and that index.html links to that exact path.

### npm ci fails

Confirm that package.json and package-lock.json are both present and committed. If package.json was edited manually and no longer matches the lockfile, run npm install, review the updated lockfile, and commit the intentional dependency change.

### A Sass deprecation warning appears

Read the warning and identify its source. If it points to your SCSS, follow the migration guidance. If it points to a dependency or build tool, check whether that tool has a compatible release or configuration.

## Try these changes

1. Run npm ls sass before and after npm ci. Compare the reported local version.
2. Change the Sass source and run npm run build:css.
3. Compare npm pkg get devDependencies.sass with npm ls sass.
4. Intentionally change the stylesheet link, reload the page, and locate the failed request in browser developer tools.
5. Review the package and lockfile diff after an intentional Sass version update.
