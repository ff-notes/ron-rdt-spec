# RON RDT General Terms and Algorithms

In the [RON] world, a replicated data type (RDT) is simultaneously:

1. a [CmRDT][CRDT], or op-based RDT
2. a [CvRDT][CRDT], i. e. a [semilattice], or state-based RDT

In other words, it must be representable both as a set of operations changing a value and as a value itself.

## 1. Common requirements for raw ops and chunk header ops

1.  **type** must identify object type and reducer function.
    Different op types corresponding to the same RDT must have the same type
    field.

    E. g. RGA type offers operations _Add_ and _Remove_,
    both are encoded with `rga` op type and are differentiated by other means.

2.  **object** must identify object.

## 2. Single op representation

An RDT must provide the single op representation.

A single (unreduced) op is represented as a RON raw op chunks

```
*type #object @event :location payload ;
```

where

1.  The pair (**event**, **location**) must be unique within ops of the
    same _type_ and _object_.

2.  If <em>op</em>₁ causally precedes <em>op</em>₂

    > <em>op</em>₁ ≺ <em>op</em>₂

    then their **events** must be increasing:

    > <em>event</em>(<em>op</em>₁) < <em>event</em>(<em>op</em>₂)

    Particularly, if a local clock is used to generate events,
    it may be needed to increase their values a bit to generate a
    causally-connected event.

## 3. Value representation

An RDT must provide the value representation.

A reduced value must be represented as a reduced chunk of such structure:

1.  Chunk header

    ```
    *type #object @version :ref !
    ```

    1.  **version** must be not less than *event* of any ops causally
        preceding this value.

        **version** should be equal to the maximum *event* of all ops causally
        preceding this value.
        A framework may provide a means to calculate maximum event and set chunk
        version automatically.

    2.  **version** must be *greater* than *version* of all values causally
        preceding this value.

    3.  **ref** must 0.

2.  followed by zero or more reduced ops

    ```
    *type #object @event :location payload ,
    ```

    1.  **type** and **object** must be equal to the *type* and *object* of the
        chunk header.

## 4. Patch representation

TODO

## 5. Reduction

TODO

[CRDT]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
[monoid]: https://en.wikipedia.org/wiki/Monoid
[RON]: https://github.com/gritzko/ron
[semilattice]: https://en.wikipedia.org/wiki/Semilattice
