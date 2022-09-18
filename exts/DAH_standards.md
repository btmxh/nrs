# Extension specification: DAH_standards

Version 1.0

Vendor: DAH

Dependencies

```
NRS 2.0+
DAH_factors 1.0+
```

## 1. Overview

This extension helps the implementor of NRS giving impact/relation scores in the
fairest way possible, by introducing standards (i.e. rules) on how
impacts/relations should be given to entries, as well as how the score vector
of such impacts/relations should be calculated.

The standards will mostly consider the reactions of the consumer of the NRS
system. Some other factors are also noted, but they won't make so much of an
impact on how the process works.

## 2. Impacts

### 2.1. Emotional impacts (EI)

An EI can be the result of multiple contributing emotions, so the first parameter
that all EI takes in is a weight list of all contributing emotions.

These weights can be used to form the contributing emotion weight vector, which
is then normalized using the "combine" metric:
```d = combine(v, EmotionSubscoreWeight)```, and scales up by the base score.

The base score will depend on the kind of emotion impact.

There are 3 kinds of EI:

#### 2.1.1. Noticeable Emotional Impact (NEI)

This is given to the entry when the impact was "noticeable": you know it's
there, but it's not strong enough to be appreciated.

A
 NEI takes in a relative score parameter, which can range from -1.0 to 1.0.
It's then interpolated to the -2.0 to 2.0 range to create the base score.

#### 2.1.2. Appreciable Emotional Impact (AEI)

This is the kind of impact when it's not just "noticeable". The scoring process
is basically the same as NEI. The only difference is how the base score is
interpolated from the relative score parameter.

Instead of a simple interpolation, this process for AEI is a little bit more complex.

The relative score parameter, which also can range from -1.0 to 1.0, is split
into two part: the sign, and the absolute value. The mapped score is the absolute
value of the relative score, interpolated to the 2.0 to 3.0 range, with the sign
replaced to the relative score sign. This makes AEI(0.0) and AEI(-0.0) different.
Although IEEE 754 has positive zero and negative zero, and every modern
programming language follows this standard, it is advisable to split the score
parameter into two parameters.

### 2.1.3. Exceptional Emotional Impact (EEI)

This comes in 4 forms:

#### 2.1.3.1. Cry (self-explanatory) (C)

This gives a base score of 4

#### 2.1.3.2. PADS (depressed after watching/consuming an entry) (PADS)

A PADS depends only on how long it lasts, the concept of "PADS strength" does
not exist. The number of days is clamped into the 1 to 5 range, then interpolate
to the score range of 3.0 to 5.0.

Implementations can also enable "negative" PADS, by flipping the sign of the
score value.

#### 2.1.3.3. Exceptional Humor Impact (EHI)

This gives a base score of 3.5, and only applicable for AP emotion.

#### 2.1.3.4. Exceptional Plot Impact (EPI)

Unlike Cry or EHI, this takes in a plot score argument, which is in the range
0 to 10, then interpolated into the range 3.5 to 4.5 to get the base score.
This is only applicable for AP emotion.

### 2.2. Visual impact

Visual impacts is used to rate how things in an entry looks.
It depends on three factors:

* the **quality** (a number from 0 to 1): how good the visual looks
* the **uniqueness** (a number from 0 to 1): how unique the visual looks
* the **visual kind**: what kind of visual it is

The visual kind is then converted into a number (the **kind factor**) to
get the maximum base score of the impact (i.e. the base score if quality
and uniqueness both equals 1).

Here is a table of some **kind factors** and its respective **visual kind**.

