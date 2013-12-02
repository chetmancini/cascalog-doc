---
title: "Introducing Cascalog: a Clojure-based query language for Hadoop"
layout: article
---

# Introducing Cascalog: a Clojure-based query language for Hadoop

Note: This guide is adapted from [Nathan Marz's blog post](http://nathanmarz.com/blog/introducing-cascalog-a-clojure-based-query-language-for-hado.html) introducing the Cascalog project back in April 2010.

## Highlights

- **Simple** – Functions, filters, and aggregators all use the same syntax. Joins are implicit and natural.
- **Expressive** – Logical composition is very powerful, and you can run arbitrary Clojure code in your query with little effort.
- **Interactive** – Run queries from the Clojure REPL.
- **Scalable** – Cascalog queries run as a series of MapReduce jobs.
- **Query anything** – Query HDFS data, database data, and/or local data by making use of Cascading's "Tap" abstraction
Careful handling of null values - Null values can make life difficult. Cascalog has a feature called "non-nullable variables" that makes dealing with nulls painless.
- **First class interoperability with Cascading** - Operations defined for Cascalog can be used in a Cascading flow and vice-versa
- **First class interoperability with Clojure** - Can use regular Clojure functions as operations or filters, and since Cascalog is a Clojure DSL, you can use it in other Clojure code.

OK, let's jump into Cascalog and see what it's all about! I'm going walk us through Cascalog with a series of examples. These examples all make use of the "playground" that comes with the project. I recommend that you download Cascalog and follow along in your REPL (only takes a few minutes to get up and running - instructions are in the [Getting Started guide](/articles/getting_started.html)).

## Basic queries

First, let's start the REPL and load the playground:

```clj
lein repl
user=> (use 'cascalog.playground) (bootstrap)
```

This will import everything we need to run the examples. You can view the datasets we're going to be querying by looking at the playground.clj file. Let's run our first query and find the people in our dataset who are 25 years old:

```clj
user=> (?<- (stdout) [?person] (age ?person 25))
```

This query can be read as "Find all ?person for which ?person has an age that is equal to 25". You'll see logging from Hadoop as the job runs and after a few seconds the results of the query will print.

OK, let's try something more involved. Let's do a range query and find all the people in our dataset who are younger than 30:

```clj
user=> (?<- (stdout) [?person] (age ?person ?age) (< ?age 30))
```

That's pretty simple too. This time we bound the age of the person to the variable ?age and then added the constraint that ?age is less than 30.

Let's run that query again but this time include the ages of the people in the results:

```clj
user=> (?<- (stdout) [?person ?age] (age ?person ?age)
               (< ?age 30))
```

All we had to do was add the ?age variable into the vector within the query.

Let's do another query and find all the male people that Emily follows:

```clj
user=> (?<- (stdout) [?person] (follows "emily" ?person)
               (gender ?person "m"))
```

You may not have noticed, but there's actually a join happening in this query. The value of ?person must be the same wherever it is used, and since "follows" and "gender" are separate sources of data, Cascalog will use a join to resolve the query.

## Structure of a query

Let's look at the structure of a query in more detail. Let's deconstruct the following query:

```clj
user=> (?<- (stdout) [?person ?a2] (age ?person ?age)
              (< ?age 30) (* 2 ?age :> ?a2))
```

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

- (age ?person ?age)
- (< ?age 30)
- (* 2 ?age :> ?a2)

The :> keyword is used to separate input variables from output variables. If no :> keyword is specified, the variables are considered input variables for operations and output variables for generators and aggregators.

The "age" predicate refers to a tap defined in playground.clj, so it's a generator. That means that the "age" predicate emits variables "?person" and "?age".

The "<" predicate is a Clojure function. Since we didn't specify any output variables, the predicate will act as a filter and filter out any records where ?age is less than 30. If we had specified:

```clj
(< ?age 30 :> ?young)
```

In this case, "<" will act as a function and bind a new variable ?young as a boolean variable representing whether the person's age is less than 30.

The ordering of predicates doesn't matter. Cascalog is purely declarative.

## Variables and constant substitution

Variables are symbols that begin with either ? or !. Sometimes you don't care about the value of an output variable and can use the symbol "_" to ignore the variable. Anything else will be evaluated and inserted as a constant within the query. This feature is called "constant substitution" and we've already been making heavy use of it so far. Using a constant as an output variable acts as a filter on the results of the function. For example:

```clj
(* 4 ?v2 :> 100)
```

There are two constants being used here: 4 and 100. 4 substitutes for an input variable, while 100 acts as a filter only keeping the values of ?v2 that equal 100 when multiplied by 4. Strings, numbers, other primitives, and any objects that have Hadoop serializers registered can be used as constants.

Let's get back to the examples.

Let's find all follow relationships where someone is following a younger person:

```clj
user=> (?<- (stdout) [?person1 ?person2] 
    (age ?person1 ?age1) (follows ?person1 ?person2)
    (age ?person2 ?age2) (< ?age2 ?age1))
```

Let's do that query again and emit the age difference as well:

```clj
user=> (?<- (stdout) [?person1 ?person2 ?delta] 
    (age ?person1 ?age1) (follows ?person1 ?person2)
    (age ?person2 ?age2) (- ?age2 ?age1 :> ?delta)
    (< ?delta 0))
```

## Aggregators

Now let's check out our first aggregator. Let's find the number of people less than 30 years old:

```clj
user=> (?<- (stdout) [?count] (age _ ?a) (< ?a 30)
              (c/count ?count))
```

This computes a single value about all of our records. We can also aggregate over partitions of records. For example, let's find the number of people each person follows:

```clj
user=> (?<- (stdout) [?person ?count] (follows ?person _)
              (c/count ?count))
```

Since we declared ?person as a result variable of the query, Cascalog will partition the records by ?person and apply the c/count aggregator within each partition.

You can use multiple aggregators within a single query. They will run on the exact same partitions of records. For example, let's get the average age of people living in a country by combining a count and a sum:

```clj
user=> (?<- (stdout) [?country ?avg] 
   (location ?person ?country _ _) (age ?person ?age)
   (c/count ?count) (c/sum ?age :> ?sum)
   (div ?sum ?count :> ?avg))
```

Notice that we applied the "div" operation to the results of the aggregators for our final result. Any operations that are dependent on aggregator output variables will execute after the aggregators run.

## Custom operations

Next, let's write a query to count the number of times each word appears in a set of sentences. To do this, we are going to define a custom operation to use within the query:

```clj
user=> (defmapcatop split [sentence]
       (seq (.split sentence "\\s+")))

user=> (?<- (stdout) [?word ?count] (sentence ?s)
              (split ?s :> ?word) (c/count ?count))
```

"defmapcatop split" defines an operation that takes a single field "sentence" as input and outputs 0 or more tuples as output. deffilterop defines filter operations that return a boolean indicating whether or not to filter a tuple. defmapop defines functions that return a single tuple. defaggregateop defines an aggregator. These operations can also be used directly with Cascalog's workflow API - but that's for another blog post.

Our word count query has the problem in that the same word will be counted differently if it appears with different combinations of uppercase and lowercase letters. We can fix our query as follows:

```clj
user=> (defn lowercase [w] (.toLowerCase w))

user=> (?<- (stdout) [?word ?count] 
        (sentence ?s) (split ?s :> ?word1)
        (lowercase ?word1 :> ?word) (c/count ?count))
```

As you can see, regular Clojure functions can also be used as operations. A Clojure function is treated as a filter when not given any output variables. When given output variables, it is a map operation. Operations that emit 0 or more tuples must be defined using defmapcatop.

Here's a query that will return counts of people bucketed by age group and gender:

```clj
user=> (defn agebucket [age] 
        (find-first (partial <= age) [17 25 35 45 55 65 100 200]))

user=> (?<- (stdout) [?bucket ?gender ?count] 
        (age ?person ?age) (gender ?person ?gender)
        (agebucket ?age :> ?bucket) (c/count ?count))
```

## Non-nullable variables

Cascalog has a feature called "non-nullable variables" that allows you to handle null values gracefully. We've actually been using non-nullable variables this whole time. Variables prefixed with a "?" are non-nullable variables, and variables prefixed with a "!" are nullable variables. Cascalog inserts null checks to filter out any records in which a non-nullable variable is binded to null.

To see the effect of non-nullable variables, let's compare the following two queries:

```clj
user=> (?<- (stdout) [?person ?city] (location ?person _ _ ?city))

user=> (?<- (stdout) [?person !city] (location ?person _ _ !city))
```

The second query includes some null values in the result set.

## Subqueries

Finally, let's look at some more complex queries that make use of subqueries. Let's determine all the follow relationships in which both people follow more than 2 people:

```clj
user=> (let [many-follows (<- [?person] (follows ?person _)
                               (c/count ?c) (> ?c 2))]
        (?<- (stdout) [?person1 ?person2] (many-follows ?person1)
         (many-follows ?person2) (follows ?person1 ?person2)))
```

Here, we use a let form to define a subquery "many-follows". The subquery is defined using <-, the query definition operator. We can then make use of many-follows within the query we execute in the body of the let form.

We can also run queries that have multiple outputs. If we also want the result of many-follows in the query above, we can write:

```clj
user=> (let [many-follows (<- [?person] (follows ?person _)
                             (c/count ?c) (> ?c 2))
      active-follows (<- [?p1 ?p2] (many-follows ?p1)
                       (many-follows ?p2) (follows ?p1 ?p2))]
    (?- (stdout) many-follows (stdout) active-follows))
```

Here we define both of our queries without executing them. We then use the query execution operator ?- to bind each query to a tap. ?- executes both queries in tandem.

## Conclusion

Cascalog is being actively improved. You can expect more features to allow for richer queries and query planner improvements to be added over time.