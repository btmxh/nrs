# Extension specification: DAH_anime_normalize

Version 1.0

Vendor: DAH

Dependencies:
```
NRS 2.0+
DAH_standards 1.0+
DAH_overall_score 1.0+
```

## Overview

In the current NRS implementation ([link](https://github.com/ngoduyanh/nrs-impl-kt)), the overall score of low-ranked animes are really low, compared to the standards from sites like MAL or AniList. This makes my score vote on these sites unfair for these animes.

This extension aims to convert the raw overall score into a better score, which suits these common standards.

## Mapping Algorithm

The extension provides a sample entry list, which consists of base animes. These base animes are the "keyframes" for this mapping algorithm. There are 10 animes, which represents 10 score levels in the MAL standards.

This entry list is scored using NRS but without this extension (we don't need base anime normalized score since they don't exist), giving 10 overall scores for 10 animes. Using "keyframe-like" interpolation (for scores outside the interpolation range, the score will be capped between the MAL's 1-10 range).

The entry list is not provided here, you can see it in the [nrs-impl-kt](https://github.com/ngoduyanh/nrs-impl-kt/blob/master/core/src/main/kotlin/com/dah/nrs/exts/DAH_anime_normalize.kt) repo. A set of serialized JSON files are going to be provided in the future.
