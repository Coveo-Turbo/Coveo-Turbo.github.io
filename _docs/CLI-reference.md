---
layout: doc
title:  "CLI Reference"
desc: The reference documentation for the CLI tool
---
## Installation

`npm install --save-dev @coveops/cli`

## Documentation

- Usage
- Build
- Serve
- Create a Project
- Create a Component
- Create a Stylesheet
- Create a Sandbox
- Deploy a Sandbox
- Create Locales
- Create a Translation
- Update a Translation
- Create a Readme
- Create a Docker Environment

### Usage

Replace the `COMMAND` with the appropriate one from the table below with its corresponding arguments and options.

**Installed globally:**

`coveops COMMAND`

**Installed as a dev dependency (recommended to track versioning):**

`./node_modules/.bin/coveoops COMMAND`

Alternatively on Windows: `.\node_modules\.bin\coveops CountedTabs COMMAND`

**Executed via npx:**

`npx @coveops/cli COMMAND`

### Build

Will use a standard Webpack configuration to build and bundle the project for distribution.

You can also add the command to your `package.json` scripts to continue using familiar hooks like `npm run build`.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component that will be used as the library name and added to the browser's global namespace. Note, `Coveo` will be prepended at build-time |
| template | option | string | `typescript` | no | The template of component to generate [`typescript`, `vanilla`] |
| path | option | string | `src` | no | The path where the source code will be generated |
| destination | option | string | `dist` | no | The path where the distributable code will be generated |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |
| styles-path | option | string | `src/stylesheets` | no | The path where the stylesheets are located |
| styles-type | option | string | `sass` | no | The stylesheet language that is used in the project [`sass`, `vanilla`] |
| styles-destination | option | string | `dist/css` | no | The path where the distributable code will be generated |
| dry | option | boolean | none | no | Will perform a dry run build and output generated webpack configuration to the console |
| disable-swapvar | option | boolean | none | no | Will remove thr Webpack Loader that injects the `SwapVar` utility to the root `index` file before building. By default, the build command will inject `SwapVar` to add components to the root `Coveo` object at runtime |
| watch | option | boolean | none | no | Will watch for changes and build after the `watch-timeout` |
| watch-timeout | option | number | 1000 | no | Will watch for changes and build after the `watch-timeout`. Increase this value if you change multiple files at a time. Value in milliseconds |

Examples:

Basic use case:

```bash
./node_modules/.bin/coveops build TestComponent
```

To watch:

```bash
./node_modules/.bin/coveops build TestComponent --watch
```

To use vanilla JavaScript:

```bash
./node_modules/.bin/coveops build TestComponent --template vanilla
```

To specify an alternative directory containing a index.scss located at `src/stylesheets`:

```bash
./node_modules/.bin/coveops build TestComponent --styles-path src/stylesheets
```

#### SwapVar

Coveo injects custom components into its root object by using a custom utility called `SwapVar`. With Coveo Turbo, it will be injected into the root index of the code during build time, so no additional effort or integration is necessary in a project.

Thanks to the `SwapVar` utility, the exported component will be referenced in implementation JavaScript code as:

```javascript
Coveo.TestComponent
```

Without the `SwapVar` utility, achieved by using the `disable-swapvar` option in the build, the exported component library will be referenced in implementation JavaScript code as:

```javascript
CoveoTestComponent
```

### Serve

Will start a Node server designed to:

- Serve the compiled distributable as a static resource that can be loaded onto the page.
- Serve installed components in the node_modules folder under the @coveops scope as static resources that can be loaded onto the page
- Serve all html pages in the sandbox folder
- Pass parameters from the environment or CLI arguments to the running instance to be used in the application via the `demoConfig` variable declared in `/config.js` when the server starts.

> Note: If no org-id or token are used, then a sample organization will be used.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| port | option | number | 8080 | no | The port number |
| path | option | string | `sandbox` | no | The path to the dedicated sandbox folder for testing |
| org-id | option | string | none | no | The id of the organization |
| token | option | string | none | no | The token used to authenticate to the organization |
| rest-uri | option | string | `https://platform.cloud.coveo.com/rest/search` | no | The uri used by JSUI to access the REST API |
| search-hub | option | string | none | no | The search hub |
| search-url | option | string | none | no | The url of a search page for a standalone searchbox |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Examples:

Basic use case:

```bash
./node_modules/.bin/coveops serve --org-id <ID HERE> --token "<TOKEN HERE>"
```

To use a different port:

