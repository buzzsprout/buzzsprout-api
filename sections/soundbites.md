Soundbites
==========

Create a short video clip from processed [episode](episodes.md) audio. Artwork is `episode` or `podcast`; this endpoint does not accept a custom image. 

* `POST /api/:podcast_id/episodes/:episode_id/soundbite` returns `202 Accepted`.
* `GET /api/:podcast_id/episodes/:episode_id/soundbite` returns the saved configuration and `creating_video` status, or `404` before a soundbite exists.
* `DELETE /api/:podcast_id/episodes/:episode_id/soundbite` returns `204 No Content`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/soundbite" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "start_time": 10,
    "duration": 15,
    "aspect_ratio": "square",
    "artwork": "episode"
  }'
```

Request defaults are `start_time` `0`, `duration` `30`, `aspect_ratio` `square`, and podcast artwork (anything other than `"episode"` uses podcast artwork). `aspect_ratio` is `square`, `portrait`, or `landscape`. Optional request-only styling fields are `background_color`, `text_color`, `wave_color`, `text_line_1`, and `text_line_2`.

Response:

```json
{
  "id": 88,
  "creating_video": true,
  "start_time": 10,
  "duration": 15,
  "aspect_ratio": "square",
  "artwork": "episode"
}
```

The API does not return a rendered video.  When a GET shows creating_video as false, the clip is ready to view in Buzzsprout.

Errors
------

Creation returns `422` if the episode audio is not processed yet:

```json
{
  "error": {
    "code": "unprocessable",
    "message": "Episode audio is not processed yet",
    "param": null
  }
}
```

GET before a soundbite exists returns `404`.
