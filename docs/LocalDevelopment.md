---
id: local-development
title: Local Development Setup
---

* goal
  * developers / work | Metro
    * how to test your changes | OTHER local projects

### Testing Metro Changes | React Native Project

* recommendations
  * run your Metro changes | local target project

* recommended workflow
  * register local `metro` packages | your development clone ( -- via -- [`yarn link`][1])
  * hot-switch to these versions | consuming project 

* goal
  * link a local Metro clone -- with a -- bare workflow React Native app 

    ```sh
    .
    └── Development
        ├── metro        # metro clone
        └── MetroTestApp # target project
     ```

* steps    
  1. `yarn link` | your `metro` clone -- to -- register local packages
     * | `metro` clone,
       * `yarn link` -- is responsible for -- registering local package folders / linked to ELSEWHERE
     * `npm exec --workspaces` register ALL packages | `metro` repo
       * they can be individually linked | target project
         ```sh
         npm exec --workspaces -- yarn link
         ```
  2. `yarn link` | target project folder -- to replace -- Metro packages | your target project
    ```sh
    # Links 3 packages
    yarn link metro metro-config metro-runtime
    ```
  3. configure Metro `watchFolders` / work with our linked packages
     * Reason: 🧠
       * `yarn link` has included files | outside of React Native project folder
       * -> need to inform Metro -- about the -- existence of these files 🧠
     
    ```diff, metro.config.js
    + const path = require('path');

        module.exports = {
    +   watchFolders: [
    +     path.resolve(__dirname, './node_modules'),
    +     // Include necessary file paths for `yarn link`ed modules
    +     path.resolve(__dirname, '../metro/packages'),
    +     path.resolve(__dirname, '../metro/node_modules'),
    +   ],
        ...
        };
    ```
    
    * check that you can run Metro | our target project
   
      ```sh
      yarn react-native start
      ```

  4. (Optional) Clean up
    * repeat step 1 & 2 -- with --`yarn unlink`
    * goal
      * restore the remote (== production npm) versions of `metro` packages | your target project

### Debug Logging

* TODO:
Metro uses the [debug](https://www.npmjs.com/package/debug) package to write logs under named debug scopes (for example: `Metro:WatchmanWatcher`). 
Set the `DEBUG` environment variable before starting Metro to enable logs matching the supplied pattern.

The snippet below provides a pattern matching all Metro-defined messages.
```sh
DEBUG='Metro:*' yarn metro serve
```
[1]: https://classic.yarnpkg.com/en/docs/cli/link
