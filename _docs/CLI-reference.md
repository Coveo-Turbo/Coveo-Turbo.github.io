---
layout: doc
desc: The reference documentation for the CLI tool
---

# Coveo Turbo CLI Reference

<div class="content-section" markdown="1">
## Table of content:
- [Installation](#installation)
- [Usage](#usage)
    - [Build](#build)
    - [Serve](#serve)
    - [Create a Project](#create-a-project)
    - [Create a Component](#create-a-component)
    - [Create a Stylesheet](#create-a-stylesheet)
    - [Create a Sandbox](#create-a-sandbox)
    - [Deploy a Sandbox](#deploy-a-sandbox-to-the-coveo-platform)
    - [Create Locales](#create-locales)
    - [Create a Translation](#create-a-translation)
    - [Update a Translation](#update-a-translation)
    - [Create a Readme](#create-a-readme-file)
    - [Create a Docker Environment](#create-a-docker-environment)
    - [Create a Query Pipeline](#create-a-query-pipeline)
</div>

<div class="content-section" markdown="1">
## Installation

To install the CLI, use the following command:

`npm install --save-dev @coveops/cli`
</div>

<div class="content-section" markdown="1">
## Usage

Replace the `COMMAND` with the appropriate one from the table below with its corresponding arguments and options.

**Installed globally:**

`coveops COMMAND`

**Installed as a dev dependency (recommended to track versioning):**

`./node_modules/.bin/coveops COMMAND`

Alternatively on Windows: `.\node_modules\.bin\coveops CountedTabs COMMAND`

**Executed via npx:**

`npx @coveops/cli COMMAND`
</div>

<div class="content-section" markdown="1">
## Build

This command uses a standard Webpack configuration to build and bundle the project for distribution.

You can also add the command to your `package.json` scripts to continue using familiar hooks like `npm run build`

| Argument | Command Type | Type | Default | Required | Comments |
| -------- | ------------ | ---- | ------- | -------- | -------- |
| name | argument | string | none | yes | The name of your component to be used as the library name and added to the browser global namespace. Note that `Coveo` will be prepended at build-time. |
| template | option | string | `typescript` | no | The template of the component to generate. The available options are: [`typescript`, `vanilla`] |
| path | option | string | `src` | no | The path where the source code is generated. |
| destination | option | string | `dist` | no | The path where the distributable code is generated. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |
| styles-path | option | string | `src/stylesheets` | no | The path where the stylesheets are located. |
| styles-type | option | string | `sass` | no | The stylesheet language that is used in the project. The available options are: [`sass`, `vanilla`] |
| styles-destination | option | string | `dist/css` | no | The path where the distributable code is generated. |
| dry | option | boolean | none | no | Whether to perform a dry run build and output the generated webpack configuration to the console. |
| disable-swapvar | option | boolean | none | no | Whether to remove the Webpack loader that injects the `SwapVar` utility to the root `index` file before building. By default, the build command injects `SwapVar` to add components to the root of the `Coveo` object at runtime. |
| watch | option | boolean | none | no | Whether to watch for changes and build after the `watch-timeout`. |
| watch-timeout | option | number | 1000 | no | The amount of time (in milliseconds) to watch for changes. Increase this value if you change multiple files at a time. |


#### Examples:

> 
> 
> Basic use case:
> 
> ```bash
> ./node_modules/.bin/coveops build TestComponent
>```
> 
> To watch:
> 
> ```bash
> ./node_modules/.bin/coveops build TestComponent --watch
> ```
> 
> To use vanilla JavaScript:
> 
> ```bash
> ./node_modules/.bin/coveops build TestComponent --template vanilla
> ```
>
> To specify an alternative directory containing a index.scss located at `src/stylesheets`:
>
> ```bash
> ./node_modules/.bin/coveops build TestComponent --styles-path src/stylesheets
> ```

</div>

<div class="content-section" markdown="1">
#### SwapVar

Coveo injects custom components into its root object by using a custom utility called `SwapVar`. With Coveo Turbo, this utility is injected into the root index of the code during build time, so no additional effort or integration is necessary in a project.

Thanks to the `SwapVar` utility, the exported component is referenced in implementation JavaScript code as such:

```javascript
Coveo.TestComponent
```

Without the `SwapVar` utility, achieved by using the `disable-swapvar` option in the build, the exported component library is referenced in implementation JavaScript code such:

```javascript
CoveoTestComponent
```
</div>

<div class="content-section" markdown="1">
### Serve

This command starts a Node server designed to:

- Serve the compiled distributable as a static resource that can be loaded onto the page.
- Serve installed components in the `node_modules` folder under the @coveops scope as static resources that can be loaded onto the page.
- Serve all HTML pages in the sandbox folder.
- Pass parameters from the environment or CLI arguments to the running instance to be used in the application via the `demoConfig` variable declared in `/config.js` when the server starts.

> If no org-id or token are used, then a sample Coveo organization with some already indexed content is used.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| port | option | number | 8080 | no | The port number where the site is generated. |
| path | option | string | `sandbox` | no | The path to the dedicated sandbox folder used for testing. |
| org-id | option | string | none | no | The id of the Coveo Cloud organization to use. |
| token | option | string | none | no | The token used to authenticate to the organization. |
| rest-uri | option | string | `https://platform.cloud.coveo.com/rest/search` | no | The uri used by the Coveo JavaScript Search Framework to access the REST API. |
| search-hub | option | string | none | no | The search hub used for the requests. To understand what the purpose of a search hub is, see [Setting the Search Hub](https://docs.coveo.com/en/3107). |
| search-url | option | string | none | no | The URL of a search page for the standalone search box. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Examples:
>
> Basic use case:
>
> ```bash
> ./node_modules/.bin/coveops serve --org-id <ID HERE> --token "<TOKEN HERE>"
> ```
>
> To use a different port:
>
> ```bash
> ./node_modules/.bin/coveops serve --port 3000 --org-id <ID HERE> --token "<TOKEN HERE>"
> ```

Some of the arguments have a corresponding environment variable that can also be used by the server:

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

</div>

<div class="content-section" markdown="1">
#### Compiled Component

The compiled component has its resources served at the following endpoints.

> In some cases, only JavaScript or CSS may be available.

- `./component.js`
- `./component.css`

</div>

<div class="content-section" markdown="1">
#### Installed Components

The installed components have their resources served at the following endpoints.

> In some cases, only JavaScript or CSS may be available.

- `components/<name>.js`
- `components/<name>.css`

The `<name>` represents the name of the component within the `@coveops` scope. 

For example, importing a component installed from `@coveops/test-component` will have the following assets served:

- `components/test-component.js`
- `components/test-component.css`

> Alternatively, these installed components can be bundled into the library via the corresponding `index` file in the `src` before being built.
</div>

<div class="content-section" markdown="1">
#### Injected Parameters

The serve command generates a JavaScript snippet that is injected into the sandbox page by importing the `/config.js` file, which contains some of the information that was passed through environment variables or CLI arguments in a global `demoConfig` variable. This allows a sandbox to remain decoupled from the test settings to allow portability between environments where component-level dependencies permit.

The following information is passed in the `demoConfig` object and is available to use within the application at runtime:

| Property | Argument | Environment Variable | Default |
| --- | --- | --- | --- |
| orgId | org-id | COVEO_ORG_ID | `"null"` |
| token | token | COVEO_TOKEN | `"null"` |
| restUri | rest-uri | COVEO_REST_URI | `"https://platform.cloud.coveo.com/rest/search"` |
| searchHub | search-hub | COVEO_SEARCH_HUB | `"null"` |
| searchUrl | search-url | COVEO_SEARCH_URL | `localhost:8080/` |

</div>

<div class="content-section" markdown="1">
### Create a Project

This command adds the necessary files to kick-start a project to create a shareable component and optionally the component itself.

> When ran using npx, the `create:project` utility initializes a `package.json` and installs itself as a dev dependency for you.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component. |
| template | option | string | `typescript` | no | The template of component to generate. The available options are: [`typescript`, `vanilla`] |
| create-component | option | boolean | `false` | no | Whether to create the component using the same name as the project |
| component-path | option | string | `src` | no | The path where the source code of the component is generated. |
| with-styles | option | boolean | `false` | no | Whether to create the stylesheet alongside the component. This option requires the `create-component` flag. |
| styles-path | option | string | `src/stylesheets` | no | The path where the source code of the stylesheet is generated. |
| styles-template | option | string | `sass` | no | The template of component to generate. The available options are: [`sass`, `vanilla`] |
| with-sandbox | option | boolean | `false` | no | Whether to create a sandbox in which to test your component. |
| sandbox-path | option | string | `sandbox` | no | The path where the sandbox is generated. |
| description | option | string | none | no | The description of the component. This updates the description on the README, as well as set the description field in the `package.json` file. |
| package-name | option | string | none | no | The name of the package that houses the component. By default, the param-case version of the `name` will be added under the `@coveops` scope. For example, setting the name as `TestComponent` yields `@coveops/test-component`. This option is meant to override the default behavior. |
| with-docker | option | boolean | none | no | Adds a `docker-compose.yml` file containing a NodeJS v12 environment. For more details, see [`Create a Docker Environment`](#create-a-docker-environment) section. |
| setup-locales | option | array | none | no | **[CLI Version 0.12.0+]** Adds the specified locales used for translations and sets up localization in the project. |
| locale-type | option | string | `json` | no | **[CLI Version 0.12.0+]** The filetype to use when managing translations for the locale. The available options are: [`json`, `yaml`, `js`] |
| default-locale | option | string | `en` | no | **[CLI Version 0.12.0+]** The default locale to use as the base for the dictionary. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during at run-time. |

> Example usage:
>
> To quick-start a new project:
>
> ```bash
> npx @coveops/cli create:project TestComponent --create-component --with-styles --with-sandbox
> ```
>
> To use in a project where `@coveops/cli` is installed:
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent
> ```
>
> To use vanilla JavaScript:
>
> Note: Building a component requires the same template option to be passed to the build command.
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent --template vanilla
> ```
>
> To create a component at the same time as a new project:
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent --create-component
> ```
>
> To create a component with styling at the same time as a new project:
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent --create-component --with-styles
> ```
>
> To create a sandbox at the same time:
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent --with-sandbox
> ```
>
> To setup locales at the same time:
>
> ```bash
> ./node_modules/.bin/coveops create:project TestComponent --setup-locales fr es-es
> ```

</div>

<div class="content-section" markdown="1">
### Create a Component

This command creates a blank component to be used to canvas for your needs. It is currently available as `vanilla` and `typescript` types.

- A Typescript template component creates a class and associate the build tools to use the TypeScript compiler.

- A Vanilla template component creates a simple module bridge to permit any JavaScript code to be bundled. It associates the build tools to use the JavaScript bundler.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component. |
| template | option | string | `typescript` | no | The template of component to generate. The available options are: [`typescript`, `vanilla`] |
| path | option | string | `src` | no | The path where the source code is generated. |
| init-strategy | option | string | `lazy` | no | The initialization strategy to use when initializing the component. The available options are: [`lazy`, `component`, `lazy-dependent`] |
| with-styles | option | boolean | `false` | no | Whether to create the stylesheet alongside the component. |
| styles-path | option | string | `src/stylesheets` | no | The path where the source code of the stylesheet is generated. |
| styles-template | option | string | `sass` | no | The template of component to generate. The available options are: [`sass`, `vanilla`] |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:component TestComponent
> ```
>
> To use vanilla Javascript:
>
> Note: Building a component requires the same template option to be passed to the build command.
>
> ```bash
> ./node_modules/.bin/coveops create:component TestComponent --template vanilla
> ```
>
> To create a component with styling at the same time as creating a new project:
>
> ```bash
> ./node_modules/.bin/coveops create:component TestComponent --with-styles
> ```

</div>

<div class="content-section" markdown="1">
### Create a Stylesheet

This command creates a blank stylesheet to be used to canvas for your needs. It is currently available as `sass` and `vanilla` types.

- A Sass template stylesheet creates a file per class and bundles it in a shared index.

- **[Experimental]** A Vanilla template will create a file per class and bundle it in a shared index.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of your component. |
| template | option | string | `sass` | no | The template of component to generate. The available options are: [`sass`, `vanilla`] |
| path | option | string | `src/stylesheets` | no | The path where the source code is generated. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:stylesheet TestComponent
> ```
>
> To use vanilla CSS:
>
> Note: this feature isn't fully supported and may cause issues. For now, it is recommended to use the Sass template.
>
> ```bash
> ./node_modules/.bin/coveops create:stylesheet TestComponent --template vanilla
> ```

</div>

<div class="content-section" markdown="1">
#### Component Initialization

The Coveo component registration allows for two main strategies to load a component once the scripts and markup are present on the page: [Eager and Lazy](https://docs.coveo.com/en/295).

The `@coveops/turbo-core` library contains useful decorators that make it simple to choose the initialization structure without requiring boilerplate code. All of the strategies fallback to `component` if the `LazyInitialization` class isn't present when importing `Coveo.Lazy.js`

> This feature is currently only available for TypeScript Components.

```bash
./node_modules/.bin/coveops create:component TestComponent --init-strategy component
```
</div>

<div class="content-section" markdown="1">
### Create a Sandbox

This command creates a folder with a generated search page to be used for basic debugging.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | `index` | no | The name of the sandbox page to be generated. The page is available at the path of the local url. |
| path | option | string | `sandbox` | no | The path where the sandbox code is generated. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:sandbox
> ```
>
> Note: once served, the page is available at localhost:<PORT>/index.html
>
> To add another page
>
> ```bash
> ./node_modules/.bin/coveops create:sandbox knowledge
> ```
>
> Note: once served, the page is available at localhost:<PORT>/knowledge.html
>
> To use a different path
>
> Note: changing the path of the sandbox requires using the same path when serving it.
>
> ```bash
> ./node_modules/.bin/coveops create:sandbox --path test
> ```

</div>

<div class="content-section" markdown="1">
### Deploy a Sandbox to the Coveo Platform

This command creates a new page and deploys the specified sandbox and its minified JavaScript and CSS there.

> This feature requires the project to already be built. It only deploys the compiled code.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | `index` | no | The name of the sandbox page to be generated. The page is available at the path of the local URL. |
| page-name | argument | string | `index` | no | The name of the page to be created on the Coveo Platform. The page is available within the Search Pages section of the Coveo Platform. |
| path | option | string | `sandbox` | no | The path where the sandbox code is generated. |
| org-id | option | string | none | yes | The ID of the Coveo Cloud organization. |
| token | option | string | none | yes | The token used to authenticate to the organization. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops deploy
> ```
>
> To deploy a specific sandbox to a specific Search page in Coveo:
>
> ```bash
> ./node_modules/.bin/coveops deploy index page
> ```

By default, creating a sandbox creates an HTML page called `index`. The page in the Coveo Platform can be named arbitrarily and differently from the sandbox.

Some of the arguments have a corresponding environment variable that can be used with the deploy command.

| Argument | Environment Variable |
| --- | --- |
| path | COVEO_SANDBOX_PATH |
| org-id | COVEO_ORG_ID |
| token | COVEO_TOKEN |
| name | COVEO_SANDBOX_NAME |

</div>

<div class="content-section" markdown="1">
### Create Locales

This command creates and scaffolds standardized locale dictionaries for the project.

> After running this command, a build is necessary for changes to take effect.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| locales | argument | string[] | [] | no | An array of the list of locales to create to use for the translation dictionaries. |
| type | option | string | `json` | no | The filetype to use when managing translations for the locale. The following formats are supported: `json|yaml|js` |
| default | option | string | `en` | no | The default locale to use as the base for the dictionary. |
| setup | option | boolean |  | no | **[CLI Version 0.12.0+]** Installs the `@coveops/localization-manager` component and updates the markup of each page to include the necessary code snippets. |
| component-template | option | string | `typescript` | no | **[CLI Version 0.12.0+]** The component template to generate. The available options are: [`typescript`, `vanilla`] |
| component-path | option | string | `src` | no | **[CLI Version 0.12.0+]** The path where the source code of the component is generated. |
| sandbox-path | option | string | `sandbox` | no | **[CLI Version 0.12.0+]** The path where the sandbox is generated. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:locales fr es
> ```
>
> To also set up the `LocalizationManager`:
>
> ```bash
> ./node_modules/.bin/coveops create:locales fr es-es --setup
> ```
>
> To use a different default locale as a base, you can specify it as follows:
>
> ```bash
> ./node_modules/.bin/coveops create:locales en es --default fr
> ```
>
> To use a different filetype to manage the dictionary, you can specify it as follows:
>
> ```bash
> ./node_modules/.bin/coveops create:locales en fr --type yaml
> ```

</div>

<div class="content-section" markdown="1">
### Create a Translation

This command creates all the necessary translation entries or empty placeholders for a given word.

This operation does not replace values that already exist. To update those, use the [`update:translation` command](#update-a-translation).

> For changes to take effect, a build will be necessary after running this command.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| word | argument | string |  | yes | The word or key to be replaced with the corresponding translation value. |
| target | option | string |  | no | The component name (e.g., `DynamicFacet`) or element id to target with the translation. |
| type | option | string | `json` | no | The filetype to use when managing translations for the locale. It should correspond to the filetype that was used to create the locale. The following formats are supported: `json|yaml|js` |
| <locale> | option | string | | no | An option to be generated for each locale created using the `create:locales` command. The value passed to the option is the translation of the word for that given locale. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:translation Word --en Word --fr Mot
> ```
>
> Note: In order to recognize English and French, the `create:locales` command was ran prior, as such.
> 
> Since, by default, English is the default language, only `fr` needed to be specified.
> ```bash 
> ./node_modules/.bin/coveops create:locales fr 
> ```
>
> To target a specific component:
>
> ```bash
> ./node_modules/.bin/coveops create:translation Word --en Word --fr Mot --target DynamicFacet
>```
>
> If you used a different filetype than json when creating the locale, you can specify it as follows:
>
> ```bash
> ./node_modules/.bin/coveops create:translation Word --type yaml --en Word --fr Mot
> ```

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

</div>

<div class="content-section" markdown="1">
### Update a Translation

This command updates the translation entries for a given word, only for the locales specified in the options.

> For the changes to take effect, a build is necessary after running this command.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| word | argument | string |  | yes | The word or key to be replaced with the corresponding translation value. |
| target | option | string |  | no | The component name (e.g., `DynamicFacet`) or element id to target with the translation. |
| <locale> | option | string | | no | An option to be generated for each locale created using the `create:locales` command. The value passed to the option is the translation for the word for that given locale. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops update:translation Word --fr Mot
> ```
>
> Note: In order to recognize English and French, the `create:locales` command was ran prior.
> Since, by default, English is the default language, only `fr` needed to be specified.
>```bash 
>./node_modules/.bin/coveops create:locales fr 
>```
>
> To update a translation on a targeted component:
>
> ```bash
> ./node_modules/.bin/coveops update:translation Word --en Word --fr Mot --target DynamicFacet
> ```
>
> If you used a different filetype than JSON when creating the locale, you can specify it as follows:
>
> ```bash
> ./node_modules/.bin/coveops update:translation Word --type yaml --en Word --fr Mot
> ```

</div>

<div class="content-section" markdown="1">
### Create a README file

This command creates or overwrites an existing README.md file with a standard template that uses contextual information provided to it.

> This command shouldn't be necessary when creating a new project, as this functionality is bundled by default.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | no | The name of the component being shared in this project. |
| description | option | string | none | no | The description of the component. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:readme TestComponent --description "This is a sample description"
> ```

</div>

<div class="content-section" markdown="1">
### Create a Docker Environment

This command creates or overwrites an existing docker-compose.yml file with a basic compose setup that includes a server running NodeJS v12 and uses the bundled Makefile to install, build, and serve the project with the configured environment variables.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:docker
> ```
>
> To get started, run:
>
> ```bash
> docker-compose up -d
> ```
>
> To access a bash shell within the environment:
>
> ```bash
> docker-compose exec server bash
> ```


> To only run the server during the `up` phase, and handle the install and build commands manually, you can remove the `setup` and `build` directives from the `entrypoint` field in the `docker-compose.yml` file.
</div>

<div class="content-section" markdown="1">
### Create a Query Pipeline

**[CLI Version 0.12.0+]**

This command creates a query pipeline in the Coveo organization. This pipeline has a condition set to the search hub specified in the command.

| Argument | Command Type | Type | Default | Required | Comments |
| --- | --- | --- | --- | --- | --- |
| name | argument | string | none | yes | The name of the pipeline to create in the organization. |
| search-hub | option | string | none | no | The searchHub to use when referring to the pipeline. By default, the `search-hub` uses the value of `name`. |
| description | option | string | none | no | Add a description to the pipeline, for reference within the Coveo Platform as the User Note. |
| without-search-hub | option | boolean | none | no | Opt out of the automatic creation of the searchHub condition on this pipeline. |
| org-id | option | string | none | yes | The id of the Coveo organization. |
| token | option | string | none | yes | The token used to authenticate to the organization. |
| verbosity | option | string | none | no | Adjusts the verbosity of error logging during the run-time. |

> Example usage:
>
> ```bash
> ./node_modules/.bin/coveops create:pipeline SearchPage
> ```
>
> To use a different searchHub:
>
> ```bash
> ./node_modules/.bin/coveops create:pipeline SearchPage --search-hub SearchPageSearchHub
> ```
>
> To not use a searchHub:
>
> ```bash
> ./node_modules/.bin/coveops create:pipeline SearchPage --without-search-hub
> ```
</div>
