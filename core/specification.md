# NRS (New Rating System) Specification

Version 2.0

## Overview

NRS is a scoring system (although the name is "New Ranking System"), which tries to be as powerful as possible.

It decides how scores are calculated for **entries**, and this score depends on **impacts** and **relations**.

Personally, my implementation of NRS is used to rank anime, manga and other similar content (not limited to Japan, but still mostly weeb stuff).

Since NRS 2.0, the concept of extensions is introduced into NRS. This effectively modularize NRS, and allow NRS to be more flexible for different implementations. The most notable result is that it remove the need of any "Japanese word" from this specification (except for the Overview and History part, and some examples).

## Extensions

An extension is something that affect the NRS score calculation process. It must have a unique **name**, i.e. there must not be two extensions with the same name (case-insensitive), and that **name** is a string of ASCII characters, which can be a letter (A-Z, a-z), a digit (0-9), or a underscore symbol (_).

The name can be named however you want, but to make the name fits nicely with my list of extension, it should be named the OpenGL format:  `{VENDOR_SHORT_NAME}_{extension_name}`. For example, `DAH_factors`, `DAH_code_generation`, etc. (vendor is basically the person/entity decide how the extension works)

The specifications for all (documented) extensions will be in the `exts` directory.

An extension can have dependencies, which can be other extensions or requirements of NRS core version.

## Score Vector, Score Matrix, Factor Scores

NRS scores are represented using **vectors**, these are called score vectors. Components of that vector are called **factor scores**. Score matrix are used to "scale" these vectors.

The number of dimensions in these vectors are the same throughout the implementation, and it must be the same as the size of the score matrices (the matrices are square btw).

Using the `combine(a, w)` function defined above, score vectors can be combined with the help of a vector called the combine weight vector `CWV`

```
combine(vectors)[i] = combine(vectors.map(v => v[i]), CWV[i])
for all i in the range 0 to NUM_DIMENSIONS - 1
```

## Entries

An **entry** is something that'll be scored by the system. 

An entry can contains other entries. A number called the **contain weight** is used to determine how "strong" this "relation" is. The contain weight must be a number between 0 and 1 (0 is the same as A not containing B, and 1 means that A completely contains B, all impacts and relations of B are directly added to A)

(Let CW(A-B) be the contain weight of A on B)

```
For example, X and Y formed a band B (two people bands don't exist, but ignore that)
X is the vocalist of the band, and the main reason why I listened to B.
Y is the drummer, and only few people cared about them.

B published a song, S, and it got 10.0 NRS score.

Since B completely owns S, it also got all of 100% S's impacts.
But the same can't be said for X and Y. Both of them only partially owns B.

This is where this "contain weight" comes to play.
Because X is the main reason of B's success, X will have a higher contain weight than Y.

For example, CW(X, B) = 0.75, CW(Y, B) = 0.25
```

This "contain" relation has turned NRS entries into a weighted directed graph, and to make thing easier to phrase, graph theory terminology is going to be extensively used in the following part.

Let E be the set of all entries, CW be the contain weight function, G be the entry graph (G's vertex set is E, G's edge set is `{(u, v) for u, v in E; u != v; CW(u,v) > 0}`)

These conditions must be satisfied:

* `CW(X, X) = 1.0` for all X in E
- `0.0 <= CW(X, Y) <= 1.0` for every pair of (X, Y) in E^2
* `CW(X, Y) = max { CW(X, E) * CW(E, Y) } for all entries E` for every pair of (X, Y) in E^2

* `CW(X, Y) * CW(Y, X) = 0` for every pair of (X, Y) in E^2, X != Y (equivalent to G has no loop)

## Impacts

An **impact** is something that gives score to entries.

An impact may be caused by multiple entries, and the amount of contribution from each entries is stored using a number called the contribution weight of that entry.

The amount of score an impact has is called the score of that impact.

## Relations

An **relation** describes the relationship between entries.

Similarly to impacts, a relation may be caused by multiple entries, and for every contributing entry, there is a contribution weight.

Relations give score to contributing entries, but this score is not static. It depends on other entries, which are called referenced entries. For every referenced entries, there is  a score matrix (which will be multiplied with the overall score of that entry, summed over all entries, to get the score of the relation).

## Combine function

The combine function is a function that takes two argument:

* An array of number, which can be empty

* A number called weight, which is in the range 0.0

The function must satisfy some conditions:

* `combine(a, w) = combine(a+, w) + combine(a-, w)`, where `a+` is `a` without all non-positive values, `a-` is `a` without all non-negative values

* `combine(a, 0.0) = max(max(a), 0.0) + min(min(a), 0.0)`

* `combine(a, 1.0) = sum(a)`

It's easy to prove that if `combine(a, w)` satisfies all 3 above conditions, there exists one and only one function `combineUnsigned(a, w)` such that:

```
combine(a, w) = combineUnsigned(a+, w) - combineUnsigned(abs(a-), w)
```

Therefore, all implementations of NRS will only have to implement `combineUnsigned(a, w)`.

## Calculation

```python
# not python code, only here for better syntax highlighting

# The overall score vector is the sum of the impact score vector and the relation score vectorOverallScore = ImpactScore + RelationScore
OverallScore = ImpactScore + RelationScore

# The impact score is the combination of all per-impact score
# The relation score is the combination of all per-relation score
ImpactScore = combine(AllImpacts.map(x => PerImpactScore(x)))
RelationScore = combine(AllRelations.map(x => PerRelationScore(x))

# per-impact score is the product of the raw impact score with the contributing weight
PerImpactScore(impact) = impact.Score * ContributingWeight(thisEntry, impact)

# per-relation score is the product of the raw relation score with the contributing weight
PerRelationScore(relation) = relation.Score * ContributingWeight(thisEntry, relation)

# contributing weight is basically the sum of all contain weight from this entry to all contributors
# BuffWeight(rawWeight) is implementation-defined
ContributingWeight(impactOrRelation) = BuffWeight(RawContributingWeight(impactOrRelation))
RawContributingWeight(impactOrRelation) = sum(CW(thisEntry, contributor) * contributorWeight for (contributor, contributorWeight) in entryOrRelation.contributors)

# raw relation score is the sum of products between contributor score matrix and its overall score
relation.Score = sum(contributorMatrix * contributor.OverallScore for (contributor, contributorMatrix) in relation.contributors)
```

Note: This algorithm may cause infinite recursion (`thisEntry.OverallScore` depends on `contributor.OverallScore`). If this happens, it's undefined behavior (but every implementation should have an approach to fix this problem).
