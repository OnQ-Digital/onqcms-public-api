# Content

The onQ CMS API allows users to fetch, upload, edit and delete content. Content is stored in folders (which can contain sub-folders), and the API allows the creation and deletion of content folders. A single piece of content on the CMS is called an **asset**.

- [Fetch Content](#fetch-content)
- [Fetch File](#fetch-file)
- [Create Content](#create-content)
- [Create Content from URL](#create-content-from-url)
- [Update Content](#update-content)
- [Delete Content](#delete-content)
- [Create Content Folder](#create-content-folder)
- [Delete Content Folder](#delete-content-folder)

[Home](README.md)

## Content Parameters

### Asset Parameters

| Parameter  | Type                      |Description                                                                                                                           |
|-------------|--------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| name        | string                   | The name of the asset (as it appears on CMS). Does not need to be unique. |
| uid         | string                   | The unique ID that identifies the asset to the onQ CMS. Created by onQ CMS when the asset is created. |
| file_width  | integer                  | The pixel dimensions of the asset. Set by the CMS when an asset is created on the system. |
| file_height | integer                  | The pixel dimensions of the asset. Set by the CMS when an asset is created on the system. |
| error       | object (see table below) |
| type        | string                   | The type of the asset, currently only "jpeg", "mp4" and "png" are supported via the API. |
| folder      | array (string)           | The folder that the asset is stored in. Contains the full folder path, with the 'leftmost' item being the closest to root level. For  example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of ["marketing","campaigns"]. |
| references  | array (string)           | A list of additional names that refer to this asset. Used to provide an additional name/tag against the asset in the proof of play reports. |
| validity    | object (see table below) | An object that defines whether an asset is valid to play, regardless of whether it is an active playlist or not. |
| player_check | Array ({id:"Player ID",valid:Boolean}) | A list of players specified in **player_check** when creating a new asset - returns whether the specified players aspect ratio is suitable for the asset, accounting for the aspect tolerance set on the player. |

### Validity Parameters

| Parameter             | Type   | Description                                                                                                                                  |
|-----------------------|--------|----------------------------------------------------------------------------------------------------------------------------------------------|
| start_date            | string | The asset will not play before this date. YYYY-MM-DD format, eg 2022-07-15. |
| end_date              | string | The asset will not play after this date. YYYY-MM-DD format, eg 2022-09-15. |
| start_time            | string | The asset will not play before this time. HH:MM format, 24 hour time, eg 07:00. |
| end-time              | string | The asset will not play after this time. HH:MM format, 24 hour time, eg 17:00. |
| date_time_independent | boolean | If true, the date and time conditions are evaluated separately - the asset will only play between the valid times between the valid dates. If false, the asset will play constantly after the valid time and date until the end time and date. |

### Example Asset JSON Object

```
{
    "name":"Demo Move.mp4",
    "uid":"demo_uid_1234",
    "file_width":1920,
    "file_height":1080,
    "type":"mp4",
    "folder":["marketing","campaigns"],
    "references":["demo","movies"],
    "validity":{
        "start_date":"2022-07-15",
        "end_date":"2022-09-15",
        "start_time":"07:00",
        "end_time":"17:00",
        "date_time_independent":true
    },
    player_check:[{id:"123",valid:true}]
}
```
[Top](#content)

## Fetch Content

### Endpoint: content/fetch
### Response: JSON array of [Asset Objects](#asset-parameters)

### Request Parameters

| Parameter  | Type           | Description |
|------------|----------------|-------------|
| name       | string         | Only assets that contain **name** in their name. EG **demo.mp4** will match a file called "demo.mp4" as well as "my demo.mp4". |
| uid        | string         | Returns the asset with the matching UID. Name is ignored if the UID parameter is present. |
| type       | string         | Only returns assets matching the type.
| folder     | array (string) | Full path to the folder to filter on. The items should be ordered left-to-right based on how close the folder is to the root level. If left empty, all folders will be searched. |
| recurse    | boolean        | Check child folders of the search folder. Default is true. |
| references | array (string) | Only return assets that contain *all* references in the **references** array. |
| check_players | array (string) | The system will check the dimensions of the uploaded content against the aspect ratio of the players identified by their IDs in the array, accounting for their tolerance settings. |

### Example Request

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
"name":"demo",
"type":"mp4",
"path":["marketing","campaigns"],
"recurse":false,
"references":["demo","demo_vid"]
}' \
https://onqcms.com/api/content/fetch
```
This request will fetch all mp4 content with "demo" in its name in the campaigns folder (which is in the marketing folder), but not any of its sub-folders. The assets must have the references "demo" and "demo_vid" to be returned.

## Fetch File

### Endpoint: file/fetch
### Response: JSON array of [Asset Objects](#asset-parameters)

### Request Parameters

| Parameter  | Type           | Required    |Description |
|------------|----------------|-------------|-------------|
| uid       | integer/array   |  False      | UID (file_uid) Value. Null for all group files |

### Example Request

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
"uid":"uid_1"
}' \
https://onqcms.com/api/file/fetch
```
This request will fetch file content with file download URL details

### Example of return values
 ```
 [
     {
        "file_content_id": "111",
        "file_group_id": "111",
        "file_user_id": "11",
        "file_title": "abc.mp4",
        "file_name": "3-5-111-xxx.mp4",
        "file_ext": "mp4",
        "file_width": "4080",
        "file_height": "1920",
        "file_size": "33306454",
        "file_hash": "ecd7c5b7axxxxxxxxxxx106e58c51891",
        "file_thumb_name": "3-5-111-xxx-thumb.jpg",
        "file_icon_name": "3-5-111-xxx-icon.jpg",
        "file_uploaded": "1600164009",
        "encode_wait": "0",
        "valid_from": null,
        "valid_until": null,
        "file_folder_id": null,
        "file_json_name": null,
        "file_uid": "uid_1",
        "download_url": "https://xxxx.blob.core.windows.net/xxx/",
        "file_download_url": "https://xxxx.blob.core.windows.net/..../3-5-111-xxx.mp4",
        "file_thumb_download_url": "https://xxxx.blob.core.windows.net/..../3-5-111-xxx-thumb.jpg",
        "file_icon_download_url": "https://xxxx.blob.core.windows.net/..../3-5-111-xxx-icon.jpg"
    },
    ..
 ]
 ```

## Create Content

### Endpoint: content/create
### Encoding: multi-part/form
### Response: JSON array of [Asset Objects](#asset-parameters)

The request to create an asset must have the multi-part/form encoding.

### Request Parameters

| Parameter | Type   | Required | Description |
|-----------|--------|----------|-------------|
| file      | binary | True     | The media file to upload to the server. Type will be automatically detected by the server. |
| data      | Object | True     | The attributes of the asset on onQ CMS. |

### Data JSON Object

| Parameter     | Type                     | Required          | Description |
|---------------|--------------------------|-------------------|-------------|
| name          | string                   | True              | The name of the asset (as it appears on CMS). Does not need to be unique. |
| folder        | array (string)           | False             | The folder that the asset is stored in. Contains the full folder path, with the 'leftmost' item being the closest to root level. For example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of ["marketing","campaigns"]. Asset will be stored at the root level if not present.
| references    | array (string)           | False             | A list of additional names that refer to this asset. Used to provide an additional name/tag against the asset in the proof of play reports.
| validity      | [Validity JSON Object](#validity-parameters) | False | An object that defines whether an asset is valid to play, regardless of whether it is an active playlist or not. |
| check_players | array (string) | The system will check the dimensions of the uploaded content against the aspect ratio of the players identified by their IDs in the array, accounting for their tolerance settings. |

### Example Request
```
curl -H "Authorization: bearer demo1234" \
-F file='@demo video.mp4' \
-F data='{
    "name":"demo background",
    "folder":["marketing","campaigns"],
    "references":["demo","demo_vid"],
    "validity":{
        "start_date":"2022-07-15",
        "end_date":"2022-09-15",
        "start_time":"07:00",
        "end_time":"15:00",
        "date_time_independent":true
    }
}' \
https://onqcms.com/api/content/create
```
The above snippet adds the **demo video.mp4** file as an asset to the onqCMS. It has the name **demo video** and the additional references **demo** and **demo_vid** which will appear in any proof of play reports.. The asset will be located in the **campaigns** content folder, which is in the **marketing** folder. The asset will only play between 7AM and 3PM every day after 2022-07-15 and before 2022-09-15 even if it is in an assigned and active playlist.

[Top](#content)

## Create Content from URL

### Endpoint: content/create-from-url
### Response: Json Object with key-value {"job_id":"job123"}. Job result can be fetched via the [API job request endpoint](api-job.md). The [Asset Object](#asset-parameters) are set as the payload of the completed job.

The system will fetch an asset from the provided URL and add it to the content
library. We can also choose to use the URL as a UID for future referencing of the file. NOTE - if you choose to use the URL as the UID, any future attempts to download from that URL will be ignored as duplicates.

### Request Parameters

| Parameter  | Type | Required          | Description |
|------------|------|-------------------|-------------|
| name       | string                   | True | The name of the asset (as it appears on CMS). Does not need to be unique. |
| url        | string                   | True  | The url to download the asset from. |
| url_is_uid | boolean                  | False | False by default. If false, the system will generate an id for the new content. New assets can be created by downloading from the provided url. If true, the system won't create a new asset if there is one that already has this url. The url can be used to reference the asset where specified in the API. |
| folder     | array (string)           | False | The folder that the asset is stored in. Contains the full folder path, with the 'leftmost' item being the closest to root level. For example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of ["marketing","campaigns"]. Asset will be stored at the root level if not present.
| references | array (string)           | False | A list of additional names that refer to this asset. Used to provide an additional name/tag against the asset in the proof of play reports.
| validity   | [Validity JSON Object](#validity-parameters) | False | An object that defines whether an asset is valid to play, regardless of whether it is an active playlist or not. |
| check_players | array (string) | The system will check the dimensions of the uploaded content against the aspect ratio of the players identified by their IDs in the array, accounting for their tolerance settings. |

### Example Request
```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "name":"URL Demo",
    "folder":["Demo"],
    "url":"https://demo.com/demo_file123.mp4",
    "url_is_uid":true,
    "references":["from url"],
    "validity":{
        "start_date":"2024-01-01",
        "start_time":"07:00"
    }
}' \
https://onqcms.com/api/content/create-from-url
```
The above snippet adds the file stored at **https://demo.com/demo_file123.mp4** as an asset to the onQ CMS. It has the name **URL Demo** and the additional reference **from url** which will appear in any proof of play reports. The asset will be located in the **Demo** content folder. The "url_is_uid" has been set, allowing us to refer to this file by its URL where indicated in other API requests. Additionally, any attempts to upload an asset with the url **https://demo.com/demo_file123.mp4** will be rejected unless this file is deleted.

[Top](#content)

## Update Content

### Endpoint: content/update

Update the fields on an existing asset.

#### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | string | False | The new name of the asset. |
| uid | string | True | The unique ID that identifies the asset to be updated |
| folder | array (string) | False | The folder to move the asset to. Contains the full folder path, with the 'leftmost' item being the closest to root level. For example, an asset stored in a 'campaigns' folder, which is itself stored in a 'marketing' folder would have a folder value of ["marketing","campaigns"]. To move the asset to the root folder, provide an array with a single empty string ([""]).
| references | array (string) | False | The new references to assign to the asset - will replace whatever references were already assigned. An empty array will clear all references from the asset.
| validity | object (see table below) | False | The new validity data for the asset. If this is set to false, then the validity check will be removed from the object. |

### Validity JSON Object

| Parameter | Type | Description |
|-----------|------|-------------|
| start_date | string | The asset will not play before this date. YYYY-MM-DD format, eg 2022-07-15. |
| end_date | string | The asset will not play after this date. YYYY-MM-DD format, eg 2022-09-15. |
| start_time | string | The asset will not play before this time. HH:MM format, 24 hour time, eg 07:00. |
| end-time | string | The asset will not play after this time. HH:MM format, 24 hour time, eg 17:00. |
| date_time_independent | boolean | If true, the date and time conditions are evaluated separately - the asset will only play between the valid times between the valid dates. If false, the asset will play constantly after the valid time and date until the end time and date. |

### Example Request
```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "uid":"demo123",
    "name":"old demo",
    "folder":["recycled"],
    "references":["old demo"],
    "validity":{
        "start_date":"2024-01-01",
        "start_time":"07:00"
    }
}' \
https://onqcms.com/api/content/update
```
The above request updates the asset with the uid **demo123** so that its name is now **old demo**, it is moved to the **recycled** folder, and it has validity set such that it won't play before 2024-01-01 at 7:00, overwriting the original validity.

[Top](#content)

## Delete Content

### Endpoint: content/delete
### Response: JSON array of asset UIDs

### Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| uid | string | True | The id of the asset to be deleted |

### Example Request
```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "uid":"demo123"
}' \
https://onqcms.com/api/content/delete
```

## Create Content Folder

### Endpoint: content_folder/create

### Request Parameters

| Parameter | Type   | Description |
|-----------|--------|-------------|
| folder    | object | Full path of folder/folder's to create, with folders ordered left to right in ancestry (grandparent, parent, child). Any folders in the array that do not exist will be created.

### Example Request

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "folder":["marketing","campaigns"]
}' \
https://onqcms.com/api/content_folder/create
```
This request creates a new folder **campaigns** in the marketing folder. If the **marketing** folder doesn't exist, it will be created first, and the **campaigns** folder will then be created inside it.

[Top](#content)

## Delete Content Folder

### Endpoint: content_folder/delete

**WARNING** This request will also delete any assets inside the targeted folder.

### Request Parameters

| Parameter | Type   | Description |
|-----------|--------|-------------|
| folder    | object | Full path of folder to delete, with folders ordered left to right in ancestry (grandparent, parent, child).

```
curl -H "Authorization: bearer demo1234" -X POST -d '{
    "folder":["marketing","campaigns"]
}' \
https://onqcms.com/api/content_folder/delete
```
This request deletes the folder **campaigns** in the marketing folder. All assets within this folder will also be deleted.


[Top](#content)
