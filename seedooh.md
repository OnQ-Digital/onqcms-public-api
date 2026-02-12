# Seedooh

The Seedooh API provides integration endpoints for the Seedooh verification platform to interact with onQ's CMS.

- [Seedooh](#seedooh)
  - [Campaigns Fetch](#campaigns-fetch)
  - [Players Fetch](#players-fetch)
  - [Campaign Playlogs Generator](#campaign-playlogs-generator)
  - [Campaign Playlogs Fetch](#campaign-playlogs-fetch)

[Back to Home](README.md)

## Campaigns Fetch

### Endpoint: seedooh/campaigns_fetch

Retrieves campaign data for Seedooh Front End with player associations and campaign details.

**Request Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| campaign_id | integer | False | Campaign ID to fetch specific campaign |
| start_date_time | string | False | Start date time filter (YYYY-MM-DD HH:MM:SS) |
| end_date_time | string | False | End date time filter (YYYY-MM-DD HH:MM:SS) |
| min_bookings | integer | False | Minimum campaign bookings filter |
| max_bookings | integer | False | Maximum campaign bookings filter |
| campaign_statuses | array | False | Array of campaign statuses to filter |
| campaign_tag_ids | array | False | Array of campaign tag IDs to filter |
| mediaplayer_ids | array | False | Array of media player IDs to filter |
| mediaplayer_tag_ids | array | False | Array of media player tag IDs to filter |
| mediaplayer_cat_ids | array | False | Array of media player category IDs to filter |
| sort_field | string | False | Field to sort by (campaign_id, campaign_name, start_date, end_date, created_at) |
| sort_order | string | False | Sort order (asc, desc) |
| by_group | integer | False | Group by type (0 for division, 1 for group, 2 for details) |

**Request Example**

```json
{
  "campaign_id": 123,
  "start_date_time": "2024-01-01 00:00:00",
  "end_date_time": "2024-12-31 23:59:59",
  "campaign_statuses": ["live"],
  "mediaplayer_ids": [1144, 1153],
  "sort_field": "campaign_id",
  "sort_order": "asc",
  "by_group": 1
}
```

**Success Response**

- Status: 200 OK

```json
[
  {
    "campaign_id": 123,
    "campaign_title": "Summer Campaign 2024",
    "advertiser": "Brand Name",
    "media_buyer": "Agency Name",
    "playlist_ids": [456, 457],
    "schedules": [
      {
        "schedule_id": "sch_456",
        "campaign_sub_id": 456,
        "playlist_id": 456,
        "start_date": "2024-01-01",
        "end_date": "2024-12-31",
        "asset_ids": [33361, 33362],
        "assets": [
          {
            "asset_id": 33361,
            "name": "Creative Asset 1",
            "download_url": "https://cdn.example.com/media/asset1.mp4"
          },
          {
            "asset_id": 33362,
            "name": "Creative Asset 2",
            "download_url": "https://cdn.example.com/media/asset2.mp4"
          }
        ],
        "player_ids": [1144, 1153]
      }
    ],
    "seedooh_verified": true,
    "filter_request_data": {
      "record_output_type": {
        "Seedooh": 123
      },
      "datetime_range": {
        "start": "2024-01-01T00:00:00",
        "end": "2024-12-31T23:59:59"
      },
      "event_types": null,
      "active_hours_record_only": true
    }
  }
]
```

**Error Response**

- Status: 400 Bad Request

```json
{
  "success": false,
  "error": "Validation failed",
  "details": {
    "campaign_id": "Campaign ID must be a positive integer"
  },
  "code": 400
}
```

- Status: 401 Unauthorized

```json
{
  "success": false,
  "error": "User is not logged in",
  "code": 401
}
```

- Status: 404 Not Found

```json
{
  "success": false,
  "error": "Campaign not found",
  "code": 404
}
```

[Top ↑](#seedooh)

## Players Fetch

### Endpoint: seedooh/players_fetch

Retrieves player data for Seedooh Front End with campaign associations and screenshot information.

**Request Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| name | string | False | Player name filter (partial match on site_name or store_name) |
| uid | string | False | Player unique ID (exact match on player_id). If provided, name filter is ignored |
| folder_path | array | False | Array of folder path strings for hierarchical folder navigation |
| folder_recurse | boolean | False | Include subfolders when filtering by folder (default: true) |
| folder_name | string | False | Alternative to folder_path - single folder name |
| tags | array | False | Array of tag names to filter by (all tags must match) |
| timezone | string | False | Timezone for date formatting (default: UTC) |
| company_group_id | integer | False | Company group ID for validation |

**Request Example**

```json
{
  "name": "demo",
  "folder_path": ["Australia", "Melbourne"],
  "folder_recurse": true,
  "tags": ["demo", "AU"],
  "timezone": "Australia/Melbourne"
}
```

**Success Response**

- Status: 200 OK

```json
[
  {
    "player_name": "Demo Player",
    "player_id": 1144,
    "address": "123 Main Street",
    "region": "Victoria",
    "state": "VIC",
    "format": "Digital Billboard",
    "category": "Retail",
    "longitude": "144.9631",
    "latitude": "-37.8136",
    "pixel_width": 1920,
    "pixel_height": 1080,
    "max_slots_per_loop": 10,
    "default_slot_duration": 10,
    "n_screens": 1,
    "operating_hours": ["06:00-22:00"]
  }
]
```

**Error Response**

- Status: 400 Bad Request

```json
{
  "success": false,
  "error": "Validation failed",
  "details": {
    "folder_path": "Folder path must be an array"
  },
  "code": 400
}
```

- Status: 401 Unauthorized

```json
{
  "error": "User is not logged in."
}
```

[Top ↑](#seedooh)

## Campaign Playlogs Generator

### Endpoint: seedooh/campaign_playlogs_generator

Initiates playlog generation for a specific campaign.

**Request Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| record_output_type | object | True | Record output type configuration |
| record_output_type.Seedooh | integer | True | Campaign ID for Seedooh output |
| datetime_range | object | True | Date range for playlog generation |
| datetime_range.start | string | True | Start date time (YYYY-MM-DDTHH:MM:SS) |
| datetime_range.end | string | True | End date time (YYYY-MM-DDTHH:MM:SS) |
| player_ids | array | False | Array of player IDs to filter report. If not provided, all campaign players are included. Only players assigned to the campaign will be included |

**Request Example**

```json
{
  "record_output_type": {
    "Seedooh": 123
  },
  "datetime_range": {
    "start": "2024-01-01T00:00:00",
    "end": "2024-01-31T23:59:59"
  },
  "player_ids": [1144, 1153]
}
```

**Success Response**

- Status: 200 OK

```json
{
  "id": "report_20240115_143022_a1b2c3d4"
}
```

**Error Response**

- Status: 400 Bad Request

```json
{
  "success": false,
  "error": "Validation failed",
  "details": {
    "validation_errors": {
      "record_output_type": "record_output_type.Seedooh is required and must be numeric"
    }
  },
  "request_id": "req_20240115_143022_a1b2c3d4"
}
```

- Status: 401 Unauthorized

```json
{
  "success": false,
  "error": "Unauthorized",
  "request_id": "req_20240115_143022_a1b2c3d4"
}
```

- Status: 404 Not Found

```json
{
  "success": false,
  "error": "Campaign not found or not available for Seedooh reporting",
  "request_id": "req_20240115_143022_a1b2c3d4"
}
```

- Status: 503 Service Unavailable

```json
{
  "success": false,
  "error": "Failed to initiate report generation",
  "request_id": "req_20240115_143022_a1b2c3d4"
}
```

[Top ↑](#seedooh)

## Campaign Playlogs Fetch

### Endpoint: seedooh/campaign_playlogs_fetch

Retrieves generated playlog data by report ID, checking status and returning parsed playlog data.

**Request Parameters**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | True | Report ID returned from playlogs generator |

**Request Example**

```json
{
  "id": "report_20240115_143022_a1b2c3d4"
}
```

**Success Response**

The endpoint returns a status object with the report state. The `state` field indicates the report status: "Complete", "InProgress", or "Failed". When complete, the `url` field contains the download URL for the report data.

- Status: 200 OK (Complete)

```json
{
  "id": "report_20240115_143022_a1b2c3d4",
  "state": "Complete",
  "url": "https://storage.example.com/reports/report_20240115_143022_a1b2c3d4.sql",
  "error_message": null,
  "size_bytes": 102400,
  "generated_at": "2024-01-15T14:35:00Z"
}
```

- Status: 200 OK (InProgress)

```json
{
  "id": "report_20240115_143022_a1b2c3d4",
  "state": "InProgress",
  "url": null,
  "error_message": null,
  "size_bytes": null,
  "generated_at": null
}
```

- Status: 200 OK (Failed)

```json
{
  "id": "report_20240115_143022_a1b2c3d4",
  "state": "Failed",
  "url": null,
  "error_message": "Report generation timed out",
  "size_bytes": null,
  "generated_at": null
}
```

**Error Response**

- Status: 400 Bad Request

```json
{
  "error": "Report ID is required"
}
```

- Status: 401 Unauthorized

```json
{
  "error": "User is not logged in"
}
```

- Status: 404 Not Found

```json
{
  "error": "Report not found or access denied"
}
```

- Status: 500 Internal Server Error

```json
{
  "error": "An unexpected error occurred while fetching playlogs",
  "message": "Please try again later or contact support if the issue persists"
}
```

- Status: 503 Service Unavailable

```json
{
  "error": "Failed to fetch report status"
}
```

[Top ↑](#seedooh)

[Back to Home](README.md)
