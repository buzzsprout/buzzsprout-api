Episodes
========

Create, inspect, update, schedule, and unpublish episodes for a podcast.

Typical workflow
----------------

1. Create a private episode (`POST`).
2. [Upload audio or video](uploads.md), then complete the upload.
3. `GET .../episodes/:id` after processing finishes (`duration` will be a positive integer).
4. Optional: [chapters](chapters.md), [contributors](contributors.md).
5. Publish with `PATCH` and `"private": false` (optionally set `published_at` to schedule).

Unpublish episodes with `{ "private": true }`.

List and show
-------------

- `GET /api/:podcast_id/episodes`
- `GET /api/:podcast_id/episodes/:id`

Both return the episode representation. The list endpoint returns an array.

```json
{
  "id": 788881,
  "title": "Too small or too big?",
  "audio_url": "https://www.buzzsprout.com/140447/788881-filename.mp3",
  "artwork_url": "https://example.com/artwork.jpg",
  "description": "",
  "artist": "Muffin Man",
  "tags": "",
  "published_at": "2026-09-12T03:00:00.000-04:00",
  "duration": 1236,
  "guid": "Buzzsprout788881",
  "custom_url": null,
  "episode_number": 5,
  "season_number": 2,
  "episode_type": "full",
  "explicit": false,
  "private": false,
  "total_plays": 150
}
```

Create
------

- `POST /api/:podcast_id/episodes`
- Returns `201 Created` with the episode.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "title": "Too many or too few?",
    "description": "What we learned this week.",
    "private": true,
    "episode_number": 6,
    "season_number": 2,
    "episode_type": "full",
    "explicit": false
  }'
```

When `published_at` is omitted, the episode is private by default and `published_at` is set to the start of the current hour in the podcast's timezone. A future `published_at` with `"private": false` schedules publication.

Accepted write fields: `title`, `description`, `artist`, `tags`, `published_at`, `season_number`, `episode_number`, `episode_type`, `explicit`, `private`, and `custom_url`. For artwork, send `artwork_url`, or an `artwork_file` in a multipart request.

Field notes
-----------

- `tags` — comma-separated string, not a JSON array (for example `"news,interview"`).
- `published_at` — include a timezone offset (for example `2026-09-12T09:00:00-04:00`). Without an offset, the podcast timezone is used.
- `episode_type` — `full`, `trailer`, or `bonus`.
- `inactive_at` — set when an episode is removed from the active library. It is not the same as unpublishing; use `"private": true` to unpublish.
- `explicit` — on update, an omitted or blank value is treated as `false`.

Update or unpublish
-------------------

- `PATCH /api/:podcast_id/episodes/:id`
- `PUT /api/:podcast_id/episodes/:id`
- Returns `200 OK` with the updated episode.

```bash
curl -sS -X PATCH "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "private": false,
    "published_at": "2026-09-12T09:00:00-04:00",
    "explicit": false
  }'
```

Media and artwork
-----------------

Prefer [Upload audio or video](uploads.md) to attach or replace media. Create the episode first, then start → PUT → complete; without **complete**, the episode has no media.

### Deprecated audio fields

`audio_url` (Buzzsprout fetches a remote file) and `audio_file` (sent on the episode create/update request) still work for existing integrations, but new work should use the upload endpoints. Sending `audio_file` through create/update is deprecated: those responses include a `Deprecation` header and a `warning` object. The legacy-only `email_user_after_audio_processed` flag applies to that older audio path, not to the upload endpoints.

Errors
------

Episode create and update use a legacy error format: validation failures are field hashes (HTTP `400`), while bad credentials and inaccessible podcasts are plain-text bodies. Newer resources elsewhere in the API use `{ "error": { "code", "message", "param" } }`.
