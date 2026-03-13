# External Import

The External Import API allows external systems (e.g., VAMS) to programmatically create, update, and deactivate campaigns on the CMS.

**Sync model**: Full-state. Each call sends the complete list of active bookings; the system reconciles to match. Expired campaigns are invisible to the sync process.

**Two-phase workflow**: The Sync endpoint handles campaign booking (create/update/deactivate). Content downloading, encoding, and dimension validation happen out-of-band — use the Status endpoint to check content readiness and warnings.

[Back to Home](README.md)

# TOC
- [1. Sync Endpoint](#1-sync-endpoint)
  - [1.1. Request Format](#11-request-format)
  - [1.2. Response Format](#12-response-format)
  - [1.3. Warnings](#13-warnings)
  - [1.4. Errors](#14-errors)
  - [1.5. Sync Behavior](#15-sync-behavior)
  - [1.6. Context Passthrough](#16-context-passthrough)
  - [1.7. Dry Run](#17-dry-run)
- [2. Status Endpoint](#2-status-endpoint)
  - [2.1. Request Format (GET)](#21-request-format-get)
  - [2.2. Request Format (POST)](#22-request-format-post)
  - [2.3. Response Format](#23-response-format)
  - [2.4. Content Status Values](#24-content-status-values)
  - [2.5. Warnings](#25-warnings)
- [3. Dry Run Endpoint](#3-dry-run-endpoint)

---

# 1. Sync Endpoint

**Endpoint**: `POST /api/external_import/sync`

**Authentication**: Bearer token (API key).

```
Authorization: Bearer {api_key}
```

Receives booking items, translates them into campaigns, and reconciles with existing CMS state. Campaign booking (create/update/deactivate) is handled synchronously. New content is registered and queued for asynchronous download — use the [Status endpoint](#2-status-endpoint) to check content readiness and dimension warnings.

## 1.1. Request Format

**Headers**:
| Header | Value | Required |
|--------|-------|----------|
| Content-Type | application/json | Yes |
| Authorization | Bearer {api_key} | Yes |

**Request Body**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| system | string | No | External system identifier. Default: `"vams"` |
| company_group_id | integer | No | Must match authenticated user's group |
| dry_run | boolean | No | If `true`, returns what would happen without making any changes. Useful for testing. |
| items | array | Yes | Array of booking items from the external system |

**VAMS Item Schema**:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| orderNumber | string/integer | Yes | Order identifier. Items with the same orderNumber form one campaign. |
| code | string | Yes | Player/store code. Matched against `store_name` and `site_name` via LIKE search. |
| startDate | string | Yes | Start date in `YYYY-MM-DD` format |
| endDate | string | Yes | End date in `YYYY-MM-DD` format |
| slot | string/integer | Yes | Slot identifier |
| url | string or array | Yes | Content URL(s). CloudFront signed URLs are supported. The filename UUID is extracted as the stable content identifier. |
| brand | string | No | Brand/advertiser name |
| advertiserName | string | No | Alternative to `brand` |
| category | string | No | Category name |
| vaType | string | No | External slot type name (used for slot mapping) |
| name | string | No | Alternative external slot name |
| context | object | No | Opaque passthrough object. Stored and echoed back in sync and status responses. Max 4 KB per item. See [Context Passthrough](#16-context-passthrough). |

**Example Request**:

```json
{
    "system": "vams",
    "items": [
        {
            "orderNumber": "3940",
            "code": "3201DS0061",
            "startDate": "2026-01-12",
            "endDate": "2026-02-15",
            "slot": 2,
            "vaType": "Behind POS Counter",
            "brand": "TOM FORD",
            "category": "Beauty",
            "url": "https://example.com/path/4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa.mp4?Expires=...",
            "context": {
                "groupKey": "abc-123"
            }
        },
        {
            "orderNumber": "3940",
            "code": "3201DS0072",
            "startDate": "2026-01-12",
            "endDate": "2026-02-15",
            "slot": 2,
            "vaType": "Behind POS Counter",
            "brand": "TOM FORD",
            "category": "Beauty",
            "url": "https://example.com/path/4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa.mp4?Expires=...",
            "context": {
                "groupKey": "abc-123"
            }
        }
    ]
}
```

**Dry Run Example**:

```json
{
    "system": "vams",
    "dry_run": true,
    "items": [ ... ]
}
```

A dry run performs translation and classification (which campaigns would be created, updated, or deactivated) but does not make any changes. The response will have `"result": "dry_run"` instead of `"ok"`. The `campaigns` array includes full details for each affected campaign — see [1.7. Dry Run](#17-dry-run). A dedicated dry run endpoint is also available — see [3. Dry Run Endpoint](#3-dry-run-endpoint).

## 1.2. Response Format

The sync response covers booking outcomes only. Use the [Status endpoint](#2-status-endpoint) to check content download/encoding status and dimension warnings.

```json
{
    "result": "ok",
    "summary": {
        "total_items_received": 2,
        "campaigns_created": 1,
        "campaigns_updated": 0,
        "campaigns_ended": 0,
        "campaigns_deleted": 0,
        "skipped_manual": 0
    },
    "campaigns": [
        {
            "campaign_name": "3940",
            "brand": "TOM FORD",
            "campaign_id": 456,
            "action": "created",
            "sub_campaigns": 1,
            "players_matched": 2,
            "context": [
                {
                    "groupKey": "abc-123"
                }
            ]
        }
    ],
    "warnings": [],
    "errors": []
}
```

**Result Values**:

| Value | Description |
|-------|-------------|
| `ok` | Sync completed successfully |
| `dry_run` | Dry run completed; no changes made |
| `error` | Sync could not run (e.g., no config found) |

**Campaign Action Values**:

| Action | Description |
|--------|-------------|
| `created` | New campaign created |
| `updated` | Existing live campaign updated (all sub-campaigns rebuilt) |
| `deleted` | Campaign hard-deleted (campaign had never been played, no data to preserve) |
| `ended` | Campaign end dates set to now (campaign has playback history and is preserved for reporting) |
| `skipped_manual` | Campaign is tagged as manually managed; not modified by sync |

After a successful sync, use the `campaign_id` from the response to query the [Status endpoint](#2-status-endpoint) for content readiness and any dimension warnings.

## 1.3. Warnings

Warnings indicate non-fatal booking issues. The sync continues despite warnings, but the affected items may not behave as expected. Warnings are returned in the top-level `warnings` array.

### Unmatched Player

A player code from the payload could not be matched to any player in the CMS (via `store_name` or `site_name` LIKE search). The campaign is still created, but without this player assigned.

```json
{
    "type": "unmatched_player",
    "code": "UNKNOWN123",
    "campaign": "3940"
}
```

**Action required**: Verify the player code is correct and that the player exists in the CMS with a matching `store_name` or `site_name`.

### Unmapped Slot

An external slot name (e.g., `vaType`) has no mapping configured in the CMS admin. The sub-campaign is still created but assigned to the `"default"` slot group.

```json
{
    "warning": "Unmapped external slot",
    "slot": "Digital Screen - Escalator",
    "campaign": "3940"
}
```

**Action required**: Add the slot mapping in the CMS admin under External Import configuration.

### Content ID Extraction Failed

A URL in the payload could not be parsed to extract the content identifier (filename UUID). The content item is skipped for that sub-campaign.

```json
{
    "item_index": 3,
    "warning": "Could not extract content_id from URL",
    "url": "https://example.com/invalid-path",
    "orderNumber": "3940"
}
```

**Action required**: Check that the URL has a valid file path with a filename and extension (e.g., `.mp4`, `.png`).

### Context Exceeded

An item's `context` object exceeds the 4 KB limit. The context is dropped for that item but the item is otherwise processed normally.

```json
{
    "item_index": 2,
    "warning": "Context exceeds 4 KB limit and was dropped",
    "orderNumber": "3940"
}
```

**Action required**: Reduce the size of the `context` object for this item.

### Ended Not Deleted

An engine-owned campaign was removed from the payload but has playback history, so it cannot be hard-deleted. Instead, its end dates were set to now, making it "finished". The campaign is preserved for reporting.

```json
{
    "type": "ended_not_deleted",
    "campaign": "3941",
    "campaign_id": 457,
    "reason": "has playback history"
}
```

**Note**: This is expected behavior. Campaigns with playback history are preserved for reporting purposes. Campaigns that have never played are hard-deleted.

## 1.4. Errors

Errors indicate items that were rejected during translation and not processed. Errors are returned in the top-level `errors` array.

### Missing Required Fields

A booking item is missing one or more required fields and was skipped entirely.

```json
{
    "item_index": 5,
    "error": "Missing required fields: startDate, url",
    "orderNumber": "3940"
}
```

### No Valid Content URLs

All URLs for a booking item failed content_id extraction. The item was skipped.

```json
{
    "item_index": 7,
    "error": "No valid content URLs found",
    "orderNumber": "3940"
}
```

## 1.5. Sync Behavior

**Grouping rules**:
- Items with the same `orderNumber` become one CMS campaign (named by the `orderNumber`, with the `brand` set as the campaign client)
- Items with identical `startDate + endDate + slot + content` merge into one sub-campaign (players are merged)
- Items with different schedules, slots, or content become separate sub-campaigns
- If one external slot maps to multiple internal slot groups, one sub-campaign per group is created

**Lifecycle rules**:
- New orders with no matching live campaign: CREATE
- Orders matching a live campaign: UPDATE (rebuild all sub-campaigns)
- Orders matching a manually managed campaign: SKIP (opt-out)
- Engine-owned campaigns missing from payload, never played: HARD DELETE
- Engine-owned campaigns missing from payload, has played: SET end_date = NOW (campaign preserved for reporting)
- Orders matching only expired campaigns: CREATE new (expired campaigns are invisible)

**Content dedup**: Content is identified by the filename UUID extracted from the URL path (the `content_id`). If the same `content_id` already exists in the CMS, the existing content is reused without re-downloading.

## 1.6. Context Passthrough

Each item can include an optional `context` object — an opaque JSON value that the CMS stores and echoes back in both sync and status responses. This allows external systems to attach their own identifiers (e.g., VAMS `groupKey`) without the CMS needing to understand them.

**Rules**:
- Maximum 4 KB per item (items exceeding this will produce a warning and the `context` will be dropped for that item)
- The CMS treats `context` as opaque — it is stored and returned as-is, never parsed or validated
- In the **sync response**, each campaign's `context` is an array of all distinct `context` values from the items that formed that campaign (since multiple items with the same `orderNumber` are grouped into one campaign)
- In the **status response**, each content entry's `context` is the `context` from the item that originally provided that content URL. Warnings (e.g., `dimension_mismatch`) also include the `context` for the affected content, so callers can route warnings to the correct booking
- If no items include `context`, the field is omitted from responses

**Example**: Two items with the same `orderNumber` but different `context` values:

```json
{
    "items": [
        { "orderNumber": "3940", "context": { "groupKey": "abc-123" }, "..." : "..." },
        { "orderNumber": "3940", "context": { "groupKey": "def-456" }, "..." : "..." }
    ]
}
```

The sync response campaign will include both:

```json
{
    "campaign_name": "3940",
            "brand": "TOM FORD",
    "campaign_id": 456,
    "action": "created",
    "context": [
        { "groupKey": "abc-123" },
        { "groupKey": "def-456" }
    ]
}
```

## 1.7. Dry Run

A dry run (`"dry_run": true` on the sync endpoint, or via the dedicated [Dry Run Endpoint](#3-dry-run-endpoint)) performs translation and full classification without making any changes. Unlike a live sync, it returns the detailed list of affected campaigns with their planned actions.

**Dry run response**:

```json
{
    "result": "dry_run",
    "summary": {
        "total_items_received": 4,
        "campaigns_created": 1,
        "campaigns_updated": 1,
        "campaigns_ended": 0,
        "campaigns_deleted": 0,
        "skipped_manual": 0
    },
    "campaigns": [
        {
            "campaign_name": "3940",
            "brand": "TOM FORD",
            "action": "create",
            "sub_campaigns": 2,
            "context": [{ "groupKey": "abc-123" }]
        },
        {
            "campaign_name": "3941",
            "brand": "DIOR",
            "campaign_id": 457,
            "action": "update",
            "sub_campaigns": 1
        },
        {
            "campaign_name": "3942",
            "brand": "CHANEL",
            "campaign_id": 458,
            "action": "end"
        }
    ],
    "warnings": [],
    "errors": []
}
```

**Dry run campaign actions**:

| Action | Description |
|--------|-------------|
| `create` | Would create a new campaign |
| `update` | Would update an existing campaign (includes `campaign_id`) |
| `delete` | Would hard-delete an engine-owned campaign not in payload (never played; includes `campaign_id`) |
| `end` | Would set end dates to now on an engine-owned campaign not in payload (has playback history; includes `campaign_id`) |
| `skipped_manual` | Campaign is tagged as manually managed; would not be modified |

**Note**: `campaign_id` is included for existing campaigns (`update`, `delete`, `end`) but not for `create` (since the campaign doesn't exist yet). The `context` array is included when the incoming items had context values.

---

# 2. Status Endpoint

**Endpoint**: `GET /api/external_import/status` or `POST /api/external_import/status`

**Authentication**: Bearer token (API key).

```
Authorization: Bearer {api_key}
```

Check content download status, encoding progress, and dimension warnings after a sync. Use this to poll for video encoding completion and to verify content dimensions match assigned players.

Query by `content_ids` (filename UUIDs) and/or `campaign_id`. When `campaign_id` is provided, the endpoint automatically resolves the campaign's assigned content and players, and includes dimension mismatch warnings.

## 2.1. Request Format (GET)

**By content IDs**:
```
GET /api/external_import/status?content_ids=4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa,a1b2c3d4-e5f6-7890-abcd-ef1234567890
```

**By campaign ID** (includes dimension warnings):
```
GET /api/external_import/status?campaign_id=456
```

**Both** (content_ids are merged with the campaign's content):
```
GET /api/external_import/status?campaign_id=456&content_ids=4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa
```

## 2.2. Request Format (POST)

```json
{
    "content_ids": [
        "4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa",
        "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
    ],
    "campaign_id": 456
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| content_ids | array | No* | Array of content ID strings (filename UUIDs) |
| campaign_id | integer | No* | Campaign ID from the sync response. Resolves content and players automatically. |

*At least one of `content_ids` or `campaign_id` must be provided.

## 2.3. Response Format

```json
{
    "result": "ok",
    "content_status": [
        {
            "content_id": "4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa",
            "file_content_id": 1234,
            "status": "ready",
            "width": 1920,
            "height": 1080,
            "format": "mp4",
            "context": {
                "groupKey": "abc-123"
            }
        },
        {
            "content_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
            "file_content_id": 1235,
            "status": "encoding_queued",
            "width": 1920,
            "height": 1080,
            "format": "mp4",
            "context": {
                "groupKey": "abc-123"
            }
        }
    ],
    "warnings": [
        {
            "type": "dimension_mismatch",
            "content_id": "4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa",
            "file_content_id": 1234,
            "content_dimensions": "1920x1080",
            "player_id": 567,
            "player_store": "3201DS0061",
            "player_dimensions": "1080x1920",
            "tolerance": 5,
            "context": {
                "groupKey": "abc-123"
            }
        }
    ]
}
```

The `warnings` array is only present when `campaign_id` is provided and dimension mismatches are detected. When querying by `content_ids` only, no dimension check is performed (no player context available).

## 2.4. Content Status Values

| Status | Description |
|--------|-------------|
| `ready` | Content is ready for playback |
| `download_queued` | Content has been registered and is queued for download. The background worker will download, validate, and upload it. |
| `encoding_queued` | Content has been downloaded and uploaded. Video is queued for encoding to h264 baseline. |
| `encoding_error` | Encoding failed (includes timeouts) |
| `not_found` | Content ID not found in the system (not yet downloaded or invalid) |

## 2.5. Warnings

### Dimension Mismatch

Returned when `campaign_id` is provided. Content dimensions (aspect ratio) do not match an assigned player's dimensions, even accounting for the player's configured tolerance. Content outside the tolerance will not play — the player will skip the campaign during playback until content with a matching aspect ratio is provided.

```json
{
    "type": "dimension_mismatch",
    "content_id": "4vwhq2gg-zju2-lqlm-m1hn-e93xwgqrp9aa",
    "file_content_id": 1234,
    "content_dimensions": "1920x1080",
    "player_id": 567,
    "player_store": "3201DS0061",
    "player_dimensions": "1080x1920",
    "tolerance": 5,
    "context": {
        "groupKey": "abc-123"
    }
}
```

| Field | Description |
|-------|-------------|
| `content_id` | The filename UUID of the content |
| `file_content_id` | Internal CMS content ID |
| `content_dimensions` | Content width x height |
| `player_id` | Internal CMS player ID |
| `player_store` | Player store name for identification |
| `player_dimensions` | Player configured width x height |
| `tolerance` | Aspect ratio tolerance percentage configured for this player |
| `context` | The `context` from the item that provided this content (omitted if none) |

**Action required**: The affected player will skip this campaign during playback. Provide content with a matching aspect ratio to resolve.

---

# 3. Dry Run Endpoint

**Endpoint**: `POST /api/external_import/dry_run`

**Authentication**: Bearer token (API key).

```
Authorization: Bearer {api_key}
```

A convenience endpoint that accepts the same payload format as the [Sync Endpoint](#1-sync-endpoint) and always forces `dry_run: true`. This avoids the need to include the `dry_run` parameter in the request body.

The request and response formats are identical to the sync endpoint with `dry_run: true` — see [1.7. Dry Run](#17-dry-run) for the response format and campaign action values.
