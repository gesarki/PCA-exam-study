# Promql Operators
Pormql support logical and arithmetic operators

## Arithmetic binary operators
- + (addition)
- - (subtraction)
- * (multiplication)
- / (division)
- % (modulo)
- ^ (power/exponentiation)

### b/n two scalars
Obvious result: evaluate to another scalar that is the result of the operator applied to both scalar operands

### b/n instant vector and scalar
operator applied to every data sampe in the vector. **Metric name is dropped**

### between two instant vectors
arithmetic operator applied to each entry in the left had side vector and its matching element in the right hand vector. Result is propagated into the result vector with the grouping labels becoming the ouput label set. **Metric name is dropeed**


## Comparison binary operators
- == (equal)
- != (not-equal)
- > (greater-than)
- < (less-than)
- >= (greater-or-equal)
- <= (less-or-equal)

Defined between scalar/scalar, vector/scalar and vector/vector value paris. By default they filter. Their behaviour can be changed by providing `bool` after the operator, which will return 0 or 1 for the value rather than filtering

### b/n instant vector and a scalar
operators applied to the value of every data sample in the query, and vector elements between which the comparison result is false get dropped from the result vector.

### between two instant vectors
behave as a filter by default, applied to matching entries. Vector elements for which the expresion is not true or which do not find a match on the other side of the expression get dropped from the result, while others are propagated into a result vector with the grouping labels becoming the ouput label set.

## Logical/set binary operators
- and (intersection)
- or (union)
- unless (complement)

`vector1 and vector2` results in a vector consisting of the elements of vector1 for which there are elements in vector2 with exactly matching label sets. Other elements are dropped. **The metric name and values are carried over from the left-hand side vector**

`vector1 or vector2` results in a vector that contains all original elements (label sets + values) of `vector1` and additionally all elements of vector2 which do not have matching label sets in vector1