---
layout: post
title: Testing distributed systems using Python Hypothesis
date:   2026-06-05
categories: [technology, programming, testing, notes]
tags: [property-based testing, hypothesis, python]
description: TODO
---

## Background / The Problem

I've been building a [privacy, local-first recipe management app][smulbook] for a while, one of its main features is
being able to synchronize data between your personal devices. This literally makes this software system a **distributed
system**.

The most challenging part about designing and implementing a distributed system is keeping state in multiple devices consistent.
To achieve this, there are quite some important details to take care of, including (but not limited to):

1. Deciding on the data synchronization strategies: syncing data vs. syncing operations.
2. the data model which allows data to eventually converge among devices after data synchronization (think of things like **Conflict-free Replicated Data Type** (**[CRDT][crdt]**)). 
3. The data system is **idempotent** after applying the same *operation* ([CRUD][crud]) multiple times.
4. Deciding on the **conflict resolution** strategy and semantics; one common strategy is **Last-Write-Wins** (**[LWW][lww]**)
5. How to deal with deletion operation?
6. For syncing operations, how do you determine the order of operations living on different devices?
7. How do you keep track of syncing progress?

... and so on.

As you can see, this is not a trivial problem, there are a lot of details and nuances, so it is not easy to get it right.
And because of this, I wanted to *TEST* the system to make sure the behavior is as expected.

To test such system with a lot of state changes and edge cases, I decided to use a tool called [`hypothesis`][hypothesis]. It
is Python's [property-based testing library][hypothesis], which I think is the right tool for this job.


## What is `hypothesis` and its features

TODO


## How I use hypothesis to solve my testing problems

TODO


## What I Learned From solving the problem

TODO
hypothesis cannot be used / mixed with pytest parameterization? Describe your experience.


[smulbook]: https://keenhenry.gitlab.io/smulbook-website/
## Conclusion

TODO



[hypothesis]: https://hypothesis.readthedocs.io/en/latest/
[crdt]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
[crud]: https://en.wikipedia.org/wiki/Create,_read,_update_and_delete
[lww]: https://oneuptime.com/blog/post/2026-01-30-last-write-wins/view
