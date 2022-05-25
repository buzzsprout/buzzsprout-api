
Podcasts
========

Get podcasts
------------

* `GET /podcasts.json` will return all the episodes for a podcast. 
* This should be called without the podcast_id in the URL. The URL should look like  `/api/podcasts.json`.

```json
[
 {
   "id": 10,
   "title": "Life Empowerment",
   "author": "Motivational Mike",
   "description": "Let me tell you how to live your best life today!",
   "website_address": null,
   "contact_email": "team+mike@buzzsprout.com",
   "keywords": null,
   "explicit": false,
   "main_category": null,
   "sub_category": null,
   "main_category2": null,
   "sub_category2": null,
   "main_category3": null,
   "sub_category3": null,
   "language": "en-us",
   "timezone": "Eastern Time (US \u0026 Canada)",
   "artwork_url": "http://www.buzzsprout.test/images/artworks_large.jpg",
   "background_url": null
 },
 {
   "id": 20,
   "title": "Youth Podcast",
   "author": "Pastor Tim Tom",
   "description": "Our super cool youth podcast",
   "website_address": null,
   "contact_email": "team+tim@buzzsprout.com",
   "keywords": null,
   "explicit": false,
   "main_category": null,
   "sub_category": null,
   "main_category2": null,
   "sub_category2": null,
   "main_category3": null,
   "sub_category3": null,
   "language": "en-us",
   "timezone": "Eastern Time (US \u0026 Canada)",
   "artwork_url": "http://www.buzzsprout.test/images/artworks_large.jpg",
   "background_url": null
 }
]
```