<!-- markdownlint-disable MD013 -->
| Visual kind                   | Kind factor | Description                                                                                        | Example                                                                                                                                        |
| ----------------------------- | ----------- | -------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Anime                         | 3.0         | Cartoon (not necessarily Japanese/Asian origins)                                                   | [Show By Rock!!](https://myanimelist.net/anime/27441/Show_By_Rock)                                                                             |
| RPG 3D game                   | 2.0         | May be applicable for entries that require the same amount of visual and similar producing methods | [Atelier Ayesha: The Alchemist of Dusk](https://store.steampowered.com/app/1152300/Atelier_Ayesha_The_Alchemist_of_Dusk_DX/)                   |
| Animated short                | 1.0         | Short animated content, like short animes                                                          | [Porter Robinson & Madeon - Shelter](https://www.youtube.com/watch?v=fzQ6gRAEoy0)                                                              |
| Animated Gacha card art       | 1.0         | self-explanatory                                                                                   | [A random Lapis Re:LiGHTs card](https://twitter.com/lapi_staff/status/1555750517887975425)                                                     |
| Animated music video          | 1.0         | self-explanatory                                                                                   | [Howan & Cyan - How To Fly](https://www.youtube.com/watch?v=IqdpYyaLnNc)                                                                       |
| Semi-animated music video     | 0.8         | Music videos that mostly utilizes low-FPS visual and simple transformations                        | [Plasmagica - Bokura no Neiro](https://www.youtube.com/watch?v=vyaGNvuVDuM)                                                                    |
| (not animated) Gacha card art | 0.8         | self-explanatory                                                                                   | [Re:Stage! card arts](https://lldetail.ml/Restage/card/index_secret.php)                                                                       |
| Manga visual                  | 0.8         | self-explanatory                                                                                   | [Oshi no Ko](https://myanimelist.net/manga/126146/Oshi_no_Ko)                                                                                  |
| Visual novel visual           | 0.8         | self-explanatory                                                                                   | [Miagete Goran, Yozora no Hoshi o](https://vndb.org/v16560)                                                                                    |
| Light novel visual            | 0.5         | self-explanatory                                                                                   | [A random COTE illustration](https://danbooru.donmai.us/posts/2905913)                                                                         |
| Static music video            | 0.25        | "Music videos" that basically is just some static images                                           | [Stellamaris - Clematis -クレマチス- (note: this is not the official thing)](https://www.youtube.com/watch?v=hj_4YAVmmuI)                      |
| Album art                     | 0.25        | self-explanatory                                                                                   | [Stellamaris - Clematis -クレマチス- (note: this is the official thing)](https://medium-media.vgm.io/albums/86/121168/121168-de8dcf1b4ceb.jpg) |
<!-- markdownlint-enable MD013 -->

To get the actual base score from the maximum base score, simply multiply the
maximum base score with the result of some function, called the visual function,
with input being the quality and uniqueness factor. This function is
implementation-defined, but there is one requirement. That is, if both the
arguments don't decrease, the result of the function must also not decrease.
In other terms, it must be a non-decreasing function, with respect to both of
the input arguments. Enabling other extensions can be a way to specify this function.

### 2.3. Waifu impact

A *waifu* is a character that is loved by the consumer in an entry. NRS does not
make any assumption about this character, and only care about how long they can be
considered a waifu.

If the period of time when this character can be considered a waifu is *notable*
(decided by the implementor), a waifu impact is given to all entries with this
character, and the contribution factor of each entry is determined by how much
that waifu was featured and loved.

A waifu can have multiple periods, and the total number of days of all periods
is used to calculate the base score of the impact. First, take the ratio of that
day count and 90. Then, raise this number to the power of the weight of `MP`.
Finally take the product of the previous result and 1.2.

Waifu impacts can only give `MP` score.

### 2.4. Horror-based impact

There are two kinds of impact for horror (MU emotion), "jumpscare" and
"sleepless night". Both of them are self-explanatory, and give a base score of
1.0 and 4.0, respectively.

### 2.5. Boredom-based impacts

There are two kinds of boredom-based impacts: Consumed and Dropped.
Both of them only give `Boredom` score.

#### 2.5.1. Consumed

The amount of time taken by an entry's consumption by the consumer (the consume
length), along with a number from 0 to 1 that is used to determine the overall
boredom during this process (the boredom factor), is used to calculate the base score.

Using the consume length, the base consume length and the maximum base score
can be calculated by evaluating some conditionals:

* If the consume length is less than  10 minutes, then the base consume length
and the maximum base score are 5 minutes and 0.1, respectively.
* If the consume length is less than 2 hours, then the base consume length and
the maximum base score are 2 hours and 0.3, respectively.
* If both of the above conditionals evaluate to false, then the base consume
length and the maximum base score are 240 minutes (6 hours) and 1.0, respectively.

The base score of the impact determined by taking the ratio of the consume
length and the base consume length, raising to the power of the weight of
the `Boredom` factor score and multiply the result to the maximum base score
and the boredom factor.

#### 2.5.2. Dropped

When the consumption of an entry was halted and there were no plans of
continuing that entry before the completion of that entry (by the consumer),
the entry will be given a `Dropped` impact. This has a base score of -0.5.

### 2.6. Meme impact

An entry may become a meme, and therefore contribute to the consumer's culture.
If so, it will be given a meme impact (NRS are supposed to grade entries
according to how influential they were after all), which will depending on how
much this contribution was. There are two factors, the meme duration score and
the meme strength score.

The meme duration score is calculated by taking the number of days the entry
was meme'd to the power of AP's factor score, while the meme strength score
is scored by the grader in the range from 0 to 1. After that, the two scores
are multiplied together and scaled up by 4 to get the score for this impact.

Since this is a meme impact, it will only give AP emotion score.

### 2.7. Additional impact

Additional impact is the only kind of impact allowed to give additional
subscores. For example, these impacts can be used to buff/nerf entries that
was underrated or has been abusing the system.

### 2.8. Music impact

Music impacts take in a `musicScore` parameter, which is in the range 0 to 10.
It is then mapped to the 0-3 range to get the base score of the impact.

### 2.9. osu! song impact

For consumers who play the rhythm game
[osu!](https://en.wikipedia.org/wiki/Osu!), some songs contribute to the gaming
experience, and their NRS entry will get a `osu! song impact`.

This impact takes in two parameters, the personal and community factor, which
are both in the 0-10 range. They are then mapped to the 0-0.5 and 0-0.2 range
respectively, and the base score is calculated by taking the sum of the mapped values.

## 3. Relations

### 3.1. Remix (R)

When an entry is a derivative work of some other entry (aka a remix), the
original entry will be given a `Remix` relation that references only the
remix entry with the `Remix` matrix.

The `Remix` matrix is 0.2 times the identity matrix (or the diagonal matrix
with every entries in the diagonal being 0.2).

### 3.2. Feature music (FM)

When an entry features another music entry, the featuring entry will be given
a `Feature music` relation. This will reference the featured music entry with
the `Feature music` matrix.

The `Feature music` matrix entries are almost all zero, with the exception of
the entry in the row and column for Music factor score being 0.2.

### 3.3. Killed-by (KB)

The process of killing is defined as an entry making the consuming experience
worse for another entry, resulting in lower NRS score for the latter. This
relation acts as a compensation for that entry, and take in two parameters:
`potential` and `effect` factor. It only references one entry, the entry that
ruined the experience etc., with the `Killed-by` matrix, multiplied by
`potential`, `effect` and 2.0.

Assuming the order of all factor scores to be `AP,AU,CP,CU,MP,MU,AV,AL,AM,B,A`,
the `Killed-by` matrix is a diagonal matrix with diagonal vector being
`[0.2, 0.1, 0.05, 0.05, 0.2, 0.1, 0.0, 0.1, 0.1, 0.1, 0.0]`.

### 4. Music contribution rules

A piece of music is divided into three parts: **vocal**, **instrumental**,
and **image**. **Vocal** and **instrumental** are self-explanatory,
and **image** is something like PR material (e.g. The `Criticrista` part
in `M-VGMDB-AL-76155`'s title: `Loop Shiteru/Asuiro Koi Moyou / Criticrista`).

The vocal and instrumental contribution factor is 0.4, and the remaining 0.2
is for the image contribution factor. If there are multiple people fulfilling
the same roles, the contribution factor will be divided. If someone is in
charge of multiple things, the contribution factors are added.

When a song doesn't have one of the aforementioned parts, for example
something like a song with no vocal, the remaining contribution factors
values are scaled up by a common factor to get a total of 1.0, so that
*vocalless* song will have instrumental factor being `0.4 / (0.4 + 0.2) = 2/3`,
and image factor being `0.2 / (0.4 + 0.2) = 1/3`.

## Additional definitions

* Implementor: The entity in charge of implementing the current NRS system
* Entry consumption: The act of consuming entries is implementation-defined and
dependent on which kind of entry was it. Some entries may not be consumable.
There are no restriction on how this process may be like, but the implementor
should make it reasonable (consuming an anime is watching it, consuming a manga
is reading it, etc.)
* Consumer: The entity in charge of consuming entries. While consuming entries,
this entity will give a multitude of reactions. These reactions will then be
used to determine which impacts/relations should be given to an entry, by the
rules specified above.
* Entry completion: When the consumption of an entry reached a threshold (which
may vary depending on the entry), the entry may be marked as completed. This act
is known as entry completion. This definition will form the basis of how boredom
impacts will be given. Some entries will not have a "threshold" to be marked as
completed, therefore making them *uncompletable*, even though they may be
consumable or not.
* Data provider: The entity in charge of providing data (entries, impacts and
relations) to the NRS system.
