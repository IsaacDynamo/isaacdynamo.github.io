---
layout: post
title:  Async Memory Read
date:   2022-06-02 21:54:34 +0200
categories: async rust
published: false
---

# Intro

> "rust should make memory access asynchronous because it's technically a blocking io operation"

This was the witty comment that someone on discord made, that motivated me to create this post.
The comment reminded me of a talk that I had seen a couple years ago, where they used C++ coroutines to improve performance of code that was hindered by cache misses.

I wondered if this could also be done with async Rust.

I took this as an good opportunity to have some fun, get some practice in Rust, to write a small async executer, interact with x86 caches and memory, and use Criterion.

# The Talk

In the talk, [Nano-coroutines to the Rescue!](https://www.youtube.com/watch?v=j9tlJAqMV7U), Gor Nishanov explains that some workloads can lose a lot of performance due to cache misses. One example is binary search in a big database. A search will perform a chain of depended loads that will stall the processor because the indexed data is unlikely to be cached if the database is big.

During a memory-stall a lot of processor time is just lost while waiting on the memory. Can something useful be done while waiting on the memory. One single search is sequential, so there is no other work to do until we know the value of the read. But if we need to perform multiple searches, which is common in a database JOIN operation, the searches can be done in parallel. An with this extra parallelism we can perform other useful work while waiting on the memory.

This key insight is nicely visualized on the following slide of the talk.

![](/assets/nano_coroutines_slide_20.png)




# Recreation in Rust