Feed verifications
==================

Manage the `<podcast:txt>` values in your RSS feed.

* `GET /api/:podcast_id/feed_verifications`
* `POST /api/:podcast_id/feed_verifications`
* `PATCH /api/:podcast_id/feed_verifications/:id`
* `DELETE /api/:podcast_id/feed_verifications/:id`

POST creates the podcast's generic `verify` value, or replaces the existing one. `purpose` is returned but cannot be set or changed through this API. Writable fields are `value` and optional `expires_at`.

Create returns `201 Created`. Delete returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/feed_verifications" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "value": "owner@example.com",
    "expires_at": "2027-09-01T00:00:00Z"
  }'
```

```json
{
  "id": 7,
  "value": "owner@example.com",
  "purpose": "verify",
  "expires_at": "2027-09-01T00:00:00.000Z"
}
```

List returns an array of the same objects. `expires_at` may be `null` when not set.

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "Value can't be blank",
    "param": "value"
  }
}
```
