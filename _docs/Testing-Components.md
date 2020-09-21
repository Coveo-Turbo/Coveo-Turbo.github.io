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
