---
title: "Upgrading Your Project to Cascalog 2.0"
layout: article
---

## Purpose

Cascalog 2.0 is a major improvement over 1.x. The underlying query parser remains the same but there are several notable breaking API changes. This is a guide to point out the major differences between 2.x and 1.x versions of Cascalog to help you make the upgrade.

## Breaking API changes at a glance

- Most namespaces have been moved into either `cascalog.cascading.*` or `cascalog.logic.*`
- Higher-order functions, extra vector around a function name, like `(defmapop [times [x]] [y] (* x y)) is no longer supported
- Built-in ops use anonymous function to set pameters. e.g. `(c/limit [1] ...)` is now `((c/limit 1) ...)`