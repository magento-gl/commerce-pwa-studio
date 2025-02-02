---
title: Modify the Footer component
---

# Modify the Footer component

One way to customize a storefront is to modify its UI components.

## Overview

This tutorial provides the steps for modifying Venia's **Footer** component by adding a link to the existing content.
By the end of this tutorial, you will know how to override different pieces of Venia to add your customizations.

<InlineAlert variant="info" slots="text"/>

This tutorial requires you to have a project set up using the steps provided in the [Project Setup][] tutorial.

[project setup]: /tutorials/setup-storefront/

When modifying any storefront component from the default Venia storefront, follow these basic steps:

1. Identify the component you want to update and its render chain
1. Make a copy of the target component and the components in its render chain in your project
1. Update dependencies in your project to use the new copies

<InlineAlert variant="warning" slots="text"/>

The steps outlined in this tutorial explain the render chain replacement approach.
This is a valid pattern in some situations, but you can make similar customizations with less code using [Targets and Targetables][].

[targets and targetables]: /tutorials/targets/

## Identify the target component

The first step in modifying anything in the Venia storefront is to identify the component whose content you want to modify.

### Using React DevTools

[React DevTools][] is a browser plugin for React developers.
It gives you the ability to navigate and inspect component nodes in the React DOM tree.

[react devtools]: https://reactjs.org/blog/2019/08/15/new-react-devtools.html

After you install React DevTools in your browser, open your storefront and your browser's web developer tools.

<InlineAlert variant="info" slots="text"/>

For this tutorial, Chrome's web developer tools are used, but these steps can be generally applied to other browsers.

![Chrome dev tools](images/web-dev-tools.png)

In Chrome, the React DevTools is accessed through a dropdown on the top right of the developer tools panel.
Read the React DevTools plugin documentation find out how to access this tool in your browser.

![React DevTools in Chrome](images/react-dev-tools.png)

Use the React DevTools to select content in the footer element to see which component renders it.
For this tutorial, select the **Footer** component.

![Footer component selected](images/footer-component-selected.png)

## Identify the render chain

The render chain is the path in the React DOM tree that starts in the **App** or a **Root Component** and ends at the target component.
These are the components you need to copy into your project to make modifications.

Use the React DevTools to navigate the React DOM tree and find the render chain of the target component.
Ignore React context providers and consumers because they are often just used as a content wrapper.

For this tutorial, the render chain for the Footer component in the Venia storefront is `Adapter -> App -> Main -> Footer`.
You can verify this by looking at the source for the [Adapter][], [Main][] and [App][] components.
Main imports and renders the Footer component, and App imports and renders the Main component.

[Main]: https://github.com/magento/pwa-studio/blob/develop/packages/venia-ui/lib/components/Main/main.js
[App]: https://github.com/magento/pwa-studio/blob/develop/packages/venia-ui/lib/components/App/app.js
[Adapter]: https://github.com/magento/pwa-studio/blob/develop/packages/venia-ui/lib/components/Adapter/adapter.js

### Root components

Static imported components, such as Header, Footer, and side Navigation, have render chains that begin in **Adapter** and then **App**, but content that is specific to a route have render chains that begin at a **Root Component**.

Root components are dynamically loaded components associated with an Adobe Commerce or Magento Open Source page type or route.
A list of Venia's root components can be found in the [RootComponent][] directory in the PWA Studio project.

[rootcomponent]: https://github.com/magento/pwa-studio/tree/develop/packages/venia-ui/lib/RootComponents

## Create component directories

If your project does not already have one, create a `components` directory under your `src` directory.
This directory will hold your project's components.

```sh
mkdir -p src/components
```

Next, create directories for each component in the render chain.
These directories will hold copies of the component source code from Venia.

```sh
mkdir -p src/components/App && \
mkdir -p src/components/Main && \
mkdir -p src/components/Footer && \
mkdir -p src/components/Adapter
```

## Copy components

Make a copy of the components in the render chain from the `node_modules` directory.

### Copy Adapter component

`src/index.js` uses Adapter and this capsulating App component.
Copy this component from `node_modules` into your project.

```sh
cp node_modules/@magento/venia-ui/lib/components/Adapter/adapter.js src/components/Adapter
```

### Copy App component

The App component is the main entry point for the storefront application.
It imports and renders the Main component, which renders the Footer component.

```sh
cp node_modules/@magento/venia-ui/lib/components/App/app.js src/components/App
```

If you look at the [`index.js`][] file for Venia's App component, its default export is not `app.js`.
The default export for this component is `container.js`, which is a container for the `app.js` module, so copy the `container.js` file into your project.

[`index.js`]: https://github.com/magento/pwa-studio/blob/develop/packages/venia-ui/lib/components/App/index.js

```sh
cp node_modules/@magento/venia-ui/lib/components/App/container.js src/components/App
```

### Copy Main component

The Main component imports and renders the Header, Footer, and route-specific components.
Copy this component from `node_modules` into your project.

```sh
cp node_modules/@magento/venia-ui/lib/components/Main/main.js src/components/Main
```

### Copy Footer component

The Footer component is the target component you will modify for this tutorial.
Copy this component from the `node_modules` directory into your project.

