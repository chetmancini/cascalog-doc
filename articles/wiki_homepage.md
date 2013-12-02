---
title: "Wiki homepage"
layout: article
---

# This is the Wiki homepage on Cascalog repository

Cascalog is a fully-featured data processing and querying library for Clojure. The main use cases for Cascalog are processing "Big Data" on top of Hadoop or doing analysis on your local computer from the Clojure REPL. Cascalog is a replacement for tools like Pig, Hive, and Cascading.

Cascalog operates at a significantly higher level of abstraction than a tool like SQL. More importantly, its tight integration with Clojure gives you the power to use abstraction and composition techniques with your data processing code just like you would with any other code. It's this latter point that sets Cascalog far above any other tool in terms of expressive power.

Follow the getting started steps, check out the tutorials, and you'll be running Cascalog queries on your local computer within 5 minutes.

Cascalog is hosted on Github at [[http://github.com/nathanmarz/cascalog]].

## Getting help

- [Cascalog Google Group](http://groups.google.com/group/cascalog-user)
- `#cascalog` room on Freenode
- [API Documentation](http://nathanmarz.github.com/cascalog/)

## Documentation

- [[Getting Started]]
- [[Introductory Tutorial, Part 1]]
- [[Introductory Tutorial, Part 2]]
- [[JCascalog]]
- [[Defining and executing queries]]
- [[How Cascalog executes a query]]
- [[Guide to custom operations]]
- [[Predicate operators]]
- [[Methods for handling wide sources]]
- [[Joins in Cascalog]]
- [[Built-in operations]]
- [[Predicate macros]]
- [[Sink functions and Cascalog taps]]
- [[Cascalog and Hadoop Security]]
* [[Troubleshooting, testing and live coding]]

## Articles around the web

### Overviews

- [Developing and deploying a Cascalog query on a Hadoop cluster](http://nathanmarz.com/blog/news-feed-in-38-lines-of-code-using-cascalog.html)
- [Why Yieldbot chose Cascalog over Pig for Hadoop processing](http://tech.backtype.com/52456836)
- [Which operation def macro should I use in Cascalog?](http://entxtech.blogspot.com/2010/12/which-operation-def-macro-should-i-use.html)
- [Cascalog made easier](http://jimdrannbauer.com/2011/02/04/cascalog-made-easier/)

### Tutorials

- [Cascalog for the Impatient](https://github.com/Quantisan/Impatient)
- [News Feed in 38 lines of Cascalog](http://nathanmarz.com/blog/news-feed-in-38-lines-of-code-using-cascalog.html)
- [Summarizing next-gen sequencing variation statistics with Hadoop using Cascalog](http://bcbio.wordpress.com/2011/07/04/summarizing-next-gen-sequencing-variation-statistics-with-hadoop-using-cascalog/)
- [Generator as filter / negations](http://groups.google.com/group/cascalog-user/browse_thread/thread/17bbe772159b8ffa)
- [Catching errors with traps](http://groups.google.com/group/cascalog-user/browse_thread/thread/f9257bf8002e053a)
- [Using Cascalog for Extract, Transform and Load](http://ianrumford.github.io/blog/2012/09/29/using-cascalog-for-extract-transform-and-load/)
- [JCascalog Trevni Example](https://github.com/mykidong/jcascalog-trevni-example)
- [JCascalog Parquet Example](https://github.com/mykidong/jcascalog-parquet-example)

### Clever Gists

- [Manipulating Clojure Maps in Cascalog](https://gist.github.com/sritchie/1444898)
- [A Thrush operator on Cascalog](https://gist.github.com/sritchie/1675672)
- [Keyword arguments and local-only JobConf](https://gist.github.com/sritchie/954c86f15962e3bd7928)

### Testing

- [Testing Cascalog with Midje](http://sritchie.github.com/2011/09/30/testing-cascalog-with-midje.html)
- [Testing Cascalog with Midje, Part 2](http://sritchie.github.com/2012/01/22/cascalog-testing-20.html)
- [Screen6: Testing Cascalog with Midje](http://blog.screen6.io/post/57428073723/introduction-to-testing-cascalog-with-midje)

## Documentation Todo

- [[Option predicates]]
- [[Building queries dynamically]]
- [[Query builders]]
- [[Global sorting]]
- [[Negations and generators-as-sets]]
- [[Tuple Serialization]]

## Other

- [[Who's using Cascalog]]