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

## NRS-ID

The NRS-ID is a case-insensitive string of alphanumeric characters (from 0-9 and A-Z) and the dash symbol (-) to separate different parts of the ID, which is unique for every NRS entry.

It's the only thing guaranteed to be different among entries, since a lot of manga/light novel/visual novel and its anime adaptation share the same title.

NRS-ID should be short and informative, so it has a limit of 24 characters.

### 1. Anime ID

The ID for anime is specified as follow:

* If the anime has an MyAnimeList (MAL, [website](myanimelist.net)) entry, the ID is the same anime ID as the MAL ID. (For example: Idoly Pride's MyAnimeList website is https://myanimelist.net/anime/40842/Idoly_Pride, so the MAL ID is 40842, therefore the NRS-ID is 40842). There is no prefixes for MAL animes, due to backward compability.
* Otherwise, fallback anime websites like AniList(AL, [website](anilist.co)), AniDB(ADB, [website](anidb.net)), Kitsu(KS, [website](kitsu.io)) ID is taken (priority order: AniList, AniDB, Kitsu) with their prefixes. (For example: The above anime, Idoly Pride, has the AniList website: https://anilist.co/anime/113814/IDOLY-PRIDE, so its ID should be AL113814 if its MAL page got taken down (which is nearly impossible))
* If there is no entry for this anime on MAL, AL, ADB or KS, its ID will be specified using a custom ID system with the prefix A. The suffix ID must only contains only digits. (A1, A023 are accepted, but AL123 is not)

### 2. Manga/Light Novel ID

The ID for manga/light novel is specified as follow:

* If the manga/light novel has an MAL entry, the ID is the same ID as the MAL ID, but with the prefix L.
* Otherwise, the ID will be specified using a custom ID system (which can only contains digits) with the prefix LP

### 3. Visual Novel ID

The ID for visual novel (VN) is basically the same as Manga/Light Novel, but with some differences:

* MAL is replaced by vndb (stands for (the) visual novel database, [website](vndb.org)).
* The prefix for a visual novel ID is V instead of L
* If an entry doesn't have a vndb entry, it's not considered as an visual novel entry, but rather a game entry.

#### ID for routes

Although visual novel routes are not separate from each other (each route contribute to the same NRS entry), there is a ID system for VN routes. The current system only supports VN with routes gotten by narrative choices only, and there are at most 5 choices for every question/situation.

In order to get the ID for route, you'll have to do these following steps:

* Record the choice string, which is a string of digits storing the choices indices (starts from 1) in chronological order.

  For example, the first recorded Aokana route is Route 211111..., which can be achieved by selecting the second choice in the first question (Worry about her.), the first choice in the next question (Because I want you to know how to fly.), etc.

* Compress the choice string as an alphanumeric string. Using the following python script:

  ```python
  def compress_route_string(string):
      ret = ''
      for i in range(0, len(string), 2):
          # for each pair of characters (as a digit)
          first = int(string[i])
          second = 0 if i + 1 >= len(string) else int(string[i + 1])
          if second == 0:
              # swap `first` and `second` if second is 0 (to make the compressed character make sense)
              second = first
              first = second
          # compress `first` and `second` into an alphanumeric character
          ret += '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'(first, second)
      return ret
  
  # print the compressed route string for 211111
  # print(compress_route_string('211111'))
  ```

### 4. Music/Game ID

Both of these entries will use a custom ID system (which can only contains digits), with prefixes M and G respectively.

Note: A visual novel (must have a vndb entry), which can be considered as a game, is always indexed by the visual novel rule.

## Rating System

### Overall score

An entry overall score is calculated as the sum of all its "subscores".

A subscore is calculated using its weight, which is constant for all NRS entries, multiplied with the raw score. 

### Emotion Score

Emotion score is an important part of an entry (weight: 0.85), and it depends on two factors: peak momentary emotion score and the PADS length.

The peak momentary emotion score is specified by a standard (the NRS-PMES standard).

PADS is an abbreviation for Post Anime Depression Syndrome. The meaning of that word is also expanded a lot. In the official PADS specs, PADS is a state of mind when negative emotion (sadness, loneliness, etc. but not hate or anger) appear because of an NRS-entry.

The formula for raw emotion score is calculated as follow:

```mathematica
RawEmotionScore = PeakMomentaryEmotionScore * 0.75 + min(5, PADSLengthInDays) * 0.5
```

### Meme Score

Meme score has a weight of 0.15. It depends on two major factors: the meme strength score and the duration score.

Since meme strength score doesn't have a standard and a rule, it has a pretty low weight (0.1) (also, its range is 0-10), This makes the duration vital in meme scores.

There is also a score named meme compensation score. If an anime's meme is killed by another anime, the compensation score will be that another anime's raw meme score, multiplied by 0.1. Otherwise, it's just left as 0.

Meme score formulas:

```mathematica
RawMemeScore = MemeStrengthScore * 0.1 + MemeDurationScore * 0.9 + MemeCompensationScore
MemeCompensationScore = 0.1 * RawMemeScoreOfTheAnimeKilledThisAnimeMeme
```

### Music Score

Music score has no standards or rules, range 0-10, has a weight of 0.05.

### Waifu Score

Waifu is a minor factor of an anime, so it only has a weight of 0.05.

NRS defined a waifu as "a character from a NRS-entry which was/is influential in a period of time". This is an expansion of the original definition for waifu, since all NRS waifus (till now) are seasonal waifus.

In order to be a NRS waifu, a character has to be a "best girl" of at least one entry (which is the most influential character in the entry). The next requirement is to be influential. There is a not-well-defined threshold of how influential for someone to be a NRS waifu.

Previously, it's defined that the character with the longest influential time is going to be the official waifu. But after Hatsune Miku took the reign, this will not be a thing anymore. Also, this event make waifu score obsolete, and in NRS version 1.1, waifu score will be removed from NRS (or merge into meme score).

To calculate a waifu score of an entry, you need a rather complicated formula, which depends on how long the waifu is influential.

The formula is defined as follow:

```mathematica
RawWaifuScore = -10 + 20 / (1 + exp(-InfluentialTimeInDays / MaxInfluentialTimeInDays * 6))

(* MaxInfluentialTimeInDays used to be the longest influential time of all waifus, but now it's hardcoded to 180 days to make implementing NRS a bit more easier. *)
```

The formula is basically a logistic function. It's to balance everything out and make the waifu score balanced.

### Legacy/Future/Reputation score (intended for NRS1.1)

A lot of anime with high score has a bad legacy/reputation, most notably Love Live Nijigaku. Meanwhile, there are a lot of underrated animes with good legacy/reputation. A "reputation" of an entry is defined to be the feeling of me for it in a relatively long time after watching/reading/etc. it.

Its weight is not specified (WIP), but it will be ranged from 0.01 to 0.05 due to the lack of standard and a rule. Its range is 0-10.

### Additional Score

Additional scores are scores that are specific for an entry. It doesn't have a standard or a rule, usually range from 0-1. There are some common additional score category like gate opening score or short anime buff.

## Standards

### Peak Momentary Emotion Standard

|                  Impact                   | Score |
| :---------------------------------------: | :---: |
|                Was dropped                |   3   |
|                Was on-hold                |   4   |
|   Completed without major interruption    |   5   |
| Was hyped for new episode (seasonal only) |   6   |
|       Has a minor emotional impact        |  6.5  |
|                Good comedy                |   7   |
|        Heartwarming (CGDCT/Idols)         |  7.5  |
|          Shocking plot (Action)           |   8   |
|       Has an major emotional impact       |   8   |
|            Good drama/romance             |  8.5  |
|               Almost cried                |   9   |
|                   Tears                   |  10   |

### Meme duration standard

|           Duration            | Score |
| :---------------------------: | :---: |
|        Less than a day        |   0   |
|           1-3 days            |   1   |
|           4-7 days            |   3   |
|           1-2 weeks           |   5   |
|           2-3 weeks           |   6   |
|       3 weeks - 1 month       |   7   |
|           1-2 month           |   8   |
|          2-3 months           |   9   |
| More than 3 months (a season) |  10   |

