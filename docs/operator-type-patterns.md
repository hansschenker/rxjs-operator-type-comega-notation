# Operator Type Patterns

This document is a compact reference for recurring RxJS operator type shapes.

## Primitive shapes

```text
T!        exactly one T
T?        zero or one T
T+        one or more T values
T*        zero or more T values

A x B     product
A + B     sum
A -> B    function mapping

(T*)*     stream of streams
1         unit / lifecycle signal
```

## Notification and Stream

```text
Notification(T, E)
= T! + E! + 1
```

```text
Stream(T, E)
= T* x (1? + E?)
```

## Representative operators

```text
map
(T! -> R!) -> T* -> R*

filter
(T! -> Boolean!) -> T* -> T*
per value: T! -> T?

tap
(T! -> 1) -> T* -> T*

scan
(A! x T! -> A!) x A! -> T* -> A*

reduce
(A! x T! -> A!) x A! -> T* -> A!

take(1)
T* -> T?

toArray
T* -> Array<T>!

mergeAll / concatAll / switchAll / exhaustAll
(T*)* -> T*

mergeMap / concatMap / switchMap / exhaustMap
(T! -> R*) -> T* -> R*

combineLatest
A* x B* -> (A! x B!)*

zip
A* x B* -> (A! x B!)*

withLatestFrom
A* x B* -> (A! x B!)*

buffer
T* -> Array<T>*

window
T* -> (T*)*

materialize
Stream(T, E) -> Notification(T, E)*

dematerialize
Notification(T, E)* -> Stream(T, E)

share
T* -> T*

observeOn / subscribeOn
T* -> T*
```

## Rule

Equal structural types do not imply equal operator behavior.

```text
type
= what can flow

behavior
= state + trigger + action policy + time + concurrency
  + cancellation + lifecycle + sharing + scheduling
```
