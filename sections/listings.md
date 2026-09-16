Listings
========

Directory listings for a podcast. There is no directory search. GET every directory (listed and unlisted) and inspect `status`, then submit remaining one-click directories in one request. Submitting requires **admin or owner**.

List directories
----------------

* `GET /api/:podcast_id/listings`

`status` is one of `listed`, `submitted`, `unlisted`, or `failed`. Pandora is omitted. Unlisted directories that have never been submitted have `id: null`.

```json
[
  {
    "id": 10,
    "directory": "Apple",
    "name": "Apple Podcasts",
    "status": "listed",
    "url": "https://podcasts.apple.com/podcast/id123"
  },
  {
    "id": null,
    "directory": "Spotify",
    "name": "Spotify",
    "status": "unlisted",
    "url": null
  }
]
```

Submit all
----------

* `POST /api/:podcast_id/listings/submit_all` — **admin or owner only**.
* Submits every remaining one-click directory that is not already listed or submitted. Directories that require a manual process are left alone; GET listings again later to see `listed` or `failed`.
* Returns `202 Accepted` and the listings array (same shape as GET).

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/listings/submit_all" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

Errors
------

If the authenticated user's email is unverified, or the podcast lacks a published episode, artwork, description, author, or category:

```json
{
  "error": {
    "code": "ineligible",
    "message": "Podcast is not eligible to submit to directories",
    "param": null
  }
}
```

Editors who attempt submit receive `403` with `code: "forbidden"`.
