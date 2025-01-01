---
id: bundling
title: Bundle Formats
---

* | bundle,
  * 💡EACH module -- gets assigned a -- numeric id 💡
    * == ❌NO support dynamic requires ❌
    * require -- are changed by -- its numeric version
    * modules are stored | DIFFERENT possible formats 

* supported bundling formats
  * [plain](#plain-bundle)
  * [indexed RAM](#indexed-ram-bundle)
  * [file RAM](#file-ram-bundle)

## Plain bundle

* == standard bundling format 
* ALL files 
  * -- are wrapped with a -- function call
  * added | global file 
* uses
  * environments / expect 1! JS bundle (== browser)
* requirements
  * 👀entry point / `.bundle` extension -- should trigger a -- build of it 👀

## Indexed RAM bundle

* bundle == binary file /
  * parts (-- expressed in -- Little Endian)
    * magic number
      * == `uint32` / 
        * located | beginning of the file
        * value `0xFB0BD1E5`
      * verify the file
    * offset table
      * == sequence of `uint32` pairs + header
        * header
          * == length of the table & length of the startup code
            * BOTH are `uint32`
            * Startup code
              * ALWAYS found | `file[sizeof(uint32)]`
        * pairs,
          * == file's offset  & length of code module / expressed | bytes
      * allows
        * loading ANY module | constant time /
          * code / module `x` -> located | `file[(x + 3) * sizeof(uint32)]` 
    * null byte (`\0`)
      * | END
      * / EACH module 
        * == separate ALL modules ->
          * length does NOT even need to be used
          * module -- can be loaded directly as an -- ASCIIZ string

```
` 0                   1                   2                   3                   4                   5                   6
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          Magic number                         |                          Header size                          |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                       Startup code size                       |                        Module 0 offset                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Module 0 length                        |                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                                               +
|                                                                                                                               |
+                                                              ...                                                              +
|                                                                                                                               |
+                                                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |                        Module n offset                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Module n length                        | Module 0 code | Module 0 code |      ...      |       \0      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Module 1 code | Module 1 code |      ...      |       \0      |                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                                               +
|                                                                                                                               |
+                                                              ...                                                              +
|                                                                                                                               |
+                                                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               | Module n code | Module n code |      ...      |       \0      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+`
```

* uses
  * environment / able to load ALL code in memory at once 
    * optimal structure 
  * NORMALLY, by iOS

## File RAM bundle

* EACH module -- is stored as a --
  * file / name `js-modules/${id}.js` +
  * ANOTHER file / called `UNBUNDLE` 
    * 👀ONLY content == magic number, `0xFB0BD1E5` 👀
    * created | root

* uses
  * by Android
    * Reason: 🧠package contents are zipped & access to a zipped file is much faster 🧠
    * if the indexed format was used instead -> ALL bundled -- should be -- unzipped at once / get the code -- for the -- corresponding module
