Chapters
========

Replace an [episode](episodes.md)'s chapter list in one request. There is no single-chapter create/update/delete.

GET the current list, edit it, then PUT the replacement. Send either `{ "chapters": [...] }` or a raw array. An empty array deletes every non-ad chapter. Omitting the array or sending another type returns `422` without changing existing chapters.

List chapters
-------------

* `GET /api/:podcast_id/episodes/:episode_id/chapters`

```json
[
  {
    "id": 1,
    "title": "Intro",
    "start_time": "00:00:00",
    "url": null
  }
]
```

Replace chapters
----------------

* `PUT /api/:podcast_id/episodes/:episode_id/chapters`

```bash
curl -sS -X PUT "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/chapters" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "chapters": [
      { "title": "Intro", "start_time": "00:00:00" },
      { "title": "Main", "start_time": "00:01:30", "url": "https://example.com" }
    ]
  }'
```

A successful replace returns the new chapter list (same shape as GET). `start_time` may be `HH:MM:SS`, `MM:SS`, or seconds. `url` and `link` are both accepted on write; the response always uses `url`.

Errors
------

Invalid payloads return:

```json
{
  "error": {
    "code": "invalid",
    "message": "chapters must be an array",
    "param": "chapters"
  }
}
```
