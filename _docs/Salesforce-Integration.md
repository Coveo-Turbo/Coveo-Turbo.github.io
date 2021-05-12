---
layout: doc
title:  "Using Coveo Turbo with Coveo for Salesforce"
desc: Use Turbo for local scaffolding and translate your work to Salesforce components.
---

# Using Coveo Turbo with Coveo for Salesforce

Coveo Turbo provides the tooling to build and share components, as well as assemble entire projects.

This article explains how to use Coveo Turbo to build customizations that will be deployed to the Salesforce cloud. All the translation steps are currently manual.

> To learn how to get started with Coveo Turbo, see [Getting Started]({{ site.baseurl }}/docs/Getting-Started).

<div class="content-section" markdown="1">
## Table of contents:
- [Setup](#set-up-a-project)
- [Customizing Lightning Components](#customizing-lightning-components)
    - [Creating a Static Resource](#creating-a-static-resource)
    - [Wrapping the Lightning Coveo component to include Static Resources](#wrapping-the-lightning-coveo-component-to-include-static-resources)
    - [Add your markup to a Visualforce Component](#add-your-markup-to-a-visualforce-component)
- [Deploying the Salesforce Solution](#deploying-the-salesforce-solution)
    - [Managing the changes directly within the sandbox](#managing-the-changes-directly-within-the-sandbox)
    - [Synchronizing local environments using sfdx and Visual Studio Code](#synchronizing-local-environments-using-sfdx-and-visual-studio-code)
</div>

<div class="content-section" markdown="1">
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
</div>

<div class="content-section" markdown="1">
## Customizing Lightning Components

To integrate custom Javascript and CSS that is compiled by Turbo in the `dist` folder into a Lightning component, there are a few main steps that are necessary to get started:

### Creating a Static Resource

Create a new static resource (preferably a zip type) in Salesforce with the compiled assets from Coveo Turbo.

When you use the `build` command, Coveo Turbo compiles your `src` folder and adds the final minified code to the `dist` folder. Here is the relative mapping of files you want to include in the static resource:

> The static resource file does not have to have the same name as the output in the `dist` folder, but for simplicity this guide will refer to them on a one-to-one basis.

| Local Path | Static Resource Path | Description |
| --- | --- |
| dist/index.min.js | index.min.js | The compiled and minified Javascript containing all components created within the project and installed with npm to the project wrapped in a function that the Salesforce library will execute. See [Handling Javascript migrations between Turbo and a Static Resource.](#handling-javascript-migrations-between-turbo-and-a-static-resource) |
| dist/css/index.min.css | index.min.css | The compiled and minified CSS containing all styles created within the project and installed with npm to the project. |
| dist/images/* | images/* | A directory containing any images that are referenced in the solution. Note that any images referenced in css in the local environment will leverage the server's routing to adjust the local path. Therefore, in an actual Salesforce environment, the compiled CSS file must be moved up a level in order to keep the relative image file mappings in tact. |

#### Handling CSS migrations between Turbo and a Static Resource

Copy the `index.min.css` from the `dist/css` folder to the static resource folder.

#### Handling Javascript migrations between Turbo and a Static Resource

The `index.min.js` file in the root of the static resource path must be wrapped in the following code snippet. Copy-paste the code from the `dist/index.min.js` within the code block as shown. Replace `'default'` with the name of the visualforce component using this script if necessary.

```javascript
window.coveoCustomScripts['default'] = function (promise, component) {
    // contents of dist/index.min.js replace this comment.
}
```

More concretely, the minified Javascript generated by Coveo Turbo will look similar to the following screenshot - which also replicates the default folder structure.

![Compiled Javascript found in Turbo dist folder](/assets/images/salesforce-integration/js-dist.png){:class="img-responsive"}

In order for this code to work within Salesforce, the block in the dist file will sit within a function call which returns a promise.

![Compiled Javascript placed in Static Resource in Salesforce](/assets/images/salesforce-integration/js-staticresource.png){:class="img-responsive"}

### Wrapping the Lightning Coveo component to include Static Resources

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

Here's an example of a wrapped AgentPanel to see all the pieces in place. We'll call it SampleAgentPanel.cmp and suppose we added a static resource called StyledInsightPanel.

```
<aura:component implements="force:hasRecordId,force:hasSObjectName,flexipage:availableForRecordHome" access="global">
    <aura:attribute name="name" type="String" access="global" />
    <aura:attribute name="searchHub" type="String" access="global" />
    <aura:attribute name="title" type="String" access="global" />
    <aura:attribute name="recordFields" type="String" access="global" />
    <aura:attribute name="openResultsInSubTab" type="Boolean" default="false" access="global" />
    <aura:attribute name="fullSearchComponent" default="" type="String" access="global" />
    <aura:attribute name="debug" type="Boolean" default="false" access="global" />

    <ltng:require styles="{!$Resource.StyledInsightPanel + '/index.min.css'}"/>

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
        customScripts="{!$Resource.StyledInsightPanel + '/index.min.js'}"
    />
</aura:component>
```

### Add your markup to a Visualforce Component

When you add the custom lightning component, there are some options which can be customized. The `name` option corresponds to the name of the Visualforce component that will be found and loaded. If no component exists, the default behavior is a component will be created and a pre-made layout will be added to it.

If you have a VisualForce component that you want to modify locally, you will want to copy the contents within the `<apex:component >` tags to your local page within the `<body>` tags and vice-versa. No modifications made in the `<head>` tag locally should be deployed to Salesforce.

> As of CLI version 0.14.2 you can create templated pages that are ready to use (including the Coveo for Salesforce Local Development Kit), see [CLI Reference - Create a Page]({{ site.baseurl }}/docs/CLI-reference.html#create-a-page)
>
> An example command to create an insight panel for Salesforce lightning would be as follows:
> ```
> ./node_modules/.bin/coveops create:page insight-panel --layout salesforce-agent-panel
> ``` 
> 
> This command will create a new `insight-panel.html` page with the markup that corresponds to the markup found in a new Insight Panel's Visualforce component and is accessible at `/insight-panel.html` from your localhost.

The `<apex:component >` in the Visualforce component and `<body>` in the Turbo page can be interchanged one-per-one as long as both started from the same layout. There may be cases where the markup won't have a one-to-one match. In these cases, the Coveo markup should follow by dedicated sections.

The most frequently customized sections include:

- The tab section, usually found within the `div` containing the `coveo-tab-section` class. The code found therein is able to be surgically modified in both the local and Visualforce templates.

![Tab Section](/assets/images/salesforce-integration/tab-section.png){:class="img-responsive"}

- The facet section, usually found within the `div` containing the `coveo-facet-column` class. The code found therein is able to be surgically modified in both the local and Visualforce templates.

![Facet Section](/assets/images/salesforce-integration/facet-section.png){:class="img-responsive"}

- The result templates are usually found within the `div` containing the `CoveoResultList` class. The cpde found therein is able to be surgically modified in both the local and Visualforce templates - usually on a per result template basis. With the Coveo for Salesforce, differing components such as `ResultLink` and `Quickview` do not need to be modified between local and Visualforce templates.

![Result Template Section](/assets/images/salesforce-integration/result-template-section.png){:class="img-responsive"}

>An insight panel will have an additional `CoveoResultActionsMenu` component under the `div` with the `coveo-result-frame` class:
>
>```html
><script id="Sample" class="result-template" type="text/html" data-layout="list" data-field-fieldname="FieldValue">
>    <div class="coveo-result-frame">
>        <div class="CoveoResultActionsMenu">
>        <div class="CoveoAttachToCase" data-display-tooltip="true"></div>
>        <div class="CoveoSalesforceQuickview"></div>
>        <div class="CoveoResultActionsSendEmail"></div>
>        <div class="CoveoResultActionsPostToFeed"></div>
>        <div class="CoveoResultActionsSendLiveAgent"></div>
>    </div>
>    <div class="coveo-result-cell" style="vertical-align:top;text-align:center;width:32px;">
>```

#### Adding the Salesforce Local Development Kit

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
</div>

<div class="content-section" markdown="1">
## Deploying the Salesforce Solution

Once the translations are made in previous steps, the solution will be ready to deploy. You can use whichever tool you like to achieve this. Here's some common approaches to consider.

### Managing the changes directly within the sandbox

If you're handling the Salesforce development directly in Salesforce then you will want to use the Developer Console to edit the Lightning and Visualforce components.

The Aura component is derived from this guide and is exclusively used in Salesforce for the purpose of configuring UIs within Salesforce's visual editor. See [Wrapping the Lightning Coveo component to include Static Resources](#wrapping-the-lightning-coveo-component-to-include-static-resources).

On Salesforce, the Aura Component is created as a new Lightning Component.

![Developer Console Creation Menu](/assets/images/salesforce-integration/sf-developer-console.png){:class="img-responsive"}

The Visualforce component is derived from the Turbo project's page. See [Add your markup to a Visualforce Component](#add-your-markup-to-a-visualforce-component).

The Static Resource is derived from the Turbo's output to `dist`. See [Creating a Static Resource](#creating-a-static-resource).

On Salesforce, the Static Resource is managed within Salesforce's Setup. In order to upload a folder as a static resource, it must be compressed to a zip file. Navigate to the static resources page and fill in the form with the name being the name of the static resource you're using within the Aura component.

![Static Resources Form Within Salesforce Setup](/assets/images/salesforce-integration/sf-static-resources.png){:class="img-responsive"}

### Synchronizing local environments using sfdx and Visual Studio Code

If you're using Visual Studio Code as your IDE, Salesforce has an extension pack that will take care of integrating sfdx into your IDE experience. The extension lets you browse and download resources from Salesforce, as well as create and deploy them from context menus.

To integrate Salesforce with Visual Studio Code, see [Salesforce Extension Pack](https://marketplace.visualstudio.com/items?itemName=salesforce.salesforcedx-vscode)

Once installed and you've authenticated to the Salesforce Sandbox, you can use the context menu to manage deployment and retrieval of your Salesforce components.

![The context menu within Visual Studio Code](/assets/images/salesforce-integration/vsc-deploy.png){:class="img-responsive"}
</div>

### Implementing custom localization

In order to implement localization for translations that aren't out of the box in your Coveo for Salesforce components, we will leverage the component's static resources.

In order to simplify development, you can and should use Turbo's CLI commands regarding localization. Let us assume we created the following translation files using the CLI commands as an example:

```
// en.js
{
    "All": "All",
    "Articles": "Articles"
}
```

```
// fr.js
{
    "All": "Tout",
    "Articles": "Articles"
}
```

Note that every key matches a `data-caption` or the value captured by the `Coveo.l` function.

In your Salesforce component's static resources folder, create the following folder and files:

```
+ locales
| + en.js
| + fr.js
```

Every file name should match the language code you are trying to translate (i.e. English is en, French is fr, Spanish is es-es, etc.). We will keep the folder name `locales` for simplicity's sake.

The contents of `<language-code>.js` are as follows:

```
var LOCALES = {
    "CancelLastAction": "Previous",
    "All": "All",
    "Articles": "Articles",
}

/**
 * Do not delete or modify the following code!!!
 */

!function(e,t){"object"==typeof exports&&"object"==typeof module?module.exports=t(require("coveo-search-ui")):"function"==typeof define&&define.amd?define(["coveo-search-ui"],t):"object"==typeof exports?exports.CoveoLocalizationManager=t(require("coveo-search-ui")):e.CoveoLocalizationManager=t(e.Coveo)}(window,(function(e){return function(e){var t={};function n(o){if(t[o])return t[o].exports;var r=t[o]={i:o,l:!1,exports:{}};return e[o].call(r.exports,r,r.exports,n),r.l=!0,r.exports}return n.m=e,n.c=t,n.d=function(e,t,o){n.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:o})},n.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},n.t=function(e,t){if(1&t&&(e=n(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var o=Object.create(null);if(n.r(o),Object.defineProperty(o,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var r in e)n.d(o,r,function(t){return e[t]}.bind(null,r));return o},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},n.p="",n(n.s=0)}([function(e,t,n){e.exports=n(1)},function(e,t,n){e.exports=n(2)},function(e,t,n){const o=n(3);e.exports=function(e,t,n,r){t=t||[];const i=(r=r||{}).disableTargetting,c=n?[n]:document.querySelectorAll(".CoveoSearchInterface");String.locale;var u={};u[String.locale]=e,String.toLocaleString(u),i||c.forEach((function(n){n.addEventListener(o.InitializationEvents.beforeInitialization,(function(n,r){o.options(n.target,function(e,t){t=t||{};const n=function(e){let t=function(e){return Object.entries(e).filter((function(e){return"string"==typeof e[1]})).reduce((function(e,t){return e[t[0]]=t[1],e}),{})}(e),n=function(e){return Object.entries(e).filter((function(e){return"string"!=typeof e[1]})).reduce((function(e,t){return e[t[0].replace(/^Coveo/,"")]=t[1],e}),{})}(e);return{locales:t,overrides:n}}(e)||{},o=n.locales||{},r=n.overrides||{};return t=(t=t.map((function(e){e.replace(/^Coveo/,"")}))).concat(Object.keys(r)).filter((function(e,t,n){return n.indexOf(e)==t})),console.log(t,r),t.reduce((function(e,t){return e[t]={valueCaption:Object.assign({},o,r[t])},e}),{})}(e,t))}))}))},window.NodeList&&!NodeList.prototype.forEach&&(NodeList.prototype.forEach=function(e,t){t=t||window;for(var n=0;n<this.length;n++)e.call(t,this[n],n,this)}),Object.entries||(Object.entries=function(e){for(var t=Object.keys(e),n=t.length,o=new Array(n);n--;)o[n]=[t[n],e[t[n]]];return o}),"function"!=typeof Object.assign&&Object.defineProperty(Object,"assign",{value:function(e,t){"use strict";if(null==e)throw new TypeError("Cannot convert undefined or null to object");for(var n=Object(e),o=1;o<arguments.length;o++){var r=arguments[o];if(null!=r)for(var i in r)Object.prototype.hasOwnProperty.call(r,i)&&(n[i]=r[i])}return n},writable:!0,configurable:!0})},function(t,n){t.exports=e}])}));

CoveoLocalizationManager(LOCALES);
```

Let's break it down:

First, we have the locale JSON object we generated in our project. We will assign it to a variable named `LOCALES`.

```
var LOCALES = {
    "CancelLastAction": "Previous",
    "All": "All",
    "Articles": "Articles",
}
```

The following piece of code comes directly from the localization-manager turbo component. Make sure you're using the latest code from [https://unpkg.com/@coveops/localization-manager/dist/index.min.js](https://unpkg.com/@coveops/localization-manager/dist/index.min.js).

```
!function(e,t){"object"==typeof exports&&"object"==typeof module?module.exports=t(require("coveo-search-ui")):"function"==typeof define&&define.amd?define(["coveo-search-ui"],t):"object"==typeof exports?exports.CoveoLocalizationManager=t(require("coveo-search-ui")):e.CoveoLocalizationManager=t(e.Coveo)}(window,(function(e){return function(e){var t={};function n(o){if(t[o])return t[o].exports;var r=t[o]={i:o,l:!1,exports:{}};return e[o].call(r.exports,r,r.exports,n),r.l=!0,r.exports}return n.m=e,n.c=t,n.d=function(e,t,o){n.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:o})},n.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},n.t=function(e,t){if(1&t&&(e=n(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var o=Object.create(null);if(n.r(o),Object.defineProperty(o,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var r in e)n.d(o,r,function(t){return e[t]}.bind(null,r));return o},n.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return n.d(t,"a",t),t},n.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},n.p="",n(n.s=0)}([function(e,t,n){e.exports=n(1)},function(e,t,n){e.exports=n(2)},function(e,t,n){const o=n(3);e.exports=function(e,t,n,r){t=t||[];const i=(r=r||{}).disableTargetting,c=n?[n]:document.querySelectorAll(".CoveoSearchInterface");String.locale;var u={};u[String.locale]=e,String.toLocaleString(u),i||c.forEach((function(n){n.addEventListener(o.InitializationEvents.beforeInitialization,(function(n,r){o.options(n.target,function(e,t){t=t||{};const n=function(e){let t=function(e){return Object.entries(e).filter((function(e){return"string"==typeof e[1]})).reduce((function(e,t){return e[t[0]]=t[1],e}),{})}(e),n=function(e){return Object.entries(e).filter((function(e){return"string"!=typeof e[1]})).reduce((function(e,t){return e[t[0].replace(/^Coveo/,"")]=t[1],e}),{})}(e);return{locales:t,overrides:n}}(e)||{},o=n.locales||{},r=n.overrides||{};return t=(t=t.map((function(e){e.replace(/^Coveo/,"")}))).concat(Object.keys(r)).filter((function(e,t,n){return n.indexOf(e)==t})),console.log(t,r),t.reduce((function(e,t){return e[t]={valueCaption:Object.assign({},o,r[t])},e}),{})}(e,t))}))}))},window.NodeList&&!NodeList.prototype.forEach&&(NodeList.prototype.forEach=function(e,t){t=t||window;for(var n=0;n<this.length;n++)e.call(t,this[n],n,this)}),Object.entries||(Object.entries=function(e){for(var t=Object.keys(e),n=t.length,o=new Array(n);n--;)o[n]=[t[n],e[t[n]]];return o}),"function"!=typeof Object.assign&&Object.defineProperty(Object,"assign",{value:function(e,t){"use strict";if(null==e)throw new TypeError("Cannot convert undefined or null to object");for(var n=Object(e),o=1;o<arguments.length;o++){var r=arguments[o];if(null!=r)for(var i in r)Object.prototype.hasOwnProperty.call(r,i)&&(n[i]=r[i])}return n},writable:!0,configurable:!0})},function(t,n){t.exports=e}])}));
```

Finally, this line initializes the translations.

```
CoveoLocalizationManager(LOCALES);
```

Copy and paste that code for every translation you have, replacing the contents of the `LOCALES` variable with the respective translations.

Finally, we will need to load those files as custom scripts in our Salesforce aura component:

```
<CoveoV2:CommunitySearch 
    searchHub="{!v.searchHub}" 
    name="{!v.name}"
    debug="{!v.debug}"
    customScripts="{!join(',',
        $Resource.CoveoV2__searchUi + '/js/cultures/' + $Locale.language + '.js',
        $Resource.<COMPONENT_NAME> + '/locales/' + $Locale.language + '.js',
        $Resource.<COMPONENT_NAME> + '/index.min.js'
    )}"
/>
```
It's as simple as that!

Now, if you have language codes that differ from Coveo's language codes, like Spanish, you can do the following:

```
    <aura:if isTrue="{!$Locale.language == 'es'}">
        <CoveoV2:CommunitySearch 
            searchHub="{!v.searchHub}" 
            name="{!v.name}"
            debug="{!v.debug}"
            customScripts="{!join(',',
                $Resource.CoveoV2__searchUi + '/js/cultures/es-es.js',
                $Resource.<COMPONENT_NAME> + '/locales/es-es.js',
                $Resource.<COMPONENT_NAME> + '/index.min.js'
            )}"
        />
        <aura:set attribute="else">
            <CoveoV2:CommunitySearch 
                searchHub="{!v.searchHub}" 
                name="{!v.name}"
                debug="{!v.debug}"
                customScripts="{!join(',',
                    $Resource.CoveoV2__searchUi + '/js/cultures/' + $Locale.language + '.js',
                    $Resource.<COMPONENT_NAME> + '/locales/' + $Locale.language + '.js',
                    $Resource.<COMPONENT_NAME> + '/index.min.js'
                )}"
            />
        </aura:set>
    </aura:if>
```

In this instance, Salesforce defines the Spanish language code as `es` and Coveo defines it as `es-es`.

For multiple conditions:

```
<aura:if isTrue="{!$Locale.language == 'en'}">
    .. do something ..
</aura:if>
<aura:if isTrue="{!$Locale.language == 'fr'}">
    .. do something ..
</aura:if>
<aura:if isTrue="{!$Locale.language == 'es'}">
    .. do something ..
</aura:if>
```