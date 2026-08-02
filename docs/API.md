---
id: api
title: Bundling API
---

## Quick Start

* Compile a file

  ```js
  const config = await Metro.loadConfig();

  await Metro.runBuild(config, {
    entry: 'index.js',
    out: 'bundle.js',
  });
  ```

* Run a server & watch the filesystem for changes

  ```js
  const config = await Metro.loadConfig();

  await Metro.runServer(config);
  ```

* Create a Connect middleware & plug it | server

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

## Reference

* `config`
  * == object / is [Metro configuration](CLI.md) 
    * exposed by your `metro.config.js` file
    * you can obtain it -- via -- `Metro.loadConfig` 
  * uses
    * ALL next functions below -- can accept it as -- additional option 


### `loadConfig(<options>)`

* **Basic options**
  * `config`
  * `cwd`
    * traverse the directory hierarchy to the root / finding the metro file configuration (by default `metro.config.js`) 
* return Metro configuration /
  * normalized
  * merged with Metro's default values

### `async runMetro(config)`

* creates a Metro server -- based on the -- config
* uses
  * middleware | your EXISTING server

### `async runBuild(config, <options>)`

* TODO:
**Required options:** `entry`, `out`

**Basic options:** `dev`, `minify`, `platform`, `sourceMap`, `sourceMapUrl`, `assets`

Bundles `entry` for the given `platform`, and saves it to location `out`. 
If `sourceMap` is set, also generates a source map.
The source map will be inlined, unless `sourceMapUrl` is also defined. In the latter case, a new file will be generated with the basename of the `sourceMapUrl` parameter.
If `assets` is `true`, an array of `AssetData` will be generated and returned in the `assets` property of the result object.

### `async runServer(config, <options>)`

**Basic options:** `host`, `port`, `secureServerOptions`, `secure (DEPRECATED)`, `secureKey (DEPRECATED)`, `secureCert (DEPRECATED)`

* start a FULL Metro HTTP server /
  * listen | specified `host:port`
  * can be queried -- to -- retrieve bundles / VARIOUS entry points
  * if you specify `secureServerOptions` -> server will be exposed | HTTPS

### `createConnectMiddleware(config, <options>)`

* **Basic options**
  * `port`
    * OPTIONAL 
    * uses
      * login purposes
  * `onBundleBuilt`
    * OPTIONAL 
    * == function /
      * called | finish creating the bundle
      * 's argument: bundle name
* create a Connect middleware /
  * answers -- to -- bundle requests
  * can be plugged | your OWN servers