```sh
cp node_modules/@magento/venia-ui/lib/components/Footer/footer.js src/components/Footer
```

## Add a link to the Footer

Open `src/components/Footer/footer.js` and make the following modifications to add a link to the footer element.

Use the Link component to create a link to an internal route defined in the [Add a static route tutorial][]:

[add a static route tutorial]: /tutorials/basic-modifications/add-static-route/

```diff
    <footer className={classes.root}>
      <div className={classes.links}>
+       <div className={classes.link}>
+         <Link to="/foo">
+           <span className={classes.label}>Foo Demo Page</span>
+         </Link>
+       </div>
        {linkGroups}
      </div>
      <div className={classes.callout}>
        <h3 className={classes.calloutHeading}>{"Follow Us!"}</h3>
```

## Connect everything together

Some of the `import` statements in the copied components use relative paths for dependent components, but these components are not available in your project.
To use these dependent components without copying them into your project, you must update their import statements to import from Venia.

### Update Footer import statements

Update the relative imports in `src/components/Footer/footer.js`.

```diff
- import { mergeClasses } from '../../classify';
- import defaultClasses from './footer.css';
- import { DEFAULT_LINKS, LOREM_IPSUM } from "./sampleData";
- import GET_STORE_CONFIG_DATA from '../../queries/getStoreConfigData.graphql';
+ import { mergeClasses } from '@magento/venia-ui/lib/classify';
+ import defaultClasses from '@magento/venia-ui/lib/components/Footer/footer.css';
+ import { DEFAULT_LINKS, LOREM_IPSUM } from "@magento/venia-ui/lib/components/Footer/sampleData";
+ import GET_STORE_CONFIG_DATA from '@magento/venia-ui/lib/queries/getStoreConfigData.graphql';
```

### Export Footer component

Create a `src/components/Footer/index.js` file with the following content to set the default component export for the `Footer` directory.

```js
export { default } from "./footer";
```

### Update Main import statements

Update the relative imports in `src/components/Main/main.js`.
Skip updating the Footer import statement to use your project's modified Footer component.

```diff
- import { mergeClasses } from '../../classify';
+ import { mergeClasses } from '@magento/venia-ui/lib/classify';
  import Footer from '../Footer';
- import Header from '../Header';
- import defaultClasses from './main.css';
+ import Header from '@magento/venia-ui/lib/components/Header';
+ import defaultClasses from '@magento/venia-ui/lib/components/Main/main.css';
```

### Export Main component

Create a `src/components/Main/index.js` file with the following content to set the default component export for the `Main` directory.

```js
export { default } from "./main";
```

### Update App import statements

Update the relative imports in `src/components/App/app.js`.
Skip updating the Main import statement to use your project's copy of the Main component.

```diff
- import { HeadProvider, Title } from '../Head';
+ import { HeadProvider, Title } from '@magento/venia-ui/lib/components/Head';
  import Main from '../Main';
- import Mask from '../Mask';
- import MiniCart from '../MiniCart';
- import Navigation from '../Navigation';
- import Routes from '../Routes';
- import ToastContainer from '../ToastContainer';
- import Icon from '../Icon';
+ import Mask from '@magento/venia-ui/lib/components/Mask';
+ import MiniCart from '@magento/venia-ui/lib/components/MiniCart';
+ import Navigation from '@magento/venia-ui/lib/components/Navigation';
+ import Routes from '@magento/venia-ui/lib/components/Routes';
+ import ToastContainer from '@magento/venia-ui/lib/components/ToastContainer';
+ import Icon from '@magento/venia-ui/lib/components/Icon';
```

Update the relative import in `src/components/App/container.js`.

```diff
  import App from './app';
- import { useErrorBoundary } from './useErrorBoundary'
+ import { useErrorBoundary } from '@magento/venia-ui/lib/components/App/useErrorBoundary'
```

### Export App component

Create a `src/components/App/index.js` file with the following content to set the default component export for the `App` directory.
Instead of directly exporting the module in `app.js` in the `index.js` file, it is wrapped inside an `AppContainer` component in `container.js`.
This is the default export for the App component.

```js
export { default } from "./container";
```

### Update Adapter import statements

```diff
- import App, { AppContextProvider } from '@magento/venia-ui/lib/components/App';
+ import { AppContextProvider } from '@magento/venia-ui/lib/components/App';
+ import App from '../App';
```

### Export Adapter component

Create a `src/components/Adapter/index.js` file with the following content to set the default component export for the `Adapter` directory.

```js
export { default } from "./adapter";
```

### Import new Adapter component

First, copy the Adapter component into the `src/components` directory:

```sh
cp node_modules/@magento/venia-ui/lib/components/Adapter/adapter.js src/components/Adapter
```

Next, modify the import statement in `adapter.js`:

```diff
- import App, { AppContextProvider } from '@magento/venia-ui/lib/components/App';
+ import { AppContextProvider } from '@magento/venia-ui/lib/components/App';
+ import App from '../App';
```

Finally, export the default module:

```diff
+ export { default } from './adapter';
```

## Congratulations

You just customized the Footer component in your storefront project!

![foo footer link](images/foo-footer-link.png)
