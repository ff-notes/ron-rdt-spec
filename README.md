# RON Replicated Data Types

## Data Format — RON

See [gritzko/ron](https://github.com/gritzko/ron)

## Specification

* [General terms and algorithms](spec/RDT.md)
* Concrete types
    * [LWW-per-field](spec/lww.md)
    * [OR-Set](spec/set.md)
    * [RGA](spec/rga.md)
    * [Version vector](spec/vv.md)

## Test Data

See [ff-notes/ron-test](https://github.com/ff-notes/ron-test)

## Implementations

* **Go**
    *   [gritzko/ron/rdt](https://github.com/gritzko/ron/tree/master/rdt)
* **Haskell**
    *   [ff-notes/swarm/ron](https://github.com/ff-notes/swarm/tree/master/ron),
        modules `RON.Data`, `RON.Data.*` and others
* **JavaScript**
    *   [gritzko/swarm/rdt](https://github.com/gritzko/swarm/tree/master/packages/rdt)
