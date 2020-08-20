---
layout: doc
title:  "Getting Started with Coveo Turbo"
desc: Get started with the Coveo Turbo CLI-based tool.
---

Coveo Turbo is a CLI-based tool designed to speed up development by standardizing and automating common tasks to quick start the creation of a reusable custom component or implementation project.

<!-- more -->

## Creating a new Project

Creating a project is straightforward and the amount of automation depends on the usage of the arguments you can pass with the CLI.

> It is discouraged to clone and modify an existing repository to make a new project. This method is not only prone to human error, it will take more time than using the CLI.

With the create project command and the appropriate flags where applicable, you can automate the following startup tasks:

- Installation of core dependencies for the project

- Create a component class templated for the use case with `--create-component`

- Create corresponding stylesheets for a given component class with `--with-styles`

- Create a sandbox with a basic Coveo search page to test in with `--with-sandbox`

- Create and and fill in basic information for a basic Readme

> The create project command does not provide credentials to a Coveo organization to use in the Sandbox at this time, but does work with any existing organization or new test organization.

The create project command also provides some flexibility in the tool stack for development. By default the project and build tools are configured to use TypeScript and SASS, but vanilla JavaScript and CSS can also be used. To use vanilla Javascript instead of TypeScript, add the following to the create project command: `--template vanilla`.

> If during the setup you realize you forgot a piece, don’t fret; the flags above are just shortcuts for the corresponding commands. To customize the default behavior of these flags, use the individual CLI commands instead.