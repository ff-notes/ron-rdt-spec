# RON RDT: Version Vector

> https://en.wikipedia.org/wiki/Version_vector

A version vector is a set of timestamps, one timestamp for each replica.
In RON, this set is formed of UUIDs.

## 1. Type UUID

RON text:

    vv

Hexadecimal:

    0x 0eba 0000 0000 0000
       0000 0000 0000 0000

## 2. Single op representation

**event** encodes timestamp. **ref** is 0. **payload** is empty.

### 2.1. Examples

1.  Replica `alfa` measures its local time as 23 and creates a version vector:

        *vv #23+alfa @23+alfa :0 !

2.  Replica `bravo` measures its local time as 27 and adds it to the object
    `#23+alfa`:

        *vv #23+alfa @27+bravo :0 !

## 3. Value representation

1.  Reduced chunks ops use the same format as raw ops.
2.  The order of fields doesn't matter.
    For reproducibility, ops should be sorted by *event origin*.

### 3.1. Examples

1.  Version vector object is normally created with exactly one measurement:

        *vv #42+charlie @42+charlie :0 !
        *vv #42+charlie @42+charlie :0 ,

    the same with compression:

        *vv #42+charlie @42+charlie :0 !
                                       ,

    or with even more compression

        *vv#42+charlie@`!,

## 4. Patch representation

The same as the value encoding.

## 5. Reduction

1.  For each replica in a value or patch there must be exactly one op with
    corresponding *event origin* if the replica has commited to the vector,
    and none otherwise.
2.  The result op for each origin is selected by the LWW rule — which
    *event value* is greater.
