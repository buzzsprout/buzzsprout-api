Brand affiliations
==================

Sponsor and affiliate links for a podcast. Create them on the show, then assign them to [episodes](episodes.md). Assigned links are appended to the episode description in RSS.

`artwork` is an optional image file on create or update (multipart form field alongside the other attributes).

List and create
---------------

* `GET /api/:podcast_id/brand_affiliations`
* `POST /api/:podcast_id/brand_affiliations`

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/brand_affiliations" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "headline": "Acme",
    "description": "10% off with code PODCAST",
    "url": "https://acme.example"
  }'
```

Create returns `201 Created`. A missing `headline` returns `422` with `param: "headline"`.

```json
{
  "id": 45,
  "headline": "Acme",
  "description": "10% off with code PODCAST",
  "url": "https://acme.example",
  "artwork_url": null
}
```

When artwork is attached, `artwork_url` is a URL to that image. List returns an array of the same objects.

Show, update, delete
--------------------

* `GET /api/:podcast_id/brand_affiliations/:id`
* `PATCH /api/:podcast_id/brand_affiliations/:id`
* `DELETE /api/:podcast_id/brand_affiliations/:id`

Deleting a brand affiliation also removes it from **every** episode. Prefer episode unassign (below) when you only want it off one episode. Delete returns `204 No Content`.

Assign to an episode
--------------------

* `POST /api/:podcast_id/episodes/:episode_id/brand_affiliations` with `{ "brand_affiliation_id": 123 }` — returns `201 Created` and the brand affiliation object.
* `DELETE /api/:podcast_id/episodes/:episode_id/brand_affiliations/:id` — unassigns without deleting the affiliation. Returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/brand_affiliations" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{ "brand_affiliation_id": 123 }'
```

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "Headline can't be blank",
    "param": "headline"
  }
}
```
