# Extension specification: DAH_standards

Version 1.0

Vendor: DAH

Dependencies
```
NRS 2.0+
DAH_factors 1.0+
```

## Overview

This extension helps the implementor of NRS giving impact/relation scores in the fairest way possible. It describes multiple standards to introduce impacts and relations, along with formulas to calculate the score vector.

## Boredom impacts

Boredom impacts are given to all "completable" entries (described in DAH_factors specification). The kind of impact is defined by how much the entries were "completed":

* Completed: the entry was finished 100%, this gives 1.0 boredom factor score
* Completed with noticeable boredom: the entry was finished 100%, but noticeable boredom was detected, this gives 0.5 boredom factor score
* Watching: the entry was in-progress, this gives 0.75 boredom factor score
* Dropped: the entry was dropped (not finished, and no plans of finishing it), this gives (or takes) -1.0 boredom factor score.
* Temporarily On-Hold: like "Dropped", but there is a plan of finishing the entry, this gives (or takes) -0.5 boredom factor score.

In the future, "Dropped" maybe replaced by "Partial Dropped", which will be fairer for different statuses of "Dropped".

## Emotional impacts (EI)

An EI can be the result of multiple contributing emotions, so the first parameter that all EI takes in is a weight list of all contributing emotions.

These weights can be used to form the contributing emotion weight vector, which is then normalized using the "combine" metric: ```d = combine(v, EmotionSubscoreWeight)```, and scales up by the base score.

The base score will depend on the kind of emotion impact.

There are 3 kinds of EI:

### 1. NEI: Noticeable Emotional Impact

This is given to the entry when the impact was "noticeable": you know it's there, but it's not strong enough to be appreciated.

An EI takes in a relative score parameter, which can range from -10.0 to 10.0. It's then interpolated to the -2.0 to 2.0 range to create the base score.

### 2. AEI: Appreciable Emotional Impact

This is the kind of impact when it's not just "noticeable". The scoring process is basically the same as NEI. The only difference is how the base score is interpolated from the relative score parameter.

Instead of a simple interpolation, this process for AEI is a little bit more complex.

The relative score parameter, which also can range from -10.0 to 10.0, is splited into two part: the sign, and the absolute value. The mapped score is the absolute value of the relative score, interpolated to the 2.0 to 3.0 range, with the sign replaced to the relative score sign. This makes AEI(0.0) and AEI(-0.0) different. Although IEEE 754 has positive zero and negative zero, and every modern programming language uses this, it is advisable to split the score parameter into two parameters.

### 3. EEI: Exceptional Emotional Impact

This comes in 4 forms:

#### a. Cry (self-explanatory)

This gives a base score of 4

#### b. PADS (depressed after watching/consuming an entry)

A PADS depends only on how long it lasts, the concept of "PADS strength" does not exist. The number of days is clamped into the 1 to 5 range, then interpolate to the score range of 3.0 to 5.0.

Implementations can also enable "negative" PADS, by flipping the sign of the score value. 

#### c. EHI (exceptional humor impact)

This gives a base score of 3.5, and only applicable for AP emotion

#### d. EPI (exceptional plot impact)

Unlike Cry or EHI, this takes in a plot score argument, which is in the range 0 to 10, then interpolated into the range 3.5 to 4.5 to get the base score. This is only applicable for AP emotion.

## Waifu impact

A waifu is a character that is loved by me from an entry. 

## Horror-based impact

There are two kinds of impact for horror (MU emotion), "jumpscare" and "sleepless night". Both of them are self-explanatory, and give a base score of 1.0 and 4.0, respectively.

## Info-based impact (deprecated, use additional impacts instead)

Political impact (something that change how I think and act) gives a base score of 0.75 to the info-politics subscore.

Field-interest impact (something that made me interested into something new) gives a base score of 2.0 (if it's a new field) or 1.0 otherwise

## Boredom impact

An entry may be *finished* by *consuming* it. Depending on how much of this process has happened, an appropriate boredom impact will be given to the entry. Some entries, despite being able to be *consumed*, will not have a boredom impact given to them (music entries in the nrs-impl-kt for example).

Boredom impact will only give score of boredom subscore, but with different values depending on the type of boredom impact.

If the process has ended without any suspension/boredom from the consumer (me), a **Completed** impact will be given. This impact gives a score of 1.0.  
If the process was ended but the consumer has felt some boredom, a **Completed with noticeable boredom** impact will be given. This impact gives a score of 0.5.  
If the process is happening and unable to end soon because of media releasing schedules (e.g. seasonal anime), a **Watching** impact will be given. When the process ends or gets interrupted, this impact may be replaced by a more suitable one. It gives a score of 0.75.  
If the process is temporarily paused, a **Temporary on-hold** impact will be given, which will "give" a score of -0.5.  
If the process is stopped (but may still be resumed in the future), a **Dropped** impact will be given, which has a score value of -1.0. This may be replaced by a **Temporary on-hold** impact when the process is continued, or **Completed with noticeable boredom** when the process ends.

## Meme impact

An entry may become a meme, and therefore contribute to the consumer's culture. If so, it will be given a meme impact (NRS are supposed to grade entries according to how influential they were after all), which will depending on how much this contribution was. There are two factors, the meme duration score and the meme strength score.

The meme duration score is calculated by taking the number of days the entry was meme'd to the power of 0.25, while the meme strength score is scored by the grader in the range from 0 to 1. After that, the two scores are multiplied together and scaled up by 3 to get the score for this impact.

Since this is a meme impact, it will only give AP emotion score.

## Additional impact

Additional impact is the only kind of impact allowed to give additional subscores. For example, these impacts can be used to buff/nerf entries that was underrated or has been abusing the system.
