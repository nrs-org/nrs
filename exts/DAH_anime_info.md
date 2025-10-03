# Extension specification: DAH_anime_info

Version 1.0

Vendor: DAH

Dependencies:
```
NRS 2.0+
DAH_meta 1.0+
DAH_animanga_info 1.0+
```

## 1. Overview

This extension provides anime-specific metadata fields for entries. The data is stored in a dictionary where each key represents a data source, parallel to the structure of `DAH_animanga_info`. The source identifiers should follow the abbreviations defined in `DAH_entry_id_impl`.

Conflict resolution follows the dynamic priority system defined in `DAH_animanga_info`.

## 2. Fields

The `DAH_anime_info` object is a dictionary where keys are source identifiers (strings) and values are `anime_info` objects. Each `anime_info` object has the following fields:

- `lastUpdated`: string (ISO 8601) — The timestamp when the information was last updated from the source.
- `episodes`: number (override)
- `animeSeason`: object with `season` (string) and `year` (number) (override)
- `duration`: string | number (override)
- `studios`: string[] (merge)
- `producers`: string[] (merge)

## 3. Example

```json
{
  "DAH_meta": {
    "DAH_animanga_info": {
      "MAL": {
        "lastUpdated": "2025-09-20T18:00:00Z",
        "title": "Attack on Titan",
        "type": "TV",
        "status": "FINISHED",
        "picture": "https://cdn.myanimelist.net/images/anime/10/47347.jpg",
        "thumbnail": "https://cdn.myanimelist.net/images/anime/10/47347t.jpg",
        "synonyms": ["shingeki no kyojin"],
        "tags": ["action", "drama", "fantasy"]
      }
    },
    "DAH_anime_info": {
      "MAL": {
        "lastUpdated": "2025-09-20T18:00:00Z",
        "episodes": 25,
        "animeSeason": { "season": "Spring", "year": 2013 },
        "duration": "24 min",
        "studios": ["wit studio"],
        "producers": ["production i.g"]
      }
    }
  }
}
```
