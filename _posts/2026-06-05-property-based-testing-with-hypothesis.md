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
7. What is the protocol for syncing data among device?
8. How do you keep track of syncing progress?

... and so on.

As you can see, this is not a trivial problem, there are a lot of details and nuances, so it is not easy to get it right.
And because of this, I wanted to *TEST* the system to make sure the behavior is as expected.

To test such system with a lot of state changes and edge cases, I decided to use a tool called [`Hypothesis`][hypothesis].


## What is `Hypothesis` and its features

[`hypothesis`][hypothesis] is Python's [property-based testing library][hypothesis]. Property-based testing is a testing
technique that verifies general properties of code rather than specific examples. It generates random inputs to ensure that
certain conditions hold true across a wide range of scenarios, making it more efficient than traditional example-based testing.

In my case of testing a distributed system, a system with many possible state changes and edge cases, property-based testing is
exactly what I need.

[`Hypothesis`][hypothesis] is inspired by Haskell's [`QuickCheck`][quickcheck]. How does hypothesis work?

TODO: more details / introduction about the library.

The high level idea of using `Hypothesis` is the user specify only the *conditions* / *criteria* / *boundaries* of the inputs
for SUT, and `Hypothesis` randomly generates *GOOD* input test cases for you to exercise the SUT.

To describe the input criteria, `Hypothesis` introduces a concept called [**`strategy`**][strategy]:

A *strategy* is basically an algorithm / a way to generate test inputs. Of course, there are many ways / algorithms to produce
test inputs; for example, an input of *random integers* is one strategy, an input of *random strings* is another strategy.

If you're familiar with [**Strategy Design Pattern**][strategy-design-pattern], the term "strategy" here in Hypothesis framework
means exactly the same! It's literally the strategy for producing test inputs in the testing context.

To write a `Hypothesis` test, we use [`@given`][given] decorator with some *strategies* as its parameters to decorate
a testing function:

```python
from hypothesis import given
from tests.strategies import single_operation

@given(single_operation())
def test_operation_idempotency(op_data):
    """Test that applying the same operation multiple times does not cause duplicate entries
    or state changes.
    """

    ...
```

In the code above, there is a custom strategy function, `single_operation`, defined by me. There are also many [**built-in**
strategies][builtin-strategies] we can use. With `@given`, you're writing hypothesis tests yourself, `hypothesis` only supplies
test input data randomly.

There's another type of `hypothesis` test: [**Stateful Tests**][stateful]. In this type of test, `hypothesis` generates test sequences
itself to find failures. A state machine that runs itself to find bugs! How nice! To give a bit more details, you write the basic actions
the SUT can perform in combination, and also write some preconditions, invariants for each action, `Hypothesis` then figure out
the sequence of actions and perform the tests many times, trying its best to find bugs.


## How I use hypothesis to solve my testing problems

A Hypothesis stateful test fits the use case of testing a distributed system very well, because a distributed system is a complex
state machine! A tool like stateful test can help you automatically to find many different corner cases.

In a [Stateful Test][stateful], we need to define a few methods methods, which describe the behavior of the state machine:

### `Initialize`
### `Preconditions`
### `Rules`
### `Invariants`


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
[quickcheck]: https://www.cse.chalmers.se/~rjmh/QuickCheck/
[given]: https://hypothesis.readthedocs.io/en/latest/reference/api.html#hypothesis.given
[strategy-design-pattern]: https://en.wikipedia.org/wiki/Strategy_pattern
[strategy]: https://hypothesis.readthedocs.io/en/latest/reference/strategies.html
[builtin-strategies]: https://hypothesis.readthedocs.io/en/latest/tutorial/builtin-strategies.html
[stateful]: https://hypothesis.readthedocs.io/en/latest/stateful.html
