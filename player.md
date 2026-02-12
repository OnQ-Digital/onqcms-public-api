# Player

OnQ CMS players are the cloud endpoint for the physical media players that run the screen. The API allows interaction with the media players to get their IDs, see their status and fetch their screenshots.

- [Fetch Player](#fetch-player)
- [Update Player](#update-player)

[Home](README.md)

## Player Parameters

### Player Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | The name of the player (as it appears on CMS). Does not need to be unique. |
| uid | string | The unique ID that identifies the player to the onQ CMS. |
| resolution_width | integer | The resolution width of the player's display in pixels. |
| resolution_height | integer | The resolution height of the player's display in pixels. |
| folder | array (string) | The folder that the player is stored in. Contains the full folder path, with the 'leftmost' item being the closest to root level. For example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of \["marketing","campaigns"\]. |
| tags | array (string) | A list of tags that control what assets will playback on this player. |
| last_update | string | A timestamp of when the player last checked in with onQ CMS. Format "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'" - eg 2024-04-14T10:35:53+10:00 |
| screenshot_update | string | A timestamp of when the player's screenshot was last updated. Format "yyyy-MM-dd'T'HH:mm:ss.SSS'Z'" - eg 2024-04-14T10:35:53+10:00. |
| screenshot |string | A URL pointing to the latest screenshot from the player (allowing download).

### Player JSON object
```
{
    "name":"Demo Player",
    "uid":"demo123",
    "resolution_width":1920,
    "resolution_height":1080,
    "folder":["Australia","Demo"],
    "tags":["demo","AU"],
    "last_update":"2022-04-14T10:35:53+10",
    "screenshot_update":"2022-04-14T10:35:53+10",
    "screenshot":"https://onqcms.com/screenshots..."
}
```

## Fetch Player

### Endpoint: player/fetch
### Response: JSON array of Player Objects (see 4.1)

### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| name | string | Only players that contain **name** in their name. EG **demo.mp4** will match a file called "demo.mp4" as well as "my demo.mp4". |
| uid | string | Returns the players with the matching UID. Name is ignored if the UID parameter is present. Player uid can be set in the OnQ CMS by going to the players page, clicking view then edit on the player, and then updating the player id field. |
| path | array (string) | Full path to the folder to filter on. The items should be ordered left-to-right based on how close the folder is to the root level. Defaults to root level. |
| recurse | boolean | True if players in sub-folders of the target folder should be returned. Defaults to true. |
| tags | array (string) | Only return players that contain *all* tags in the **tags** array. |

### Example Request

```
curl -H "Authorization: bearer demo1234" -X POST -d '{ \
"name":"demo",
"path":["Australia"],
"recurse":false,
"tags":["demo","AU"]
}' \
https://onqcms.com/api/player/fetch
```
The above code gets all players with "demo" in their name that are in the "Australia" folder but not any of its subfolders. The players must have the reference "demo" and must have the "demo" and "AU" tags.

[Top](#player)


## Fetch Player - Campaign Details

### Endpoint: player/detail
### Response: JSON array of Player Objects
### Request Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| company_group_id | integer | Company Group ID |
| player_id | integer/array | Single Player ID or Array of multiple player IDs |


### Example Request

```
curl -H "Authorization: bearer demo1234" -X POST -d '{ \
"company_group_id": 3,
"player_id":[1144, 1413]
}' \
https://onqcms.com/api/player/detail
```
```
curl -H "Authorization: bearer demo1234" -X POST -d '{ \
"company_group_id": 3,
"player_id":1144
}' \
https://onqcms.com/api/player/detail
```

### Example Return
```
{
    "categories": [
        {
            "key": "brand",
            "value": [
                "hugo boss"
            ]
        }
    ],
    "tags": [
        "test 2"
    ],
    "campaign_details": [
        {
            "campaign_id": "142",
            "campaign_name": "DJ Campaign",
            "campaign_sub": [
                {
                    "campaign_sub_id": "5",
                    "playlist_content": [
                        33361
                    ]
                },
                {
                    "campaign_sub_id": "6",
                    "playlist_content": [
                        33362
                    ]
                }
            ]
        },
        {
            "campaign_id": "143",
            "campaign_name": "DJ Campaign",
            "campaign_sub": [
                {
                    "campaign_sub_id": "7",
                    "playlist_content": [
                        33361,
                        33362
                    ]
                },
                {
                    "campaign_sub_id": "8",
                    "playlist_content": [
                        33362
                    ]
                }
            ]
        }
    ],
    "file_contents": [
        {
            "file_content_id": 33361,
            "file_title": "CR_May22_MumDay_1920x1080px_10.jpg",
            "file_name": "3-237-1651629762-74e0412.jpg",
            "file_thumb_name": "3-237-1651629762-74e0412-thumb.jpg",
            "file_ext": "jpg",
            "file_width": 1920,
            "file_height": 1080,
            "file_hash": "b7e0f08f88ca7c75982f98ffd01cd417",
            "file_uid": null
        },
        {
            "file_content_id": 33362,
            "file_title": "CR_May22_MumDay_1920x1080px_11.jpg",
            "file_name": "3-237-1651629763-304480f.jpg",
            "file_thumb_name": "3-237-1651629763-304480f-thumb.jpg",
            "file_ext": "jpg",
            "file_width": 1920,
            "file_height": 1080,
            "file_hash": "feb7b8f3f798b145596f920ab5a9ac11",
            "file_uid": null
        }
    ]
}
```

[Top](#player)

## Update Player

### Endpoint: content/update

Update the fields on an existing player. Request can either be an array of player objects for a bulk update, or a single player object.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| uid | string | True | The unique ID that identifies the player to be updated |
| references | array (string) | False | The new references to assign to the player - will replace whatever references were already assigned. An empty array will clear all references from the player.
| tags | array (string) | False | The new tags to assign to the player - will replace whatever tags were already assigned. An empty array will clear all tags from the player.

### Example Request
```
curl -H "Authorization: bearer demo1234" -X POST -d '{ \
    "uid":"demo123", \
    "references":["old demo"] \
    "tags":["old","demo"] \
}' \
https://onqcms.com/api/content/update
```
The above request updates the player with the uid "demo123" and replaces the references on the player to "old demo" and replaces the tags on the player with "old", and "demo".

[Top](#player)
