Podrolls
========

Recommended podcasts that appear in your RSS feed. They can be any podcast (not only Buzzsprout). There is no search endpoint—send `guid` and `feed_url` yourself (for example from [Podcast Index](https://podcastindex.org)). 

* `GET /api/:podcast_id/podrolls`
* `POST /api/:podcast_id/podrolls`
* `PATCH /api/:podcast_id/podrolls/:id`
* `DELETE /api/:podcast_id/podrolls/:id`

Create returns `201 Created`. Delete returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/podrolls" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "guid": "29cdca4a-32d8-56ba-b48b-09a011c5daa9",
    "feed_url": "https://feeds.example.com/show.rss"
  }'
```

```json
{
  "id": 12,
  "guid": "29cdca4a-32d8-56ba-b48b-09a011c5daa9",
  "feed_url": "https://feeds.example.com/show.rss",
  "position": 1
}
```

List returns an array of the same objects. On update, send `{ "position": 1 }` (1-based; `0` is treated as `1`) and optionally `feed_url` / `guid`.

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "Feed url can't be blank",
    "param": "feed_url"
  }
}
```