```bash
./node_modules/.bin/coveops serve --port 3000 --org-id <ID HERE> --token "<TOKEN HERE>"
```

Some of the arguments have a corresponding environment variable that can also be used by the server.

| Argument | Environment Variable |
| --- | --- |
| port | SERVER_PORT |
| path | COVEO_SANDBOX_PATH |
| org-id | COVEO_ORG_ID |
| token | COVEO_TOKEN |
| rest-uri | COVEO_REST_URI |
| search-hub | COVEO_SEARCH_HUB |
| search-url | COVEO_SEARCH_URL |
| name | COVEO_SANDBOX_NAME |

#### Compiled Component

The compiled component will have its resources served at the following endpoints.

> Note that in some cases only Javascript or CSS may be available.

- `./component.js`
- `./component.css`

#### Installed Components

The installed components has its resources served at the following endpoints.

> Note that in some cases only Javascript or CSS may be available.

- `components/<name>.js`
- `components/<name>.css`

The `<name>` represents the name of the component within the `@coveops` scope. 

Example: Importing a component installed from `@coveops/test-component` will have the following assets served:

- `components/test-component.js`
- `components/test-component.css`

> Note: Alternatively, these installed components can be bundled into the library via the corresponding `index` file in the `src` before being built.

#### Injected Parameters

The serve command generates a JavaScript snippet that is injected into the sandbox page by importing the `/config.js` file that provides some of the information that was passed through environment variables or CLI arguments in a global `demoConfig` variable. This allows a sandbox to remain decoupled from the test settings to allow portability between environments where component-level dependencies permit.

The following information is passed in the `demoConfig` object and is available to use within the application runtime:

| Property | Argument | Environment Variable | Default |
| --- | --- | --- | --- |
| orgId | org-id | COVEO_ORG_ID | `"null"` |
| token | token | COVEO_TOKEN | `"null"` |
| restUri | rest-uri | COVEO_REST_URI | `"https://platform.cloud.coveo.com/rest/search"` |
| searchHub | search-hub | COVEO_SEARCH_HUB | `"null"` |
| searchUrl | search-url | COVEO_SEARCH_URL | `localhost:8080/` |

### Create a Project

Will add the necessary files to kick-start a project to create a shareable component and optionally the component itself.

> When ran using npx, the `create:project` utility initializes a `package.json` and installs itself as a dev dependency for you.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component |
| template | option | string | `typescript` | no | The template of component to generate [`typescript`, `vanilla`] |
| create-component | option | boolean | `false` | no | Whether to create the component using the same name as the project |
| component-path | option | string | `src` | no | The path where the source code of the component will be generated |
| with-styles | option | boolean | `false` | no | Whether to create the stylesheet alongside the component. Requires the `create-component` flag. |
| styles-path | option | string | `src/stylesheets` | no | The path where the source code of the stylesheet will be generated |
| styles-template | option | string | `sass` | no | The template of component to generate [`sass`, `vanilla`] |
| with-sandbox | option | boolean | `false` | no | Whether to create a sandbox |
| sandbox-path | option | string | `sandbox` | no | The path where the sandbox will be generated |
| description | option | string | none | no | The description of the component. This will update the description on the README, as well as set the description field in the `package.json` file. |
| package-name | option | string | none | no | The name of the package that will house the component. By default, the param-case version of the `name` will be added under the `@coveops` scope. Example, setting name as `TestComponent` will yield `@coveops/test-component`. This option is meant to override the default behavior. |
| with-docker | option | boolean | none | no | Adds a `docker-compose.yml` file containing a NodeJS v12 environment. See more details in the `Create a Docker Environment` section. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

To quick-start a new project:

```bash
npx @coveops/cli create:project TestComponent --create-component --with-styles --with-sandbox
```

To use in a project where `@coveops/cli` is installed:

```bash
./node_modules/.bin/coveops create:project TestComponent
```

To use vanilla JavaScript:

> Building a component requires the same template option to be passed to the build command.

```bash
./node_modules/.bin/coveops create:project TestComponent --template vanilla
```

To create a component at the same time as a new project:

```bash
./node_modules/.bin/coveops create:project TestComponent --create-component
```

To create a component with styles at the same time as a new project:

```bash
./node_modules/.bin/coveops create:project TestComponent --create-component --with-styles
```

To create a sandbox at the same time:

```bash
./node_modules/.bin/coveops create:project TestComponent --with-sandbox
```

### Create a Component

Will create a blank component to be used to canvas for your needs. It is currently available as `vanilla` and `typescript` types.

- A Typescript template component will create a class and associate the build tools to use the typescript compiler

