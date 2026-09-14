Transcripts
===========

[Episodes](episodes.md) can have two transcript resources:

1. **High-fidelity** — provided by Buzzsprout (generated automatically after an episode is processed). Segment (word)-level text with timestamps you can read and edit.
2. **Custom public** — a transcript file you upload to replace what is shown publicly.

They are separate. Uploading a custom transcript does not start Buzzsprout transcription, and deleting a custom transcript does not remove the high-fidelity one. Any podcast member can call these endpoints.

High-fidelity transcripts
-------------------------

Buzzsprout provides the high-fidelity transcript. The API can fetch and replace its segments, but it cannot create or delete one.

### Get

* `GET /api/:podcast_id/episodes/:episode_id/high_fidelity_transcript`
* Returns `404` if the episode does not have a high-fidelity transcript.

The response is the JSON segment array:

```json
[
  {
    "start_time": 0.547,
    "end_time": 0.627,
    "body": "Hi,",
    "speaker": "Speaker 01",
    "type": "text"
  }
]
```

### Replace

* `PUT /api/:podcast_id/episodes/:episode_id/high_fidelity_transcript`
* `PATCH /api/:podcast_id/episodes/:episode_id/high_fidelity_transcript`
* The request body is the raw segment array shown above, without a containing object.
* Returns `200` and the normalized segment array after a successful update.
* Returns `404` if the episode does not have a high-fidelity transcript.

```bash
curl -sS -X PUT "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/high_fidelity_transcript" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '[
    {"start_time": 0.0, "end_time": 1.2, "body": "Hello", "speaker": "Host", "type": "text"}
  ]'
```

Each segment supports these fields:

| Field | Required | Format |
| --- | --- | --- |
| `start_time` | Yes | A non-negative number of seconds. |
| `body` | Yes | A non-empty string. |
| `end_time` | No | A non-negative number of seconds that is not before `start_time`. |
| `speaker` | No | A string. |
| `type` | No | `text` or `punc`; defaults to `text`. |

The array must contain at least one segment. Unsupported fields, missing required fields, and invalid field values return `422 Unprocessable Entity` without changing the existing transcript.

```json
{
  "error": {
    "code": "invalid",
    "message": "File segment 1 must include body",
    "param": "transcript"
  }
}
```

Custom public transcripts
-------------------------

Upload your own transcript file when you want to replace the public transcript. Use this path for create/replace/delete of that custom file.

`GET /transcript` returns whatever is public right now: the custom transcript if one exists, otherwise the published high-fidelity transcript. For the editable segment array, use the high-fidelity endpoints above.

### Get

* `GET /api/:podcast_id/episodes/:episode_id/transcript`
* Returns `404` when neither a custom nor a published high-fidelity transcript is available.

```json
{
  "id": 99,
  "status": "transcribed",
  "published": true,
  "download_url": "https://www.buzzsprout.com/transcripts/..."
}
```

### Create or replace

* `POST /api/:podcast_id/episodes/:episode_id/transcript`
* Multipart form fields:
  * `transcript_file` — the transcript file
  * `transcript_format` — accepted formats: `srt` or `json`
* Returns `201 Created` and the JSON object above.
* Missing `transcript_file` or `transcript_format` returns `400 Bad Request`.
* A file that cannot be parsed for the given format returns `422` with `code: "invalid"` and `param: "transcript_file"`.

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/transcript" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json" \
  -F "transcript_file=@episode.srt" \
  -F "transcript_format=srt"
```

### Delete

* `DELETE /api/:podcast_id/episodes/:episode_id/transcript`
* Removes the custom public transcript only, not a high-fidelity transcript.
* Returns `204 No Content`, or `404` when no custom transcript exists.
