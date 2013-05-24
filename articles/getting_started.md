---
title: "Getting Started with Cascalog"
layout: article
---

## About this guide

This guide is a quick tutorial to help you get started with using Cascalog. It should take about 15 minutes to go through these examples. This guide covers:

1. How to add Cascalog dependency to your project
2. Word count example in both Java and Clojure
3. Run a Cascalog query from Clojure nREPL
4. Introduction to basic concepts
5. Pointers for where to go next

## What version of Cascalog does this guide cover?

This guide covers Cascalog 1.10.1.

## Why Cascalog?

Work in progress

## Prerequisites

1. A Hadoop installation. You can [setup a local single-node Hadoop](http://wiki.apache.org/hadoop/GettingStartedWithHadoop) on your computer.
2. [Leiningen](https://github.com/technomancy/leiningen) or Maven.

## Adding Cascalog dependency to your project

Clojure artifacts are released to [Clojars](https://clojars.org/cascalog) repository.

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

## Word count example in JCascalog

### JCascalog overview

JCascalog is a pure-Java interface to Cascalog that comes bundled with Cascalog as of version 1.8.7. All the functionality available in Cascalog is available via the JCascalog interface. Moreso, Cascalog and JCascalog are perfectly interoperable: JCascalog subqueries, operations, and predicate macros can be used in regular Cascalog code and vice-versa.

### Example

Work In Progress

## Word count example in Cascalog

Ready to begin? Bring up the Clojure prompt using Leiningen with `lein repl` in your project. Once you see `user=>`, type in the following to use the main Cascalog API.

```clj
(use 'cascalog.api)
```

The prompt should print `nil` and return control to you. For ease of demonstration, also load the built-in example namespace.

```clj
(use 'cascalog.playground)
```

Now that we have everything ready, let's get some words to be counted. Conveniently, the `cascalog.playground` namespace contains a var `sentence` containing sentences. Take a look at it by typing:

```clj
sentence
```

Which prints out a series of vectors each of size 1.

All Cascalog queries contain a source and a sink. The most basic of which is just to read your data and directly write it back out.

```clj
(?- (stdout)
    sentence)
```

Which kicks off a job in your local hadoop node and prints off the result. `?-` is the query execution operator in Cascalog. It takes a sequence of \<output tap, query\> pairs, and executes all supplied queries in parallel. Here, we have defined the **output tap** on the first line as `(stdout)`, which prints to the standard output. The second line contains the query. In this example though, there is no querying as we merely specified the in-memory **generator** as the source. This particular query is similar to doing `SELECT * FROM sentence` in SQL.

To actually perform a query, we introduce the query creation operation -- `<-`.

```clj
(?- (stdout)
    (<- [?line]
        (sentence :> ?line)))
```

The result from this query is the same as the previous one. However, here we have an explicit query with **output field** between square brackets, `[?line]`, **generator** `sentence`, **input field**, also named `?line`. `:>` is one of two predicate operators in Cascalog which treats variables on its right as input to what is on its left.

This query is similar to performing `SELECT line FROM sentence` with SQL.

In a Cascalog query, we tell it what we want and the logic solver figures out how to do it. In this example, we wanted output `[?line]`. Which happens to be the same as input `?line`. The fact that the two fields have the same name is treated as though they are the same entity. Thus, given that we want `[?line]`, and input `?line` is fed by `sentence`, this query reads in data from our **generator** `sentence` and pipes it directly out.

WIP

In Cascading/Cascalog, the basic data structure is a **Tuple**. So in Cascalog terms, `sentence` is a series of Tuple of width one, or 1-Tuples.

