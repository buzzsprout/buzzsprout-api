Fan mail
========

Inbound listener messages. You cannot create fan mail or send an SMS reply through this API.

* `GET /api/:podcast_id/fan_mail` — unblocked messages, newest first
* `GET /api/:podcast_id/fan_mail/:id` — also marks the thread as read
* `PATCH /api/:podcast_id/fan_mail/:id` with `{ "published": true }` or `{ "published": false }`
* `POST /api/:podcast_id/fan_mail/:id/read`
* `POST /api/:podcast_id/fan_mail/:id/block` — blocks this sender

```bash
curl -sS -X PATCH "https://www.buzzsprout.com/api/PODCAST_ID/fan_mail/FAN_MAIL_ID" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{ "published": true }'
```

```json
{
  "id": 501,
  "status": "unread",
  "published": false,
  "blocked": false,
  "text": "Love the show",
  "sender_info": "2222",
  "city": "Atlanta",
  "state": "GA",
  "country": "US",
  "created_at": "2026-09-01T15:04:05.000Z",
  "published_at": null,
  "message_type": "TextMessage"
}
```

| Field | Notes |
| --- | --- |
| `status` | `unread` or `read` |
| `published` | `true` when the message is published and not blocked |
| `blocked` | `true` after block |
| `text` | Message body (or voice transcript text) |
| `sender_info` | Short sender label (for example last four digits of a phone number) |
| `city`, `state`, `country` | Optional geo; may be `null` |
| `message_type` | `TextMessage` or `VoiceMessage` |
| `published_at` | Timestamp when published, or `null` |

Show, update, read, and block each return the fan mail object above. Block sets `blocked` to `true` for this sender's messages; there is no unblock endpoint.

Errors
------

Missing fan mail IDs return `404`.
