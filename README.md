Buzzsprout API
==============

The Buzzsprout API is a REST API for managing podcasts and episodes. Requests and responses use JSON unless otherwise documented.

Base URL
--------

All requests use HTTPS. Podcast-scoped endpoints start with:

```text
https://www.buzzsprout.com/api/:podcast_id
```

Use the numeric podcast ID shown in Buzzsprout. Extensionless paths are canonical; an optional `.json` suffix is also accepted.

Authentication
--------------

Send the API token in the `Authorization` header:

```text
Authorization: Token token=YOUR_API_TOKEN
```

Find the token under **Profile → API** in Buzzsprout.

Use an identifiable `User-Agent` on every request. Generic library defaults may be blocked.

For automated clients
---------------------

- Send an identifiable `User-Agent` (not a generic library default).
- Always **complete** an upload after putting the file bytes; otherwise the episode has no media.

Publish an episode
------------------

A typical publish flow:

1. `GET /api/podcasts` — note the numeric podcast `id`.
2. `POST /api/:podcast_id/episodes` with at least a `title` and `"private": true` (private is the default when `published_at` is omitted).
3. Upload audio or video:
   - [Start the upload](sections/uploads.md)
   - `PUT` the file bytes to the returned URL(s)
   - **Complete** the upload (required before the episode has media)
4. `GET /api/:podcast_id/episodes/:id` when encoding finishes (`duration` is positive integer).
5. Optional: add [chapters](sections/chapters.md), [contributors](sections/contributors.md), artwork
6. `PATCH /api/:podcast_id/episodes/:id` with `"private": false` (and `published_at` to a future date if scheduling).

See [Episodes](sections/episodes.md) and [Upload audio or video](sections/uploads.md) for field details and examples.

Resources
---------

- [Podcasts](sections/podcasts.md) — list, show, and update a podcast
- [Episodes](sections/episodes.md) — create, inspect, update, schedule, and unpublish episodes
- [Upload audio or video](sections/uploads.md) — attach media to an episode
- [Stats](sections/stats.md) — download totals and details for a single date
- [Team](sections/team.md) — manage people with Buzzsprout login access
- [Contributors](sections/contributors.md) — manage on-show credits and assign them to episodes
- [Brand affiliations](sections/brand_affiliations.md) — manage sponsor or affiliate links and assign them to episodes
- [Chapters](sections/chapters.md) — replace an episode's chapter list
- [Transcripts](sections/transcripts.md) — manage public transcripts and update high-fidelity transcripts
- [Listings](sections/listings.md) — inspect directory status and submit eligible directories
- [Podrolls](sections/podrolls.md) — manage recommended podcasts in the RSS feed
- [Feed verifications](sections/feed_verifications.md) — manage `<podcast:txt>` values in the RSS feed
- [Fan mail](sections/fan_mail.md) — list, publish, read, and block listener messages
- [Insertion points](sections/insertion_points.md) — manage mid-roll timestamps
- [Soundbites](sections/soundbites.md) — create short video clips from episode audio

Team members can log into Buzzsprout. Contributors are public names and roles that appear on the show; they are separate resources.

Curl example
------------

```bash
curl -sS "https://www.buzzsprout.com/api/podcasts" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

Ruby example
------------

This example uses only Ruby's standard library. Keep the token in an environment variable rather than in source code.

```ruby
require "json"
require "net/http"

API_TOKEN = ENV.fetch("BUZZSPROUT_API_TOKEN")
PODCAST_ID = ENV.fetch("BUZZSPROUT_PODCAST_ID")

def buzzsprout_request(request_class, path, body: nil)
  uri = URI("https://www.buzzsprout.com#{path}")
  request = request_class.new(uri)
  request["Accept"] = "application/json"
  request["Authorization"] = "Token token=#{API_TOKEN}"
  request["User-Agent"] = "ExamplePodcastClient/1.0"

  if body
    request["Content-Type"] = "application/json"
    request.body = JSON.generate(body)
  end

  response = Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
    http.request(request)
  end
  raise "Buzzsprout API error #{response.code}: #{response.body}" unless response.is_a?(Net::HTTPSuccess)

  response.body.empty? ? nil : JSON.parse(response.body)
end

podcasts = buzzsprout_request(Net::HTTP::Get, "/api/podcasts")

episode = buzzsprout_request(
  Net::HTTP::Post,
  "/api/#{PODCAST_ID}/episodes",
  body: { title: "My new episode", private: true }
)
```

Request and response format
---------------------------

For JSON request bodies, send `Content-Type: application/json; charset=utf-8`. File and transcript endpoints describe their multipart or raw-upload requirements separately.

Successful creates return `201 Created`; asynchronous operations may return `202 Accepted`; deletes generally return `204 No Content`.

Most newer endpoints return errors in this shape:

```json
{
  "error": {
    "code": "invalid",
    "message": "Title can't be blank",
    "param": "title"
  }
}
```

The original episode create and update endpoints retain their legacy error format: validation errors are field hashes, while invalid credentials and inaccessible podcasts are plain-text bodies.

Rate limits and retries
-----------------------

Requests are limited to 60 per minute per authorization value. A request over the limit returns `429 Too Many Requests`. Retry `429` and transient `5xx` responses with exponential backoff; do not retry validation or permission errors unchanged.

HTTP caching
------------

When a response includes `ETag` or `Last-Modified`, store it and send it back as `If-None-Match` or `If-Modified-Since`. An unchanged resource returns `304 Not Modified` without a response body.

Support
-------

Use [GitHub issues](https://github.com/Buzzsprout/buzzsprout-api/issues) for API questions, bugs, and feature requests.
