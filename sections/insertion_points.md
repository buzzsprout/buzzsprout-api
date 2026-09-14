Insertion points
================

Mid-roll timestamps on processed [episode](episodes.md) audio. Returns `422` with `code: "unprocessable"` if the episode has no processed audio yet.

* `GET /api/:podcast_id/episodes/:episode_id/insertion_points`
* `POST /api/:podcast_id/episodes/:episode_id/insertion_points` with `{ "timestamp": "00:15:00" }` or `{ "seconds": 900 }`. Omitting both returns `422` with `param: "seconds"`.
* `PATCH /api/:podcast_id/episodes/:episode_id/insertion_points/:id`. Omitting `seconds` / `timestamp` leaves the existing time unchanged.
* `DELETE /api/:podcast_id/episodes/:episode_id/insertion_points/:id` — returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/insertion_points" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{ "timestamp": "00:15:00" }'
```

```json
{
  "id": 3,
  "seconds": 900,
  "timestamp": "00:15:00",
  "eligible": true
}
```

List returns an array of the same objects, ordered by `seconds`. New and updated insertion points default to `eligible: true`; include `{ "eligible": false }` on an update when that state must be preserved.

Manual placements must sit after the first 20% of the episode, before the last 20%, and at least six minutes apart.

Errors
------

```json
{
  "error": {
    "code": "unprocessable",
    "message": "Episode audio is not processed yet",
    "param": null
  }
}
```

Missing `seconds` / `timestamp` on create returns `code: "invalid"` with `param: "seconds"`.
