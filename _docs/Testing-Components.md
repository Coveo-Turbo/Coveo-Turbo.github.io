---
layout: doc
title:  "Testing Coveo Turbo Components"
desc: Quickly get a sense of how an existing Coveo Turbo component works.
---

Coveo Turbo offers a bunch of components that work hand in hand with the [Coveo JavaScript Search Framework](https://github.com/coveo/search-ui).

> You can review the list of components on the Turbo [home page]({{ site.baseurl }}/#components).

To use an existing Coveo Turbo component, it is usually recommended to install it through npm. However, when you just want to quickly test a component to see if it answers your requirements, you can use a reference to the compiled JavaScript instead.

The format of the compiled JavaScript looks like this, where `{component-name}` is replaced with the name of the component:

```
https://unpkg.com/@coveops/{component-name}@latest/dist/index.min.js
```

> You are discouraged from using unpkg as a production CDN. However, to ease the early steps of development, you are welcome to reference the unpkg code directly.

Once you have loaded the script on your page, you can reference the Coveo Turbo component like any other Coveo component. The exact name is usually indicated in the readme of the component, but it typically looks like this:

```html
<div class="CoveoTurboComponent"></div>
```

> Example:
>
> You want to test the [counted-tabs](https://github.com/Coveo-Turbo/counted-tabs) component on your page.
>
> You add the following tag in the header of your page, so the script loads from unpkg. Since this is a test environment on your local machine, you are not worried about being throttled.
> ```html
> <script src="https://unpkg.com/@coveops/counted-tabs@latest/dist/index.min.js"></script>
> ```
>
> In the CoveoSearchInterface component of the search page where you are testing the component, you add the following markup:
> ```html
> <div class="CoveoCountedTabs"></div>
> ```
>
> Because the counted-tabs component expects a field to be used, you add it in the data attributes, as such:
> ```html
> <div class="CoveoCountedTabs" data-field="@mytabfield"></div>
> ```