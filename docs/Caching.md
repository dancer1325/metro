---
id: caching
title: Caching
---

* local cache of [transformed modules](./Concepts.md#transformation)
  * speeds up builds
  * ONLY if the source code changes -> Metro retransforms modules
  * _Example:_ `FileStore`

* remote cache
  * speed up builds 
  * use cases
    * larger teams 
    * larger codebases
      * _Example:_ Meta
  * typical setup
    1. storage backend / team-specific
      * _Example:_ S3 bucket
    2. Run [`metro build`](./CLI.md#build-entry) PERIODICALLY
      * _Example:_ | CI
      * populate the cache -- , via `HttpStore`  OR custom read/write cache store, --  | your Metro config
    3. Configure Metro | your development machines -- , via `HttpGetStore` OR CUSTOM read-only cache store | your Metro config, to -- read -- from the -- cache
  * _Example:_ `HttpCache`

* [`cacheStores`](./Configuration.md#cachestores)
  * == Metro cache's main configuration option

* recommendations | list caches
  * local cache 
  * remote cache 

## Built-in cache stores

* uses
  * \+ [`cacheStores`](./Configuration.md#cachestores) config option
    * **`FileStore({root: string})`** will store cache entries as files under the directory specified by `root`.
    * **`AutoCleanFileStore()`** <div class="label deprecated">Deprecated</div> is a `FileStore` that periodically cleans up old entries. It accepts the same options as `FileStore` plus the following:
  * **`options.intervalMs: number`** is the time in milliseconds between cleanup attempts. Defaults to 10 minutes.
  * **`options.cleanupThresholdMs: number`** is the minimum time in milliseconds since the last modification of an entry before it can be deleted. Defaults to 3 days.
* **`HttpStore(options)`** is a bare-bones remote cache client that reads (`GET`) and writes (`PUT`) compressed cache artifacts over HTTP or HTTPS.
  * **`options.endpoint: string`** is the base URL for the cache server. For example, an `HttpStore` with `'http://www.example.com/endpoint'` as the endpoint would issue requests to URLs such as `http://www.example.com/endpoint/c083bff944879d9f528cf185eba0f496bc10a47d`.
  * **`options.timeout: number`** is the timeout for requests to the cache server, in milliseconds. Defaults to 5000.
  * **`options.family: 4 | 6`** is the same as the `family` parameter to Node's [`http.request`](https://nodejs.org/api/http.html#httprequesturl-options-callback).
  * **`options.cert`, `options.ca`, `options.key`**: HTTPS options passed directly to [Node's built-in HTTPS client](https://nodejs.org/api/https.html).
* **`HttpGetStore(options)`** is a read-only version of `HttpStore`.

You can import these classes from the `metro-cache` package or get them through the function form of `cacheStores`:

```js
// metro.config.js
const os = require('node:os');
const path = require('node:path');

module.exports = {
  cacheStores: ({ FileStore }) => [
    new FileStore({
      root: path.join(os.tmpdir(), 'metro-cache'),
    }),
  ],
};

```

## Custom cache stores

* steps to implement It
  *  pass an instance of a class with the following interface into [`cacheStores`](./Configuration.md#cachestores):

```flow
interface CacheStore<T: Buffer | JsonSerializable> {
  // Read an entry from the cache. Returns `null` if not found.
  get(key: Buffer): ?T | Promise<?T>;

  // Write an entry to the cache (if writable) or do nothing (if read-only)
  set(key: Buffer, value: T): void | Promise<void>;

  // Clear the cache (if possible) or do nothing
  clear(): void | Promise<void>;
}

type JsonSerializable = /* Any JSON-serializable value */;
```

The value of a cache entry is either an instance of [`Buffer`](https://nodejs.org/api/buffer.html#buffer) or a JSON-serializable value (with unspecified internal structure in both cases). For a given cache key, `get()` *must* return the same type of value that was originally provided to `set()`.
