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

## Memory sources

LFS tap is used to reference local filesystem. It's especially useful when you're just starting up, or
experimenting with a dataset that comes in raw file format, or when your application is usually processing
files from local filesystem. Because local filesystem is neither distributed, nor replicated, using LFS
taps will force execution to single node.
