# Extension specification: DAH_serialize

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
DAH_entry_id 1.0+
```

## Overview

This extension describes all the requirements to serialize **entries**, **impacts**, **relations** and **scores**. These requirements are described in the [Modification] section and in the form of dependencies (`DAH_entry_id` must be enabled).

## Modification

A two-way **conversion** between type A and type B is a surjective function `f: A -> B`. Since it's surjective, the inverse function, `inverse_f = f^-1: B -> A + {null}` is well-defined. When A is the same type as B, the identity function will be implicitly taken as the conversion between the two types.

There must be a two-way conversions between:

* Entry ID type and the String type

## See Also

Extension `DAH_serialize_json` for the JSON implementation of this extension.
