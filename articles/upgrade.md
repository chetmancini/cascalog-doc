---
title: "Upgrading Your Project to Cascalog 2.0"
layout: article
---

## Purpose

Cascalog 2.0 is a major improvement over 1.x. The underlying query parser remains the same but there are several notable breaking API changes. This is a guide to point out the major differences between 2.x and 1.x versions of Cascalog to help you make the upgrade.

## Breaking API changes at a glance

- Most namespaces have been moved into either `cascalog.cascading.*` or `cascalog.logic.*`
  - `cascalog.ops` &#8594; `cascalog.logic.ops`
  - `cascalog.vars` &#8594; `cascalog.logic.vars`
- Higher-order functions using extra vector around a function name, like `(defmapop [times [x]] [y] (* x y))` is no longer supported in favour of using anonymous functions
- Built-in ops use anonymous functions to set pameters. For example, `(c/limit [1] ...)` is now `((c/limit 1) ...)`
- Functions can be passed directly instead of as vars. For example,

```
(def src [1 2 3 4 5]) 
(defn square [x] (* x x))
(defn my-query [op]
  (??<- [!x !y]
        (src !x)
        (op !x :> !y)))

(my-query #'square)
```

now becomes
```
(my-query square)
```

## Non-breaking changes
Cascalog operations are now just functions, so the `def*op` names have been deprecated in favor of names that sound like functions, `def*fn`. A few examples:
- `deffilterop` &#8594; `deffilterfn`
- `defbufferop` &#8594; `defbufferfn`
- `defmapop` &#8594; `defmapfn`
