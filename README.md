# API Documentation

Source code can be found here: https://github.com/leggo15/Echosu/blob/main/echo/views/api.py

# Table of Contents

- [Authentication](#authentication)
  - [Obtaining an API Token](#obtaining-an-api-token)
    - [Generate Token](#generate-token)
    - [Store the Token Securely](#store-the-token-securely)
  - [Using the API Token](#using-the-api-token)
- [API Endpoints](#api-endpoints)
  - [1. Tag Applications for a Beatmap (with includes)](#1-tag-applications-for-a-beatmap-with-includes)
  - [2. Beatmap ViewSet](#2-beatmap-viewset)
  - [3. Tag ViewSet](#3-tag-viewset)
  - [4. Tag Application ViewSet](#4-tag-application-viewset)
  - [5. User Profile ViewSet](#5-user-profile-viewset)
- [Examples](#examples)
  - [1. Fetch Tags for a Specific Beatmap](#1-fetch-tags-for-a-specific-beatmap)
  - [2. Toggle Tags on a Beatmap](#2-toggle-tags-on-a-beatmap)
  - [3. Fetch Filtered Beatmaps](#3-fetch-filtered-beatmaps)
- [Example Scripts](#example-scripts)
- [Error Handling](#error-handling)

## API Token

The API uses Token-Based Authentication to secure endpoints. To use the API, you must include a valid token in the request headers.

#### Generate Token:

- Login to echosu.com (through the osu authentication api)
- Click your profile picture and go to settings.
- Click generate API token. You can generate as many tokens as you want, but only the latest one will be active.

#### Store the Token Securely:

- Once generated, copy the token key.
- Store it securely; it won't be displayed again.

### Using the API Token

Include the token in the `Authorization` header of your API requests as follows:

```python
headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}
```

## API Endpoints

### 1. Tag Applications for a Beatmap

All tag data for a beatmap is exposed via the tag applications endpoint, filtered by `beatmap_id`. `include` flags allow attaching derived data per tag.

- **URL:** `/api/tag-applications/?beatmap_id=<beatmap_id>&include=tag_counts,tag_timestamps,predicted_tags&user=me`
- **Methods:** `GET`
- **Authentication:** Required (Token or logged-in session)
- **Query Parameters:**

  - `beatmap_id` (required): string, the beatmap ID to fetch applications for
  - `include` (optional): comma-separated extras. Supported:
    - `tag_counts`: attach `count` to each `tag` (applications per tag on this beatmap; excludes predictions by default)
    - `tag_timestamps`: attach `consensus_intervals` to each `tag` (aggregated intervals across users)
    - `predicted_tags`: include predicted tags
  - `user` (optional): if `me`, attach `user_intervals` for the current user under each `tag`
- **Response (lite entries with per-tag extras):**

```json
[
  {
    "id": 11155,
    "user": {"id": 1, "username": "testuser"},
    "tag": {
      "id": 178,
      "name": "aim",
      "count": 1,
      "consensus_intervals": [[83.18, 93.75], [127.34, 135.47]],
      "user_intervals": [[90.0, 92.0]]
    },
    "created_at": "2025-08-13T19:46:56.480951Z"
  }
]
```

### 2. Beatmap ViewSet

Interact with Beatmap data. Provides read-only operations and filtering.

- **Base URL:** `/api/beatmaps/`
- **Methods:** `GET`
- **Authentication:** Required (Token or logged-in session)
- **Actions:**
  - **List Beatmaps**
    - **URL:** `/api/beatmaps/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all beatmaps.
  - **Retrieve a Beatmap**
    - **URL:** `/api/beatmaps/{beatmap_id}/`
    - **Method:** `GET`
    - **Description:** Retrieve details of a specific beatmap by its beatmap ID.
  - **Filtered Beatmaps**
    - **URL:** `/api/beatmaps/filtered/`
    - **Method:** `GET`
    - **Description:** Retrieve beatmaps filtered by a query string.
    - **Parameters:**
      - `query` (required): The search term to filter beatmaps by title.
        - **Type:** string
        - **Example:** `/api/beatmaps/filtered/?query=Test`

```json
{
    "id": 1,
    "beatmap_id": "1244293",
    "title": "Test Beatmap",
    "artist": "Test Artist",
    "tags": [
        {
            "id": 1,
            "name": "aim"
        }
    ]
}
```

### 3. Tag ViewSet

Interact with Tag data. Provides standard read-only operations.

- **Base URL:** `/api/tags/`
- **Methods:** `GET`
- **Authentication:** Required (Token or logged-in session)
- **Actions:**
  - **List Tags**
    - **URL:** `/api/tags/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all tags.
  - **Retrieve a Tag**
    - **URL:** `/api/tags/{id}/`
    - **Method:** `GET`
    - **Description:** Retrieve specific tag by its ID.

```json
{
    "id": 1,
    "name": "aim"
}
```

### 4. Tag Application ViewSet

Manage tag applications to beatmaps. Provides standard read-only operations, and a custom action to toggle tags on given beatmap ID's.

- **Base URL:** `/api/tag-applications/`
- **Methods:** `GET`
- **Authentication:** Required (Token or logged-in session)
- **Actions:**
  - **List Tag Applications**
    - **URL:** `/api/tag-applications/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of tag applications. When filtered with `?beatmap_id=<id>`, returns a lite representation and supports `include=tag_counts,tag_timestamps` and `user=me` to attach derived data to each `tag`.
  - **Retrieve a Tag Application**
    - **URL:** `/api/tag-applications/{id}/`
    - **Method:** `GET`
    - **Description:** Retrieve details of a specific tag application by its ID.
  - **Toggle Tags**
    - **URL:** `/api/tag-applications/toggle/`
    - **Method:** `POST`
    - **Description:** Toggle multiple tags for a specific beatmap. Applies the tag if not already applied by the user, or removes it if already applied. If the beatmap does not exist in the database, it fetches it from the osu! API and adds it to the database before toggling the tags.
    - **Request Payload:**
      - **Content-Type:** `application/json`
      - **Body:**

```json
{
    "beatmap_id": "2897724",
    "tags": ["aim", "streams"]
}
```

```json
{
    "status": "success",
    "results": [
        {
            "tag": "aim",
            "action": "applied"
        },
        {
            "tag": "streams",
            "action": "removed"
        }
    ]
}
```

```json
{
    "beatmap_id": ["2897724 isn't a valid beatmap ID."]
}
```

```json
{
    "tags": [
        "Tags must be 1-25 characters long and can only contain letters, numbers, spaces, hyphens, and underscores."
    ]
}
```

### 5. User Profile ViewSet

Manage user profiles. Provides standard read-only operations.

- **Base URL:** `/api/user-profiles/`
- **Methods:** `GET`
- **Authentication:** Required (Token or logged-in session)
- **Actions:**
  - **List User Profiles**
    - **URL:** `/api/user-profiles/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all user profiles.
  - **Retrieve a User Profile**
    - **URL:** `/api/user-profiles/{id}/`
    - **Method:** `GET`
    - **Description:** Retrieve details of a specific user profile by its ID.

```json
{
    "id": 1,
    "user": {
        "id": 1,
        "username": "testuser"
    },
    "osu_id": "123456",
    "profile_pic_url": "https://example.com/profile.jpg"
}
```

## Examples

### 1. Fetch Tag Data for a Specific Beatmap

**Endpoint:**

```bash
GET /api/tag-applications/?beatmap_id=2897724&include=tag_counts,tag_timestamps
```

**Headers:**

```python
headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}
```

**Response (excerpt):**

```json
[
  {
    "id": 11155,
    "user": {"id": 1, "username": "testuser"},
    "tag": {
      "id": 3,
      "name": "aim",
      "count": 4,
      "consensus_intervals": [[12.3, 25.0]]
    },
    "created_at": "2025-08-13T19:46:56.480951Z"
  }
]
```

### 2. Toggle Tags on a Beatmap

**Endpoint:**

```bash
POST /api/tag-applications/toggle/
```

**Headers:**

```python
headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}
```

**Request Body:**

```json
{
    "beatmap_id": "2897724",
    "tags": ["aim", "farm", "diff spike"]
}
```

**Response:**

```json
{
    "status": "success",
    "results": [
        {
            "tag": "aim",
            "action": "applied"
        },
        {
            "tag": "farm",
            "action": "removed"
        },
        {
            "tag": "diff spike",
            "action": "applied"
        }
    ]
}
```

**Explanation:**

- **action:** Indicates whether the tag was applied, removed, or if an error occurred.
- If the beatmap does not exist in the database, the API will attempt to fetch it from the osu! API, add it to the database, and then proceed to toggle the tags.

### 3. Fetch Filtered Beatmaps

**Endpoint:**

```bash
GET /api/beatmaps/filtered/?query=aim
```

**Headers:**

```python
headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}
```

**Response:**

```json
[
    {
        "id": 1,
        "beatmap_id":"2897724",
        "title":"Last Exit To Brooklyn",
        "artist":"Modern Talking",
        "tags": [
            {
                "id": 1,
                "name": "aim"
            },
            {
                "tag": "diff spike",
                "action": "applied"
            }
        ]
    }
]
```

## Example Scripts

Basic script where any of the endpoint URL's can be used:

```python
import requests
import json

API_TOKEN = 'YOUR_TOKEN'
BASE_URL = 'https://www.echosu.com'

headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}

resp = requests.get(f'{BASE_URL}/api/tag-applications/', params={
    'beatmap_id': '2897724',
    'include': 'tag_counts,tag_timestamps',
}, headers=headers)

if resp.status_code == 200:
    apps = resp.json()
    # Derive unique tag counts
    counts = {}
    for item in apps:
        t = item.get('tag') or {}
        if not t: continue
        key = (t.get('id'), t.get('name'))
        counts[key] = t.get('count', 0)
    print(json.dumps({str(k): v for k, v in counts.items()}, indent=2))
```

Script applying tags to a map (and formating the print for readability):

```python
import requests
import json


API_TOKEN = 'YOUR_TOKEN'
BASE_URL = 'https://www.echosu.com'

headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}

def toggle_tags(beatmap_id, tags):

    url = f'{BASE_URL}/api/tag-applications/toggle/'
  
    payload = {
        'beatmap_id': beatmap_id,
        'tags': tags, 
    }
  
    try:
        response = requests.post(url, headers=headers, data=json.dumps(payload))
  
        if response.status_code == 200:
            return response.json()
        else:
            print(f'Failed to toggle tags. Status code: {response.status_code}')
            print(f'Response: {response.text}')
            return None
    except requests.exceptions.RequestException as e:
        print(f'An error occurred: {e}')
        return None

if __name__ == '__main__':
    beatmap_id = '2897724'
    tags_to_toggle = ['aim', 'jumps', 'farm']
  
    result = toggle_tags(beatmap_id, tags_to_toggle)
  
    if result:
        print(json.dumps(result, indent=4))

```

## Error Handling

The API provides meaningful error messages and HTTP status codes to help understand what went wrong. (most of the time)

### Common Error Responses:

- **400 Bad Request:**

  - **Cause:** Invalid input data, such as non-conforming tag formats or missing required fields.
- **404 Not Found:**

  - **Cause:** Requested resource does not exist
- **500 Internal Server Error:**

  - **Cause:** Unexpected server-side errors.
