# Extension specification: DAH_factors

Version 1.0

Vendor DAH

Dependencies:
```
NRS 2.0+
```

## Overview

Since the core NRS specifications doesn't explicitly define what the factor scores should be, i.e. they're implementation-defined, this extension aims to fill in this gap.

## Subscores

Factor scores are grouped into different groups, based on their relations. This is due to the fact that some factor groups are similar, and therefore an impact will be likely to give similar factor scores. This is used to nerf impacts that gave multiple similar factor scores. Additionally, these factor scores and subscores come with their unique identifier (noted next to their names)

The groups are:

### 1. Emotion (E)

This is the factor group responsible for emotional impact (like crying, depressed, etc.)

The group consists of 6 emotions: AU, AP, MU, MP, CU, CP. Each emotion is a factor score.

An emotion is defined by two traits:
* (A)ctivated, (M)ild, or (C)alm (by how activated you are)
* (U)npleasant or (P)leasant (if you feel good or not)

Some example of emotion classification:

* AU: angry, panic
* MU: hate
* CU: sad, empathy
* AP: interested, 
* MP: love
* CP: happy (for the characters, idk it's hard to put into words)

### 2. Art (A)

Art comes in 3 forms, and must not go with any emotions (there already is an emotion subscore)

The factor scores of this subscore are:
* Music (AM)
* Illustration (AI) (Visual, this will be the new name very soon)
* Language (AL)

The names should be self-explanatory.

### 3. Boredom (B)

Boredom is the amount of interest given to the entry. The subscore has only one factor score with the same name.

This factor score is only given to entries that are "completable", which are anime, manga, light novels, completable games (like visual novels). Music is excluded because it took literally like only five minutes to complete listening to one normal song. Gacha games will take something like 5 years to "complete", and there is no good indicator of how "completed" the game was. 

### 4. Additional (AD)

Like the boredom subscore, this subscore has only one factor score with the same name. Its aim is to provide additional score that isn't emotion-related, art-related, info-related or interest-related, something like a short anime buff, compensation score, etc.

## Combine factors

The combine factors for all factor scores and subscores are as follows:

Subscores:
* E: 0.6
* A: 0.4
* B: 0.0
* AD: 1.0

Factor scores:
* AU: 0.3
* AP: 0.4
* MU: 0.35
* MP: 0.35
* CU: 0.4
* CP: 0.5
* AL: 0.3
* AI: 0.3
* AM: 0.4
* B: 0.0
* AD: 1.0

Putting the factor score combine weights into a vector, we got the combine weight vector: \[0.3, 0.4, 0.35, ...\] (the factor score order must be AU, AP, MU, ...)

## See Also

The ```DAH_overall_score``` extension is used to combined the factor scores into a single number called the overall score.
