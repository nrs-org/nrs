# Extension specification: DAH_manga_info

Version 1.0

Vendor: DAH

Dependencies:
```
NRS 2.0+
DAH_meta 1.0+
DAH_animanga_info 1.0+
```

## 1. Overview

This extension provides manga/light novel-specific metadata fields for entries. The data is stored in a dictionary where each key represents a data source, parallel to the structure of `DAH_animanga_info`. The source identifiers should follow the abbreviations defined in `DAH_entry_id_impl`.

Conflict resolution follows the dynamic priority system defined in `DAH_animanga_info`.

## 2. Fields

The `DAH_manga_info` object is a dictionary where keys are source identifiers (strings) and values are `manga_info` objects. Each `manga_info` object has the following fields:

- `lastUpdated`: string (ISO 8601) — The timestamp when the information was last updated from the source.
- `chapters`: number (override)
- `volumes`: number (override)

## 3. Example

```json
{
  "DAH_meta": {
    "DAH_animanga_info": {
      "MAL": {
        "lastUpdated": "2025-10-02T20:00:00Z",
        "title": "One Piece",
        "type": "Manga",
        "status": "ONGOING",
        "picture": "https://cdn.myanimelist.net/images/manga/3/55539.jpg",
        "thumbnail": "https://cdn.myanimelist.net/images/manga/3/55539t.jpg",
        "synonyms": ["op"],
        "tags": ["adventure", "action", "comedy"]
      }
    },
    "DAH_manga_info": {
      "MAL": {
        "lastUpdated": "2025-10-02T20:00:00Z",
        "chapters": 1100,
        "volumes": 105
      }
    }
  }
}
```
