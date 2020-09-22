---
layout: doc
title:  "Getting Started with Coveo Turbo"
desc: Get started with the Coveo Turbo CLI-based tool.
---

Coveo Turbo is a CLI-based tool designed to speed up development by standardizing and automating common tasks to quick start the creation of a reusable custom component or implementation project.

This article explains how to get started using Coveo Turbo to create, modify, and publish a component.

> To learn how to test an existing component without cloning an existing project, see [Testing Coveo Turbo Components]({{ site.baseurl }}/docs/Testing-Components).

> For the full reference of the Coveo Turbo CLI, see [CLI Reference]({{ site.baseurl }}/docs/CLI-Reference).

## Install Coveo Turbo

To install Coveo Turbo, you need a Node environment with npm. You can download and install both on the [node.js website](https://nodejs.org/en/).

> Coveo Turbo requires at least version 12 of Node.js. While using previous versions may work for certain commands, it's possible that other commands will not work properly.

Once you have installed Node.js, you can run the following command to install and prepare the Coveo Turbo CLI:

```
npm install --save-dev @coveops/cli
```

## Creating a New Project

Once the Coveo Turbo CLI is installed, you can create a new project using the following command:

```
npx @coveops/cli create:project TestComponent --create-component --with-styles --with-sandbox
```
> Remember to change `TestComponent` with the name of your component.

> If you want to use JavaScript instead of TypeScript, you should add `--template vanilla` to the command. Similarly, to use pure CSS instead of Sass, add `--styles-template vanilla` to the command.

This command does the following things:
- Create a component class with the appropriate template.
- Create associated stylesheets to help style your component.
- Create a sandbox environment that contains a basic Coveo search page, where you can test your component.
- Create and fill a readme.md file with basic usage information, to help you have basic usage documentation available for each component.

> The sandbox environment is not automatically connected to a Coveo Cloud organization; this can be added when serving the component.

## Modifying a Component

Once you have created your component, you can start modifying the created template to develop your functionality.

In the generated `src` folder, you should be able to find a `TestComponent.ts` (where `TestComponent` is replaced with the name you gave your component). This file handles the logic of your component, and is written in Typescript.

> To learn how to create custom TypeScript components, see [Advanced Integration With Custom TypeScript Component (Implementation)](https://docs.coveo.com/en/355/).

> If you've created a JavaScript component instead, see [Implementing a Custom Component in JavaScript](https://docs.coveo.com/en/305/).

You can also change the styling of the component by changing the `src\stylesheets\TestComponent.scss` [Sass](https://sass-lang.com/) file (where `TestComponent` is replaced with the name you gave your component).

## Build the Component

Once you have modified your component and are ready to test it, you need to run the build command, so the code can be built.

To do so, run the following command, replacing `TestComponent` with the name of your component:

```
coveops build TestComponent
```

> For a full list of all build options, see [CLI Reference - Build]({{ site.baseurl }}/docs/CLI-Reference#build).

> You should consider enabling watch on your builds, so you don't have to manually run the build command every time you make changes. The command would look like this:
> ```
> coveops build TestComponent --watch
> ```
>
> Note that the watch option does not serve the component.

## Serving the Component

During development, you're likely to want to see your component in action. By serving your code, you are deploying it locally, allowing you to test your component in a working Coveo search page.

> You are encouraged to use your own Coveo organization to test your components, as some components can expect a certain item structure to work properly. However, if you do not have one, you can use a sample Coveo Cloud organization by omitting an `org-id` and `token` value.

To serve your component, enter the following command:

```
coveops serve org-id yourorgid token "your-token"
```

> Remember to change `yourorgid` with the id of your organization, and `your-token` with a valid search token or API key. For more information on how to get these values, see [Managing Coveo Organization Settings and Limits](https://docs.coveo.com/en/1562/cloud-v2-administrators/managing-coveo-organization-settings-and-limits#organization-tab) and [Adding and Managing API Keys](https://docs.coveo.com/en/1718/cloud-v2-administrators/adding-and-managing-api-keys).

The page is then served and available at `localhost:8080`.