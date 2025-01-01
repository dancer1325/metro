---
id: concepts
title: Concepts
---

* Metro
  * see [README.md](/README.md)
  * bundling process' stages
    1. Resolution
    2. Transformation
    3. Serialization

### Resolution

* allows
  * build a graph of ALL modules / -- required from the -- entry point
* happens parallel to transformation stage

### Transformation

* allows
  * module -- is converted (transpiled) to a -- format / understandable by the target platform 
* 👀based on the # cores / you have -> transform modules happens in parallel 👀 

### Serialization

* 👀ONCE ALL modules have been transformed -> they will be serialized 👀
* allows
  * 💡modules -- are combined, to generate -- >=1 bundles 💡 
* bundle == bundle of modules / combined into 1! JS file

## Modules

* Metro 
  * == MULTIPLE modules / 
    * 👀-- correspond to -- EVERY flow's step 👀
      * [metro-resolver](/packages/metro-resolver)
      * metro-transformer (TODO: Which one?)
      * metro-serializer (TODO: Which one?)
    * OWN responsibility 
