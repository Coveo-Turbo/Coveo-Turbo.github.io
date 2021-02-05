---
layout: doc
title:  "Using Coveo Turbo with Coveo for Salesforce"
desc: Use Turbo for local scaffolding and translate your work to Salesforce components.
---

# Using Coveo Turbo with Coveo for Salesforce

Coveo Turbo provides the tooling to build and share components, as well as assemble entire projects.

This article explains how to use Coveo Turbo to build customizations that will be deployed to the Salesforce cloud. All the translation steps are currently manual.

> To learn how to get started with Coveo Turbo, see [Getting Started]({{ site.baseurl }}/docs/Getting-Started).

## Set up a project

Start by creating a new Turbo project - you can use the `--page-layout` option as a shortcut to specify a starter layout.

> For a full list of available page layouts, see [CLI Reference - Create a Page]({{ site.baseurl }}/docs/CLI-reference.html#create-a-page).

### Ease the translation with the Coveo for Salesforce Local Development Kit

If you use any of the Salesforce layouts, then an adapter component will be bundled with the markup.

> If you're using a basic page layout, see [Coveo for Salesforce Local Development Kit](https://github.com/Coveo-Turbo/c4sf-local-development-kit)

### View all content in your local environment

To ease development, you may need to use the `viewAllContent` argument locally so that templates you otherwise won't see are accessible during development.

To enable this in your local environment, add the following piece of Javascript code within the `DOMContentLoaded` event handler found in the `<head>` section of the markup of your page.

```javascript
Coveo.SearchEndpoint.endpoints['default'].options.queryStringArguments.viewAllContent = 'true';
```

> Your local API key will require the View All Content permission to be granted, see [Privilege Reference - View all Content Domain](https://docs.coveo.com/en/1707/manage-an-organization/privilege-reference#view-all-content-domain)

## Customizing Lightning Components

To integrate custom Javascript and CSS that is compiled by Turbo in the `dist` folder into a Lightning component, there are a few main steps that are necessary to get started:

### Creating a Static Resource

Create a new static resource (preferably a zip type) in Salesforce with the compiled assets from Coveo Turbo.

When you use the `build` command, Coveo Turbo compiles your `src` folder and adds the final minified code to the `dist` folder. Here is the relative mapping of files you want to include in the static resource:

| Local Path | Static Resource Path | Description |
| --- | --- |
| dist/index.min.js | index.min.js | The compiled and minified Javascript containing all components created within the project and installed with npm to the project wrapped in a function that the Salesforce library will execute. |
| dist/css/index.min.css | index.min.css | The compiled and minified CSS containing all styles created within the project and installed with npm to the project. |
| dist/images/* | images/* | A directory containing any images that are referenced in the solution. Note that any images referenced in css in the local environment will leverage the server's routing to adjust the local path. Therefore, in an actual Salesforce environment, the compiled CSS file must be moved up a level in order to keep the relative image file mappings in tact. |

The `index.min.js` file in the root of the static resource path must be wrapped in the following code snippet. Copy-paste the code from the `dist/index.min.js` within the code block as shown.

```javascript
window.coveoCustomScripts['default'] = function (promise, component) {
    // contents of dist/index.min.js replace this comment.
}
```

### Wrap the Lightning Coveo component to include Static Resources.

Create a custom lightning component to wrap the out of the box Coveo for Salesforce lightning components and add references to your static resource.

> For more information on creating a custom lightning component to wrap an out of the box component, see [Integrating Coveo Components in a Custom Lightning for your community](https://docs.coveo.com/en/1193/coveo-for-salesforce/integrating-coveo-components-in-a-custom-lightning-component-for-your-community)

> For a reference of available components to wrap, see [Lightning Components](https://docs.coveo.com/en/1209/coveo-for-salesforce/lightning-components)

Within the component file, you will want to include the following code snippet to add the styles. Replace `__RESOURCE_NAME__` with the name of your static resource.

```
<ltng:require styles="{!$Resource.__RESOURCE_NAME__ + '/index.min.css'}"/>
```

Coveo's Aura components include a parameter called `customScripts` which permit additional Javascript to be executed within that context. Replace `__RESOURCE_NAME__` with the name of your static resource.

```
customScripts="{!$Resource.__RESOURCE_NAME__ + '/index.min.js'}"
```

Here's an example of a wrapped AgentPanel to see all the pieces in place. We'll call it SampleAgentPanel.cmp and suppose we added a static resource called SampleResource.

```
<aura:component implements="force:hasRecordId,force:hasSObjectName,flexipage:availableForRecordHome" access="global">
    <aura:attribute name="name" type="String" access="global" />
    <aura:attribute name="searchHub" type="String" access="global" />
    <aura:attribute name="title" type="String" access="global" />
    <aura:attribute name="recordFields" type="String" access="global" />
    <aura:attribute name="openResultsInSubTab" type="Boolean" default="false" access="global" />
    <aura:attribute name="fullSearchComponent" default="" type="String" access="global" />
    <aura:attribute name="debug" type="Boolean" default="false" access="global" />

    <ltng:require styles="{!$Resource.SampleResource + '/index.min.css'}"/>

    <CoveoV2:AgentPanel
        aura:id="AgentInsightPanel"
        recordId="{!v.recordId}"
        sObjectName="{!v.sObjectName}"
        name="{!v.name}"
        searchHub="{!v.searchHub}"
        title="{!v.title}"
        recordFields="{!v.recordFields}"
        openResultsInSubTab="{!v.openResultsInSubTab}"
        fullSearchComponent="{!v.fullSearchComponent}"
        debug="{!v.debug}"       
        customScripts="{!$Resource.SampleResource + '/index.min.js'}"
    />
</aura:component>
```

### Add your markup to a Visual Force Component

When you add the custom lightning component, there are some options which can be customized. The `name` option corresponds to the name of the Visualforce component that will be found and loaded. If no component exists, the default behavior is a component will be created and a pre-made layout will be added to it.

If you have a VisualForce component that you want to modify locally, you will want to copy the contents within the `<apex:component >` tags to within the `<body>` tags and vice-versa. No modifications made in the `<head>` tag locally should be deployed to Salesforce.

With the [Coveo for Salesforce Local Development Kit](https://github.com/Coveo-Turbo/c4sf-local-development-kit) set up, ResultLink, Quickview components and the ResultMenu styling won't break locally. In Salesforce you must always use the SalesforceQuickview, or corresponding SalesforceResultLink, ConsoleResultLink or AdaptiveResultLink. 

> The development kit is not an end-to-end solution and is meant to fill some common translation gaps that can lead to bugs and extra work if not accounted for. 
>
> Do not use it to test core Coveo for Salesforce functionalities outside the Salesforce environment. The kit will add local definitions for the following components so they can be used in markup outside of the Salesforce environment:
> 
> | Salesforce JSUI Component | Local JSUI Component Substituted |
> | --- | --- |
> | SalesforceQuickview | Quickview |
> | SalesforceResultLink | ResultLink |
> | ConsoleResultLink | ResultLink |
> | AdaptiveResultLink | ResultLink |

## Deploying the Salesforce Solution

Once the translations are made in previous steps, the solution will be ready to deploy. You can use whichever tool you like to achieve this. Here's some common approaches to consider.

### Managing the changes directly within the sandbox

If you're handling the Salesforce development directly in Salesforce then you will want to use the Developer Console to edit the Lightning and Visualforce components and then upload a zip of the static resources to a relevantly named static resource entry within the Static Resources section in the Salesforce Setup. 

### Synchronizing local environments using sfdx and Visual Studio Code

If you're using Visual Studio Code as your IDE, Salesforce has an extension pack that will take care of integrating sfdx into your IDE experience. The extension lets you browse and download resources from Salesforce, as well as create and deploy them from context menus.

To integrate Salesforce with Visual Studio Code, see [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode)
