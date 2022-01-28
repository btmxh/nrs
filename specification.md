# NRS (New Rating System) Specification

## Overview

NRS is a rating system for entries named NRS-entry, which consists of animes, mangas, light novels, visual novels, and a lot of weird combined entry types.

NRS also have an ID system named NRS-ID, originally used to connect entry rows from different sheets for the same entry using the VLOOKUP function. 

## Name

Originally, NRS is not named, the original GitHub repository is anime-rebalance, and when it was moved to Google Docs, the filename is Anime Rating Rework, and the header is New Anime Rating System Rework.

When I decided to find a abbreviation for it, NRS is picked, and it stands for "New Rework System" or "New Rating System".

## History

In late-2020, I got bored of the old emotion-based rating system. Then, I created a new system to replace it. This is NRS's old form. Its specification is planned to written on the anime-rebalance repository hosted on GitHub.

The specification and the implementation was planned to be written in two markdown documents. But I realized that Google Sheets is a better place to host all of this, since the results are calculated by formulas and spreadsheets are the perfect format for all of that calculations.

After the "Autism Movement" and some drawbacks found after implementing NRS on a spreadsheet, it's declared that NRS should be hosted offline on my own Linux machine as a git repository. The specs should be written in Markdown (as you can see here), and the implementation is not included.

## Versioning

NRS has no versions when it was first launched, and this still held when it was hosted on Google Sheets. This migration has led to the first official version of NRS, version 1.0, which is what the original NRS hosted on Google Sheets is like.  

NRS 2.0 is then published to fix all of the problems in NRS 1.0

## NRS-ID

The NRS-ID is a case-insensitive string of alphanumeric characters (from 0-9 and A-Z) and the dash symbol (-) to separate different parts of the ID, which is unique for every NRS entry.

It's the only thing guaranteed to be different among entries, since a lot of manga/light novel/visual novel and its anime adaptation share the same title.

NRS-ID should be short and informative, so it has a limit of 24 characters.

From NRS 2.0, the (full) ID is in the following format:

```
NRS_ID = TypePrefix + "-" + DatabaseID + ("-" + SubtypePrefix) + "-" + EntryIDInDatabase + ("-" + OptionalSuffix)
// when OptionalSuffix is omitted, it's
NRS_ID = TypePrefix + "-" + DatabaseID + "-" + EntryIDInDatabase
// when there are no databases containing the entry, we use a custom ID system (or base on other entries).
// the custom ID needs to be digit-only
NRS_ID = TypePrefix + "-" + CustomID
```

This has made NRS IDs more clear and easy to read. Take A-MAL-23847 for example, it's an anime entry (A), with ID 23847 taken from MyAnimeList (MAL).

### Type Prefixes

* A: Anime
* L: Light Novel/Manga
* V: Visual Novel
* G: Video Game
* F: Franchise
* M: Music

### Databases (and its priority)

