# Extension specification: DAH_combine_pow

Version 1.0

Vendor: DAH

Dependencies
```
NRS 2.0+
```

## Overview

This extension provides the combineUnsigned function using exponentiation functions

## Implementation (python code)

```py
epsilon = 1e-4 # implementation-defined, but should be a small value
def combineUnsigned(arr, w):
  if w < epsilon:
    return max(arr, default = 0)

  return pow(sum([pow(x, 1/w) for x in arr]), w)
```

## See also

```DAH_combine_pp``` for an alternative combineUnsigned implementation.
