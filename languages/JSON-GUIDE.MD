# JSON Guide

## 1. General Guidelines

- **Use JSON objects** (not arrays) for collections, unless ordering is essential.
- **No comments** in JSON.
- **UTF-8 encoding** for all data.
- **Indentation:** 2 spaces.
- **Property names:** Double quotes (`"`) around all keys.
- **No trailing commas**.
- **Avoid nulls unless necessary**; prefer omitting optional fields.

Example:

```json
{
  "name": "John Doe",
  "id": 123
}
```

---

## 2. Property Naming Conventions

- **camelCase** for all JSON properties.
- Treat acronyms as words: `"httpUrl"`, not `"HTTPURL"`.
- Use **English words** or abbreviations.
- Plurals for collections: `"items"`, `"errors"`.
- Nested objects follow the same convention.

---

## 3. Property Values

- **Strings:** `"value"`; escape as needed (`\"`, `\\`, `\n`).
- **Numbers:** JSON numbers (no leading zeros). Use strings for very large integers.
- **Booleans:** `true` / `false`.
- **Nulls:** `null` for missing/optional values.

---

## 4. Field Types and Formats

| Type        | Format                         | Example                              |
| ----------- | ------------------------------ | ------------------------------------ |
| Date        | RFC 3339 UTC                   | `"2010-02-16T15:09:49Z"`             |
| Duration    | Seconds, optional fraction     | `"3.5s"`                             |
| Coordinates | Array `[longitude, latitude]`  | `[125.6, 10.1]`                      |
| Enum        | UpperCamelCase                 | `"AdminUser"`                        |
| URLs        | Absolute, HTTPS preferred      | `"https://api.example.com/user/123"` |
| IDs         | Strings for very large numbers | `"12345678901234567890"`             |

---

## 5. Reserved Properties

### Top-Level

| Field           | Description                    |
| --------------- | ------------------------------ |
| `kind`          | Type of resource               |
| `etag`          | Version identifier for caching |
| `id`            | Unique identifier              |
| `selfLink`      | URL to the resource itself     |
| `items`         | Array of child items           |
| `nextPageToken` | Token for next page            |
| `prevPageToken` | Token for previous page        |

### Nested

| Field      | Description                               |
| ---------- | ----------------------------------------- |
| `metadata` | Extra information about the resource      |
| `error`    | Detailed error info                       |
| `warnings` | Non-fatal warnings or deprecation notices |

---

## 6. Pagination

- Use `nextPageToken` / `prevPageToken` for paging.
- Clients send token as `pageToken`.

Example:

```json
{
  "kind": "youtube#playlistListResponse",
  "nextPageToken": "CAUQAA",
  "items": [{ "id": "PLBCF2DAC6FFB574DE", "title": "Playlist 1" }]
}
```

---

## 7. Error Representation

Errors are **standardized** and can appear at top-level or nested.

### Basic Error

```json
{
  "error": {
    "code": 400,
    "message": "Invalid request",
    "errors": [
      {
        "message": "Invalid value",
        "domain": "global",
        "reason": "invalid"
      }
    ]
  }
}
```

| Field     | Description                                              |
| --------- | -------------------------------------------------------- |
| `code`    | HTTP status code                                         |
| `message` | Human-readable description                               |
| `errors`  | Array of detailed issues (`message`, `domain`, `reason`) |

### Partial Failures

When some items succeed and some fail:

```json
{
  "kind": "api#batchResponse",
  "status": "partial",
  "items": [
    {
      "id": "1",
      "status": "success"
    },
    {
      "id": "2",
      "status": "error",
      "error": {
        "code": 403,
        "message": "Permission denied",
        "errors": [
          {
            "message": "Permission denied",
            "domain": "global",
            "reason": "forbidden"
          }
        ]
      }
    }
  ]
}
```

- `status`: `"success" | "partial" | "failure"`
- Individual items include success or error.

---

## 8. Batch Requests and Responses

### Batch Request

- Use `items` array for each request.
- Each item has `id` (unique client-assigned identifier), `method`, `path`, and optionally `body`.

Example:

```json
{
  "kind": "api#batchRequest",
  "items": [
    {
      "id": "1",
      "method": "GET",
      "path": "/v1/users/123"
    },
    {
      "id": "2",
      "method": "POST",
      "path": "/v1/users",
      "body": {
        "name": "Alice"
      }
    }
  ]
}
```

### Batch Response

- Mirrors request IDs.
- Each item shows success/error.

Example:

```json
{
  "kind": "api#batchResponse",
  "items": [
    {
      "id": "1",
      "status": "success",
      "body": {
        "id": "123",
        "name": "John"
      }
    },
    {
      "id": "2",
      "status": "error",
      "error": {
        "code": 400,
        "message": "Invalid data"
      }
    }
  ]
}
```

---

## 9. Ordering of Keys

- Property order **not significant**.
- Recommended for readability:

  1. `kind`
  2. `id`
  3. `etag`
  4. `selfLink`
  5. Core fields (alphabetical)
  6. `items`

---

## 10. Example: YouTube API Resource

```json
{
  "kind": "youtube#video",
  "etag": "\"m2yskBQFythfE4irbTIeOgYYfBU/axL2...\"",
  "id": "Ks-_Mh1QhMc",
  "snippet": {
    "publishedAt": "2012-06-20T22:45:24Z",
    "channelId": "UCBR8-60-B28hp2BmDPdntcQ",
    "title": "Google JSON Style Guide Explained"
  }
}
```

---

## 11. Example: Paged Response

```json
{
  "kind": "calendar#events",
  "etag": "\"1234567890\"",
  "summary": "Events",
  "nextPageToken": "GAXYZ",
  "items": [
    {
      "kind": "calendar#event",
      "id": "event123",
      "summary": "Meeting with team"
    }
  ]
}
```

---

## 12. Example: Partial Failure in Batch

```json
{
  "kind": "api#batchResponse",
  "status": "partial",
  "items": [
    {
      "id": "1",
      "status": "success",
      "body": { "id": "123", "name": "John" }
    },
    {
      "id": "2",
      "status": "error",
      "error": {
        "code": 403,
        "message": "Permission denied",
        "errors": [
          {
            "message": "Permission denied",
            "domain": "global",
            "reason": "forbidden"
          }
        ]
      }
    }
  ]
}
```

---

This covers:

- JSON formatting and naming
- Reserved properties
- Types and formats
- Pagination
- Error handling
- Partial failures
- Batch requests/responses

---
