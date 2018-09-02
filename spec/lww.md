# RON RDT: LWW-per-field

It seems very useful to implement not just plain last-write-wins (LWW) register,
but with Cassandra-inspired extension, namely the field.

LWW-per-field is the same as LWW register but identified by the pair
(**object**, **location**) in terms of RON.

Plain LWW may be expressed as a field (op location) with any specific value
(e. g. 0) of an LWW-per-field object.

From the other hand, an LWW-per-field object is a product of LWW registers.

So, with this type we can encode both a simple LWW register and a product of
LWW registers.

## 1. Type UUID

RON text:
```
lww
```

Hexadecimal:
```
0x c3be c000 0000 0000
   0000 0000 0000 0000
```

## 2. Raw Op Encoding

1.  **@event** encodes timestamp of write.

2.  **:location** encodes the object field.

### 2.1. Examples

1.  Write an integer to the default field
    ```
    *lww #38+37 @40+39 :0 =35 ;
    ```

2.  Write a tuple of an integer and a string to the field `alpha`
    ```
    *lww #38+37 @41+39 :alpha =42 'bravo' ;
    ```

3.  Simultaneous write to several fields of the same object
    ```
    *lww #38+37 @43+39 :alpha   =42     ;
    *lww #38+37 @43+39 :charlie 'bravo' ;
    ```
    the same with compression
    ```
    *lww #38+37 @43+39 :alpha   =42     ;
                       :charlie 'bravo' ;
    ```

## 3. Value Encoding
