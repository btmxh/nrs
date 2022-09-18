# Extension specification: DAH_serialize_json

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
DAH_entry_id 1.0+
DAH_serialize 1.0+
```

## Overview

This extension is the implementation of the serialization process of
**impacts**, **relations**, **entries**, and **scores**.

The serialization process will output 4 JSON file, which are named
"impacts.json", "relations.json", "entries.json", "scores.json".

Here are the structures of these 4 JSON files. See the appendix for some examples.

### entries.json

As its name suggests, this file stores entry data.

It's an ID map of all entries in the current NRS system. Each entry is an
object, with the following fields:

* id (optional): the ID of the entry. This is not necessary, since the map key
is already its ID.

* children: map of all **direct children** of this entry (some entries `entry`
such that `CW(thisEntry, entry) != 0`)

* DAH_meta (if the extension `DAH_meta` is enabled): some metadata about the
entry. This is the JSON serialization of the meta object of this entry.

### impacts.json

This one is for impacts.

The JSON file is a list of all impacts. Each impact is a JSON object with the
following fields:

* contributors: the map of all impact contributors (ID to contributor weight)
* score: the score vector of this impact (as a number array)
* DAH_meta (if the extension `DAH_meta` is enabled): similarly to that in entries.json

### relations.json

The JSON file is a list of all relations, similarly to impacts.json. A relation
is a JSON object. While the field `contributors` and `DAH_meta` has the similar
meaning as in impacts.json, the field `references` is the reference map (ID to
reference weight) of the relation.

### scores.json

The structure is similar to that of entries.json, an ID map of scores.

Score is also an JSON object, and with the field `DAH_meta` if DAH_meta
extension is present. These are its fields (except `DAH_meta` ofc):

* `impacts` and `relations` (optional): score from each impact/relation.

* totalImpact: the impact score vector of the entry

* totalRelation: the relation score vector of the entry

* overallVector: the overall score vector of the entry
