# Twitch Leaderboard API Documentation

A REST API that returns the top bits cheerers, sub gifters, and most-viewed clips for any Twitch channel.

## Table of Contents

- [Base URL](#base-url)
- [Endpoints](#endpoints)
  - [GET /twitch/leaderboard](#get-twitchleaderboard)
  - [Query Parameters](#query-parameters)
  - [Example Requests](#example-requests)
- [Success Response](#success-response)
  - [Response Fields](#response-fields)
  - [Bits Entry Fields](#bits-entry-fields)
  - [Sub Gifts Entry Fields](#sub-gifts-entry-fields)
  - [Clips Entry Fields](#clips-entry-fields)
- [Error Responses](#error-responses)
  - [400 Bad Request](#400-bad-request)
  - [404 Not Found](#404-not-found)
  - [502 Bad Gateway](#502-bad-gateway)
  - [500 Internal Server Error](#500-internal-server-error)
- [Notes](#notes)

---

## Base URL

```
https://api.sinist3r.lol
```

---

## Endpoints

### GET `/twitch/leaderboard`

Returns the top 10 leaderboard entries across three categories for a given Twitch channel.

#### Query Parameters

| Parameter   | Type   | Required | Description                             |
|-------------|--------|----------|-----------------------------------------|
| `channelId` | string | Yes      | A Twitch channel username or numeric ID |

#### Example Requests

```
GET /twitch/leaderboard?channelId=somechannel
GET /twitch/leaderboard?channelId=123456789
```

---

## Success Response

**Status:** `200 OK`

```json
{
  "channel_id": "123456789",
  "channel_login": "somechannel",
  "channel_display_name": "SomeChannel",
  "period": "ALLTIME",
  "seconds_remaining_in_period": null,
  "leaderboards": {
    "bits": {
      "count": 1,
      "entries": [
        {
          "rank": 1,
          "username": "ExampleUser1",
          "login": "exampleuser1",
          "bits_cheered": 84289
        }
      ]
    },
    "sub_gifts": {
      "count": 1,
      "entries": [
        {
          "rank": 1,
          "username": "ExampleUser2",
          "login": "exampleuser2",
          "subs_gifted": 1305
        }
      ]
    },
    "clips": {
      "count": 1,
      "entries": [
        {
          "rank": 1,
          "title": "Example Clip Title",
          "url": "https://www.twitch.tv/somechannel/clip/ExampleClipSlug",
          "views": 4690,
          "thumbnail": "https://static-cdn.jtvnw.net/twitch-clips/example-preview.jpg"
        }
      ]
    }
  }
}
```

### Response Fields

#### Top Level

| Field                         | Type            | Description                                                         |
|-------------------------------|-----------------|---------------------------------------------------------------------|
| `channel_id`                  | string          | The channel's numeric Twitch ID                                     |
| `channel_login`               | string          | The channel's lowercase username                                    |
| `channel_display_name`        | string          | The channel's display name as shown on Twitch                       |
| `period`                      | string          | The leaderboard period: `MONTH`, `WEEK`, or `ALLTIME`               |
| `seconds_remaining_in_period` | integer or null | Seconds until the leaderboard resets. `null` for `ALLTIME` channels |
| `leaderboards`                | object          | Contains the three leaderboard categories                           |

#### Leaderboard Categories

Each category (`bits`, `sub_gifts`, `clips`) contains:

| Field     | Type    | Description                         |
|-----------|---------|-------------------------------------|
| `count`   | integer | Number of entries returned          |
| `entries` | array   | Ordered list of leaderboard entries |

> [!NOTE]
> `count` may be less than 10 if the channel does not have enough activity to fill the leaderboard.

#### Bits Entry Fields

| Field          | Type    | Description                      |
|----------------|---------|----------------------------------|
| `rank`         | integer | Position on the leaderboard      |
| `username`     | string  | Display name of the user         |
| `login`        | string  | Lowercase username of the user   |
| `bits_cheered` | integer | Total bits cheered in the period |

#### Sub Gifts Entry Fields

| Field         | Type    | Description                      |
|---------------|---------|----------------------------------|
| `rank`        | integer | Position on the leaderboard      |
| `username`    | string  | Display name of the user         |
| `login`       | string  | Lowercase username of the user   |
| `subs_gifted` | integer | Total subs gifted in the period  |

#### Clips Entry Fields

| Field       | Type           | Description                       |
|-------------|----------------|-----------------------------------|
| `rank`      | integer        | Position on the leaderboard       |
| `title`     | string         | Title of the clip                 |
| `url`       | string         | Direct URL to the clip on Twitch  |
| `views`     | integer        | Total view count of the clip      |
| `thumbnail` | string or null | URL to the clip's thumbnail image |

---

## Error Responses

All errors return a JSON object with a single `error` field describing what went wrong.

```json
{
  "error": "Error message here."
}
```

### `400 Bad Request`

Returned when a required parameter is missing.

| Cause                    | Error Message                                 |
|--------------------------|-----------------------------------------------|
| `channelId` not provided | `Missing required query parameter: channelId` |

**Example:**
```
GET /twitch/leaderboard
```
```json
{
  "error": "Missing required query parameter: channelId"
}
```

---

### `404 Not Found`

Returned when the channel does not exist or has no leaderboard data.

| Cause                              | Error Message                                                                                           |
|------------------------------------|---------------------------------------------------------------------------------------------------------|
| Username not found on Twitch       | `Channel 'username' not found on Twitch.`                                                               |
| Numeric ID not found on Twitch     | `Channel ID '123456' not found on Twitch.`                                                              |
| Channel exists but has no activity | `Twitch returned no leaderboard data for channel '123456'. The channel may have no leaderboard activity yet.` |
| Could not resolve ID from username | `Could not resolve channel ID for 'username'.`                                                          |

**Example:**
```
GET /twitch/leaderboard?channelId=thischanneldoesnotexist
```
```json
{
  "error": "Channel 'thischanneldoesnotexist' not found on Twitch."
}
```

---

### `502 Bad Gateway`

Returned when the Twitch GQL API returns an unexpected response.

| Cause                             | Error Message                                              |
|-----------------------------------|------------------------------------------------------------|
| Twitch API returned an HTTP error | `Twitch GQL request failed: <http error details>`          |
| Twitch response missing a field   | `Unexpected response structure -- missing key: <key name>` |
| Twitch returned a GQL error       | `Twitch GQL error: <error details>`                        |

> [!WARNING]
> `502` errors may indicate that Twitch has changed their API. If these errors persist, dm `sin636` on Discord.

---

### `500 Internal Server Error`

Returned when an unexpected error occurs on the server.

```json
{
  "error": "An unexpected error occurred."
}
```

---

## Notes

> [!NOTE]
> Twitch determines whether a channel uses `MONTH`, `WEEK`, or `ALLTIME` leaderboards. This cannot be controlled via the API.

> [!NOTE]
> Twitch caps leaderboard requests at 10 entries for bits and sub gifts, and 5 entries for clips.

> [!TIP]
> Both a Twitch username (e.g. `somechannel`) and a numeric channel ID (e.g. `123456789`) are accepted interchangeably. Twitch usernames are also case-insensitive, so `SomeChannel`, `somechannel`, and `SOMECHANNEL` all resolve to the same channel.
