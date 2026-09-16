Stats
=====

Download activity for one podcast-local calendar date. Use the podcast endpoint to see which episodes had downloads that day, then page through one episode's normalized details. These endpoints do not accept a date range.

`date` is required and must be `YYYY-MM-DD`. It uses the podcast-local date, not the download's UTC timestamp. Invalid or missing dates return `422` with `param: "date"`.

Podcast downloads
-----------------

* `GET /api/:podcast_id/downloads?date=YYYY-MM-DD`
* Returns only episodes with at least one download on that date, ordered by episode ID.

```bash
curl -sS "https://www.buzzsprout.com/api/PODCAST_ID/downloads?date=2026-09-12" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

```json
[
  {
    "episode_id": 788881,
    "total": 2
  },
  {
    "episode_id": 788882,
    "total": 1
  }
]
```

A day with no activity returns `[]`.

Episode downloads
-----------------

* `GET /api/:podcast_id/:episode_id/downloads?date=YYYY-MM-DD`
* Returns paginated download details for one episode and date.
* Defaults to `page=1` and `per_page=100`. `per_page` may be at most `1000`.
* Invalid `page` or `per_page` values return `422` with that parameter named.

This route is `/api/:podcast_id/:episode_id/downloads`, not under `/episodes`.

```json
{
  "episodes": [
    {
      "id": 788881,
      "downloads": [
        {
          "app": "Apple Podcasts",
          "device": "Apple iPhone",
          "device_type": "Phone",
          "country_code": "US",
          "region": "Texas",
          "city": "Austin",
          "continent": "North America",
          "media_type": "audio"
        },
        {
          "app": "Spotify",
          "device": "Apple iPhone",
          "device_type": "Mobile",
          "country_code": "US",
          "region": "Florida",
          "city": "Jacksonville",
          "continent": "North America",
          "media_type": "video"
        }
      ]
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 100,
    "has_more": false,
    "next_page": null
  }
}
```

Each object is one download. Follow `pagination.next_page` while `has_more` is `true`. A full final page has no next page.

A day with no activity still returns the episode, with an empty `downloads` array. Missing episode IDs, or episodes that do not belong to the podcast, return `404`.

Normalized fields
-----------------

Missing client values become `Unknown` for `app` and `device_type`, and `Unknown Device` for `device`. Missing geography is `null`. `media_type` is `audio` or `video`. Raw user agents are not returned.

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "date is required and must be a valid date in YYYY-MM-DD format",
    "param": "date"
  }
}
```
