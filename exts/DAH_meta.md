# Extension specification: DAH_meta

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
```

## 1. Overview

`DAH_meta` allows the implementor to put more more metadata into NRS entries,
impacts and relations. This extension introduces an additional property for
all NRS objects, called the `meta` object. This object is a JavaScript-like
object, and can be deserialized into JSON. By setting the fields of this object
to certain values, the implementor may add additional metadata into entries,
impacts and relations.
