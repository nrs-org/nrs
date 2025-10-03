# Extension specification: DAH_animanga_info

Version 1.0

Vendor: DAH

Dependencies:
```
NRS 2.0+
DAH_meta 1.0+
```

## 1. Overview
This extension provides a unified metadata schema for animanga entries (anime, manga, light novel, etc.). The data is stored in a dictionary where each key represents a data source (e.g., `MAL`, `AL`). The source identifiers should follow the abbreviations defined in `DAH_entry_id_impl`. This allows for storing information from multiple external sources.

A special source key, `USER`, is reserved for user-provided overrides.

### Conflict Resolution
When resolving data from multiple sources, a dynamic priority system is used. The priority of each source is calculated at runtime based on the `lastUpdated` timestamp of its data. The formula is:
`priority = base_priority * exp(-elapsed_time / tau)`

- `base_priority`: A predefined base value for each source (e.g., MAL: 10, AL: 9).
- `elapsed_time`: The time since the data was last updated.
- `tau`: A time constant that controls the rate of decay (e.g., 30 days).

The `USER` source is a special case and always has the highest priority.

Fields are resolved using one of two strategies:
- `override`: The value from the highest-priority source is taken.
- `merge`: Values from all sources are collected into an array, with duplicates removed.

## 2. Fields
The `DAH_animanga_info` object is a dictionary where keys are source identifiers (strings) and values are `info` objects. Each `info` object has the following fields:

- `lastUpdated`: string (ISO 8601) — The timestamp when the information was last updated from the source.
- `title`: string — Main title (override)
- `type`: string — Enum: UNKNOWN or
  - Anime: TV, MOVIE, OVA, ONA, SPECIAL
  - Manga: MANGA, ONE_SHOT, DOUJINSHI, MANHWA, MANHUA, OEL
  - Novel: LIGHT_NOVEL, WEB_NOVEL
- `status`: string — Enum: FINISHED, ONGOING, UPCOMING, UNKNOWN (override)
- `picture`: string — Main image URL (override)
- `thumbnail`: string — Thumbnail image URL (override)
- `synonyms`: string[] — Alternative titles (merge)
- `tags`: string[] — Lowercase genre/tags (merge)
- `description`: string — HTML description/sypnosis of the entry (override)

## 3. Example
```json
{
  "DAH_meta": {
    "DAH_animanga_info": {
      "MAL": {
        "lastUpdated": "2025-10-01T12:00:00Z",
        "title": "Fullmetal Alchemist: Brotherhood",
        "type": "TV",
        "status": "FINISHED",
        "picture": "https://cdn.myanimelist.net/images/anime/1223/96541.jpg",
        "thumbnail": "https://cdn.myanimelist.net/images/anime/1223/96541t.jpg",
        "synonyms": [
          "FMA Brotherhood",
          "Hagane no Renkinjutsushi: Fullmetal Alchemist"
        ],
        "tags": [
          "action",
          "adventure",
          "drama",
          "fantasy",
          "magic",
          "military",
          "shounen"
        ]
      },
      "USER": {
        "title": "FMA:B",
        "tags": ["best anime ever"]
      }
    }
  }
}
```