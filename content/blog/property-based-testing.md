---
title: Property-based testing
date: 2026-09-26
slug: property-based-testing
summary: What property-based tests are, how generators, properties, and runners work together, four common property patterns, and examples in Python with Hypothesis.
---

When we think about testing we usually remember the classic test pyramid, with unit tests at the bottom and e2e tests at the top, but there is much more around testing and testing strategies than just that.

![The test pyramid: many unit tests at the bottom, fewer integration tests in the middle, and few end-to-end tests at the top](/images/property-based-testing/test-pyramid.png)

There are MANY different kinds of tests like load tests, performance tests, regression tests, and there are also property-based tests, but what are property-based tests?

Property-based tests are basic tests that, as the name suggests, are based on a given property, but what's a property? A property can be seen as a function f(x) -> boolean, so given an input it outputs true or false. For example, imagine that we're creating a sorting algorithm and we want to validate its implementation with tests. A sorting algorithm has many properties (conditions) that it must meet, but we can list at least two: for ALL consecutive x,y in the sorted list, x <= y, and the other property is that `len(sorted(list)) == len(list)`.

## How it works

Property-based testing has 3 main concepts, a generator, a property and a runner. A property is a function that must ALWAYS return true for any input to the code under test. A generator is the code responsible for generating, given a definition, an arbitrary number of inputs (usually 100, but it's configurable). The runner, as its name suggests, is responsible for validating the generated inputs against the properties defined until one fails - it does more than that, as it then tries to shrink it to the smallest example that breaks the property.

![The generator creates lists, the property checks them, and after a failure the runner shrinks the input down to [0, 0]](/images/property-based-testing/how-it-works.gif)

## Why use it

The main reason for using it is the way of thinking about tests. Usually we think about arbitrary test inputs for the function, and this has bias. When we start to think about properties we shift our way of thought to try to extract properties inherent to the code that we're trying to test. So we shift from thinking about which tests to write to writing code that generates tests from probability distributions. More inputs get tested, which makes it more likely to find edge cases before they reach the end users.

![Three hand-picked inputs all miss a bug in the negative numbers, while generated inputs spread across the whole range, pile up on edges like 0 and huge values, and hit the bug many times](/images/property-based-testing/probability-distribution.gif)

## Finding properties

The properties can be as custom as you want them to be, but there are some common patterns already, and we can highlight 4:

### 1. Round-trip

`g(f(x)) == x` -> common use case is serialization/deserialization

![A value is encoded, decoded back, and compared with the original](/images/property-based-testing/round-trip.gif)

### 2. Invariant

Something that never changes, the `len(list) == len(sorted(list))` for example

![Bars being sorted while the length, the sum, and the elements stay the same](/images/property-based-testing/invariant.gif)

### 3. Idempotency

`f(f(x)) == f(x)`. This property is essential in distributed systems and when we're handling money operations.

![The same charge request is retried three times, but the account is only charged once](/images/property-based-testing/idempotency.gif)

### 4. Oracle

`f(x) == g(x)`. This is very useful when we are refactoring some code or making changes. The same idea powers migrations in shadow mode.

![The same input goes to the legacy and the new implementation, and the outputs are compared until a negative price makes them diverge](/images/property-based-testing/oracle.gif)

## Examples with Hypothesis

In Python, the most used library for property-based testing is [Hypothesis](https://hypothesis.readthedocs.io/). It plays the generator and the runner roles, so we only need to describe the inputs with strategies and write the property as a regular test. It works with pytest out of the box:

```bash
pip install hypothesis pytest
```

### Round-trip

For serialization, we can generate any JSON-like value and check that encoding and decoding gives us the same value back. `st.recursive` builds nested lists and dictionaries from the basic types:

```python
import json

from hypothesis import given
from hypothesis import strategies as st

json_values = st.recursive(
    st.none() | st.booleans() | st.integers() | st.text(),
    lambda children: st.lists(children) | st.dictionaries(st.text(), children),
)


@given(json_values)
def test_json_round_trip(value):
    assert json.loads(json.dumps(value)) == value
```

`json_values` is the generator, the `assert` is the property, and `@given` hands it to the runner.

Try adding `st.floats()` to the base types: Hypothesis will usually find `nan`, and since `nan != nan`, the property fails (and `json.dumps` writes it as `NaN`, which isn't even valid JSON). That's exactly the kind of edge case we wouldn't pick by hand.

### Invariant

Now a sorting function that has a bug: it uses a `set` under the hood, so it drops duplicated values.

```python
def my_sort(xs):
    return sorted(set(xs))  # bug: drops duplicates


@given(st.lists(st.integers()))
def test_sort_keeps_length(xs):
    assert len(my_sort(xs)) == len(xs)
```

Running `pytest` fails with:

```text
E       assert 1 == 2
E        +  where 1 = len([0])
E        +    where [0] = my_sort([0, 0])
E        +  and   2 = len([0, 0])
E       Failing test case: test_sort_keeps_length(
E           xs=[0, 0],
E       )
```

The first list that broke the property was probably something bigger and noisier, but Hypothesis shrank it to `[0, 0]`, the smallest input that shows the bug: two equal values.

The length alone doesn't prove that a function sorts, though. For that, we check that each element is less than or equal to the next one, and that the result has exactly the same elements as the input:

```python
from collections import Counter


@given(st.lists(st.integers()))
def test_sort_is_ordered(xs):
    result = my_sort(xs)
    assert all(a <= b for a, b in zip(result, result[1:]))
    assert Counter(result) == Counter(xs)
```

The order check passes, but the `Counter` check fails on `[0, 0]` too, since the duplicate is gone. Once `my_sort` returns `sorted(xs)`, both tests pass.

### Idempotency

For a charge, retrying with the same `order_id` must leave the ledger in the same state as charging once:

```python
def charge(ledger, order_id, amount):
    if order_id not in ledger:
        ledger[order_id] = amount
    return ledger


@given(st.uuids(), st.integers(min_value=1), st.integers(min_value=1, max_value=10))
def test_charge_is_idempotent(order_id, amount, retries):
    once = charge({}, order_id, amount)

    ledger = {}
    for _ in range(retries):
        ledger = charge(ledger, order_id, amount)

    assert ledger == once
```

### Oracle

When rewriting a function, the old implementation becomes the source of truth for the new one. Here the new version looks simpler, but it forgot that the legacy one never returns a negative price:

```python
def legacy_discount(price):
    if price <= 0:
        return 0
    return round(price * 0.9, 2)


def new_discount(price):
    return round(price * 0.9, 2)  # forgot the negative case


@given(st.floats(min_value=-1_000, max_value=1_000, allow_nan=False))
def test_new_discount_matches_legacy(price):
    assert new_discount(price) == legacy_discount(price)
```

```text
E       assert -0.9 == 0
E        +  where -0.9 = new_discount(-1.0)
E        +  and   0 = legacy_discount(-1.0)
E       Failing test case: test_new_discount_matches_legacy(
E           price=-1.0,
E       )
```

We didn't write a single test case with a negative price, and it was found anyway.

### Settings and replay

By default, Hypothesis runs 100 examples per test. We can raise it for the properties that matter the most:

```python
from hypothesis import given, settings


@settings(max_examples=500)
@given(st.floats(min_value=-1_000, max_value=1_000, allow_nan=False))
def test_new_discount_matches_legacy(price):
    assert new_discount(price) == legacy_discount(price)
```

It also saves the failing examples in a local database and replays them first on the next run, so a bug it found once doesn't disappear on the next random run.

## Wrapping up

Property-based tests don't replace the tests we already have. They add another way of looking at the same code. Instead of asking "which inputs should I test?", we ask "what must always be true?", and let the generator explore inputs we wouldn't think of. To start:

- Look for round-trips: anything we encode, we should be able to decode.
- Look for invariants: what doesn't change after the function runs.
- Look for idempotency: operations that can be retried, especially with money.
- Use the old code as an oracle when refactoring or migrating.
- Trust the shrinking: the smallest failing example usually points straight to the bug.

A good first step is picking a function that already has a few example-based tests and asking which property those examples were trying to check.
