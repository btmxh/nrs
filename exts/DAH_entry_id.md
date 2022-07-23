# Extension specification: DAH_entry_id

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
```

## Overview

This extension introduces the concept of entry ID.

A valid entry ID mapping is a function, which takes an entry as input, and outputs something called the entry ID. This function is surjective (every entry in the entry set must have different ID).

ID is useful since entries is reference everywhere in the data model of NRS. 

An impact can be modeled as the following struct (in C++)

```cpp
// c++20 modules
import std;

struct ScoreVector;
struct Entry;

struct Impact {
    std::unordered_map<std::shared_ptr<Entry>, float> contributors;
    ScoreVector score;
}
```

But the problem is how we will serialize this data model, into JSON for example. There are two type of JSON containers, JSON array and JSON object, and both of them can't be used to model a `std::unordered_map<std::shared_ptr<Entry>, float>`.

To resolve this issue, we will model `Impact` differently, using entry IDs:

```cpp
// c++20 modules
import std;

struct ScoreVector;
struct Entry;
using EntryID = std::string;

struct Impact {
    std::unordered_map<EntryID, float> contributors;
    ScoreVector score;
}
```

## Modifications

This extension requires the existence of the entry ID mapping function, `EntryID(e)`, which takes an argument: entry `e` and outputs the entry ID for `e`. This function must be surjective.

The entry ID for `e` is not necessary a string.

## See Also

Extension `DAH_entry_id_impl` for rules to determine the entry ID from the entry.