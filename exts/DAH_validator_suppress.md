# Extension specification: DAH_validator_suppress

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
DAH_meta 1.0+
```

## 1. Overview

The validation process is defined as a process that analyze NRS output and
find all potential issues with the implementation. It consists of multiple
*subprocesses*, called validation rules, which are identified by its name.

This extension assumes that the entry `DAH_entry_progress` in the `meta` object
of all entries, impacts and relations to be a list of suppressed validation rules
for this object, i.e. the rules will ignore this object when they're run.
