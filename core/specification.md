# NRS (New Rating System) Specification

Version 2.0

## 1. Overview

NRS is a scoring system (although the name is "New Ranking System"), which tries
to be as powerful as possible.

It decides how scores are calculated for **entries**, and this score depends on
**impacts** and **relations**.

Personally, my implementation of NRS is used to rank anime, manga and other
similar content (not limited to Japanese origin, but still mostly weeb stuff).

Since NRS 2.0, the concept of extensions is introduced into NRS. This
effectively modularize NRS, and allow NRS to be more flexible for different
implementations. The most notable result is that it remove the need of any
"Japanese word" from this specification (except for the Overview and History
part, and some examples).

## 2. Fundamental concepts

### 2.1. NRS systems and NRS contexts

This set of files, containing the core specification and extensions specification
documents are hosted on a [Git](https://git-scm.com/) reposistory. The current
official reposistory is hosted on [GitHub](https://github.com), with the
reposistory URL being
[https://github.com/ngoduyanh/nrs.git](https://github.com/ngoduyanh/nrs.git).
This Git reposistory may be referred as "the official NRS reposistory".

An NRS system is an entity that is capable of executing certain tasks described
by an NRS specification. This is the specification for a *core* NRS system,
therefore any *core* NRS system must be an NRS system. By modifying this
document using a set of [extensions](2.2. Extensions), which each has its own
specification in the `exts` directory of the NRS specification git repository,
a modified NRS specification can be derived from this document, then used to
create a new NRS system. If the modification change how the system behave,
this new system is no longer a *core* NRS system.

An NRS context can be created from an NRS system, which may or may not exist
physically. NRS contexts act as an environment for the tasks to be executed.
Different contexts from the same system may have different extensions enabled,
therefore their processes may be incompatible. A *core* NRS context is created
from a *core* NRS system, and since there are no restrictions on extension
support for *core* system, *core* contexts also may not be incompatible with
each other. After the creation of an NRS context, the set of enabled extensions
can not be modified.

> Context incompability is defined to be the inablity to interfere with each
other's processes. (i.e. sharing data)

### 2.2. Extensions

An extension is a set of modification to this document.

All extensions must have a unique **name**. This name must be a string of ASCII
characters, which may be an alphanumeric character (A-Z, a-z, 0-9) or the
underscore character `'_'`. The use of any other characters is strictly
forbidden. The uniqueness of extension names must be case-insensitive ('Abc' and
'abc' must not be names for different extensions).

The entity that specify an extension is called the vendor of that extension,
and may have a name used to prefix their extension names using the
OpenGL-inspired extension name convention. This naming convention is defined as
such: the vendor name and the base extension must be written in
[SCREAMING_SNAKE_CASE](https://en.wiktionary.org/wiki/screaming_snake_case)
and [snake_case](https://en.wikipedia.org/wiki/Snake_case), respecitively;
then joined by the underscore separator character. This helps minimizing
extension naming collisions where different vendors implementing the same
functionalities with minor differences.

This name is then used to name the specification files of the extension,
which is in the `exts` directory of the official NRS reposistory.

Extensions may interact with each other. There are two types of interactions
defined in this document:

* dependence: Some extensions, in order to be enabled, require some other
extensions to be enabled.
* incompatible: Some extensions, in order to be enabled, require some other
extensions to be disabled.

### 2.3. Mathematical concepts

The (score calculation process)[] depends on some basic concepts of linear
algebra: vectors, matrices, dot product, matrix multiplication, etc. All of
the vectors and matrices used in the process will be simple vectors and
matrices (ordered sets of real numbers).

Score vectors and matrices are simple vectors and matrices with a restriction:
in a context, all score vectors must have the same number of dimensions `n`,
and all score matrices must be `n x n` square matrices. This number `n` is
called the number of factor scores.

A process called **clamping** exists for every matrices. This makes sure that
weight matrices in the system could not exceed a certain "range" (the concept
of ranges does not exist for general matrices, hence the process couldn't be
clearly defined here). This process is implementation-defined, and a no-op
implementation may be used. A common clamping algorithm is clamping every
elements in the matrix in some numerical range, which is a well-defined for
the real numbers. A matrix is called to be **clamped** if clamping it does not
change its elements.

Factor scores are fancy indices to access elements of a score vector. The number
of factor scores are globally constant in a context, and every factor score can
be used to access elements of any score vectors. Similarly, a pair of factor
scores can be used to index a matrix. One can interpret an element of a matrix,
at factor score row `i` and column `j`, as how much `i`-factor-score that the
result would get from the matrix-vector multiplication per `j`-factor-score in
the multiplied vector.

An unique function, called **the combine function** must be defined in every
context. This function takes in a multiset of number and an additional number,
called the combine weight, as input, then gives out a single number as output.
Alternatively, passing a simple vector or a score vector to this function in
the place of the number multiset will result in the same output as collecting
all elements of the vector into a new number multiset, then passing it to
the function. In any case, the combine weight is obligatory.

From this line, the result of calling the function using the number multiset
(or vector) `a` and the combine weight `w` will be shortened into
`combine(a, w)` using the function call notation.

The combine function has several requirements:

* `combine(a, 1)` is the sum of all elements in `a`
* `combine(a, 0)` is the sum of the largest positive value in `a` (or 0 if it
doesn't exist) with the smallest negative value in `a` (or 0 if it doesn't
exist). In other words, `combine(a, 0) = max(max(a), 0) + min(min(a), 0)`
* `combine(a, w)` is the sum of `combine(p, w)` and `combine(n, w)`, where `p`
and `n` are filtered submultiset of `a` with only *positive* and *negative*
values, respectively.
* `combine(a, w)` is the quotient of `combine(sa, w)` to `s`, where `sa` is the
multiset of all element of `a`, scaled by `s`, for every non-zero value of `s`.

Using the mentioned property, we can achieve
`combine(a, w) = combine(p, w) - combine(-n, w)`
(using the same notation as the third requirement, `-n` is the multiset of all
absolute values of `n`. The right hand side only computes combine with a multiset
of real positive numbers, therefore it's sufficient for implementation to define
`combine` with the assumption that all elements of `a` is positive. This
specialization of `combine` is called the `combineUnsigned` function.

### 2.4. Context entities

Context entities are entities that belong to an NRS context. These contain:
entries, impacts, and relations.

#### 2.4.1. Entries

An **entry** is a context entity that is scored in the
[score calculation process](#3.2. Score calculation)

An entry can contains other entries. For each pair of entry `A` and `B`, there
is a number named the **direct contain weight**, that determine how much of `B`
was contained directly by `A`. This weight being a matrix is to be able to more
accurately and flexibly model the containing relationships between two entries.

These values are used to determine the **contain weight** of entry pairs. The
process is covered in the
[contain weight solving](#3.2.2. Contain weight solving) process.

#### 2.4.2. Impacts

An **impact** is a context entity that's used to give constant score to entries.

An impact is defined using two properties:

* The impact contribution map, mapping entries to a clamped score matrix, called
the **direct contributing weight**. This number determines how much of the impact
was a result of the entry.
* The impact score vector, which will be scaled to get the score added
each contributing entry.

#### 2.4.3 Relations

A **relation** is a context entity that's used to give score to entries based on
scores of other entries. The score added to entries will depend on scores of
other entries.

A relation is defined using two properties:

* The relation referencing map, mapping entries to a clamped score matrix, called
the **relation score transform matrix**.

* The relation contribution map, mapping entries to a clamped score matrix, called
the **direct contributing weight**. Similarly to the weight in impacts, it
determines how much of the relation affects the entry.

## 3. Required capabilities

An NRS system must be able to carry out two operations: creating contexts and
calculating entries' scores.

### 3.1. Creating contexts

An NRS context must be able to be created from a system. In this context
creation process, the set of all enabled extensions will be specified. This
extension set is not required to be accessible from the newly created context,
but must stay the same in the lifetime of the context. The definition of this
lifetime is implementation-defined.

### 3.2. Score calculation

An NRS context doesn't exactly need to follow along these steps. The only
requirements for the calculation process is that the result must match
(numeric errors are allowed).

#### 3.2.1. Combining score vectors

The function `combineVectors` takes in a multiset of vectors and a combine
weight, in the form of a vector, then performs component-wise `combine`-ing
to get the output vector.

The combine weight is globally constant, and it's called the
**factor score combine weight vector**. It will be defined by the implementation.

#### 3.2.2. Contain weight solving

Entries in the context and their relationship can be modeled as a weighted
directed graph. This graph, called the **entry graph**, is constructed as follows:

* The vertex set is the set of all entries.
* Let `A` and `B` be vertices (entries of the context). The directed edge
`(A, B)` is an edge of the entry graph if and only if the
**direct contain weight** of the pair `(A, B)` is a non-zero matrix. This weight
is also used as the weight for this edge.

After establishing the entry graph, the **contain weight** of the pair `(A, B)`
can be calculated as follows:

* If `A` and `B` is the same entry, the result is the identity matrix.
* Otherwise, the result is sum of matrix-matrix products between the direct contain
weight of `(A, C)` and the contain wights of `(C, B)` for every out-neighbor `C` of
`A`.

The process can be recursively calculated using tree traversing algorithms like DFS,
BFS, etc. As long as there are no loops in the entry graph, i.e. the entry graph is
a tree, the process can end. The behavior of this process when there are loops is
implementation-specific, with forbiding the existence of loops in this graph being
the recommend one.

#### 3.2.3. Contributing weight solving

Consider the entity `IR` that is either an impact or a relation. Since both
impacts and relations have a contribution map, we will let that map for `IR`
be `M`.

Using this map and the containing relationship between entries, the contribution
weight of an entry `E` to `IR` may be calculated by taking the sum of all
products between the contain weight of the pair `(E, A)` and the
**direct contributing weight** of `A` in `IR` for every `A` in the map `M`.

This weight is then passed into an function, called the weight-buffing function
to get the buffed contributing weight of `E` in `IR`. The buffed weight must be
something that could be multiplied with a vector and yield another vector as a
result, i.e. scalars, matrices, vectors (with element-wise multiplication), etc.

#### 3.2.4. Impact score calculation

For an impact `I` and entry `E`, the amount of score that `I` give to `E` is the
product of the I's impact score vector and the buffed contributing weight of `E`
in `I`.

The total impact score of an entry `E` is then calculated by performing
`combineVectors` on the multiset of all score vectors that `I` gives to `E`,
for every impact `I`.

#### 3.2.5. Relation score calculation

For a relation `R`, the relation base score is calculated by taking the sum of
all matrix products between every referenced entry overall score and its overall
score.

Using this base score, for every relation `R` and entry `E`, the score vector
that `R` give to `E` is calculated by taking the product of `R`'s base score and
the buffed contributing weight of `E` in `R`.

The total relation score of an entry `E` is then calculated by performing
`combineVectors` on the multiset of all score vectors that `R` gives to `E`,
for every relation `R`.

> Note: This algorithm may cause infinite recursion (relation score of this
> entry may depend on the overall score of another entries). If this happens, the
> behavior is implementation-defined.

#### 3.2.5. Overall score calculation

The overall score is simply the sum of the impact score and the relation score
of an entry.
