# Extension specification: DAH_entry_id_impl

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
DAH_entry_id 1.0+
```

## Overview

This extension implement the entry ID mapping function from the extension `DAH_entry_id`.

## Modification

### ID syntax

An entry ID must be a string, which is the concatenation of all ID components using the separator `-`. The list of ID components for an entry is determined as follow:

```python
# XYZ* means that XYZ is optional
# standard id
IDComponents = [TypePrefix, DatabaseID, SubTypePrefix*, EntryIDInDatabase, Suffix*]
# custom id (entries with no database)
IDComponents = [TypePrefix, CustomID]
```

### Type Prefixes:

* A: Anime

* M: Music

* L: Manga/Light Novel

* V: Visual Novel

* F: Franchise

* G: Game

* GF: Game from Franchise

### Database IDs and Priorities

Anime:

- MAL: [MyAnimeList](https://myanimelist.net)
- AL: [AniList](https://anilist.co)
- ADB: [AniDB](https://anidb.net)
- KS: [Kitsu](https://kitsu.io)

Visual Novel:

- VNDB: [vndb](https://vndb.org)

Video Game, Franchise, Music:

- VGMDB: [vgmdb](https://vgmdb.net)
  
  VGMDB has two subtype prefixes: AL for album, AR for artists

### Notes

- Visual Novels are defined as entries with a vndb entry, so there are no custom ID system for Visual Novels.
- Music track entries will use its album ID base ID, and the track number as its suffix, so the ID of `OvertuRe:` will be "M-VGMDB-AL-89363-2" (it's in the album `DRe:AmEr (KiRaRe ver.)`, id "M-VGMDB-AL-89363").
- For franchises with multiple games, and none of them have a separate VGMDB entries, the games will share the same database ID with the owner franchise, but using the suffix as the index of that game in the game list (start from 1).
- 
