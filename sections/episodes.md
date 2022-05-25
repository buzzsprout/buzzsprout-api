Episodes
========
Run the following examples in Postman [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/6eae6ad6bc679e8ad112)

Get episodes
------------

* `GET /episodes.json` will return all the episodes for a podcast.

```json
[

  {
    "id":788881,
    "title":"Too small or too big?",
    "audio_url":"https://www.buzzsprout.com/140447/788881-filename.mp3",
    "artwork_url":"https://storage.buzzsprout.com/variants/NABbMDx7JN5bSLzLPXyj67jA/8d66eb17bb7d02ca4856ab443a78f2148cafbb129f58a3c81282007c6fe24ff2",
    "description":"",
    "summary":"",
    "artist":"Muffin Man",
    "tags":"",
    "published_at":"2019-09-12T03:00:00.000-04:00",
    "duration":12362,
    "hq":true,
    "magic_mastering":true,
    "guid":"Buzzsprout788881",
    "inactive_at":null,
    "episode_number":5,
    "season_number":5,
    "explicit":false,
    "private":false,
    "total_plays":150
  },
  {
    "id":788880,
    "title":"Too fast or too slow?",
    "audio_url":"https://www.buzzsprout.com/140447/788880-filename.mp3",
    "artwork_url":"https://storage.buzzsprout.com/variants/NABbMDx7JN5bSLzLPXyj67jA/8d66eb17bb7d02ca4856ab443a78f2148cafbb129f58a3c81282007c6fe24ff2",
    "description":"",
    "summary":"",
    "artist":"Muffin Man",
    "tags":"",
    "published_at":"2019-09-12T03:00:00.000-04:00",
    "duration":23462,
    "hq":false,
    "magic_mastering":false,
    "guid":"Buzzsprout788880",
    "inactive_at":null,
    "episode_number":4,
    "season_number":5,
    "explicit":true,
    "private":false,
    "total_plays":150
  }
]
```

Get episode
------------

* `GET /episodes/788881.json` will return the specifed episode.

```json

{
  "id":788881,
  "title":"Too small or too big?",
  "audio_url":"https://www.buzzsprout.com/140447/788881-filename.mp3",
  "artwork_url":"https://storage.buzzsprout.com/variants/NABbMDx7JN5bSLzLPXyj67jA/8d66eb17bb7d02ca4856ab443a78f2148cafbb129f58a3c81282007c6fe24ff2",
  "description":"",
  "summary":"",
  "artist":"Muffin Man",
  "tags":"",
  "published_at":"2019-09-12T03:00:00.000-04:00",
  "duration":12362,
  "hq":true,
  "guid":"Buzzsprout788881",
  "inactive_at":null,
  "episode_number":5,
  "season_number":5,
  "explicit":false,
  "private":false,
  "total_plays":150
}
```


Create episode
-------------
* `POST /episodes.json` will create a new episode with the included parameters.

```json
{
  "title":"Too many or too few?",
  "description":"",
  "summary":"",
  "artist":"Muffin Man",
  "tags":"",
  "published_at":"2019-09-12T03:00:00.000-04:00",
  "duration":23462,
  "guid":"Buzzsprout788880",
  "inactive_at":null,
  "episode_number":4,
  "season_number":5,
  "explicit":true,
  "private":true,
  "email_user_after_audio_processed": true,
  "audio_url": "https://www.google.com/my_audio_file.mp4",
  "artwork_url": "https://www.google.com/my_artwork_file.jpeg"
}
```

This will return `201 Created`,  with the current JSON representation of the episode if the creation was a success.

**To send an audio file or artwork file**:  Instead of sending the *audio_url* param you can send the actual file as an attachment with the *audio_file* param. Likewise, instead of sending the *artwork_url* param you can send the actual file as an attachment with the *artwork_file* param

**Email user after audio processed**: Sends the user an email once your file has finished processing or if an error occurred during processing. It defaults to true and can be set to false to not send any emails.


Update episode
-------------
* `PUT /episodes/1.json` will update an existing episode with the included parameters.

```json
{
  "title":"The New Republic",
  "private":false
}
```

This will return `200 OK` along with the current JSON representation of the updated episode if the creation was a success.
