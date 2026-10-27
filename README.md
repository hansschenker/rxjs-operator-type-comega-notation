# RxJS Operator Type — Cω-Inspired Notation

A compact type notation for describing the **structural behavior of RxJS operators**.

The notation is inspired by the stream/cardinality notation associated with Erik Meijer and the Cω (C Omega) research language. The goal here is not to reproduce Cω as a language. The goal is to borrow its small, readable vocabulary for teaching and documenting RxJS operator types.

## Core idea

An RxJS operator description has at least two distinct layers:

1. **Type / dataflow shape** — what can flow through the operator.
2. **Behavioral policy** — how values are handled over time.

This repository focuses on the first layer.

```text
Cω-style type
    ↓
What can flow?

Mealy / BDD
    ↓
How does it behave over time?
```

This separation is important because many RxJS operators have the same structural type but different runtime policies.

For example:

```text
mergeMap
concatMap
switchMap
exhaustMap

(T! -> R*) -> T* -> R*
```

Their types are structurally identical, but their policies differ:

```text
mergeMap   = allow overlap
concatMap  = queue
switchMap  = only latest
exhaustMap = ignore while busy
```

---

## 1. Base Symbols

| Notation | Meaning |
|---|---|
| `T!` | exactly one required value of type `T` |
| `T?` | zero or one value of type `T` |
| `T+` | one or more values of type `T` |
| `T*` | zero or more values of type `T` |
| `A x B` | product type: both `A` and `B` |
| `A + B` | sum type: either `A` or `B` |
| `A -> B` | function from `A` to `B` |
| `(T*)*` | stream whose values are streams |
| `1` | unit/lifecycle signal carrying no domain value |

### Product type

```text
A! x T!
```

means:

> one value of type `A` together with one value of type `T`.

This is useful for reducer/state-transition functions:

```text
A! x T! -> A!
```

Read as:

> previous state and current input produce the next state.

### Sum type

```text
T! + E!
```

means:

> either a value of type `T` or an error of type `E`.

---

## 2. RxJS Notification Algebra

An Observable notification can be represented as:

```text
Notification(T, E)
= T! + E! + 1
```

Meaning:

```text
Next(T)
or Error(E)
or Complete
```

This matches the Observable lifecycle protocol:

```text
next*
then either
complete
or error
```

---

## 3. Stream Definition

A compact stream model is:

```text
Stream(T, E)
= T* x (1? + E?)
```

Read as:

> zero or more values of type `T`, followed by at most one terminal event: completion or error.

For everyday operator notation, the short form is usually enough:

```text
T*
```

Lifecycle/error details are included only when they matter to the operator being described.

---

## 4. General RxJS Operator Shape

A normal pipeable operator can be described as:

```text
T* -> R*
```

More explicitly:

```text
Stream(T, E) -> Stream(R, E')
```

Many operators also receive user functions. Their complete notation therefore includes both the user function and the stream transformation.

---

## 5. Core Operator Examples

### map

User function:

```text
T! -> R!
```

Operator:

```text
(T! -> R!) -> T* -> R*
```

Interpretation:

> each source value produces exactly one output value.

---

### filter

Predicate:

```text
T! -> Boolean!
```

Per-value cardinality:

```text
T! -> T?
```

Operator:

```text
(T! -> Boolean!) -> T* -> T*
```

Interpretation:

> each input value produces either zero or one output value.

---

### tap

Side effect:

```text
T! -> 1
```

Operator:

```text
(T! -> 1) -> T* -> T*
```

The stream shape is preserved.

---

## 6. scan

The reducer/accumulator function is:

```text
A! x T! -> A!
```

Read as:

> previous accumulator and current source value produce the next accumulator.

### scan with seed

```text
(A! x T! -> A!) x A! -> T* -> A*
```

Example:

```ts
scan((acc, curr) => acc + curr, 0)
```

For numbers:

```text
Number! x Number! -> Number!
```

and:

```text
(Number! x Number! -> Number!)
x Number!
->
Number* -> Number*
```

### scan without seed

When the first source value becomes the initial accumulator:

```text
(T! x T! -> T!) -> T+ -> T+
```

---

## 7. scan vs reduce

This notation makes the difference especially clear.

```text
scan:
(A! x T! -> A!) x A! -> T* -> A*

reduce:
(A! x T! -> A!) x A! -> T* -> A!
```

Meaning:

```text
scan   = evolving state over time
reduce = final state after completion
```

---

## 8. Cardinality-Changing Operators

### take(1)

```text
T* -> T?
```

### first

Ignoring the empty-stream error:

```text
T+ -> T!
```

Including the empty-stream behavior:

```text
T* -> T! + E!
```

### toArray

```text
T* -> Array<T>!
```

Many emitted values become one collection value.

---

## 9. Higher-Order Streams

A higher-order Observable is:

```text
(T*)*
```

Meaning:

> a stream whose emitted values are themselves streams.

This is the central structural shape behind flattening operators.

---

## 10. Flattening Operators

A projection function has the shape:

```text
T! -> R*
```

Mapping a stream with that function produces:

```text
T* -> (R*)*
```

Flattening then produces:

```text
(R*)* -> R*
```

Therefore all four mapping/flattening operators have the same structural type:

```text
(T! -> R*) -> T* -> R*
```

### mergeMap

```text
(T! -> R*) -> T* -> R*
```

Policy:

```text
allow overlap
```

### concatMap

```text
(T! -> R*) -> T* -> R*
```

Policy:

```text
queue
```

### switchMap

```text
(T! -> R*) -> T* -> R*
```

Policy:

```text
only latest
```

### exhaustMap

```text
(T! -> R*) -> T* -> R*
```

Policy:

