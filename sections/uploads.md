Upload audio or video
=====================

Attach audio or video to an [episode](episodes.md). Create the episode first, then follow the three steps below.

This is the preferred way to add or replace media. Completing the upload is what attaches the file to the episode—if you skip that step, the episode has no media. After encoding finishes, you can add [chapters](chapters.md), [contributors](contributors.md), etc.

Workflow
--------

1. **Start** an upload with file metadata.
2. **PUT** the exact file bytes to the returned URL(s). Do not send the Buzzsprout `Authorization` header on those requests.
3. **Complete** the upload so Buzzsprout attaches the file and starts encoding.

After complete, encoding runs asynchronously. Poll `GET /api/:podcast_id/episodes/:id` until `duration` is not `-1`.

Start an upload
---------------

- `POST /api/:podcast_id/episodes/:episode_id/uploads`
- Required JSON fields: `filename`, `byte_size`, and the file's media type
- Prefer `type` for the file media type (aliases `media_type` and `content_type` are also accepted; `type` avoids colliding with the request's own `Content-Type`)
- Optional: `"multipart": true` for large files (see below)
- Returns `201 Created`

```json
{
  "upload": {
    "id": "upload-id",
    "multipart": false,
    "upload_url": "https://uploads.example.com/..."
  }
}
```

PUT the file bytes
------------------

`PUT` exactly `byte_size` bytes to `upload_url`. Match the size you declared when starting the upload; a mismatch fails at complete.

Do not send Buzzsprout authentication on this request. Upload URLs expire after 6 hours.

Complete the upload
-------------------

- `POST /api/:podcast_id/episodes/:episode_id/uploads/:upload_id/complete`
- Returns `200 OK` with the episode

URL-encode `upload_id` if it contains special characters. After a successful complete, poll the episode until encoding finishes (`duration` is no longer `-1`).

Large files (multipart)
-----------------------

For large files, start with `"multipart": true`. The response includes `part_size` and a `parts` array. Upload each byte range to its matching `parts[].url` (parts may run in parallel; failed parts may be retried). Each `PUT` must contain exactly that part's `content_length` bytes. Then call the same complete endpoint.

Multipart is optional up to 5 GiB. For eligible video larger than 5 GiB, multipart is required (`multipart_required` if omitted).

```json
{
  "upload": {
    "id": "upload-id",
    "multipart": true,
    "part_size": 20971520,
    "parts": [
      {
        "part_number": 1,
        "url": "https://uploads.example.com/...",
        "content_length": 20971520
      },
      {
        "part_number": 2,
        "url": "https://uploads.example.com/...",
        "content_length": 7891357
      }
    ]
  }
}
```

Examples
--------

Replace `PODCAST_ID`, `EPISODE_ID`, and `YOUR_API_TOKEN`. Ruby examples use the `buzzsprout_request` helper from the main [README](../README.md).

### Single-file upload

#### Bash example

```bash
# 1. Start
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/uploads" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "filename": "episode.m4a",
    "type": "audio/mp4",
    "byte_size": 12345678
  }'

# 2. PUT file bytes (no Authorization header)
curl -sS -X PUT "UPLOAD_URL" \
  -H "Content-Length: 12345678" \
  --data-binary @episode.m4a

# 3. Complete
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/uploads/UPLOAD_ID/complete" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

#### Ruby example

```ruby
file_path = "episode.m4a"
size = File.size(file_path)
episode_id = episode.fetch("id")

# 1. Start the upload
upload = buzzsprout_request(
  Net::HTTP::Post,
  "/api/#{PODCAST_ID}/episodes/#{episode_id}/uploads",
  body: {
    filename: File.basename(file_path),
    type: "audio/mp4",
    byte_size: size
  }
).fetch("upload")

# 2. PUT the file bytes (no Buzzsprout Authorization header)
uri = URI(upload.fetch("upload_url"))
request = Net::HTTP::Put.new(uri)
request.content_length = size
File.open(file_path, "rb") do |file|
  request.body_stream = file
  response = Net::HTTP.start(uri.host, uri.port, use_ssl: true) { |http| http.request(request) }
  raise "Upload failed: #{response.code}" unless response.is_a?(Net::HTTPSuccess)
end

# 3. Complete — attaches the file to the episode
upload_id = URI.encode_www_form_component(upload.fetch("id"))
buzzsprout_request(
  Net::HTTP::Post,
  "/api/#{PODCAST_ID}/episodes/#{episode_id}/uploads/#{upload_id}/complete"
)
```

### Multipart upload

#### Bash example

```bash
# 1. Start with multipart: true
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/uploads" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "filename": "episode.mp4",
    "type": "video/mp4",
    "byte_size": 6000000000,
    "multipart": true
  }'

