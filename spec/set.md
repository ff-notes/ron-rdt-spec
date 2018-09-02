# RON RDT: Observed-Remove Set (OR-Set)

The idea of Observed-Remove set is that a node can remove only values that it
has observed.
In practice, that means removal operations require ids of corresponding addition
operations.

## 1. Type UUID

RON text:
```
set
```

Hexadecimal:
```
0x 0de9 e000 0000 0000
   0000 0000 0000 0000
```

## 2. Single op representation

1.  Add:
    1.  **location** is 0
    2.  **payload** is the value
2.  Remove:
    1.  **location** is the version being deleted
    2.  **payload** is empty

### 2.1. Examples

1.  Replica `alfa` adds string `"bravo"` to the object `#32+charlie`

        *set #32+charlie @35+alfa :0 'bravo' ;

2.  Replica `delta` removes that value specifying its version

        *set #32+charlie @38+delta :35+alfa ;

## 3. Value representation

1.  Reduced op format:
    1.  Alive version (version that has not been removed)
        1.  **event** is the version
        2.  **location** is 0
        3.  **payload** is the value
    2.  Tombstone (removed version)
        1.  **event** is the original version
        2.  **location** is the event of the last removal op
        3.  **payload** is the value, for the purpose of undo
2.  The order of fields doesn't matter.
    For reproducibility, ops should be sorted by *event*.

### 3.1. Examples

1.  Empty object

        *set #32+charlie @32+charlie :0 !

2.  Replica `alfa` adds string `"bravo"` to the object `#32+charlie`

        *set #32+charlie @35+alfa :0         !
        *set #32+charlie @35+alfa :0 'bravo' ,

    the same with compression

        *set #32+charlie @35+alfa :0         !
                                     'bravo' ,

3.  Replica `echo` adds string `"bravo"`, too

        *set #32+charlie @72+echo :0         !
        *set #32+charlie @72+echo :0 'bravo' ,

    the same with compression

        *set #32+charlie @72+echo :0         !
                                     'bravo' ,

4.  Replicas `alfa` and `echo` exchange their ops (or states) and both got the
    same result value:

        *set #32+charlie @72+echo :0         !
        *set #32+charlie @35+alfa :0 'bravo' ,
        *set #32+charlie @72+echo :0 'bravo' ,

    the same with compression

        *set #32+charlie @72+echo :0         !
                         @35+alfa :0 'bravo' ,
                         @72+echo :0 'bravo' ,

5.  Replica `delta` got value from the replica `alfa`, but no from the replica
    `echo`. Then `delta` removes the string by its version known to it.

        *set #32+charlie @38+delta :35+alfa          ;

        *set #32+charlie @38+delta :0                !
                         @35+alfa  :38+delta 'bravo' ,

6.  When all replicas recieve all changes, the value at all replicas will be

        *set #32+charlie @7200000001+echo :0                !
                         @35+alfa         :38+delta 'bravo' ,
                         @72+echo         :0        'bravo' ,

## 4. Patch representation

The same as the value encoding.

## 5. Reduction

1.  For each item version in a value or patch there must be exactly one op with
    corresponding *event*.
2.  Removal has precedence over addition.
    For the same item version, the op with the maximum *location* is chosen.
