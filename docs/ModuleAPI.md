---
id: module-api
title: Module API
---

* Metro
  * 's design
    * allow code written for Node (or for bundlers targeting the Web)
    * run mostly unmodified 

# Metro's main APIs / -- available to -- application code

## `require()`

* == Node's [`require()`](https://nodejs.org/api/modules.html#requireid) function 
* `require(moduleNameOrPath)`
  * `moduleNameOrPath`
    * == constant | compile-time
      * if it's NOT a constant & based on [`dynamicDepsInPackages`](./Configuration.md#dynamicdepsinpackages) -> will fail | 
        * build time or
        * runtime
    * 👀these modules -- will be -- added | bundle 👀
  * returns the result of evaluating that module's code

```js
const localModule = require('./path/module');
const asset = require('./path/asset.png');
const jsonData = require('./path/data.json');
const {View} = require('react-native');
```

### Advanced usage: `require` | runtime

* | build time,
  * Metro 
    * [resolves](./Resolution.md) module names -- to -- absolute paths
    * [assigns an opaque module ID](./Configuration.md#createmoduleidfactory) / EACH one

* | runtime,
  * `require` -- refers to a -- `function(opaqueModuleID): returnsAModule`
    * `opaqueModuleID` != name or path 
    * uses
      * you ALREADY have a module ID / -- returned by -- ANOTHER module API
        * _Example:_ [`require.resolveWeak`](#require-resolveweak)

```js
const localModule = require('./path/module');
const id = require.resolveWeak('./path/module');
// Bypass the restriction on non-constant require() arguments
const dynamicRequire = require;
dynamicRequire(id) === localModule; // true
```

## `module.exports`

* == [`module.exports`](https://nodejs.org/api/modules.html#moduleexports)

## ES Modules syntax (`import` and `export`)

* recommendations
  * use [`@babel/plugin-transform-modules-commonjs`](https://babeljs.io/docs/babel-plugin-transform-modules-commonjs) | Metro projects
    * Reason: 🧠 support `import` and `export` 🧠

* if you use `@react-native/babel-preset` | React Native projects -> `import` and `export` supported out of the box

## `import()` (dynamic import)

* TODO:

[`import()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) calls are supported out of the box. 
In React Native, using `import()` automatically splits your application code so that it loads faster during development, without affecting release builds.

:::info
**For framework implementers**:
1. Enable [lazy bundling](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0605-lazy-bundling.md) by adding `&lazy=true` to the initial HTTP bundle URL your framework requests from Metro.
2. At runtime, `import()` calls a framework-defined function to [fetch and evaluate](https://github.com/react-native-community/discussions-and-proposals/blob/main/proposals/0605-lazy-bundling.md#__loadbundleasync-in-metro) the split bundle. 
   1. Your framework **must** implement this function if it uses the `lazy=true` parameter, or runtime errors will occur.
   :::

## `require.resolveWeak()`

Takes a module name (or path) and returns that module's opaque ID, without including it in the bundle. 
This is a specialised API intended to be used by frameworks; application code will rarely need to use it directly. 
See the section about [using `require` at runtime](#advanced-usage-require-at-runtime).
