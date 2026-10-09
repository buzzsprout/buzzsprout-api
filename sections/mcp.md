# MCP for supported AI assistants

Buzzsprout's MCP server provides authenticated, read-only access to podcasts, episodes, and download statistics. It is intended for supported AI-assistant integrations and requires a registered Buzzsprout OAuth application. It is not currently a self-service replacement for the public REST API.

## At a Glance

| Item | Details |
| --- | --- |
| MCP endpoint | `https://www.buzzsprout.com/mcp` |
| Transport | Streamable HTTP over HTTPS |
| Authentication | OAuth authorization-code flow with PKCE (`S256`), using a registered Buzzsprout OAuth application |
| OAuth resource identifier | `https://www.buzzsprout.com/mcp` |
| Required scope | `api` |
| Capabilities | Four read-only tools for listing accessible podcasts and episodes and retrieving aggregate download totals; the `buzzsprout://guidance` resource |
| Authorization | The connected podcaster's current roles and podcast permissions apply to every request |

## Authentication and connection setup

### Prerequisites

Use a compatible remote MCP client configured with a registered Buzzsprout OAuth application. OAuth authentication requires the `api` scope. Tools and resources require authenticated MCP access.

The `api` scope is shared with OAuth access to the REST API, which may authorize write operations. **The MCP tools themselves are read-only.** MCP access tokens are issued for the MCP endpoint and are not accepted by the REST API. REST-only OAuth tokens are not accepted at `/mcp`.

### Connection procedure

1. Add `https://www.buzzsprout.com/mcp` as a remote MCP server in the client configured with the registered application.
2. Complete Buzzsprout OAuth sign-in and approval. The client uses the authorization-code flow with PKCE (`S256`) and `resource=https://www.buzzsprout.com/mcp`.
3. The client authenticates MCP requests with an OAuth bearer access token, not the API token from the podcaster's profile.

Existing Buzzsprout roles and podcast permissions apply to every request. See [Errors and limits](#errors-and-limits) for authentication failures and OAuth discovery.

## Shared guidance

Read `buzzsprout://guidance` with MCP `resources/read` for shared podcaster terminology, download terminology, inclusive date ranges, episode visibility and state caveats, and zero-download omission behavior. The resource is served as `text/plain` and requires the same authenticated access as the tools.

## Tools

### Tools summary

All listed inputs are required unless the tool takes no inputs.

| Tool | Purpose | Required inputs |
| --- | --- | --- |
| `list_podcasts` | List podcasts the connected podcaster can access | None |
| `list_episodes` | List a podcast's episodes | `podcast_id` (integer) |
| `get_podcast_downloads` | Get total downloads across all episodes of a podcast | `podcast_id` (integer), `start_date`, `end_date` (strings) |
| `get_episode_downloads` | Get download totals for selected episodes of one podcast | `podcast_id` (integer), `episode_ids` (array of one or more integers), `start_date`, `end_date` (strings) |

### Permissions and read-only behavior

All four tools use the connected podcaster's current permissions. Podcast-scoped tools require access to that podcast; requested episode IDs must belong to it.

All are read-only with no modification side effects: none creates, edits, renames, publishes, or deletes episodes, changes billing, or changes plans.

### Privacy and accessible information

None of the tools returns listener identities, listener email addresses, or payment data. Podcast records can include the podcast's `contact_email`; episode records can include non-public episode information.

### Response format

Successful tool results contain JSON encoded in `result.content[0].text`. The Returns entries below describe that decoded JSON, not the MCP response envelope.

### Download and date semantics

Both download tools return aggregate downloads, not unique listeners or retention. Dates must be valid `YYYY-MM-DD` strings; ranges include both endpoints and use the podcast's local calendar dates, with `start_date` on or before `end_date`.

### `list_podcasts`

Lists the podcasts the connected podcaster can access.

**Inputs:** None.

**Returns:** Array of `PodcastSerializer` records; see [podcast fields](podcasts.md#list-and-show).

**Empty result:** `[]` when no podcasts are accessible.

**Errors:** No tool-specific not-found error; shared errors below apply.

### `list_episodes`

Lists a podcast's episodes.

**Inputs:** `podcast_id` (integer).

**Returns:** An object with `count` and `episodes`, where `episodes` is an array of `EpisodeSerializer` records, ordered by `published_at` descending, then ID descending. `count` is exactly the length of `episodes`.

**Empty result:** When the podcast has no episodes:

```json
{
  "count": 0,
  "episodes": []
}
```

**Errors:** `Podcast not found` for an inaccessible or unknown podcast.

`list_episodes` may include private, scheduled, inactive, premium, or still-processing episodes. `published_at` is a stored release or scheduled timestamp, not proof that an episode is currently public or playable.

### `get_podcast_downloads`

Returns total downloads across all episodes of a podcast, including audio and video in any state.

**Inputs:** `podcast_id` (integer), `start_date`, `end_date` (strings).

**Returns:**

```json
{
  "downloads": 456
}
```

**Empty result:** `{"downloads":0}` when no downloads match.

**Errors:** `Podcast not found`, or the invalid-date-range error below.

### `get_episode_downloads`

Returns download totals for selected episodes of one podcast.

**Inputs:** `podcast_id` (integer), `episode_ids` (array of one or more integers), `start_date`, `end_date` (strings).

**Returns:** An array of episode download totals; repeated IDs are counted once. For example:

```json
[
  {
    "episode_id": 123,
    "downloads": 45
  }
]
```

**Empty result:** `[]` when all requested valid episodes have zero downloads in the range.

**Errors:** `Podcast not found`, `Episodes not found`, or the invalid-date-range error below. Any inaccessible, unknown, or wrong-podcast episode causes the whole call to fail.

`get_episode_downloads` omits episodes with zero downloads. A valid requested episode missing from a **successful** response had 0 downloads for that range. Empty lists and zero totals are successful results, not errors.

## Errors and limits

- Missing or incorrectly typed required tool inputs return a tool error. Tool errors set `result.isError` to `true` and include a text message.
- Inaccessible or unknown podcasts return `Podcast not found`; invalid requested episode IDs return `Episodes not found`. These messages do not distinguish lack of access from a missing record.
- Invalid or reversed date ranges return: `start_date and end_date must be valid dates in YYYY-MM-DD format, with start_date on or before end_date`.
- Unexpected server failures can return a JSON-RPC error with the message `Internal error`.
- Unauthenticated requests, including invalid, expired, revoked, or incorrectly targeted tokens, return HTTP `401` with a `WWW-Authenticate` header pointing to `https://www.buzzsprout.com/.well-known/oauth-protected-resource/mcp` for OAuth discovery.
- MCP requests are limited to 60 per minute per authorization value. Requests without an authorization value are limited by IP address. Exceeding the limit returns HTTP `429 Too Many Requests`.
