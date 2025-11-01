# REST API Design Guide

---

## 1. **General Principles**

- **Use HTTPS** for all endpoints.
- **RESTful resources**: Use nouns, not verbs.

  - ✅ `/users`
  - ❌ `/getUsers`

- **JSON format** for all requests/responses.
- **UTF-8 encoding**.
- **Version your API** via URL or header.

  - Example: `/v1/users`

- **Idempotency**: GET, PUT, DELETE should be idempotent.
- **Statelessness**: Requests contain all needed info; server does not store session state.

---

## 2. **Resource Naming**

| Convention                        | Example                                                          |
| --------------------------------- | ---------------------------------------------------------------- |
| Use plural nouns                  | `/users`, `/orders`                                              |
| Hierarchical for sub-resources    | `/users/{userId}/orders`                                         |
| No CRUD verbs in paths            | `/users/{id}` not `/deleteUser/{id}`                             |
| Use hyphens (`-`) for readability | `/user-profiles`                                                 |
| Consistent casing                 | **camelCase** for JSON fields, **kebab-case** for paths optional |

---

## 3. **HTTP Methods**

| Method | Usage                   |
| ------ | ----------------------- |
| GET    | Retrieve resource(s)    |
| POST   | Create new resource     |
| PUT    | Update/replace resource |
| PATCH  | Partial update          |
| DELETE | Remove resource         |

**Rules:**

- GET: safe & idempotent
- POST: non-idempotent (creates new resources)
- PUT: idempotent, replace resource entirely
- PATCH: idempotent, partial update

---

## 4. **Request Structure**

- **Headers**:

  - `Authorization: Bearer <token>`
  - `Content-Type: application/json`
  - Optional `Accept: application/json`

- **Body**: JSON object for POST/PATCH/PUT

Example POST body:

```json
{
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "role": "AdminUser"
}
```

---

## 5. **Response Structure**

- All responses should include **top-level `kind` field** and optional `etag`/`id`.
- **Always return JSON**, even for errors.
- **Consistent field naming**: camelCase.

Example success response:

```json
{
  "kind": "example#user",
  "id": "123",
  "etag": "\"abc123\"",
  "name": "Alice Johnson",
  "email": "alice@example.com",
  "role": "AdminUser"
}
```

### 5.1 Lists / Collections

```json
{
  "kind": "example#userList",
  "nextPageToken": "token123",
  "items": [
    {
      "id": "123",
      "name": "Alice Johnson"
    },
    {
      "id": "124",
      "name": "Bob Smith"
    }
  ]
}
```

---

## 6. **Query Parameters / Filtering**

- Use query parameters for filtering, sorting, and pagination.

| Param       | Description          | Example                   |
| ----------- | -------------------- | ------------------------- |
| `pageSize`  | Max results per page | `?pageSize=50`            |
| `pageToken` | Token for next page  | `?pageToken=abc123`       |
| `filter`    | Filtering expression | `?filter=status=active`   |
| `orderBy`   | Sorting              | `?orderBy=createdAt desc` |

**Filters**: use a simple expression like `field=value` or `field>value`.

---

## 7. **Error Handling**

- Use **standard HTTP status codes**.
- Return **structured error object**.

| Status Code | Meaning                    |
| ----------- | -------------------------- |
| 400         | Bad request (invalid data) |
| 401         | Unauthorized               |
| 403         | Forbidden                  |
| 404         | Not found                  |
| 409         | Conflict                   |
| 500         | Internal server error      |

### Error Object Example

```json
{
  "error": {
    "code": 400,
    "message": "Invalid request",
    "errors": [
      {
        "message": "Invalid email format",
        "domain": "global",
        "reason": "invalid"
      }
    ]
  }
}
```

### Partial Failures (Batch)

```json
{
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
        "message": "Permission denied"
      }
    }
  ]
}
```

---

## 8. **Pagination**

- Use `pageSize` and `nextPageToken` for paging through large collections.
- Optional `prevPageToken` for backward navigation.
- Include `totalItems` only if feasible.

Example:

```json
{
  "items": [...],
  "nextPageToken": "abc123",
  "totalItems": 125
}
```

---

## 9. **Sorting & Filtering**

- Query parameters:

  - `filter` → `"status=active AND role=AdminUser"`
  - `orderBy` → `"createdAt desc"`

- Avoid complex SQL-like syntax; keep filters simple.

---

## 10. **Batch Operations**

- **Batch request**: array of sub-requests with unique IDs.
- **Batch response**: array of results per ID.
- **Partial success** allowed.

### Example Batch Request

```json
{
  "items": [
    { "id": "1", "method": "GET", "path": "/users/123" },
    { "id": "2", "method": "DELETE", "path": "/users/124" }
  ]
}
```

### Example Batch Response

```json
{
  "items": [
    {
      "id": "1",
      "status": "success",
      "body": { "id": "123", "name": "Alice" }
    },
    {
      "id": "2",
      "status": "error",
      "error": { "code": 403, "message": "Forbidden" }
    }
  ]
}
```

---

## 11. **Authentication & Authorization**

- Use **OAuth 2.0** Bearer tokens.
- Include `Authorization` header in all requests:

  ```
  Authorization: Bearer <token>
  ```

- For service accounts: use JWT tokens.
- Avoid API keys in URLs.

---

## 12. **Idempotency**

- PUT, DELETE, GET are **idempotent**.
- POST can support idempotency via `Idempotency-Key` header:

  ```
  Idempotency-Key: <unique-key>
  ```

---

## 13. **Versioning**

- Recommended: **URL versioning** `/v1/` or `/v2/`.
- Optional: `Accept` header versioning.
- Avoid breaking changes in minor releases.

---

## 14. **Field Conventions**

| Aspect          | Recommendation                        |
| --------------- | ------------------------------------- |
| Naming          | camelCase                             |
| Optional fields | Omit when not set                     |
| Null            | Use only when necessary               |
| Enums           | UpperCamelCase strings                |
| IDs             | Strings, especially for large numbers |

---

## 15. **Example Endpoints**

### 15.1 GET User

```http
GET /v1/users/123
```

Response:

```json
{
  "kind": "example#user",
  "id": "123",
  "name": "Alice Johnson",
  "email": "alice@example.com"
}
```

### 15.2 List Users (Paged)

```http
GET /v1/users?pageSize=50&pageToken=abc123
```

Response:

```json
{
  "kind": "example#userList",
  "nextPageToken": "def456",
  "items": [
    { "id": "123", "name": "Alice Johnson" },
    { "id": "124", "name": "Bob Smith" }
  ]
}
```

### 15.3 Create User

```http
POST /v1/users
```

Request body:

```json
{
  "name": "Charlie Brown",
  "email": "charlie@example.com",
  "role": "ViewerUser"
}
```

Response:

```json
{
  "kind": "example#user",
  "id": "125",
  "name": "Charlie Brown",
  "email": "charlie@example.com"
}
```

---
