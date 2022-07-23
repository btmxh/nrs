# Extension specification: DAH_overall_score

Version 1.0

Vendor: DAH

Dependencies:
```
NRS 2.0+
```

## Overview

This extensions is used to combine factor scores in to a single number, called the overall score.

This extension requires subscores and their combine weight to be defined by another extension (see ```DAH_factors``` for an example).

The overall score calculating algorithm is fairly simple, using only the ```combine()``` function provided from the core specification.

* Overall score is the sum of all subscore values
* The subscore value is the combination of all factor score value, with the weight being the weight of the subscore
* The factor score value is the component of the overall score vector (from the core specification) at the factor index.

## See Also

Extension ```DAH_anime_normalize``` for a way to transform the overall score to a normalized score that suits common standards (5 is average, 6 is fine, etc.)
