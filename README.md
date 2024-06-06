Buzzsprout API
====================

This is version one of the Buzzsprout API.  This API has been designed around RESTful concepts with JSON for serialization.

URL
---
All requests are SSL only and the API URL is `https://www.buzzsprout.com/api/9999` where `9999` is the podcast identifier. If we change the API in backward-incompatible ways, we'll add the version marker and maintain stable support for old URLs.


Authentication
--------------

Buzzsprout uses a simple token-based HTTP Authentication scheme. For clients to authenticate, the method "Token" and the token key should be included in the Authorization HTTP header. The key should be prefixed by the string literal "token=" with no whitespace. For example:

```shell
Authorization: Token token=ApV99yzvwApV99yzvwApV99yzvwApV99yzvw
```


Additionaly you can pass the api token as a paramater in the URL
```
?api_token=ApV99yzvwApV99yzvwApV99yzvwApV99yzvw
```

To retrieve your token check out the my account section in your Buzzsprout admin at [buzzsprout.com](https://www.buzzsprout.com "www.buzzsprout.com").

User Agent
----------
Ensure you set the user agent header. Failing to do so may result in a blocked request since many bots and spammers use the default user agent set by the libraries used with making requests.

Body Format
----------
All data is serialized with JSON and UTF-8 encoded.  This means that you have to send `Content-Type: application/json; charset=utf-8` when you're POSTing or PUTing data into Buzzsprout. All API URLs end in .json to indicate that they accept and return JSON.

You'll receive a `415 Unsupported Media Type` response code if you attempt to use a different URL suffix or leave out the `Content-Type` header.

Sample GET
-------
To make a request for all the Episodes on your account, you'd append the episodes index path to the base url to form something like https://www.buzzsprout.com/api/9999/episodes.json. In cURL, that looks like:

```shell
curl -H "Authorization: Token token=ApV99yzvwApV99yzvwApV99yzvwApV99yzvw" \
  https://www.buzzsprout.com/api/9999/episodes.json
```
Sample POST
------------
To create something, it's the same deal except you also have to include the `Content-Type` header and the JSON data:

```shell
curl -H "Authorization: Token token=ApV99yzvwApV99yzvwApV99yzvwApV99yzvw" \
     -H 'Content-Type: application/json' \
     -d '{ "title": "My new episode!" }' \
  https://www.buzzsprout.com/api/9999/episodes.json
```

Use HTTP caching
----------------

You must make use of the HTTP freshness headers to lessen the load on our servers (and increase the speed of your application!). Most requests we return will include an `ETag` or `Last-Modified` header. When you first request a resource, store this value, and then submit them back to us on subsequent requests as `If-None-Match` and `If-Modified-Since`. If the resource hasn't changed, you'll see a `304 Not Modified` response, which saves you the time and bandwidth of sending something you already have.


Handling errors
---------------

If Buzzsprout is having trouble, you might see a 5xx error. `500` means that the app is entirely down, but you might also see `502 Bad Gateway`, `503 Service Unavailable`, or `504 Gateway Timeout`. It's your responsibility in all of these cases to retry your request later.


Thanks!
----------------------

Thank you for checking out Buzzsprout's API and please don't hesitate to reach out and let us know how you are using the API.  Feel free to use GitHub issues to post any issues or feature requests.

Please tell us how we can make the API better. If you have a specific feature request or if you found a bug, please use GitHub issues. Fork these docs and send a pull request with improvements.

To talk with us and other developers about the API, [post a question on StackOverflow](http://stackoverflow.com/questions/ask) tagged `Buzzsprout` or [check out our support section](http://www.buzzsprout.com/help).
