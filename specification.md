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

From version 2.0, emotion score is calculated as the weighted sum of all base emotion scores. The base emotion scores are calculated by combining all of the representative score for that base emotion (for example, Sana's backstory and Madokami's MGS scores are Magireco's representative scores for the base emotion sad/depression) using the following function with w = 0.2

```mathematica
Combine([x_1, x_2, ..., x_n], w) = y_1 + y_2 * w + ... + y_n * w^(n-1), where y_(1..n) is x_(1..n) in descending order.
```

Example:
Let's assume Magireco only:
* made me cry because of Sana's backstory and Madokami's MGS
* have a good community

Sana's backstory score: 6 + min(6, 1.2 * 0) * 2/3 = 6 (no PADS)  
Madokami's MGS score: 6 + min(6, 1.2 * inf) * 2/3 = 10 (PADS length unknown, but longer than 5)  
Good community score: 10  

The sad/depression score: 10 + 6 * 0.2 = 11.2  
The total raw emotion score: 11.2 * 0.7 + 10 * 0.01 = 7.94
The emotion score: 7.94 * 0.85 = 6.749

#### 1. Sad/Depression Score (weight = 0.7)

> Small Note: Sad is negative emotion with a target (a character from the entry). Depressions have no targets, or you can say the target is myself. There's little need to separate the two though.

The standard to calculate representative scores is NRS-SD.

PADS is an abbreviation for Post Anime Depression Syndrome. The meaning of that word is also expanded a lot. In the official PADS specs, PADS is a state of mind when negative emotion (sadness, loneliness, etc. but not hate or anger) appear because of an NRS-entry. PADS can also happen because of other emotions (see NRS-JH and NRS-CH standards)

#### 2. Journey-Hype Score (weight = 0.5)

When an anime is able to take you on an adventure, you become one with the group (that took on that adventure), and therefore, feel the same feeling that they did. These emotions usually consist of faith and happiness.

The standard to calculate representative scores is NRS-JH.

#### 3. Comfy/Heartwarming Score (weight = 0.3)

When the interaction of a group of characters with each other is wholesome, the atmosphere of the anime become more comfy/heartwarming, therefore giving me the feeling of comfort.

The standard to calculate representative scores is NRS-CH.

#### 4. Love Score (weight = 0.2)

When I love something (a character or something) in an entry, which isn't music (since music score is a thing) or waifu (until waifu score is removed), and if that love is strong enough, it'll be taken into account.

The intensity of love is measured using the influential time of the target with this formula:

```mathematica
(* same formula as the old waifu score *)
RepresentativeLoveScore = -10 + 20 / (1 + exp(-InfluentialTimeInDays / 30))
```

#### 5. Humor Score (weight = 0.2)

> TODO

#### 6. Thriller Score (weight = 0.4)

> TODO

#### 7. Educational Score (weight = 0.1)

The standard to calculate representative scores is NRS-ED.

#### 8. Horror Score (weight = 0.05)

The standard to calculate representative scores is NRS-HR.

#### 9. Community Score (weight = 0.05)

The positivity of an entry's fandom can also affect the emotion score, since it has a great effect on how I feel of that entry (e.g. Love Live NijiGaku and Magia Record).

The standard for this is NRS-CM.

#### 10. Boringness Score (weight = 0.3)

A completed anime will get more score than an on-hold completed one, and an on-hold completed one will get more than a dropped anime.

The standard to score is NRS-BR.

#### 11. Special Emotions (weight = 1)

Emotion that was not mentioned above will be scored here. Despite the lack of standard, the weight is 1, therefore special emotion scores have to be scored fairly and carefully.

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

### Legacy/Future/Reputation score (intended)

A lot of anime with high score has a bad legacy/reputation, most notably Love Live Nijigaku. Meanwhile, there are a lot of underrated animes with good legacy/reputation. A "reputation" of an entry is defined to be the feeling of me for it in a relatively long time after watching/reading/etc. it.

Its weight is not specified (WIP), but it will be ranged from 0.01 to 0.05 due to the lack of standard and a rule. Its range is 0-10.

### Additional Score

Additional scores are scores that are specific for an entry. It doesn't have a standard or a rule, usually range from 0-1. There are some common additional score category like gate opening score or short anime buff.

## Standards

### Sad/Depression Standard (NRS-SD)

|         Impact         |                 Score                 |
| :--------------------: | :-----------------------------------: |
|  Noticable Negativity  |                  1÷2                  |
| Appreciable Negativity |                  4÷5                  |
|     PADS occurred      | 5 + min(4, 0.8 * number of PADS days) |
|         Cried          | 6 + min(4, 0.8 * number of PADS days) |

### Journey-Hype Standard (NRS-JH)

|     Impact     |                     Score                     |
| :------------: | :-------------------------------------------: |
| Noticable Hype |                      1÷2                      |
| PADS occurred  | (5 + min(4, 0.8 * number of PADS days)) * 7/5 |
|     Cried      | (6 + min(4, 0.8 * number of PADS days)) * 7/5 |
> multiply with 7/5 to balance with PADS/cry score for sad/depression

### Comfy/Heartwarming Standard (NRS-CH)

|       Impact        |                     Score                     |
| :-----------------: | :-------------------------------------------: |
| Noticable Comfiness |                      1÷2                      |
|    PADS occurred    | (5 + min(4, 0.8 * number of PADS days)) * 7/3 |
|        Cried        | (6 + min(4, 0.8 * number of PADS days)) * 7/3 |
> multiply with 7/3 to balance with PADS/cry score for sad/depression

### Horror Standard (NRS-HR)
|         Impact          | Score |
| :---------------------: | :---: |
|  Successful jumpscare   |   6   |
| Cause a sleepless night |  10   |

### Educational Standard (NRS-ED)
|                 Impact                  | Score |
| :-------------------------------------: | :---: |
| Inspire a political discovery/invention |   7   |
|    Make me interested in known field    |   8   |
|    Make me interested in a new field    |  10   |

### Boringness Standard (NRS-BR)

|    Status    | Score |
| :----------: | :---: |
|   Dropped    |   2   |
|   On-hold    |   6   |
| Kinda boring |  6-8  |
|  Completed   |  10   |
|    Other     |   5   |
> "Other" status is for stuff that can't be completed (Music, No-end video game entries, etc.)

### Community Score Standard (NRS-CM)

| Community Impression | Score |
| :------------------: | :---: |
|         Bad          |  -5   |
|       Neutral        |   0   |
|         Good         |  10   |

### Meme Duration Standard (NRS-ME)

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

