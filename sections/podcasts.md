Podcasts
========

List, inspect, and update podcasts available to the authenticated user. Any member can list and show. Updating a podcast requires **admin or owner**.

List and show
-------------

- `GET /api/podcasts` — all podcasts available to the authenticated user
- `GET /api/podcasts/:id` — one available podcast

These routes do not use a podcast ID before `podcasts`.

```bash
curl -sS "https://www.buzzsprout.com/api/podcasts" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

```json
{
  "id": 10,
  "title": "Life Empowerment",
  "author": "Motivational Mike",
  "description": "Practical ways to live your best life.",
  "website_address": "https://example.com",
  "website_url": "https://example.com",
  "contact_email": "mike@example.com",
  "keywords": "motivation, life",
  "explicit": false,
  "main_category": "Arts",
  "sub_category": "Books",
  "main_category2": null,
  "sub_category2": null,
  "main_category3": null,
  "sub_category3": null,
  "language": "en-us",
  "timezone": "Eastern Time (US & Canada)",
  "artwork_url": "https://example.com/artwork.jpg",
  "background_url": null,
  "rss_url": "https://feeds.buzzsprout.com/10.rss"
}
```

`rss_url` is the podcast's RSS feed. `website_address` is the podcast's configured external website. `website_url` uses that address when set; otherwise, it returns the custom or default Buzzsprout website URL if the plan includes a website, or `null`.

Update a podcast
----------------

- `PATCH /api/podcasts/:id`
- `PUT /api/podcasts/:id`
- Requires an admin or owner. Editors receive `403 Forbidden`.
- Send a flat JSON object, not `{ "podcast": { ... } }`.

```bash
curl -sS -X PATCH "https://www.buzzsprout.com/api/podcasts/PODCAST_ID" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "title": "Life Empowerment",
    "description": "A new description",
    "language": "en-us",
    "timezone": "America/New_York",
    "website_address": "https://example.com",
    "category": "Arts :: Books",
    "category2": "Education",
    "category3": null,
    "artwork_url": "https://example.com/artwork.jpg"
  }'
```

Writable fields are `title`, `description`, `keywords`, `author`, `explicit`, `language`, `timezone`, `website_address`, `contact_email`, `category`, `category2`, and `category3`. Write a category as `"Main :: Sub"`; the response splits it into `main_category` and `sub_category` fields.

`website_url` and `rss_url` are read-only. After changing `artwork_url`, the update response may still contain the previous image URL while the new artwork is processed. A successful update returns the podcast object shown above.

Errors
------

Validation failures use:

```json
{
  "error": {
    "code": "invalid",
    "message": "Title can't be blank",
    "param": "title"
  }
}
```

Editors who attempt an update receive `403` with `code: "forbidden"`.