- A Vanilla template component will create a simple module bridge to permit any JavaScript code to be bundled. It will associate the build tools to use the JavaScript bundler.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component |
| template | option | string | `typescript` | no | The template of component to generate [`typescript`, `vanilla`] |
| path | option | string | `src` | no | The path where the source code will be generated |
| init-strategy | option | string | `lazy` | no | The initialization strategy to use when initializing the component [`lazy`, `component`, `lazy-dependent`] |
| with-styles | option | boolean | `false` | no | Whether to create the stylesheet alongside the component |
| styles-path | option | string | `src/stylesheets` | no | The path where the source code of the stylesheet will be generated |
| styles-template | option | string | `sass` | no | The template of component to generate [`sass`, `vanilla`] |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops create:component TestComponent
```

To use vanilla Javascript:

> Building a component requires the same template option to be passed to the build command.

```bash
./node_modules/.bin/coveops create:component TestComponent --template vanilla
```

To create a component with styles at the same time as a new project:

```bash
./node_modules/.bin/coveops create:component TestComponent --with-styles
```

### Create a Stylesheet

Will create a blank stylesheet to be used to canvas for your needs. It is currently available as `sass` and `vanilla` types.

- A Sass template stylesheet will create a file per class and bundle it in a shared index

- [Experimental] A Vanilla template will create a file per class and bundle it in a shared index

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component |
| template | option | string | `sass` | no | The template of component to generate [`sass`, `vanilla`] |
| path | option | string | `src/stylesheets` | no | The path where the source code will be generated |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops create:stylesheet TestComponent
```

To use vanilla CSS:

> This feature isn't fully supported and may cause issues. For now, it is recommended to use the Sass template.

```bash
./node_modules/.bin/coveops create:stylesheet TestComponent --template vanilla
```

#### Component Initialization

Coveo's component registration allows for two main strategies to load a component once the scripts and markup are present on the page: [Eager and Lazy](https://docs.coveo.com/en/295/javascript-search-framework/lazy-versus-eager-component-loading).

The `@coveops/turbo-core` library contains useful decorators that make it simple to choose the initialization structure without requiring boilerplate code. All of the strategies fallback to `component` if the `LazyInitialization` class isn't present by importing `Coveo.Lazy.js`.

> Note: The feature is currently only available for Typescript Components.

```bash
./node_modules/.bin/coveops create:component TestComponent --init-strategy component
```

### Create a Sandbox

Will create a folder with a generated search page to be used for basic debugging.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | `index` | no | The name of the sandbox page that will be generated. The page will be available as the path of the local url. |
| path | option | string | `sandbox` | no | The path where the sandbox code will be generated |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops create:sandbox
```

> Once served will be available at localhost:<PORT>/index.html

To add another page

```bash
./node_modules/.bin/coveops create:sandbox knowledge
```

> Once served, it will be available at localhost:<PORT>/knowledge.html

To use a different path

> Changing the path of the sandbox requires using the same path when serving it.

```bash
./node_modules/.bin/coveops create:sandbox --path test
```

### Deploy a Sandbox to the Coveo Platform

Will create a new page and deploy the specified sandbox and its minified JavaScript and CSS there.

> Note: The feature requires the project to be built and will only deploy compiled code.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | `index` | no | The name of the sandbox page that will be generated. The page will be available as the path of the local url |
| page-name | argument | string | `index` | no | The name of the page that will be created on the Coveo Platform. The page will be available within the Search Pages section of the Coveo Platform |
| path | option | string | `sandbox` | no | The path where the sandbox code will be generated |
| org-id | option | string | none | yes | The id of the organization |
| token | option | string | none | yes | The token used to authenticate to the organization |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops deploy
```

To deploy a specific sandbox to a specific Search page in Coveo:

```bash
./node_modules/.bin/coveops deploy index page
```

By default, creating a sandbox will create an html page called `index`. The page in the Coveo Platform can be named arbitrarily and differently from the sandbox.

Some of the arguments have a corresponding environment variable that can also be used by the deploy command.

| Argument | Environment Variable |
| --- | --- |
| path | COVEO_SANDBOX_PATH |
| org-id | COVEO_ORG_ID |
| token | COVEO_TOKEN |
| name | COVEO_SANDBOX_NAME |

### Create Locales

Will create and scaffold standardized locale dictionaries for the project.

