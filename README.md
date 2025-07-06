# API Documentation
Source code can be found here: https://github.com/leggo15/Echosu/blob/main/echo/views/api.py
# Table of Contents

- [Authentication](#authentication)
  - [Obtaining an API Token](#obtaining-an-api-token)
    - [Generate Token](#generate-token)
    - [Store the Token Securely](#store-the-token-securely)
  - [Using the API Token](#using-the-api-token)
- [API Endpoints](#api-endpoints)
  - [1. Tags for Beatmaps](#1-tags-for-beatmaps)
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

### 1. Tags for Beatmaps

Retrieve tags associated with a specific beatmap or a batch of beatmaps.

- **URL for batch:** `/api/beatmaps/tags/`
- **URL for single map:** `/api/beatmaps/<str:beatmap_id>/tags/`
- **Methods:** `GET`
- **Authentication:** Required (Token-Based)
- **Parameters:**
  - `beatmap_id` (optional): The ID of a specific beatmap.
    - **Type:** string
    - **Usage:** To fetch tags for a single beatmap.
  - `batch_size` (optional): Number of beatmaps to fetch in a batch.
    - **Type:** integer
    - **Default:** 500
    - **Usage:** For batch retrieval of beatmaps with tags.
  - `offset` (optional): The starting point for batch retrieval.
    - **Type:** integer
    - **Default:** 0
    - **Usage:** For paginating through batches.
- **Responses:**
  - **200 OK:** Successfully retrieved tags.

```json
[
    {
        "beatmap_id": "1244293",
        "title": "Test Beatmap",
        "artist": "Test Artist",
        "tags": [
            {
                "tag": "aim",
                "count": 10
            },
            {
                "tag": "stream",
                "count": 5
            }
        ]
    }
]
```

- **404 Not Found:** Beatmap(s) not found.

```json
{
    "detail": "Beatmap not found."
}
```

or

```json
{
    "detail": "No beatmaps found."
}
```

### 2. Beatmap ViewSet

Interact with Beatmap data. Provides read-only operations and filtering.

- **Base URL:** `/api/beatmaps/`
- **Methods:** `GET`
- **Authentication:** Required (Token-Based)
- **Actions:**
  - **List Beatmaps**
    - **URL:** `/api/beatmaps/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all beatmaps.
  - **Retrieve a Beatmap**
    - **URL:** `/api/beatmaps/{id}/`
    - **Method:** `GET`
    - **Description:** Retrieve details of a specific beatmap by its ID.
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
- **Authentication:** Required (Token-Based)
- **Actions:**
  - **List Tags**
    - **URL:** `/api/tags/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all tags.
  - **Retrieve a Tag**
    - **URL:** `/api/tags/{id}/`
    - **Method:** `GET`
    - **Description:** Retrieve details of a specific tag by its ID.

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
- **Authentication:** Required (Token-Based)
- **Actions:**
  - **List Tag Applications**
    - **URL:** `/api/tag-applications/`
    - **Method:** `GET`
    - **Description:** Retrieve a list of all tag applications.
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
- **Authentication:** Required (Token-Based)
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

### 1. Fetch Tags for a Specific Beatmap

**Endpoint:**

```bash
GET /api/beatmaps/2897724/tags/
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
      "beatmap_id":"2897724",
      "title":"Last Exit To Brooklyn",
      "artist":"Modern Talking",
      "tags":[
         {
            "tag":"aim",
            "count":4
         }
         {
            "tag":"ohio",
            "count":1
         }
      ]
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

API_TOKEN = 'NoXLUxiMVGRRZTJKbk54TlarrnoMgtzl99qqwX4dTtghBjYlQp8UKw8R1iGOFY1x'
BASE_URL = 'https://www.echosu.com'

headers = {
    'Authorization': f'Token {API_TOKEN}',
    'Content-Type': 'application/json',
}

response = requests.get(f'{BASE_URL}/api/beatmaps/tags/', headers=headers)

if response.status_code == 200:
    beatmaps = response.json()
    print(beatmaps)
```

Script applying tags to a map (and formating the print for readability):

```python
import requests
import json


API_TOKEN = 'NoXLUxiMVGRRZTJKbk54TlarrnoMgtzl99qqwX4dTtghBjYlQp8UKw8R1iGOFY1x'
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
