# RON RDT General Terms and Algorithms

In the [RON] world, a replicated data type (RDT) is simultaneously:

1. a [CmRDT][CRDT], or op-based RDT
2. a [CvRDT][CRDT], i. e. a [semilattice], or state-based RDT

In other words, it must be representable both as a set of operations changing a value and as a value itself.

## 1. Common requirements for raw ops and chunk header ops

1.  **type** UUID must identify object type and reducer function.
    Different op types corresponding to the same RDT must have the same type
    field.

    E. g. RGA type offers operations _Add_ and _Remove_,
    both are encoded with `rga` op type and are differentiated by other means.

2.  **object** UUID must identify object.

## 2. Single op representation

1.  A single (unreduced) op is represented as a RON raw op chunks

    ```
    *type #object @event :location payload ;
    ```

2.  where

    1.  The UUID pair (**event**, **location**) must be unique within ops of the
        same _type_ and _object_.

    2.  If <em>op</em>₁ causally precedes <em>op</em>₂

        > <em>op</em>₁ ≺ <em>op</em>₂

        then their **event** UUIDs must be increasing:

        > <em>event</em>(<em>op</em>₁) < <em>event</em>(<em>op</em>₂)

        Particularly, if a local clock is used to generate events,
        it may be needed to increase their values a bit to generate a
        causally-connected event.

## 3. Patch representation

TODO

## 4. Value representation

TODO

[CRDT]: https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type
[monoid]: https://en.wikipedia.org/wiki/Monoid
[RON]: https://github.com/gritzko/ron
[semilattice]: https://en.wikipedia.org/wiki/Semilattice
