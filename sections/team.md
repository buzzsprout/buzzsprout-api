Team
====

Team members are people with Buzzsprout login access to this podcast. This is not the same as [contributors](contributors.md), which are on-show credits (Host, Guest, and so on).

All team endpoints require **admin or owner**. Editors receive `403` with `code: "forbidden"`.

* Roles that can be assigned: `editor` or `admin`. `owner` is rejected.
* You cannot change or remove the owner, or yourself.

List team members
-----------------

* `GET /api/:podcast_id/team`

```json
[
  {
    "id": 1,
    "user_id": 12,
    "name": "Admin User",
    "email": "owner@example.com",
    "role": "owner"
  }
]
```

`id` is the team entry ID used in update and delete URLs. `user_id` is the Buzzsprout user ID and is not used in these paths.

Invite a team member
--------------------

* `POST /api/:podcast_id/team`

```bash
curl -sS -X POST "https://www.buzzsprout.com/api/PODCAST_ID/team" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{
    "name": "Pat Editor",
    "email": "pat@example.com",
    "role": "editor"
  }'
```

New accounts receive an invitation email with a temporary password. Existing Buzzsprout users are added to the podcast and emailed a notification. Inviting an email that is already on the podcast returns `422` with `param: "email"`. On create, a missing `role` defaults to `editor`.

```json
{
  "id": 42,
  "user_id": 99,
  "name": "Pat Editor",
  "email": "pat@example.com",
  "role": "editor"
}
```

Update or remove
----------------

* `PATCH /api/:podcast_id/team/:id` with `{ "role": "admin" }`. The `:id` is the team entry's `id`, not `user_id`. Omitting `role` leaves the current role unchanged.
* `DELETE /api/:podcast_id/team/:id` — returns `204 No Content`.

```bash
curl -sS -X PATCH "https://www.buzzsprout.com/api/PODCAST_ID/team/42" \
  -H "Authorization: Token token=YOUR_API_TOKEN" \
  -H "User-Agent: ExamplePodcastClient/1.0" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "Accept: application/json" \
  -d '{ "role": "admin" }'
```

```json
{
  "id": 42,
  "user_id": 99,
  "name": "Pat Editor",
  "email": "pat@example.com",
  "role": "admin"
}
```

Errors
------

```json
{
  "error": {
    "code": "invalid",
    "message": "role must be editor or admin",
    "param": "role"
  }
}
```

Attempts to change or remove the owner (or yourself) return `403` with `code: "forbidden"` and message `Cannot change this team member`.
