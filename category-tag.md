# Category & Tag

The onQ CMS API allows users to create, manage and assign categories and tags to objects in the system (players, playlists, assets, campaigns).

# TOC
- [Category & Tag](#category--tag)
- [TOC](#toc)
- [1. Category](#1-category)
  - [1.1. Fetch Category](#11-fetch-category)
  - [1.2. Create a new category](#12-create-a-new-category)
  - [1.3. Edit category](#13-edit-category)
  - [1.4. Delete Category](#14-delete-category)
- [2. Tag](#2-tag)
  - [2.1. Fetch Tag](#21-fetch-tag)
  - [2.2. Create a new Tag](#22-create-a-new-tag)
  - [2.3. Edit Tag](#23-edit-tag)
  - [2.4. Delete Tag](#24-delete-tag)



[Back to Home](README.md)

# 1. Category

Base URL: https://onqcms.com/api/smart_category

## 1.1. Fetch Category

Endpoint: https://onqcms.com/api/smart_category/fetch

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| category_id         | integer           | False      | Category ID              |
| category_key        | string            | False     | Category key              |
| category_val        | string            | False     | Category Value              |
| object_type         | string            | False      | 'player', 'playlist', 'asset', 'campaign' |
| object_id           | integer           | False     | Object ID              |

`Example request:`

```json
{
    "category_key": "region"
}
```

`Success Response:`

```json
[
    {
        "category_id": "189",
        "category_key": "region",
        "category_value": "",
        "updated_at": "2025-01-03 16:25:36",
        "created_at": "2025-01-03 16:25:36"
    },
    {
        "category_id": "197",
        "category_key": "region",
        "category_value": "region",
        "updated_at": "2025-01-03 16:30:16",
        "created_at": "2025-01-03 16:30:16"
    },
    {
        "category_id": "237",
        "category_key": "region",
        "category_value": "apac",
        "updated_at": "2025-01-03 16:33:10",
        "created_at": "2025-01-03 16:33:10"
    },
    {
        "category_id": "238",
        "category_key": "region",
        "category_value": "melbourne",
        "updated_at": "2025-01-03 16:33:10",
        "created_at": "2025-01-03 16:33:10"
    },
    {
        "category_id": "239",
        "category_key": "region",
        "category_value": "victoria",
        "updated_at": "2025-01-03 16:33:10",
        "created_at": "2025-01-03 16:33:10"
    }
]
```

Fetch all categories (empty body):

```json
{}
```
[Top ↑](#category--tag)


## 1.2. Create a new category

Endpoint: https://onqcms.com/api/smart_category/create

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| categories          | array             | True      | Category key and array of values |
| object_type         | string            | False      | 'player', 'playlist', 'asset', 'campaign'. This must be supplied when object_id exist. |
| object_id           | integer           | False      | array of integers or strings. This must be supplied when object_type exist.  |


`Example request:`

Create categories only:
```json
{
    "categories": [
        { "key": "Client", "value": ["DJ-2"] },
        { "key": "Type",   "value": ["Marketing"] },
        { "key": "Region", "value": ["APAC", "MELBOURNE", "VICTORIA"] }
    ]
}
```

Create and assign to campaigns:
```json
{
    "categories": [
        { "key": "Client", "value": ["DJ-2"] },
        { "key": "Type",   "value": ["Marketing"] },
        { "key": "Region", "value": ["APAC", "MELBOURNE", "VICTORIA"] }
    ],
    "object_type": "campaign",
    "object_id": [111, 222, 333]
}
```

Create and assign to players:
```json
{
    "categories": [
        { "key": "Client", "value": ["DJ-2"] },
        { "key": "Type",   "value": ["Marketing"] },
        { "key": "Region", "value": ["APAC", "MELBOURNE", "VICTORIA"] }
    ],
    "object_type": "player",
    "object_id": [112, 333, 222]
}
```

Create and assign to playlists:
```json
{
    "categories": [
        { "key": "Client", "value": ["DJ-2"] },
        { "key": "Type",   "value": ["Marketing"] },
        { "key": "Region", "value": ["APAC", "MELBOURNE", "VICTORIA"] }
    ],
    "object_type": "playlist",
    "object_id": [111, 222, 333]
}
```

Create and assign to assets:
```json
{
    "categories": [
        { "key": "Client", "value": ["DJ-2"] },
        { "key": "Type",   "value": ["Marketing"] },
        { "key": "Region", "value": ["APAC", "MELBOURNE", "VICTORIA"] }
    ],
    "object_type": "asset",
    "object_id": [111, 222, 333]
}
```

`Success Response:`

```json
{
  "category_id": 162
}
```
[Top ↑](#category--tag)

## 1.3. Edit category

Endpoint: https://onqcms.com/api/smart_category/create

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| category_key        | string            | True      | Category key name         |
| category_value      | string            | True      | Category key's value      |
| object_type         | string            | True      | 'player', 'playlist', 'asset', 'campaign' |
| object_id           | integer           | True      | array of integers or strings |


`Example request:`

```json
{
    "category_key": "Client",
    "category_value": "DJ-2",
    "object_type": "campaign",
    "object_id": [111, 222, 333]
}
```

```json
{
    "category_key": "Brand",
    "category_value": "Hugo Boss",
    "object_type": "player",
    "object_id": [111, 222, 333]
}
```

```json
{
    "category_key": "Brand",
    "category_value": "Hugo Boss",
    "object_type": "playlist",
    "object_id": [111, 222, 333]
}
```

```json
{
    "category_key": "Brand",
    "category_value": "Hugo Boss",
    "object_type": "asset",
    "object_id": [111, 222, 333]
}
```

`Success Response:`

```json
{
}
```
## 1.4. Delete Category

Endpoint: https://onqcms.com/api/smart_category/delete


**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| category_id         | integer           | True      | Category ID               |
| object_type         | string            | True      | 'player', 'playlist', 'asset', 'campaign' |

`Example request:`

```json
{
    "category_id": 189,
    "object_type": "player"
}
```
[Top ↑](#category--tag)

# 2. Tag
Base URL: https://onqcms.com/api/smart_tag

## 2.1. Fetch Tag
Endpoint: https://onqcms.com/api/smart_tag/fetch

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| tag_id              | integer           | False     | Tag ID              |
| tag_name            | string            | False     | Tag name              |
| object_type         | string            | False     | 'player', 'playlist', 'asset', 'campaign' |
| object_id           | integer           | False     | Object ID              |

`Example request:`

```json
{
    "tag_id": 7,
    "object_type": "player"
}
```

```json
{
    "tag_id": 7
}
```

Fetch all tags (empty body):
```json
{}
```
[Top ↑](#category--tag)

## 2.2. Create a new Tag
Endpoint: https://onqcms.com/api/smart_tag/create

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| tag_name            | string            | True      | Tag name         |
| object_type         | string            | False      | 'player', 'playlist', 'asset', 'campaign'. This must be supplied when object_id exist. |
| object_id           | Array             | False      | array of strings. This must be supplied when object_type exist. |


`Example request:`

Create tag only:
```json
{
    "tag_name": "Hugo"
}
```

Create and assign to players:
```json
{
    "tag_name": "Hugo",
    "object_type": "player",
    "object_id": ["testaaa001", "testaaa002", "offset-test1"]
}
```

Create and assign to playlists:
```json
{
    "tag_name": "Hugo",
    "object_type": "playlist",
    "object_id": ["testaaa001", "testaaa002", "offset-test1"]
}
```

Create and assign to assets:
```json
{
    "tag_name": "Hugo",
    "object_type": "asset",
    "object_id": ["testaaa001", "testaaa002", "offset-test1"]
}
```

`Success Response:`

```json
{
  "tag_id": 5
}
```
[Top ↑](#category--tag)

## 2.3. Edit Tag
Endpoint: https://onqcms.com/api/smart_tag/create

This is to edit tag allocation.

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| tag_id              | integer/array     | True      | Tag id in either integer or array     |
| object_type         | string            | True      | 'player', 'playlist', 'asset', 'campaign' |
| object_id           | array             | True      | array of integers or strings |


`Example request:`

```json
{
    "tag_id": [144, 145],
    "object_type": "player",
    "object_id": [307]
}
```

`Success Response:`

```json
{
    "removed": [
        [3, "146", "campaign", 307]
    ],
    "added": [
        [3, 144, "campaign", 307],
        [3, 145, "campaign", 307]
    ]
}
```
## 2.4. Delete Tag
Endpoint: https://onqcms.com/api/smart_tag/delete

**Request Parameters**

| Parameter           | Type              | Required  | Description               |
|---------------------|-------------------|-----------|---------------------------|
| tag_id              | integer           | True      | Tag ID               |
| object_type         | string            | True      | 'player', 'playlist', 'asset', 'campaign' |

`Example request:`

```json
{
    "tag_id": 7,
    "object_type": "player"
}
```
[Top ↑](#category--tag)

[Back to Home](README.md)
