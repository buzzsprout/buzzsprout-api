Contributors
============

Contributors are on-show credits (Host, Co-Host, Producer, Editor, Guest). They are not [team](team.md) members and do not get a Buzzsprout login. Create a contributor on the podcast first, then assign that contributor to [episodes](episodes.md).

`bio` is an HTML string. The 1000-character limit counts the HTML, including tags.

List and create
---------------

* `GET /api/:podcast_id/contributors`
* `POST /api/:podcast_id/contributors`

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/contributors" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "name": "Guest Star",
    "role": "Guest",
    "bio": "<p>Hello</p>",
    "url": "https://example.com"
  }'
```

Roles may be sent as `Guest` or `guest`. The response uses the canonical label (`Host`, `Co-Host`, `Producer`, `Editor`, `Guest`). Create returns `201 Created`.

```json
{
  "id": 123,
  "name": "Guest Star",
  "role": "Guest",
  "bio": "<p>Hello</p>",
  "url": "https://example.com"
}
```

List returns an array of the same objects.

Show, update, delete
--------------------

* `GET /api/:podcast_id/contributors/:id`
* `PATCH /api/:podcast_id/contributors/:id`
* `DELETE /api/:podcast_id/contributors/:id`

Deleting a contributor removes them from **every** episode and deletes the contributor. Prefer episode unassign (below) when you only want them off one episode. Delete returns `204 No Content`.

Assign to an episode
--------------------

* `POST /api/:podcast_id/episodes/:episode_id/contributors` with `{ "contributor_id": 123 }` — returns `201 Created` and the contributor object.
* `DELETE /api/:podcast_id/episodes/:episode_id/contributors/:id` — unassigns without deleting the contributor. Returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/contributors" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{ "contributor_id": 123 }'
```

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "Name can't be blank",
    "param": "name"
  }
}
```
