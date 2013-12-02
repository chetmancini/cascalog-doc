---
title: "Sources and Sinks in Cascalog"
layout: article
---

## About this guide

This guide explains how to use external data sources together with Cascalog. It will cover several ways
of getting information from database and writing it back there.

1. Taps: sources and sinks
2. Memory sources, Generators
3. Using Stdout as sink
4. Using HFS, LFS taps
5. Twitter Maple
6. Cascading-cassandra

## Sources and sinks

`Tap` is a term that's used to describe both `sources` and `sinks`. Tap represents a resource, like disk,
HDFS, external database, memory or anything else. If you can obtain data from tap, it's a source. If you
can write data to it, it's a sink. Source can be a Cascading tap, Cascalog tap or a subquery. Sink can be
a Cascading tap, Cascalog tap or sink function.

Cascalog provides two extra abstractions around taps: sink functions and cascalog taps. They can be useful if you need to write or read data that isn't directly readable by the available taps, or if you just want to remove redundant code around specific taps.

## Sink functions

Sink functions can be used to add an extra step before sinking to a tap. E.g. this is the original sink function posted on the mailing list by Nathan:

```clojure
(defn count-stdout [sq] 
  [(stdout) 
   (<- [?count] 
       (sq :>> (get-out-fields sq)) 
       (c/count ?count))] ) 
 
(?<- count-stdout 
  [?person ?age ?gender] 
  (age ?person ?age) 
  (gender ?person ?gender))
```

Instead of outputting the fields in the query `count-stdout` will actually output the number of rows in the dataset (`?count`).

The possibilities of what you do with this is only limited to what you can express in a single query. There is on important thing to keep in mind though. 

#### A sink function should return a vector with two values: a Cascading tap and a Cascalog query

E.g. if you want to use `hfs-seqfile` you have get the :sink field to get the underlying Cascading tap or it will not work:

```clojure
(defn count-hfs-seqfile [sq] 
  [(:sink (hfs-seqfile "foo")) 
   (<- [?count] 
       (sq :>> (get-out-fields sq)) 
       (c/count ?count))])
```

For cascalog-taps created with the function `cascalog-tap` (discussed in the next section) this is a bit trickier. Cascalog-taps provided a sink function itself so in order to make it work we need to call this function and give the query as argument to this function. For this example I use the keyval-tap of [Elephantdb](https://github.com/nathanmarz/elephantdb/blob/develop/elephantdb-cascalog/src/clj/elephantdb/cascalog/keyval.clj)
that requires exactly such a construct (Note that this taps requires an extra ?key field):

```clojure
(defn count-keyval-tap [sq] 
  ((:sink (keyval-tap "foo")) 
   (<- [?key ?count] 
       (sq :>> (get-out-fields sq)) 
       (identity "key" :> ?key)
       (c/count ?count))))
```

Now you understand how to use sink functions, it will be trivial to understand how use to `cascalog-tap`.

## Cascalog taps

Cascalog taps are like sink functions with one important addition: source queries. This allows you to control the output of this tap. Think of it as serializers, if a sink function is a serializer, a source query can be used as deserializer. Here is a silly example. Say, for some unknown reason, all my seqfiles need to have a special string at the front of each row, and we don't want to think about this during the execution of other queries. The following would do this:

```clojure
(defn custom-hfs-seqfile [path]
  (let [source-tap (<- [?field] ((hfs-seqfile path) _ ?field))
        sink-tap (fn [data] 
                   [(:sink (hfs-seqfile path))
                   (<- [?label ?field]
                     (data ?field)
                     (identity "my-label" :> ?label))])]
    (cascalog-tap source-tap sink-tap)))
```
We can write two simple queries to see what this does:

```clojure
(?- (custom-hfs-seqfile "foo") [[1]])
(?- (stdout) (custom-hfs-seqfile "foo"))
;; RESULTS
;; -----------------------
;; 1
;; -----------------------
```
This seems like the behavior of a normal hfs-seqfile until we use the normal `hfs-seqfile`:

```clojure
(?- (stdout) (hfs-seqfile "foo"))
;; RESULTS
;; -----------------------
;; my-label 1
;; -----------------------
```

By using cascalog-tap we have abstracted away the presence of the extra (serialization) step of adding and ignoring the extra field.

## Memory sources

LFS tap is used to reference local filesystem. It's especially useful when you're just starting up, or
experimenting with a dataset that comes in raw file format, or when your application is usually processing
files from local filesystem. Because local filesystem is neither distributed, nor replicated, using LFS
taps will force execution to single node.