Anime:
* MAL: [MyAnimeList](https://myanimelist.net)
* AL: [AniList](https://anilist.co)
* ADB: [AniDB](https://anidb.net)
* KS: [Kitsu](https://kitsu.io)

Visual Novel:
* VNDB: [vndb](https://vndb.org)

Video Game, Franchise, Music:
* VGMDB: [vgmdb](https://vgmdb.net)

  VGMDB has two subtype prefixes: AL for album, AR for artists

### Notes

* Visual Novels are defined as entries with a vndb entry, so there are no custom ID system for Visual Novels.
* Franchises without a vgmdb entry can be named after its first published sub-entry: ```"F" + FirstSubEntryID```. Here are some examples (we assume that the franchise entry doesn't exist):

  MagiReco: The franchise comprises of multiple NRS entries:

  * G-VGM-5237 - Magia Record: Mahou Shoujo Madoka☆Magica Gaiden (the game)
  * A-MAL-38256 - Magia Record: Mahou Shoujo Madoka☆Magica Gaiden (TV)
  * A-MAL-41530 - Magia Record: Mahou Shoujo Madoka☆Magica Gaiden (TV) 2nd Season - Kakusei Zenya (ss2)
  * A-MAL-49291 - Magia Record: Mahou Shoujo Madoka☆Magica Gaiden (TV) Final Season - Asaki Yume no Akatsuki (ss3)

  But the game is published first (idk the date tho), so the entry ID is FG-VGM-5237.

  > Note: The game is not always what's published first (Lapis Re:LiGHTs and Idoly Pride), so be careful.
* Entries without a vgmdb entry can be named after its franchise (the opposite situation of what happens above): ```TypePrefix + FranchiseID```, optional suffix is often needed.

  Re:Stage! Prism Step is a game from the Re:Stage! franchise. But on vgmdb there's no game entry, so the game ID is GF-VGM-7059

* Music entries are separated songs, therefore they usually need an optional suffix, and it's the track number in the first published album containing the song.

## Rating System

```
// the combine function, will be used a lot in NRS

// unsigned combine function
combine_unsigned([x_1, x_2, ..., x_n], w) = y_1 + y_2 * w + ... + y_n * w^(n-1), where y_(1..n) is x_(1..n) in descending order.

// signed combine function
combine(arr, w) = combine_unsigned(arr_positive, w) - combine_unsigned(arr_negative_abs, w),
where arr_positive is arr but without all negative values,
arr_negative_abs is arr without all positive values, but taking the absolute values
```

### Score Hierarchy

1. NRS Score: total score, with everything taken into account (as far as NRS can handle). It is calculated by taking the sum of all subscores, then added with the relation score.

2. Subscores:

  * Emotion Subscore (combine_weight = 0.6)
  * Art Subscore (combine_weight = 0.4)
  * Boredom Subscore (combine_weight = 0.0)
  * Fandom Subscore (combine_weight = 1.0)
  * Information Subscore (combine_weight = 0.5)
  * Other Score (combine_weight = 1.0)

  Subscore are calculated by combining all factor scores (using combine_weight as weight).

3. Factor scores: Basically subscores of subscores.  
  a. Emotion factor scores:

  Emotions are group by two properties: Pleasant (Unpleasant or Pleasant) and Active/Calming (Activated, Calming, Moderate).

  Some example emotions:

  * Activated Unpleasant: Hate, Anger, Disgust, etc.
  * Activated Pleasant: Interest, Amusement, Joy, etc.
  * Moderate Unpleasant: Fear
  * Moderate Pleasant: Love, Heartwarming, etc.
  * Calming Unpleasant: Sad, Depression, Compassion, etc.
  * Calming Pleasant: Journey/Hype, Supportive, etc.  

  Combine Weight Table:
  |            | Activated | Moderate | Calming |
  |------------|-----------|----------|---------|
  | Pleasant   | 0.4       | 0.35     |   0.4   |
  | Unpleasant | 0.3       | 0.35     |   0.5   |
  
  b. Art factor scores:

  * Language (translated stuff is also accounted), combine weight is 0.3
  * Illustrations (~~how cute are the anime girls~~, yk what i meant, fan arts also accounted), combine weight is 0.3
  * Music (without lyrics, music entries only), combine weight is 0.4

  c. Information factor scores:

  * Politics (change my view on life or sth), combine weight is 0.7
  * General Information (make me interested in sth), combine weight is 0.5

  d. Subscore-based factor scores:

  Boredom, Fandom, Other subscores has only one factor score each (with the same name as the subscore). Their combine weight are all 1.0.

  e. Calculation:

  Factor scores are calculated by combined all factor scores awarded from impacts (combine weights noted above.

  ```
  Impact A gave 5 Art-Language factor score, 8 Art-Music factor score
  Impact B gave 7 Art-Language factor score, 10 Information-Politics factor score

  Therefore the entry has a total of:
  combine([5, 7], 0.3) Art-Language factor score
  combine([8], 0.4) Art-Music factor score
  combine([10], 0.7) Information-Politics factor score
  ```

### Relations

Relations are kind of like impacts but towards other entries. Here are some examples:
* Magia Record (anime) "killed" RizeKoi
* Lapis Re:LiGHTS made idol anime a thing, so Re:Stage! Dream Days or Love Live! School Idol Project are watched because of it

Note: "contains" is not a relation. If entry A contains entry B, you add all impacts of B to the list of impacts from A. 

Relation score is calculated as follows:
```
RelationScore = Entry1.OverallScore * RelationWeight1 + Entry2.OverallScore * RelationWeight2 + ...
```

When circular relation happens, the behavior is undefined, but every implementation should have its own sane behavior (which can be equations, stack size limit, etc.)

Relation weight are based on two things, the chemistry (for example how an OST fit to a sad scene) and the relation itself (gate-open, revive, etc.)

### Impacts

An impact is somewhat difficult to define, so here are some examples:
* Your Lie In April made me cried
* Oregairu is the inspiration of Hikism-Yukism
* Love Live Niji is what made "Ayumuiro Days" a thing

Impacts are the building blocks of NRS scores. An impact will give factor score values.

### Standardized Impacts

**Emotional Impacts** are the first 4 standardized impacts, only giving score to the emotion group causing them

#### 1. PADS

PADS (Post Anime Depression Syndrome) is the state of mind when some extreme emotions overwhelming other normal emotions because of an entry. PADS can be caused by all of the emotion groups, and it will give factor score to that group.

A PADS is considered when its duration (how long those extreme emotions last) is at least a day.

Since PADS is an extreme emotion, its score is not scaled (doesn't depend on the emotion group).

If PADS is caused by 2 emotion groups, it's considered two PADS, and the duration must be measured separately. (this is rare, since PADSes are often merged into one emotion).

The scaled score formula: ```map(PADSLengthInDays, 1, 5, 3, 5)```

#### 2. Cry

Crying is the most rewarded emotional impact. Similarly to PADSes, crying is considered an extreme (expression of) emotion, therefore its score is not scaled (doesn't depend on the emotion group).

The scaled score is 4.

#### 3. Appreciable Emotional Impact (AEI)

"Appreciable" means something like "High Quality". An AEI is an emotional impact without PADS or tears, but a lot more than barely noticeable (basically almost cried).

Example:
* Kokoro Connect's drama: is executed well, but didn't cause any PADS or make me cry.
* Chuunibyou demo Koi ga Shitai! Ren: Shichimiya dead people almost made me cry.

The unscaled score ranges from 2 to 3 depending on how "appreciable" the impact is.

The scale for Activated Unpleasant is 0.7, both Activated Pleasant and Moderate Unpleasant are 0.8, Moderate Pleasant is 0.9, and both Calming Unpleasant and Calming Pleasant is 1.0.

#### 4. Noticeable Emotion Impact (NEI)

"Noticeable" means, well, it exists. A NEI is an emotional impact without PADS, tears and being "appreciable". For example, take any well-executed drama from an idol/music anime (bandori, idolmaster, etc.).

The score ranges from 0 to 2, depending on how "noticeable" the impact is.

The scale for Activated Unpleasant is 0.6 both Activated Pleasant and Moderate Unpleasant are 0.8, Moderate Pleasant is 0.9, both Calming Unpleasant and Calming Pleasant is 1.0.

#### 5. Waifu

When I love something (a character or something) in an entry, and if that love is strong enough, it'll be taken into account, by giving some moderate pleasant factor.

The intensity of love is measured using the influential time of the target with this formula:

```
// old waifu score formula
ModeratePleasantFactor = 1.5 * tanh(InfluentialTimeInDays / 60)
```

#### 6. Exceptional Humor and Plot

When an entry has good comedy or plot, it will be given some Activated Pleasant factor. Normally, this is a NEI/AEI, but when it's exceptional, an EHI or EPI (exceptional humor/plot impact) will be given instead.

This can be considered a Cry impact for AP emotions.

An EHI gave 3.5 AP factor score, and EPI can give from 3.5 to 4.5 factor score. (which can be more than a normal Cry impact)

#### 7. Horror

Activated Unpleasant factor are given to entries with spooky stuff.

Giving me a sleepless night is 4 score, and every jumpscare is 1 score.

#### 8. Information

Inspiring political discoveries is 0.75 politics factor score, while making me interested in a field is 1 (2 if it's a new field) politics factor score.

#### 9. Boredom

This impact will give/take boredom factor score.

Aired/Completed Entries (for gacha game, its completed time is end-of-service)
* Completed: 1
* Completed but noticable boredom: 0.5
* Dropped: -1
* Unwatched: 0

Airing/Continuing Entries:
* Watching: 0.75
* Temporarily On-Hold: -0.5
* Partially Dropped: -0.5 to 0.25

#### 10. Meme

If an entry got meta and memed, it will be given some Activated Pleasant score. This score depends on how long and how strong the meme is.

Meme score is calculated as follow:

```
ActivatedPleasantFactor = MemeStrengthScore * MemeDurationScore * 3

MemeStrengthScore is in the range 0.0 - 1.0
MemeDurationScore is specified by the duration standard below
```
|           Duration            | Score |
| ----------------------------- | ----- |
|        Less than a day        |   0   |
|           1-3 days            |  0.1  |
|           4-7 days            |  0.3  |
|           1-2 weeks           |  0.5  |
|           2-3 weeks           |  0.6  |
|       3 weeks - 1 month       |  0.7  |
|           1-2 month           |  0.8  |
|          2-3 months           |  0.9  |
| More than 3 months (a season) |   1   |
