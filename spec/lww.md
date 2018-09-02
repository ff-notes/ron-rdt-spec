# RON RDT: LWW-per-field

It seems very useful to implement not just plain last-write-wins (LWW) register,
but with Cassandra-inspired extension, namely the field.

LWW-per-field is the same as LWW register but identified by the pair
(**object**, **location**) in terms of RON.

Plain LWW register may be expressed as a field of an LWW-per-field object with
any specific op *location*.
Let is be 0.

From the other hand, an LWW-per-field object is a product (vector) of LWW
registers.

So, with this type we can encode both a simple LWW register and a product
(vector) of LWW registers.

Also, single LWW register isn't a monoid, but LWW-per-field object is.

## 1. Type UUID

RON text:

    lww

Hexadecimal:

    0x c3be c000 0000 0000
       0000 0000 0000 0000

## 2. Single op representation

1.  **event** encodes timestamp of write.
2.  **location** encodes the object field.

### 2.1. Examples

1.  Write an integer to the default field

        *lww #38+foxtrot @40+golf :0 =35 ;

2.  Write a tuple of an integer and a string to the field `alfa`

        *lww #38+foxtrot @41+golf :alfa =42 'bravo' ;

3.  Simultaneous write to several fields of the same object

        *lww #38+foxtrot @43+golf :alfa    =42     ;
        *lww #38+foxtrot @43+golf :charlie 'bravo' ;

    the same with compression

        *lww #38+foxtrot @43+golf :alfa    =42     ;
                                  :charlie 'bravo' ;

## 3. Value representation

1.  Reduced chunks ops use the same format as raw ops.
2.  The order of fields doesn't matter.
    For reproducibility, ops should be sorted by *location*.

### 3.1. Examples

1.  Empty object

        *lww #38+foxtrot @38+foxtrot :0 !

2.  Ops from example 2.1.3 applied to empty object

        *lww #38+foxtrot @43+golf :0               !
        *lww #38+foxtrot @43+golf :alfa    =42     ,
        *lww #38+foxtrot @43+golf :charlie 'bravo' ,

    the same with compression

        *lww #38+foxtrot @43+golf :0               !
                                  :alfa    =42     ,
                                  :charlie 'bravo' ,

3.  Write string `"delta"` to the field `charlie` of the object `#38+foxtrot`
    from the previous example.
    Op:

        *lww #38+foxtrot @98+hotel :charlie 'delta' ;

    Result value:

        *lww #38+foxtrot @98+hotel :0               !
                         @43+golf  :alfa    =42     ,
                         @98+hotel :charlie 'delta' ,

## 4. Patch representation

The same as the value encoding.

### 4.1. Examples

1.  Ops from example 2.1.3 combined

        *lww #38+foxtrot @43+golf :43+golf         !
        *lww #38+foxtrot @43+golf :alfa    =42     ,
        *lww #38+foxtrot @43+golf :charlie 'bravo' ,

    the same with compression

        *lww #38+foxtrot @43+golf :43+golf         !
                                  :alfa    =42     ,
                                  :charlie 'bravo' ,

2.  Write string `"delta"` to the field `charlie` of the object `#38+foxtrot`
    and merge this op into the patch from the previous example.
    Op:

        *lww #38+foxtrot @98+hotel :charlie 'delta' ;

    Result patch:

        *lww #38+foxtrot @98+hotel :43+golf         !
                         @43+golf  :alfa    =42     ,
                         @98+hotel :charlie 'delta' ,

## 5. Reduction

1.  For each field in a value or patch there must be exactly one op with
    corresponding *location* if the field was set, and none otherwise.
2.  The result op for each field is selected by the LWW rule — which *event* is
    greater.
