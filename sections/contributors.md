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
  "url": "https://example.com",
  "headshot_url": null
}
```

List returns an array of the same objects.

`headshot_url` is the image URL, or `null` when the contributor has no headshot. It is included in contributor list, show, create, and update responses.

Show, update, delete
--------------------

* `GET /api/:podcast_id/contributors/:id`
* `PATCH /api/:podcast_id/contributors/:id`
* `DELETE /api/:podcast_id/contributors/:id`

Deleting a contributor removes them from **every** episode and deletes the contributor. Prefer episode unassign (below) when you only want them off one episode. Delete returns `204 No Content`.

### Upload a headshot

To add or replace a headshot, send `headshot_file` as a file in a multipart request when creating or updating a contributor. PNG, JPEG, and WebP images up to 15 MB are accepted. Omitting the file on an update leaves the existing headshot unchanged.

```bash
curl -sS -X PATCH "https://www.buzzsprout.com/api/PODCAST_ID/contributors/CONTRIBUTOR_ID" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json" \
  -F "headshot_file=@headshot.jpg"
```

For a new contributor, use `POST /api/:podcast_id/contributors` and include fields such as `name` and `role` in the same multipart request. Invalid images return `422 Unprocessable Entity` with `code: "invalid"` and `param: "headshot"`; an invalid replacement does not change the existing headshot.

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
