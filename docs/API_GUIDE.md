# TaskFlow API Documentation Guide

**Version:** 1.0.0 | **Status:** Draft | **Last Updated:** January 2025

**Related Jira Tasks:** BHOGAR-206, BHOGAR-207, BHOGAR-208, BHOGAR-209

**GitHub Repository:** https://github.com/bhogarinc/taskflow-api

---

## Table of Contents

1. [Overview](#overview)
2. [Base URLs](#base-urls)
3. [Authentication](#authentication)
4. [API Endpoints](#endpoints)
5. [Data Schemas](#schemas)
6. [Pagination](#pagination)
7. [Error Handling](#errors)
8. [Rate Limiting](#rate-limiting)
9. [API Versioning](#versioning)

---

## 1. Overview

TaskFlow is a lightweight task management REST API designed for teams. It provides comprehensive functionality for task tracking, team collaboration, and project management.

### Key Features

- **Task Management:** Full CRUD operations with filtering, sorting, and search
- **Team Collaboration:** Boards, teams, and member management
- **Authentication:** JWT-based auth with refresh tokens
- **Comments:** Task-level discussions
- **Flexible Organization:** Labels, priorities, status workflows

### Tech Stack

| Component | Technology |
|-----------|------------|
| Framework | FastAPI (Python) |
| Database | PostgreSQL |
| Cache | Redis |
| Auth | JWT + OAuth2 |
| Documentation | OpenAPI 3.1 |

---

## 2. Base URLs

| Environment | Base URL |
|-------------|----------|
| Production | `https://api.taskflow.io/v1` |
| Staging | `https://staging-api.taskflow.io/v1` |
| Development | `http://localhost:8000/v1` |

---

## 3. Authentication

### 3.1 JWT Bearer Token

Primary authentication method using JWT access tokens:

```
Authorization: Bearer <access_token>
```

#### Token Lifecycle

| Token Type | Expiry | Usage |
|------------|--------|-------|
| Access Token | 15 minutes | API requests |
| Refresh Token | 7 days | Obtain new access token |

### 3.2 OAuth 2.0

For third-party integrations:
- **Authorization URL:** `https://auth.taskflow.io/oauth/authorize`
- **Token URL:** `https://auth.taskflow.io/oauth/token`
- **Scopes:** `read`, `write`, `admin`

### 3.3 Authentication Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/auth/register` | Register new user |
| POST | `/auth/login` | Login and get tokens |
| POST | `/auth/refresh` | Refresh access token |
| POST | `/auth/logout` | Invalidate token |
| POST | `/auth/forgot-password` | Request password reset |

---

## 4. API Endpoints

### 4.1 Users

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/users/me` | Get current user profile |
| PATCH | `/users/me` | Update user profile |
| PUT | `/users/me/password` | Change password |
| GET | `/users/{userId}` | Get public user profile |

### 4.2 Tasks

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks` | List tasks (paginated, filterable) |
| POST | `/tasks` | Create new task |
| GET | `/tasks/{id}` | Get task details |
| PATCH | `/tasks/{id}` | Update task |
| DELETE | `/tasks/{id}` | Delete task (soft) |
| POST | `/tasks/{id}/assign` | Assign task to user |
| PATCH | `/tasks/{id}/status` | Update task status |

#### Task Filtering Parameters

- `status` - Filter by task status
- `priority` - Filter by priority level
- `assignee_id` - Filter by assignee
- `board_id` - Filter by board
- `due_before` / `due_after` - Date range filter
- `search` - Full-text search (min 2 chars)

### 4.3 Boards

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/boards` | List accessible boards |
| POST | `/boards` | Create board |
| GET | `/boards/{id}` | Get board with stats |
| PATCH | `/boards/{id}` | Update board |
| DELETE | `/boards/{id}` | Delete board |
| GET | `/boards/{id}/tasks` | Get tasks grouped by status |

### 4.4 Teams

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/teams` | List user's teams |
| POST | `/teams` | Create team |
| GET | `/teams/{id}` | Get team details |
| GET | `/teams/{id}/members` | List team members |
| POST | `/teams/{id}/members` | Add member by email |
| DELETE | `/teams/{id}/members/{userId}` | Remove member |

### 4.5 Comments

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/tasks/{id}/comments` | List task comments |
| POST | `/tasks/{id}/comments` | Add comment |
| DELETE | `/comments/{id}` | Delete comment |

---

## 5. Data Schemas

### 5.1 Task Status Values

```
backlog → todo → in_progress → in_review → done
                    ↓
                cancelled
```

### 5.2 Priority Levels

- `lowest`
- `low`
- `medium` (default)
- `high`
- `highest`

### 5.3 Team Roles

| Role | Permissions |
|------|-------------|
| owner | Full control, can delete team |
| admin | Manage members, boards, tasks |
| member | Create/edit tasks, view all |
| viewer | Read-only access |

---

## 6. Pagination

All list endpoints use cursor-based pagination with the following parameters:

### Request Parameters

| Parameter | Type | Default | Max |
|-----------|------|---------|-----|
| page | integer | 1 | - |
| per_page | integer | 20 | 100 |

### Response Format

```json
{
  "items": [...],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 150,
    "total_pages": 8,
    "has_next": true,
    "has_prev": false
  }
}
```

---

## 7. Error Handling

### 7.1 Error Response Format

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Request validation failed",
  "details": [
    {
      "field": "email",
      "message": "Invalid email format"
    }
  ]
}
```

### 7.2 HTTP Status Codes

| Code | Meaning | Usage |
|------|---------|-------|
| 200 | OK | Successful GET/PUT/PATCH |
| 201 | Created | Successful POST |
| 204 | No Content | Successful DELETE |
| 400 | Bad Request | Invalid request format |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Resource already exists |
| 422 | Unprocessable | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Server Error | Internal error |

---

## 8. Rate Limiting

### 8.1 Rate Limit Tiers

| Endpoint Type | Limit | Window |
|---------------|-------|--------|
| Authentication | 5 requests | 1 minute |
| General API | 100 requests | 1 minute |
| Write Operations | 50 requests | 1 minute |
| Bulk Operations | 10 requests | 1 minute |

### 8.2 Rate Limit Headers

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Request limit per window |
| `X-RateLimit-Remaining` | Remaining requests |
| `X-RateLimit-Reset` | Reset timestamp (Unix) |

### 8.3 Rate Limit Response (429)

```json
{
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "API rate limit exceeded. Try again in 60 seconds."
}
```

---

## 9. API Versioning

### 9.1 Version Strategy

TaskFlow API uses URL path versioning: `/v1/`, `/v2/`, etc.

### 9.2 Version Lifecycle

| Status | Support | Sunset Notice |
|--------|---------|---------------|
| Current | Full support | - |
| Deprecated | Bug fixes only | 6 months |
| Sunset | No support | 12 months |

### 9.3 Breaking Changes Policy

Breaking changes only in major version updates. Minor versions add features; patch versions fix bugs.

---

## Appendix: OpenAPI Specification

The complete OpenAPI 3.1 specification is available in the GitHub repository:

[View OpenAPI Specification](https://github.com/bhogarinc/taskflow-api/blob/main/docs/openapi.yaml)

### Generating Client SDKs

```bash
# Using OpenAPI Generator
openapi-generator generate -i openapi.yaml -g python -o taskflow-python-sdk

# Using Swagger Codegen
swagger-codegen generate -i openapi.yaml -l typescript-angular -o taskflow-ts-sdk
```

---

*Document maintained by the TaskFlow Engineering Team. For questions, contact api@taskflow.io*
