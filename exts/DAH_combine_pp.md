# Extension specification: DAH_combine_pp

Version 1.0

Vendor: DAH

Dependencies
```
NRS 2.0+
```

## Overview

This extension provides the combineUnsigned function based on polynomials

## Implementation (python code)

```py
def combineUnsigned(arr, w):
  ret = 0
  for x in arr.sort():
    ret = ret * w + x
  return ret
```

## See also

```DAH_combine_pow``` for an alternative combineUnsigned implementation.
