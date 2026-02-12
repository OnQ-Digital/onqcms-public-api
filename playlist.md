# Playlist

Playlists define how content is to be played on any player assigned to the playlist. The onQ CMS has two playlist systems: **legacy playlists** (standard CMS playlists) and **campaign sub-playlists** (campaign divisions). The Playlist Registry assigns each playlist a globally unique ID so that both types can be referenced through a single API.

- [Fetch Playlist](#fetch-playlist)
- [Bulk Playlist](#create-a-bulk-playlist)

[Home](README.md)

## Playlist Object

The fetch endpoint returns a unified playlist object regardless of the underlying playlist type.

### Playlist Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| id | integer | The globally unique playlist registry ID. Use this to reference the playlist in API calls. |
| playlist_type | string | The type of the playlist. Either "legacy" or "campaign_sub". |
| source_id | integer | The ID of the playlist in its source table (playlist_id or campaign_sub_id). |
| name | string | The name of the playlist. |
| campaign_id | integer/null | The parent campaign ID. Only set for campaign_sub playlists, null for legacy. |
| parent_campaign_name | string/null | The name of the parent campaign. Only set for campaign_sub playlists, null for legacy. |
| dateCreated | string | The creation timestamp of the playlist. |
| dateModified | string | The last modified timestamp of the playlist. |
| mediaAssets | array (media asset object) | The media assets in the playlist. See Media Asset Parameters below. |
| players | array (integer) | The media player IDs assigned to this playlist. |
| schedule | object | The schedule/validity configuration. Shape varies by playlist type - see Schedule Parameters below. |

### Media Asset Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| file_content_id | integer | The unique ID of the asset on the CMS. |
| file_title | string | The display name of the asset. |
| file_name | string | The file name of the asset on the storage server. |
| file_ext | string | The file extension (eg "mp4", "jpg", "png"). |
| file_width | integer | The pixel width of the asset. |
| file_height | integer | The pixel height of the asset. |
| file_thumb_name | string | The file name of the thumbnail image. |
| file_uid | string | The unique identifier of the asset. |
| download_url | string | The full URL to download the asset. |

### Schedule Parameters (Legacy)

Legacy playlists use the following schedule shape:

| Parameter | Type | Description |
|-----------|------|-------------|
| days | array (string) | The days of the week the playlist is active. eg ["Mon", "Tue", "Wed"]. |
| from_date | string | The start date in ISO 8601 format. |
| from_time | string | The start time in HH:MM format. |
| to_date | string | The end date in ISO 8601 format. |
| to_time | string | The end time in HH:MM format. |
| date_time_independent | boolean | If true, date and time conditions are evaluated separately. |

### Schedule Parameters (Campaign Sub)

Campaign sub-playlists use the following schedule shape:

| Parameter | Type | Description |
|-----------|------|-------------|
| start_date_time | string | The start date and time (yyyy-MM-dd HH:mm:ss). |
| end_date_time | string | The end date and time (yyyy-MM-dd HH:mm:ss). |
| schedules | array | The day-of-week and time-of-day schedule rules. See Schedules Object below. |
| content_validity | array | Per-asset validity rules. See Content Validity Object below. |

#### Schedules Object

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| dow | array (string) | True | Days of the week: "mon", "tue", "wed", "thu", "fri", "sat", "sun" |
| after | datetime | False | Validity from date time |
| before | datetime | False | Validity until date time |
| daily_times | array | False | Daily play time windows for this schedule |

#### Daily Times Object

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| after | time | False | Validity from time |
| before | time | False | Validity until time |

#### Content Validity Object

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | integer | True | Content ID |
| days | array (string) | True | Days of the week: "mon", "tue", "wed", "thu", "fri", "sat", "sun" |
| valid_from_date | datetime | False | Validity from date |
| valid_from_time | datetime | False | Validity from time |
| valid_until_date | datetime | False | Validity until date |
| valid_until_time | datetime | False | Validity until time |
| tag_must_have | array | False | Tags the player must have |
| tag_any_these | array | False | Player needs any one of these tags |
| tag_none_these | array | False | Player must not have any of these tags |

### Example Playlist JSON Object (Legacy)

```json
{
    "id": 12345,
    "playlist_type": "legacy",
    "source_id": 42,
    "name": "Lobby Playlist",
    "campaign_id": null,
    "parent_campaign_name": null,
    "dateCreated": "2024-06-15T10:30:00+10:00",
    "dateModified": "2024-12-01T14:22:00+10:00",
    "mediaAssets": [
        {
            "file_content_id": 101,
            "file_title": "Welcome Video",
            "file_name": "3-237-1651629762-74e0412.mp4",
            "file_ext": "mp4",
            "file_width": 1920,
            "file_height": 1080,
            "file_thumb_name": "3-237-1651629762-74e0412-thumb.jpg",
            "file_uid": "abc123",
            "download_url": "https://xxxx.blob.core.windows.net/.../3-237-1651629762-74e0412.mp4"
        }
    ],
    "players": [10, 15, 22],
    "schedule": {
        "days": ["Mon", "Tue", "Wed", "Thu", "Fri"],
        "from_date": "2024-06-15T00:00:00+10:00",
        "from_time": "08:00",
        "to_date": "2025-06-15T00:00:00+10:00",
        "to_time": "18:00",
        "date_time_independent": false
    }
}
```

### Example Playlist JSON Object (Campaign Sub)

```json
{
    "id": 67890,
    "playlist_type": "campaign_sub",
    "source_id": 5,
    "name": "Summer Promo - Division 1",
    "campaign_id": 142,
    "parent_campaign_name": "Summer Promo",
    "dateCreated": "2024-12-01 00:00:00",
    "dateModified": "2025-01-15 14:30:00",
    "mediaAssets": [
        {
            "file_content_id": 33361,
            "file_title": "CR_May22_MumDay_1920x1080px_10.jpg",
            "file_name": "3-237-1651629762-74e0412.jpg",
            "file_ext": "jpg",
            "file_width": 1920,
            "file_height": 1080,
            "file_thumb_name": "3-237-1651629762-74e0412-thumb.jpg",
            "file_uid": null,
            "download_url": "https://xxxx.blob.core.windows.net/.../3-237-1651629762-74e0412.jpg"
        },
        {
            "file_content_id": 33362,
            "file_title": "CR_May22_MumDay_1920x1080px_11.jpg",
            "file_name": "3-237-1651629763-304480f.jpg",
            "file_ext": "jpg",
            "file_width": 1920,
            "file_height": 1080,
            "file_thumb_name": "3-237-1651629763-304480f-thumb.jpg",
            "file_uid": null,
            "download_url": "https://xxxx.blob.core.windows.net/.../3-237-1651629763-304480f.jpg"
        }
    ],
    "players": [1144, 1153],
    "schedule": {
        "start_date_time": "2024-12-01 00:00:00",
        "end_date_time": "2025-02-28 23:59:59",
        "schedules": [
            {
                "dow": ["mon", "tue", "wed", "thu", "fri"],
                "after": "2024-12-01 09:00:00",
                "before": "2025-02-28 18:00:00",
                "daily_times": [
                    {"after": "09:00:00", "before": "12:00:00"},
                    {"after": "13:00:00", "before": "18:00:00"}
                ]
            }
        ],
        "content_validity": [
            {
                "id": 33361,
                "days": ["mon", "wed", "fri"],
                "valid_from_date": "2024-12-01",
                "valid_from_time": "09:00:00",
                "valid_until_date": "2025-02-28",
                "valid_until_time": "18:00:00",
                "tag_must_have": [],
                "tag_any_these": [],
                "tag_none_these": []
            }
        ]
    }
}
```

[Top](#playlist)

## Fetch Playlist

### Endpoint: playlist/fetch
### Response: JSON playlist object or JSON object with player_id and playlists array

Fetches playlist data using one of two query modes. Provide **one** of the following: a `playlist_id` to fetch a single playlist, or a `player_id` to fetch all playlists assigned to that player.

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| playlist_id | integer | If no player_id | The playlist registry ID. Returns a single playlist object. |
| player_id | integer | If no playlist_id | The media player ID. Returns all playlists assigned to this player. |
| campaign_id | integer | False | Only used with player_id. Filters the results to only include campaign sub-playlists belonging to this campaign. Legacy playlists are not affected by this filter. |

### Example Request - Fetch by Playlist ID

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "playlist_id": 12345
}' \
https://onqcms.com/api/playlist/fetch
```
This request fetches the playlist with registry ID **12345**. Returns a single [Playlist Object](#playlist-parameters).

### Example Request - Fetch by Player ID

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "player_id": 1144
}' \
https://onqcms.com/api/playlist/fetch
```
This request fetches all playlists (both legacy and campaign sub-playlists) assigned to the player **1144**. Returns an object containing the player_id and an array of [Playlist Objects](#playlist-parameters).

### Example Return - Fetch by Player ID

```json
{
    "player_id": 1144,
    "playlists": [
        {
            "id": 12345,
            "playlist_type": "legacy",
            "source_id": 42,
            "name": "Lobby Playlist",
            "campaign_id": null,
            "parent_campaign_name": null,
            "dateCreated": "2024-06-15T10:30:00+10:00",
            "dateModified": "2024-12-01T14:22:00+10:00",
            "mediaAssets": [],
            "players": [1144, 1153],
            "schedule": {
                "days": ["Mon", "Tue", "Wed", "Thu", "Fri"],
                "from_date": "2024-06-15T00:00:00+10:00",
                "from_time": "08:00",
                "to_date": "2025-06-15T00:00:00+10:00",
                "to_time": "18:00",
                "date_time_independent": false
            }
        },
        {
            "id": 67890,
            "playlist_type": "campaign_sub",
            "source_id": 5,
            "name": "Summer Promo - Division 1",
            "campaign_id": 142,
            "parent_campaign_name": "Summer Promo",
            "dateCreated": "2024-12-01 00:00:00",
            "dateModified": "2025-01-15 14:30:00",
            "mediaAssets": [],
            "players": [1144, 1153],
            "schedule": {
                "start_date_time": "2024-12-01 00:00:00",
                "end_date_time": "2025-02-28 23:59:59",
                "schedules": [],
                "content_validity": []
            }
        }
    ]
}
```

### Example Request - Fetch by Player with Campaign Filter

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "player_id": 1144,
    "campaign_id": 142
}' \
https://onqcms.com/api/playlist/fetch
```
This request fetches playlists assigned to player **1144** that belong to campaign **142**. Only campaign sub-playlists matching the campaign_id are returned. Legacy playlists are still included in the results.

### Error Responses

`Error Response:`

- Status: 400 Bad Request
```json
{
    "error": "Invalid JSON"
}
```

- Status: 400 Bad Request
```json
{
    "error": "Missing required parameter: playlist_id or player_id"
}
```

- Status: 401 Unauthorized
```json
{
    "error": "Invalid or missing API key"
}
```

- Status: 404 Not Found
```json
{
    "error": "Playlist not found"
}
```

- Status: 404 Not Found
```json
{
    "error": "Player not found"
}
```

[Top](#playlist)

## Create a Bulk Playlist

### Endpoint: playlist_bulk/create

The bulk playlist request creates a special type of playlist that is unique to each assigned player. This playlist can be viewed, edited and deleted, but not reassigned via the CMS. If the bulk playlist request is called on a player that already has an API playlist assigned, the playlist is deleted and recreated with the settings as per the new request.

### Request Parameters
The CMS expects an array of playlist objects as the input.

### Playlist JSON Object

| Parameter | Type                          | Required | Description |
|-----------|-------------------------------|----------|-------------|
| players   | array (strings - player IDs)  | True     | The media players to be assigned to the playlist |
| playlist_width | integer	|	True | Playlist Width |
| playlist_height | integer	|	True | Playlist Height |
| playlist  | array (playlist asset object) | True     | A list of assets to playback. See below table for parameters. Assets will play back in the order that they are in the array, restarting playback from the start once the end of the playlist is reached.

### Playlist Asset JSON Object

| Parameter | Type                     | Required | Description |
|-----------|--------------------------|------------|-------------|
| uid       | string                   | If no file | The unique ID of an asset on the CMS to play back. Must not define file if using uid. |
| file      | object (see table below) | If no uid  | The details of an asset for the CMS to fetch from a remote url, if it hasn't been fetched already. Must not define uid if using file. |
| duration  | integer                  | False      | The playback duration for images, defaults to 10 seconds. |
| tags      | object (see table below) | False      | Tags to control which players play back this asset.
| validity  | object (see table below) | False      | An object that defines whether an asset is valid to play within this playlist. See table below for values. |

### File JSON Object

| Parameter  | Type                     | Required | Description |
|------------|--------------------------|----------|-------------|
| url        | string                   | True     | The unique ID of an asset on the CMS to play back. Must not define file if using uid. |
| name       | object (see table below) | False    | The details of an asset for the CMS to fetch from a remote url, if it hasn't been fetched already. Must not define uid if using file. |
| hash       | string                   | False    | The md5 hash of the asset. An error is returned if the hash of the downloaded file doesn't match this value. |
| folder     | array (string)           | False    | The folder that the asset is stored in. Contains the full folder path, with the 'leftmost' item being the closest to root level. For example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of ["marketing","campaigns"]. Asset will be stored at the root level if not present.
| references | array (string)           | False    | A list of additional names that refer to this asset. Used to provide an additional name/tag against the asset in the proof of play reports.
| validity   | object (see table below) | False    | An object that defines whether an asset is valid to play, regardless of whether it is an active playlist or not. See table below for values. |

### Validity JSON Object

| Parameter             | Type    | Description |
|-----------------------|---------|-------------|
| start_date            | string  | The asset will not play before this date. YYYY-MM-DD format, eg 2022-07-15. |
| end_date              | string  | The asset will not play after this date. YYYY-MM-DD format, eg 2022-09-15. |
| start_time            | string  | The asset will not play before this time. HH:MM format, 24 hour time, eg 07:00. |
| end-time              | string  | The asset will not play after this time. HH:MM format, 24 hour time, eg 17:00. |
| date_time_independent | boolean | If true, the date and time conditions are evaluated separately - the asset will only play between the valid times between the valid dates. If false, the asset will play constantly after the valid time and date until the end time and date. |

### Tags JSON Object

| Parameter | Type           | Description |
|-----------|----------------|-------------|
| all       | array (string) | The player must have all of these tags to play the asset. |
| any       | array (string) | The player needs any one of these tags to play the asset. |
| none      | array (string) | The player must *not* have any of these tags to play the asset. |

### Example Request
```
curl -H "Authorization: bearer demo123" -X POST -d '[
	{
		"players":["demo123"],
		"playlist_width": 1920,
		"playlist_height": 1080,
		"playlist":[
			{
				"file":{
					"url": "https://onqcms.com/api-test/test-pic.jpg",
					"hash": "00000000000000",
					"name":"demo image",
					"folder":["marketing","campaigns"],
					"references":["demo","demo_pic"],
					"validity":{
						"start_date":"2022-07-15",
						"end_date":"2022-09-15",
						"start_time":"07:00",
						"end_time":"15:00",
						"date_time_independent":true
					}
				},
				"duration":5,
				"tags":{
                    "all":["all 1","all 2"],
					"any":["any tag1","any tag2", "any tag3"],
                    "none":["exclude"]
				},
				"validity":{
					"start_date":"2022-07-15",
					"end_date":"2022-09-15",
					"start_time":"07:00",
					"end_time":"15:00",
					"date_time_independent":true
				}
			}
		]
	}
]' \
https://onqcms.com/api/playlist_bulk/create
```
The above snippet creates (or deletes and recreates) and assigns an API playlist for the player **demo123**. The playlist will loop over one piece of content that has been fetched from the url https://onqcms.com/api-test/test-pic.jpg while the validity is true. The asset will play on any player with all the below:
- either the **any tag1**, **any tag2** or **any tag3** tags; and
- which do not have the **exclude** tag; and
- which have both the **all1** and **all2** tags.

[Top](#playlist)

[Home](README.md)
