# Extension specification: DAH_entry_progress

Version 1.0

Vendor: DAH

Dependencies:

```
NRS 2.0+
DAH_meta 1.0+
```

## 1. Overview

This extension assumes that the entry `DAH_entry_progress` in the `meta` object
of all entries to be an object storing the consumption progress of the entry.

## 2. Entry statuses

There are 5 kinds of entry statuses:

* Completed: the entry is completed.
* Watching: the entry consumption is on-going.
* Dropped: the entry consumption is halted and there are no plans to resume it.
* On-hold: the entry consumption is paused and may be resume at any time.
* Unwatched: the entry consumption hasn't started.

The entry status is stored in the entry `status` of the `DAH_entry_progress` object.

## 3. Consumption length

The consumption length is the total time spent by the consumer consuming the entry.
This length in seconds is stored in the entry `length_seconds` of the
`DAH_entry_progress` object.

## 4. Anime episode count

When the entry is an anime, an additional entry `episode` is then used to store
the episode count in the `DAH_entry_progress` object. This can be used to
synchronize the NRS list to anime list services such as MyAnimeList and AniList.