> Note: A build will be necessary after running this command for changes to take effect.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| locales | argument | string[] | [] | no | The list of locales to create to use for translation dictionaries. |
| type | option | string | `json` | no | The filetype to use when managing translations for the locale. The following formats are supported: `json|yaml|js` |
| default | option | string | `en` | no | The default locale to use as the base for the dictionary. |

Example usage:

```bash
./node_modules/.bin/coveops create:locales fr es
```

To use a different default locale as a base, you can specify it as follows:

```bash
./node_modules/.bin/coveops create:locales en es --default fr
```

To use a different filetype to manage the dictionary, you can specify it as follows:

```bash
./node_modules/.bin/coveops create:locales en fr --type yaml
```

### Create Translation

Will create all the necessary translation entries or empty placeholders for a given word.

This operation will not replace values that already exist. To update, please use the `update:translation` command.

> Note: A build will be necessary after running this command for changes to take effect.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| word | argument | string |  | yes | The word or key that will be replaced with the corresponding translation value. |
| target | option | string |  | no | The component name (e.g., `DynamicFacet`) or element id to target with the translation. |
| type | option | string | `json` | no | The filetype to use when managing translations for the locale. It should correspond to the filetype that was used to create the locale. The following formats are supported: `json|yaml|js` |
| <locale> | option | string | | no | An option will be generated for each locale created using the `create:locales` command. Yhe value passed to the option will be the translation for the word for that given locale |

Example usage:

```bash
./node_modules/.bin/coveops create:translation Word --en Word --fr Mot
```

> Note: In order to recognize English and French, the `create:locales` command was ran prior:
>```bash 
>./node_modules/.bin/coveops create:locales fr 
>```
> Since, by default, English is the default language, only `fr` needed to be specified.

To target a specific component:

```bash
./node_modules/.bin/coveops create:translation Word --en Word --fr Mot --target DynamicFacet
```

If you used a different filetype than json when creating the locale, you can specify it as follows:

```bash
./node_modules/.bin/coveops create:translation Word --type yaml --en Word --fr Mot
```

> To use translations:
> 1. Ensure `@coveops/localization-manager` is installed as a dependency and registered as per instructions on the component.
> 2. Expose the translation file as per the chosen locale for the page. This file will add a variable `LOCALES` to the global namespace which can then be used throughout the process. The `LOCALES` variable will have the additional locales and is upkept with these CLI commands.
>     ```html
>     <script src="locales/fr.js"></script>
>     ```
> 3. Run the `LocalizationManager` before `Coveo` initializes.
>     ```javascript
>     CoveoLocalizationManager(LOCALES);
>     ```

### Update Translation

Will update the translation entries for a given word. 

This operation will only update translations for the locales specified in the options of the command.

> Note: A build will be necessary after running this command for changes to take effect.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| word | argument | string |  | yes | The word or key that will be replaced with the corresponding translation value. |
| target | option | string |  | no | The component name (e.g., `DynamicFacet`) or element id to target with the translation. |
| <locale> | option | string | | no | An option will be generated for each locale created using the `create:locales` command. The value passed to the option will be the translation for the word for that given locale |

Example usage:

```bash
./node_modules/.bin/coveops update:translation Word --fr Mot
```

> Note: In order to recognize English and French, the `create:locales` command was ran prior:
>```bash 
>./node_modules/.bin/coveops create:locales fr 
>```
> Since, by default, English is the default language, only `fr` needed to be specified.

To update a translation on a targeted component:

```bash
./node_modules/.bin/coveops update:translation Word --en Word --fr Mot --target DynamicFacet
```

If you used a different filetype than json when creating the locale, you can specify it as follows:

```bash
./node_modules/.bin/coveops update:translation Word --type yaml --en Word --fr Mot
```

### Create a README file

Will create or overwrite an existing README.md file with a standard template that uses contextual information provided to it.

> Note: This command shouldn't be necessary when creating a new project because the functionality is bundled by default.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | no | The name of the component that is being shared in this project |
| description | option | string | none | no | The description of the component |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops create:readme TestComponent --description "This is a sample description"
```

### Create a Docker Environment

Will create or overwrite an existing docker-compose.yml file with a basic compose setup that includes a server that runs NodeJS v12 and will use the bundled Makefile to install, build, and serve the project with the configured environment variables.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

Example usage:

```bash
./node_modules/.bin/coveops create:docker
```

To get started, run:

```bash
docker-compose up -d
```

To access a bash shell within the environment:

```bash
docker-compose exec server bash
```

> nly run the server during up and handle install and build manually, the `setup` and `build` directives can be removed from the `entrypoint` field in the `docker-compose.yml` file.