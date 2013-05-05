---
title: "Getting Started with Cascalog"
layout: article
---

## About this guide

This guide is a quick tutorial to help you get started with using Cascalog. It should take no more than 15 minutes to read and go through these examples. This guide covers:

- How to add Cascalog dependency to your project

## What version of Cascalog does this guide cover?

This guide covers Cascalog 1.10.1.

## Cascalog Overview

Cascalog is a fully-featured data processing and querying library for Clojure or Java. The main use cases for Cascalog are processing "Big Data" on top of Hadoop or doing analysis on your local computer. Cascalog is a replacement for tools like Pig, Hive, and Cascading and operates at a significantly higher level of abstraction than those tools.

## Adding Cascalog dependency to your project

Clojure artifacts are released to [Clojars](https://clojars.org/cascalog)

### With Leiningen

```clj
[cascalog "1.10.1"]
```

### With Maven

Add Clojars to your repositories.

```xml
<repository>
  <id>clojars.org</id>
  <url>http://clojars.org/repo</url>
</repository>
```

Then add Cascalog dependency to your project.

```xml
<dependency>
  <groupId>cascalog</groupId>
  <artifactId>cascalog</artifactId>
  <version>1.10.1</version>
</dependency>
```

## Use Cascalog through Java or Clojure?

The native implementation of Cascalog is in Clojure. However, Cascalog provides a pure-Java interface called JCascalog that is perfectly interoperable with the Clojure version. Choose whichever interface that is comfortable for you to get started.

## Getting started with JCascalog (Java)

### JCascalog overview

JCascalog is a pure-Java interface to Cascalog that comes bundled with Cascalog as of version 1.8.7. All the functionality available in Cascalog is available via the JCascalog interface. Moreso, Cascalog and JCascalog are perfectly interoperable: JCascalog subqueries, operations, and predicate macros can be used in regular Cascalog code and vice-versa.

## Getting started with Cascalog (Clojure)

