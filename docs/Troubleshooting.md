---
id: troubleshooting
title: Troubleshooting
---

 1. `watchman watch-del-all`
 2. `rm - rf node_modules` & `yarn install`
 3. Reset Metro's cache 
   * TODO`--reset-cache` flag,OR
   * | your Metro configuration file,
     * add `resetCache: true` 
 4. `rm -rf ${TMPDIR:-/tmp}/metro-*`
 5. Update Metro to the [latest version](https://www.npmjs.com/package/metro)