```text
ignore while busy
```

### Explicit flattening

```text
mergeAll:
(T*)* -> T*

concatAll:
(T*)* -> T*

switchAll:
(T*)* -> T*

exhaustAll:
(T*)* -> T*
```

Their structural types are equal. Their temporal/concurrency policies are not.

---

## 11. Combination Operators

### combineLatest

For two sources:

```text
A* x B* -> (A! x B!)*
```

Two streams produce a stream of product values.

### zip

```text
A* x B* -> (A! x B!)*
```

Structurally the same as `combineLatest`, but the pairing policy differs.

```text
zip
= pair by position

combineLatest
= pair using latest remembered state
```

### withLatestFrom

```text
A* x B* -> (A! x B!)*
```

Behaviorally:

```text
A triggers
B supplies remembered state
```

---

## 12. Buffering and Windowing

### buffer

```text
T* -> Array<T>*
```

### window

```text
T* -> Observable<T>*
```

Abstractly, windowing creates a higher-order stream:

```text
T* -> (T*)*
```

The distinction matters in JavaScript:

```text
buffer -> values containing collections
window -> values containing Observables
```

---

## 13. Time-Sensitive Operators

Operators such as:

```text
delay
debounceTime
throttleTime
auditTime
sampleTime
```

often retain the same structural type:

```text
T* -> T*
```

That type does not describe their timing policy.

For a suppression operator such as `debounceTime`, a useful per-candidate view is:

```text
T! -> T?
```

but its actual semantics require a time/state model.

This is exactly where Mealy/BDD descriptions complement the type notation.

---

## 14. Error Operators

### catchError

If the handler is:

```text
E! -> R*
```

then a useful compact form is:

```text
(E! -> R*) -> T* -> (T + R)*
```

A lifecycle-aware form is:

```text
Stream(T, E1) -> Stream(T + R, E2)
```

### retry

```text
Stream(T, E) -> Stream(T, E)
```

The value type stays the same.

Its important behavior is the resubscription policy, which is not visible in the structural signature.

---

## 15. materialize / dematerialize

Define:

```text
Notification(T, E)
= T! + E! + 1
```

Then:

```text
materialize:
Stream(T, E)
->
Notification(T, E)*
```

and:

```text
dematerialize:
Notification(T, E)*
->
Stream(T, E)
```

This gives a direct type-level description of turning lifecycle events into ordinary stream values and back.

---

## 16. Sharing Operators

At the value level:

```text
share:
T* -> T*
```

But the essential change is execution topology:

```text
before:

subscriber 1 -> execution 1
subscriber 2 -> execution 2

after share:

subscriber 1 \
              -> shared execution
subscriber 2 /
```

Therefore sharing requires a behavioral annotation in addition to the Cω-style type.

---

## 17. Scheduling Operators

Operators such as `observeOn` and `subscribeOn` often retain:

```text
T* -> T*
```

The structural type does not change.

The changed dimension is:

```text
when work or notifications are scheduled
```

Again, type and behavior should remain separate.

---

## 18. Operator Family Cheat Sheet

| Operator family | Cω-style shape |
|---|---|
| one-to-one | `T* -> R*` |
| filtering | `T! -> T?` / `T* -> T*` |
| accumulation | `(A! x T! -> A!) -> T* -> A*` |
| reduction | `(A! x T! -> A!) -> T* -> A!` |
| limiting | `T* -> T?`, `T* -> T*` |
| flattening | `(T*)* -> T*` |
| higher-order projection | `(T! -> R*) -> T* -> R*` |
| combining | `A* x B* -> (A! x B!)*` |
| buffering | `T* -> Array<T>*` |
| higher-order grouping | `T* -> (T*)*` |
| lifecycle materialization | `Stream(T,E) -> Notification(T,E)*` |
| error transformation | `Stream(T,E1) -> Stream(R,E2)` |
| sharing | `T* -> T*` + execution policy |
| scheduling | `T* -> T*` + scheduling policy |

---

## 19. Documentation Model

The notation is intended to be the first layer of a fuller RxJS operator description.

```text
1. Cω-style Type
   What can flow?

2. Mealy Machine
   What state exists?
   What input events cause transitions?

3. BDD
   Given state
   When event
   Then actions

4. Plain Language
   Teaching explanation
```

Example for `switchMap`:

```text
Cω-style type

(T! -> R*) -> T* -> R*
```

Then the behavioral layer can explain:

```text
Given an inner stream is active
When a new outer value arrives
Then unsubscribe from the current inner
     project the new outer value
     subscribe to the new inner
```

The type tells us **what flows**.

The behavioral model tells us **how it flows over time**.

---

## 20. Design Principle

The notation deliberately stays small.

It should answer questions such as:

- How many values can flow?
- Does one input become zero, one, or many outputs?
- Does the operator return values or streams?
- Does it combine multiple input streams?
- Does it collapse a stream into one final value?
- Does it expose lifecycle notifications as values?

It should **not** be overloaded with every RxJS semantic dimension.

Time, cancellation, concurrency, ordering, subscription, sharing, scheduling, and termination policy belong in the behavioral description.

That gives a clean separation:

```text
Cω-style notation
= dataflow shape

Mealy / BDD
= behavior over time
```

---

## Status

Initial notation and representative operator families are documented.

Next logical step:

> derive a Cω-style signature for every RxJS 7.8.2 operator and maintain an operator index.

---

## Attribution

This repository was developed collaboratively by Hans Schenker and **GPT-5.6 Sol (OpenAI)**.

GPT-5.6 Sol contributed substantially to the notation design, operator classification, examples, and distinction between structural type and behavioral policy.

## License

Documentation and examples may be reused and adapted with attribution.