# 2. For each part: PUT exactly content_length bytes to that part's url
curl -sS -X PUT "PART_URL" \
  -H "Content-Length: CONTENT_LENGTH" \
  --data-binary @part.bin

# 3. Complete (same as single-file)
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/episodes/EPISODE_ID/uploads/UPLOAD_ID/complete" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Accept: application/json"
```

#### Ruby example

```ruby
file_path = "episode.mp4"
size = File.size(file_path)
episode_id = episode.fetch("id")

# 1. Start with multipart: true
upload = buzzsprout_request(
  Net::HTTP::Post,
  "/api/#{PODCAST_ID}/episodes/#{episode_id}/uploads",
  body: {
    filename: File.basename(file_path),
    type: "video/mp4",
    byte_size: size,
    multipart: true
  }
).fetch("upload")

# 2. PUT each part's exact byte range
File.open(file_path, "rb") do |file|
  upload.fetch("parts").each do |part|
    length = part.fetch("content_length")
    file.seek((part.fetch("part_number") - 1) * upload.fetch("part_size"))
    bytes = file.read(length)

    uri = URI(part.fetch("url"))
    request = Net::HTTP::Put.new(uri)
    request.content_length = length
    request.body = bytes
    response = Net::HTTP.start(uri.host, uri.port, use_ssl: true) { |http| http.request(request) }
    raise "Part upload failed: #{response.code}" unless response.is_a?(Net::HTTPSuccess)
  end
end

# 3. Complete — attaches the file to the episode
upload_id = URI.encode_www_form_component(upload.fetch("id"))
buzzsprout_request(
  Net::HTTP::Post,
  "/api/#{PODCAST_ID}/episodes/#{episode_id}/uploads/#{upload_id}/complete"
)
```

Abort an upload
---------------

- `DELETE /api/:podcast_id/episodes/:episode_id/uploads/:upload_id/abort`
- Returns `204 No Content`

Call abort when canceling or giving up after a failure. Stop any in-flight `PUT` requests first; aborting cannot interrupt a request the client is still sending.

Limits
------

- Audio and other non-video media: 5 GiB maximum
- Video on a video plan, with a video media type: 50 GiB maximum
- Without a video plan **and** a video media type, the file is treated as audio under the 5 GiB cap
- Single-file (non-multipart) upload: 5 GiB maximum; larger eligible videos require multipart

Not accepted filenames
----------------------

These extensions are rejected (`param: "filename"`): archives and documents (`zip`, `rar`, `7z`, `tar`, `gz`, `tgz`, `pdf`, `doc`, `docx`, `pages`, `txt`, `rtf`, `ppt`, `pptx`, `xls`, `xlsx`, `csv`), images (`jpg`, `jpeg`, `png`, `gif`, `webp`, `heic`, `heif`, `svg`, `tif`, `tiff`), project/session files (`aup`, `aup3`, `band`, `logicx`, `ptx`, `ptf`, `sesx`, `cpr`, `flp`, `als`, `rpp`), incomplete downloads (`crdownload`, `part`, `partial`, `download`, `opdownload`, `tmp`), and installers or markup (`dmg`, `iso`, `exe`, `msi`, `pkg`, `app`, `json`, `html`).

Errors
------

Upload errors use:

```json
{
  "error": {
    "code": "invalid",
    "message": "Uploaded file size does not match byte_size",
    "param": "byte_size"
  }
}
```

Common codes and params:

| Situation | Typical `code` / `param` |
| --- | --- |
| Missing or invalid `filename` / type / size | `invalid` with `param` `filename`, `content_type`, or `byte_size` |
| File larger than 5 GiB without multipart (when required) | `multipart_required` (`param`: `multipart`) |
| File over the plan/size limit | `invalid` (`param`: `byte_size`) |
| Upload id past 24 hours | `expired` |
| Complete when bytes do not match `byte_size` | `invalid` (`param`: `byte_size`) |
| Could not process the uploaded object | `upload_failed` |
