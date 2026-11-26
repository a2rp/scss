# 003. Installing Dart Sass and Version Basics — Q&A

## Implementations and installation choices

### 1. What is Dart Sass?

Dart Sass is the primary implementation of the Sass language. It can be installed as a standalone command-line tool or used through JavaScript packages.

### 2. Is Sass the same thing as Dart Sass?

Sass is the language and ecosystem. Dart Sass is the primary implementation that compiles Sass. The npm package named sass distributes Dart Sass in a JavaScript form.

### 3. Which implementation should a new project use?

Use Dart Sass. For an npm project, the package named sass is a straightforward project-local default.

### 4. What is node-sass?

node-sass was a Node.js binding for LibSass, a separate C/C++ Sass implementation. It is an older, end-of-life choice and should not be selected for a new project.

### 5. Are node-sass and sass interchangeable package names?

No. They refer to different implementations and packages. New projects should install sass or a supported sass-embedded integration, not node-sass.

### 6. What is the difference between the sass and sass-embedded npm packages?

The sass package is a pure JavaScript distribution of Dart Sass and works across platforms supported by Node.js. sass-embedded communicates with the Dart Sass executable through the Embedded Sass protocol and can be faster, but supports fewer platforms. Choose the implementation supported by the project’s build tool and target environment.

### 7. Should a beginner install sass-embedded?

Usually not. Start with sass unless the build tool recommends sass-embedded or compilation performance is an identified concern.

### 8. What does a standalone Sass installation provide?

It provides the Sass executable without requiring a Node.js project. Add the extracted executable directory to PATH if it should be available from any terminal location.

### 9. Should Sass be installed globally or in the project?

For project work, install Sass locally as a development dependency. A global installation is convenient for occasional command-line use, but its version may differ from the project’s version.

### 10. Why is Sass normally a development dependency?

Sass compiles source stylesheets during development or the build process. The browser receives the generated CSS, so a browser does not need the Sass compiler at runtime.

## Installing and checking Sass

### 11. How do I install Sass in an npm project?

Run this from the project root:

~~~sh
npm install -D sass
~~~

This adds Sass to the project’s development dependencies.

### 12. How can I check the Sass version used by the project?

After installing dependencies, run:

~~~sh
npx sass --version
npm ls sass
~~~

The first command runs the Sass executable available to the project. The second shows the installed dependency in the npm tree.

### 13. Why can sass --version differ from npx sass --version?

sass may resolve to a standalone or globally installed executable. npx sass run from the project uses the local project dependency when it is installed. For work in a repository, check the local version.

### 14. What do node --version and npm --version tell me?

They show the active Node.js and npm versions. Sass’s npm package requires a supported Node.js environment, while a standalone Sass executable does not depend on a project’s npm installation.

### 15. What files should an npm project keep for repeatable Sass setup?

Keep package.json and package-lock.json. The first records the declared dependency range and scripts; the second records the exact resolved dependency tree for npm.

### 16. Should node_modules be committed?

No. It is installed from the dependency files and is normally excluded from version control.

### 17. What is the difference between npm install and npm ci?

npm install installs dependencies and can update the lockfile while resolving allowed versions. npm ci installs the versions recorded in package-lock.json and expects the lockfile to match package.json. It is useful for a clean, repeatable install.

## Version numbers and updates

### 18. What does a version such as 1.108.2 represent?

It has major, minor, and patch parts. The major number can indicate breaking changes, the minor number generally adds compatible features, and the patch number generally contains compatible fixes.

### 19. What does an exact version in package.json mean?

A selector such as 1.108.2 requests that specific version. It does not permit package.json to select another Sass release by itself.

### 20. What does the tilde selector mean?

A selector such as ~1.108.2 generally allows patch updates within the 1.108 line, without moving to the next minor version.

### 21. What does the caret selector mean for Sass 1.x?

For a stable version such as ^1.108.2, the range allows compatible versions below 2.0.0. The exact version installed is still recorded in package-lock.json.

### 22. Does package.json show the exact version installed?

Not always. It may show a range. Inspect package-lock.json or run npm ls sass to see the version currently installed in the project.

### 23. How can I update Sass within the allowed package.json range?

Run:

~~~sh
npm update sass
~~~

Review the package and lockfile changes, then run the project’s build command.

### 24. How can I request a particular Sass release?

Specify the version in the install command, for example:

~~~sh
npm install -D sass@1.108.2
~~~

That version is an example. Check current project requirements and release information before choosing a version.

### 25. Is a newer Sass version always a safe automatic upgrade?

No dependency update should be accepted without review. Read relevant release notes, inspect the lockfile changes, compile the project, and resolve any new warnings or errors.

## Compatibility and troubleshooting

### 26. What is a Sass deprecation warning?

It indicates that a language feature or API is being phased out, or that a tool uses an older integration. It may not stop the current build, but it gives maintainers time to migrate.

### 27. How do I find where a deprecation warning comes from?

Read the warning’s file, line, and message. If it points to project SCSS, update that source. If it comes from a dependency or build tool, check that tool’s supported Sass version and integration.

### 28. What should I do if a tool reports the legacy JavaScript API warning?

Check whether the build tool has an update or configuration option for the modern Sass API. The warning can come from the tool that calls Sass, rather than from the stylesheet itself.

### 29. Sass says the command is not recognized. What should I check?

If Sass is installed locally, run the command from the project root with npx sass. For a standalone or global install, confirm the executable is installed and its directory is on PATH.

### 30. Sass disappears after cloning the repository. Why?

The installed dependencies are not part of the clone. From the project root, install them from the lockfile with npm ci, then check the local Sass version.

### 31. The lockfile changed after installation. Is that necessarily an error?

Not necessarily. The package manager may have resolved a newer version that is allowed by package.json. Inspect the changes and keep them only when the update is intentional.

### 32. Where should I look after an upgrade causes compilation errors?

Start with the first Sass error and the named source file and line. Then read the relevant release or migration notes, fix the source or tool configuration, and rebuild.

## Quick review

- **Recommended new-project implementation:** Dart Sass.
- **Convenient npm package:** sass.
- **Potentially faster embedded alternative:** sass-embedded, when the tool and platform support it.
- **Avoid for new work:** node-sass / LibSass.
- **Check local version:** npx sass --version.
- **Repeatable npm install:** commit package.json and package-lock.json; use npm ci for a clean lockfile install.
