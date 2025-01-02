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

**Basic options:** `dev`, `minify`, `platform`, `sourceMap`, `sourceMapUrl`

Bundles `entry` for the given `platform`, and saves it to location `out`. 
If `sourceMap` is set, also generates a source map.
The source map will be inlined, unless `sourceMapUrl` is also defined. In the latter case, a new file will be generated with the basename of the `sourceMapUrl` parameter.

### `async runServer(config, <options>)`

**Basic options:** `host`, `port`, `secureServerOptions`, `secure (DEPRECATED)`, `secureKey (DEPRECATED)`, `secureCert (DEPRECATED)`

Starts a full Metro HTTP server. 
It will listen on the specified `host:port`, and can then be queried to retrieve bundles for various entry points.
If the `secureServerOptions` family of options are present, the server will be exposed over HTTPS.

`secure`, `secureKey`, `secureCert` are now deprecated and will be removed in a later release.
The presence of `secureServerOptions`, along with its options will make Metro run over https.

### `createConnectMiddleware(config, <options>)`

* **Basic options**
  * `port`
  * `onBundleBuilt`

Instead of creating the full server, creates a Connect middleware that answers to bundle requests.
This middleware can then be plugged into your own servers.
The `port` parameter is optional and only used for logging purposes. 
The `onBundleBuilt` function is optional, is passed the bundle name, and is called when the server has finishing creating the bundle.
