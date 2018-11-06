# RON Replicated Data Types

## Data Format — RON

See [gritzko/ron](https://github.com/gritzko/ron)

## Specification

* [General terms and algorithms](spec/RDT.md)
* Base types
    * [LWW-per-field](spec/lww.md)
    * [OR-Set](spec/set.md)
    * [RGA](spec/rga.md)
    * [Version vector](spec/vv.md)
* Composite type construction
    *   Use object UUID to embed one object into another.
    *   For any type looking like an ordered collection,
        it is recommended to use *RGA*.
    *   For any type looking like an unordered collection,
        it is recommended to use *OR-Set*.
    *   For product types (structures, objects),
        it is recommended to use *LWW-per-field*.

## Test Data

See [ff-notes/ron-test](https://github.com/ff-notes/ron-test)

## Implementations

* **Elixir**
    *   [flanfly/ronex](https://github.com/flanfly/ronex)
* **Go**
    *   [gritzko/ron/rdt](https://github.com/gritzko/ron/tree/master/rdt)
* **Haskell**
    *   [ff-notes/ron/ron](https://github.com/ff-notes/ron/tree/master/ron),
        modules `RON.Data`, `RON.Data.*` and others
* **JavaScript**
    *   [gritzko/swarm/rdt](https://github.com/gritzko/swarm/tree/master/packages/rdt)
