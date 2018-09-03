# RON RDT: Replicated Growable Array (RGA)

RGA suits well for keeping any ordered collection.

For example, strings may be expressed as an RGA of characters
(codepoints, actually).

## 1. Type UUID

RON text:

    rga

Hexadecimal:

    0x 0dab 9400 0000 0000
       0000 0000 0000 0000

## 2. Single op representation

1.  Insert:
    1.  **ref** is the parent vertex id.
        1.  0 is the special "invisible root" vertex.
    2.  **payload** is the value.
2.  Remove:
    1.  **ref** is the target vertex id.
    2.  **payload** is empty.

### 2.1. Examples

1.  Replica `alfa` creates an RGA from string `"hi"` at time 27,
    inserting each character after previous one:

        *rga #27+alfa @27+alfa         :0       'h' ;
        *rga #27+alfa @2700000001+alfa :27+alfa 'i' ;

    with compression:

        *rga #27+alfa @`     'h' ;
                      @)1 :) 'i' ;

2.  Replica `bravo` capitalizes first letter by removing `@27+alfa 'h'` and
    inserting `'H'` in the root.

        *rga #27+alfa @42+bravo         :27+alfa     ;
        *rga #27+alfa @4200000001+bravo :0       'H' ;

    with compression:

        *rga #27+alfa @42+bravo :27+alfa     ;
                      @)1       :0       'H' ;

## 3. Value representation

1.  Chunk version is calculated as the maximum *event* or *ref* of contained
    ops.
2.  Reduced op format:
    1.  **event** is the vertex id
    2.  Alive vertex (vertex that has not been removed)
        2.  **ref** is 0
        3.  **payload** is the value
    3.  Tombstone (removed vertex)
        2.  **ref** is the event of the last removal op
        3.  **payload** is the value, for the purpose of undo
3.  The order of ops is specified in RGA definition.

### 3.1. Examples

1.  Empty RGA

        *rga #71+charlie @71+charlie :0 !

1.  Replica `alfa` creates an RGA from string `"hi"` at time 27,
    inserting each character after previous one:

        *rga #27+alfa @27+alfa         :0     !
        *rga #27+alfa @27+alfa         :0 'h' ,
        *rga #27+alfa @2700000001+alfa :0 'i' ,

    with compression:

        *rga #27+alfa @`      !
                          'h' ,
                      @)1 'i' ,

2.  Replica `bravo` capitalizes first letter by removing `@27+alfa 'h'` and
    inserting `'H'` in the root.

        *rga #27+alfa @27+alfa          :0            !
        *rga #27+alfa @4200000001+bravo :0        'H' ,
        *rga #27+alfa @27+alfa          :42+bravo 'h' ,
        *rga #27+alfa @2700000001+alfa  :0        'i' ,

    with compression:

        *rga #27+alfa @27+alfa                        !
                      @4200000001+bravo           'H' ,
                      @`                :42+bravo 'h' ,
                      @)1               :0        'i' ,

## 4. Patch representation

1.  The same as the value representation. Hence, a patch is a sub-array.
2.  The chunk **ref** is the parent vertex id.

## 5. Reduction

1.  For each vertex in a value or patch there must be exactly one op with
    corresponding *event*.
2.  Removal has precedence over addition.
    For the same vertex, the op with the maximum *ref* is chosen.
3.  Insertion and value reduction is specified by the RGA definition.
4.  Since patched are RGAs, too, patches are reduced by the same algorithm,
    if applicable.
