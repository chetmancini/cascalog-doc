---
title: "Getting Started with Cascalog"
layout: article
---

## About this guide

This guide is a quick tutorial to help you get started with using Cascalog. It should take about 15 minutes to go through these examples. This guide covers:

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

### Example (TODO)

## Getting started with Cascalog (Clojure)

First, let's start the REPL and load the playground:

    lein repl
    user=> (use 'cascalog.playground) (bootstrap)

This will import everything we need to run the examples. You can view the datasets we're going to be querying by looking at the playground.clj file. Let's run our first query and find the people in our dataset who are 25 years old:

    user=> (?<- (stdout) [?person] (age ?person 25))

This query can be read as "Find all ?person for which ?person has an age that is equal to 25". You'll see logging from Hadoop as the job runs and after a few seconds the results of the query will print.

OK, let's try something more involved. Let's do a range query and find all the people in our dataset who are younger than 30:

    user=> (?<- (stdout) [?person] (age ?person ?age) (< ?age 30))

That's pretty simple too. This time we bound the age of the person to the variable ?age and then added the constraint that ?age is less than 30.

Let's run that query again but this time include the ages of the people in the results:

    user=>  (?<- (stdout) [?person ?age] (age ?person ?age)
            (< ?age 30))

All we had to do was add the ?age variable into the vector within the query.

Let's do another query and find all the male people that Emily follows:

    user=>  (?<- (stdout) [?person] (follows "emily" ?person)
            (gender ?person "m"))

You may not have noticed, but there's actually a join happening in this query. The value of ?person must be the same wherever it is used, and since "follows" and "gender" are separate sources of data, Cascalog will use a join to resolve the query.

## Structure of a query

Let's look at the structure of a query in more detail. Let's deconstruct the following query:

    user=>  (?<- (stdout) [?person ?a2] (age ?person ?age)
            (< ?age 30) (* 2 ?age :> ?a2))

The query operator we've been using is ?<-, which both defines and runs a query. ?<- wraps around <-, the query creation operator, and ?-, the query execution operator. We'll see how to use those later on to create more complex queries.

First, we tell the query where we want to emit the results. In this case, we say "(stdout)". "(stdout)" creates a Cascading tap which writes its contents to standard output after the query finishes. Any Cascading tap can be used for the output. This means you can output data in any file format you want (i.e. Sequence files, text format, etc.) and anywhere you want (locally, HDFS, database, etc.).

After we define our sink, we define the result variables of the query in a Clojure vector. In this case, we are interested in the variables ?person and ?a2.

Next, we specify one or more "predicates" that define and constrain the result variables. There are three categories of predicates:

1. Generators: A generator is a source of data. Two kinds:
    - Cascading Tap - for example, the data on HDFS at a certain path
    - An existing query defined using <-
2. Operations: Implicit relations that take in input variables defined elsewhere and either act as a function that binds new variables or a filter
3. Aggregators: Count, sum, min, max, etc.

A predicate has a name, a list of input variables, and a list of output variables. The predicates in our query above are:

    (age ?person ?age)
    (< ?age 30)
    (* 2 ?age :> ?a2)

The :> keyword is used to separate input variables from output variables. If no :> keyword is specified, the variables are considered input variables for operations and output variables for generators and aggregators.

The "age" predicate refers to a tap defined in playground.clj, so it's a generator. That means that the "age" predicate emits variables "?person" and "?age".

The `<` predicate is a Clojure function. Since we didn't specify any output variables, the predicate will act as a filter and filter out any records where ?age is less than 30. If we had specified:

    (< ?age 30 :> ?young)

In this case, `<` will act as a function and bind a new variable ?young as a boolean variable representing whether the person's age is less than 30.

The ordering of predicates doesn't matter. Cascalog is purely declarative.

## Variables and constant substitution

Variables are symbols that begin with either ? or !. Sometimes you don't care about the value of an output variable and can use the symbol "_" to ignore the variable. Anything else will be evaluated and inserted as a constant within the query. This feature is called "constant substitution" and we've already been making heavy use of it so far. Using a constant as an output variable acts as a filter on the results of the function. For example:

    (* 4 ?v2 :> 100)

There are two constants being used here: 4 and 100. 4 substitutes for an input variable, while 100 acts as a filter only keeping the values of ?v2 that equal 100 when multiplied by 4. Strings, numbers, other primitives, and any objects that have Hadoop serializers registered can be used as constants.

Let's get back to the examples.

Let's find all follow relationships where someone is following a younger person:

    user=> (?<- (stdout) [?person1 ?person2] 
        (age ?person1 ?age1) (follows ?person1 ?person2)
        (age ?person2 ?age2) (< ?age2 ?age1))

Let's do that query again and emit the age difference as well:

    user=> (?<- (stdout) [?person1 ?person2 ?delta] 
        (age ?person1 ?age1) (follows ?person1 ?person2)
        (age ?person2 ?age2) (- ?age2 ?age1 :> ?delta)
        (< ?delta 0))

## Aggregators

Now let's check out our first aggregator. Let's find the number of people less than 30 years old:

    user=> (?<- (stdout) [?count] (age _ ?a) (< ?a 30)
                  (c/count ?count))

This computes a single value about all of our records. We can also aggregate over partitions of records. For example, let's find the number of people each person follows:

    user=> (?<- (stdout) [?person ?count] (follows ?person _)
                  (c/count ?count))

Since we declared ?person as a result variable of the query, Cascalog will partition the records by ?person and apply the c/count aggregator within each partition.

You can use multiple aggregators within a single query. They will run on the exact same partitions of records. For example, let's get the average age of people living in a country by combining a count and a sum:

    user=> (?<- (stdout) [?country ?avg] 
       (location ?person ?country _ _) (age ?person ?age)
       (c/count ?count) (c/sum ?age :> ?sum)
       (div ?sum ?count :> ?avg))

Notice that we applied the "div" operation to the results of the aggregators for our final result. Any operations that are dependent on aggregator output variables will execute after the aggregators run.