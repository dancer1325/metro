---
id: getting-started
title: Getting Started
---


```bash
# NPM
npm install --save-dev metro metro-core

# YARN
yarn add --dev metro metro-core
```

## Running `metro`

* ways
  * run the [CLI](./CLI.md) or
  * call it programmatically
    * requirements
    
      ```
      const Metro = require('metro');
      ```

### Running Programmatically -- ALLOWED methods -- 

#### Method `runMetro(config)`

```js
'use strict';

const http = require('http');
const Metro = require('metro');

// We first load the config from the file system
Metro.loadConfig().then(async (config) => {
  const metroBundlerServer = await Metro.runMetro(config);

  // hook a MetroServer | HTTP(S)
  const httpServer = http.createServer(
    metroBundlerServer.processRequest.bind(metroBundlerServer),
  );

  httpServer.listen(8081);
});
```

* TODO:
In order to be also compatible with Express apps, `processRequest` will also call its third parameter when the request could not be handled by Metro. 
This allows you to integrate the server with your existing server, or to extend a new one:

```js
const httpServer = http.createServer((req, res) => {
  metroBundlerServer.processRequest(req, res, () => {
    // Metro does not know how to handle the request.
  });
});
```

If you are using [Express](http://expressjs.com/), you can just pass `processRequest` as a middleware:

```js
const express = require('express');
const app = express();

app.use(
  metroBundlerServer.processRequest.bind(metroBundlerServer),
);

app.listen(8081);
```

#### Method `runServer(config, options)`

* recommendations
  * 👀use `runMetro` instead of `runServer` 👀

##### Options

* `host (string)`
  * place | host the server on
* `onReady (Function)`
  * if server is ready to serve requests -> called
* `onClose (Function)`
  * if server and ALL OTHER Metro processes are closed -> called 
  * `httpServer.on('close', () => {});`
    * alternative to `onClose` 

```js
const config = await Metro.loadConfig();

const metroHttpServer = await Metro.runServer(config, {
  onClose: () => {console.log('metro server and all associated processes are closed')}
});

httpServer.on('close', () => {console.log('metro server is closed')});
```

TODO:
* `secure (boolean)`: **DEPRECATED** Whether the server should run on `https` instead of `http`.
* `secureKey (string)`: **DEPRECATED** The key to use for `https` when `secure` is on.
* `secureCert (string)`: **DEPRECATED** The cert to use for `https` when `secure` is on.
* `secureServerOptions (Object)`: The options object to pass to Metro's HTTPS server. The presence of this object will make Metro's server run on `https`. Refer to the [Node docs](https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener) for valid options.
* `waitForBundler (boolean)`: Whether to wait for the bundler to finish initializing before returning the server instance.

```js
const config = await Metro.loadConfig();

await Metro.runServer(config);
```

```js
const fs = require('fs');

const config = await Metro.loadConfig();

await Metro.runServer(config, {
  secureServerOptions: {
    ca: fs.readFileSync('path/to/ca'),
    cert: fs.readFileSync('path/to/cert'),
    key: fs.readFileSync('path/to/key'),
  }
});
```

#### Method `runBuild(config, options)`

* uses
  * | build time

##### Options

* options / related to the
  * server
  * bundle itself

<!-- TODO(ives): Decide whether we need to show this to the user  * `output (boolean)` -->

TODO:
* `dev (boolean)`: Create a development version of the build (`process.env.NODE_ENV = 'development'`).
* `entry (string)`: Pointing to the entry file to bundle.
* `onBegin (Function)`: Called when the bundling starts.
* `onComplete (Function)`: Called when the bundling finishes.
* `onProgress (Function)`: Called during the bundle, every time there's new information available about the module count/progress.
* `minify (boolean)`: Whether Metro should minify the bundle.
* `out (string)`: Path to the output bundle.
* `platform ('web' | 'android' | 'ios')`: Which platform to bundle for if a list of platforms is provided.
* `sourceMap (boolean)`: Whether Metro should generate source maps.
* `sourceMapUrl (string)`: URL where the source map can be found. It defaults to the same same URL as the bundle, but changing the extension from `.bundle` to `.map`. When `inlineSourceMap` is `true`, this property has no effect.

```js
const config = await Metro.loadConfig();

await Metro.runBuild(config, {
  entry: 'index.js',
  platform: 'ios',
  minify: true,
  out: '/Users/Metro/metro-ios.js'
});
```

#### Method `createConnectMiddleware(config)`

* Connect middleware
  * 👀ALTERNATIVE of creating a full server 👀
  * -- answers to -- bundle requests 
  * uses
    * plugged | your OWN servers

##### Options

* `port (number)`
  * OPTIONAL
  * := Connect middleware's port
  * uses
    * logging purposes

```js
const Metro = require('metro');
const express = require('express');
const app = express();
const server = require('http').Server(app);

Metro.loadConfig().then(async config => {
  const connectMiddleware = await Metro.createConnectMiddleware(config);
  const {server: {port}} = config;

  app.use(connectMiddleware.middleware);
  server.listen(port);
  connectMiddleware.attachHmrServer(server);
});
```

## Available options

### Configuration

* see [Configuring Metro](./Configuration.md)

## URL and bundle request

* server
  * serves
    * assets,
    * bundles
    * source maps | those bundles

### Assets

* 👀if you want to request an asset -> use `require` 👀
  * == done | ANOTHER JS file
  * treated special -- by the -- server
    * -> return the path to that file
  * asset -- is recognized by -- its extension
    * `assetExts` := array / allowed extensions

* you can specify assets / depend on the
  * platform
    * `*.specificPlatform`
      * _Example:_ `*.ios`
  * requested size  
    * `*specificSize`
      * _Example:_ `@2x`

### Bundle

* root for a bundle request
  * 👀VALID ANY JS file 👀
  * place | root of the project 
  * ALL files / it requires -> will be recursively included 

* if you want to request a bundle -> replace `.js` -- to -- `.bundle` 

* OPTIONS for building the bundle
  * 👀-> passed as query parameters / ALL are optional 👀
    * `dev`
      * build the bundle | development mode or NOT
      * <--> 1:1 to the bundles' `dev` setting 
      * allowed values
        * `true`
        * `false`
    * `platform`
      * platform -- requesting the -- bundle
      * allowed values
        * `ios`
        * `android`
      * <--> 1:1 to the bundles' `platform` setting
    * `minify`
      * == indicate whether code should be minified or not
      * <--> 1:1 to the bundles' `minify` setting
      * allowed values
        * `true`
        * `false`
    * `excludeSource`
      * == indicate whether sources should be included or not | source map
      * allowed values
        * `true`
        * `false`
  * _Example:_ if you request `http://localhost:8081/foo/bar/baz.bundle?dev=true&platform=ios`
    * -> create a bundle out of `foo/bar/baz.js` for iOS in development mode

### Source maps

* TODO:
Source maps are built for each bundle by using the same URL as the bundle (thus, the same as the JS file acting as a root). 
This will only work when `inlineSourceMap` is set to `false`.
All options you passed to the bundle will be added to the source map URL; otherwise, they wouldn't match.

## JavaScript transformer

The JavaScript transformer ([`babelTransformerPath`](./Configuration.md#babeltransformerpath)) is the place where JS code will be manipulated; useful for calling Babel. 
The transformer can export two methods:

### Method `transform(module)`

Mandatory method that will transform code. The object received has information about the module being transformed (e.g its path, code...) and the returned object has to contain an `ast` key that is the AST representation of the transformed code. The default shipped transformer does the bare minimum amount of work by just parsing the code to AST:

```js
const babylon = require('@babel/parser');

module.exports.transform = (file: {filename: string, src: string}) => {
  const ast = babylon.parse(file.src, {sourceType: 'module'});

  return {ast};
};
```

If you would like to plug-in Babel, you can simply do that by passing the code to it:

```js
const {transformSync} = require('@babel/core');

module.exports.transform = file => {
  return transformSync(file.src, {
    // Babel options...
  });
};
```

### Method `getCacheKey()`

Optional method that returns the cache key of the transformer.
When using different transformers, this allows to correctly tie a transformed file to the transformer that converted it.
The result of the method has to be a `string`.
