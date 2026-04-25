# TaskFlow API Rate Limiting & Security

**Version:** 1.0.0 | **Related Tasks:** BHOGAR-209

---

## Overview

This document outlines the rate limiting strategy and security measures for the TaskFlow API.

## Rate Limiting Strategy

### Tiered Rate Limits

| Endpoint Category | Requests | Time Window | Scope |
|-------------------|----------|-------------|-------|
| **Authentication** | 5 | 1 minute | Per IP |
| **General API (Read)** | 100 | 1 minute | Per user |
| **Write Operations** | 50 | 1 minute | Per user |
| **Bulk Operations** | 10 | 1 minute | Per user |
| **Export/Download** | 5 | 1 minute | Per user |

### Endpoint Categories

#### Authentication Endpoints
- `POST /auth/register`
- `POST /auth/login`
- `POST /auth/refresh`
- `POST /auth/forgot-password`

#### Write Operations
- All `POST`, `PUT`, `PATCH`, `DELETE` requests
- Task creation/updates
- Board modifications
- Team member management

#### Bulk Operations
- Batch task imports
- Bulk status updates
- Mass assignment operations

## Response Headers

All API responses include rate limit headers:

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1706200800
X-RateLimit-Policy: 100;w=60
```

### Header Descriptions

| Header | Description | Example |
|--------|-------------|---------|
| `X-RateLimit-Limit` | Maximum requests allowed | `100` |
| `X-RateLimit-Remaining` | Requests remaining in window | `87` |
| `X-RateLimit-Reset` | Unix timestamp when limit resets | `1706200800` |
| `X-RateLimit-Policy` | Rate limit policy descriptor | `100;w=60` |

## Rate Limit Exceeded Response

When rate limit is exceeded, the API returns HTTP 429:

```http
HTTP/1.1 429 Too Many Requests
Content-Type: application/json
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1706200800
Retry-After: 45

{
  "code": "RATE_LIMIT_EXCEEDED",
  "message": "API rate limit exceeded. Try again in 45 seconds.",
  "retry_after": 45
}
```

## Implementation Details

### Redis-Based Rate Limiting

```python
# Rate limiting middleware (conceptual)
import redis
from fastapi import Request, HTTPException

class RateLimiter:
    def __init__(self, redis_client: redis.Redis):
        self.redis = redis_client
        self.limits = {
            'auth': (5, 60),      # 5 requests per 60 seconds
            'read': (100, 60),    # 100 requests per 60 seconds
            'write': (50, 60),    # 50 requests per 60 seconds
            'bulk': (10, 60),     # 10 requests per 60 seconds
        }
    
    async def check_rate_limit(
        self, 
        key: str, 
        category: str
    ) -> dict:
        limit, window = self.limits[category]
        current = self.redis.get(key) or 0
        
        if int(current) >= limit:
            ttl = self.redis.ttl(key)
            raise HTTPException(
                status_code=429,
                detail={
                    "code": "RATE_LIMIT_EXCEEDED",
                    "message": f"Rate limit exceeded. Retry in {ttl} seconds.",
                    "retry_after": ttl
                }
            )
        
        pipe = self.redis.pipeline()
        pipe.incr(key)
        pipe.expire(key, window)
        pipe.execute()
        
        return {
            "limit": limit,
            "remaining": limit - int(current) - 1,
            "reset": int(time.time()) + window
        }
```

### Rate Limit Keys

| Key Pattern | Description |
|-------------|-------------|
| `ratelimit:auth:{ip}` | Authentication attempts per IP |
| `ratelimit:read:{user_id}` | Read requests per user |
| `ratelimit:write:{user_id}` | Write requests per user |
| `ratelimit:bulk:{user_id}` | Bulk operations per user |

## Security Measures

### Authentication Security

1. **Password Requirements**
   - Minimum 8 characters
   - At least one uppercase letter
   - At least one lowercase letter
   - At least one number
   - At least one special character

2. **Token Security**
   - Access tokens: 15-minute expiry
   - Refresh tokens: 7-day expiry
   - Tokens stored in httpOnly cookies
   - Secure flag enabled (HTTPS only)

3. **Failed Login Protection**
   - 5 attempts per IP per minute
   - Account lockout after 10 failed attempts
   - Lockout duration: 30 minutes

### API Security Headers

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Referrer-Policy: strict-origin-when-cross-origin
```

### CORS Configuration

```python
# Allowed origins
ALLOWED_ORIGINS = [
    "https://app.taskflow.io",
    "https://staging.taskflow.io",
    "https://localhost:3000"
]

# CORS settings
CORS_SETTINGS = {
    "allow_origins": ALLOWED_ORIGINS,
    "allow_credentials": True,
    "allow_methods": ["GET", "POST", "PUT", "PATCH", "DELETE"],
    "allow_headers": ["Authorization", "Content-Type", "X-Request-ID"],
    "max_age": 600
}
```

### Input Validation

All API inputs are validated using Pydantic models:

```python
from pydantic import BaseModel, EmailStr, Field, validator

class TaskCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: str = Field(None, max_length=10000)
    board_id: UUID
    
    @validator('title')
    def title_not_empty(cls, v):
        if not v.strip():
            raise ValueError('Title cannot be empty')
        return v.strip()
```

### SQL Injection Prevention

- Use ORM (SQLAlchemy) with parameterized queries
- Never concatenate user input into SQL
- Input sanitization on all string fields

### XSS Protection

- Output encoding for all user-generated content
- Content Security Policy headers
- Validation of HTML content (if allowed)

## Monitoring & Alerting

### Rate Limit Metrics

| Metric | Threshold | Alert |
|--------|-----------|-------|
| Rate limit hits | > 100/hour | Warning |
| Rate limit hits | > 500/hour | Critical |
| Failed auth attempts | > 50/minute | Security alert |
| Unique IPs hitting limits | > 20/hour | Investigation |

### Security Monitoring

```yaml
# Security alerts configuration
alerts:
  brute_force:
    condition: failed_logins > 10 per IP per hour
    action: block_ip_temporarily
    
  unusual_activity:
    condition: requests > 1000 per user per hour
    action: notify_security_team
    
  token_abuse:
    condition: token_reuse_detected
    action: revoke_all_tokens
```

## Best Practices for Clients

### Handling Rate Limits

```python
import requests
from time import sleep

def api_request_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            sleep(retry_after)
            continue
            
        response.raise_for_status()
        return response.json()
    
    raise Exception("Max retries exceeded")
```

### Exponential Backoff

```python
import random

def exponential_backoff(attempt, base_delay=1, max_delay=60):
    delay = min(base_delay * (2 ** attempt), max_delay)
    jitter = random.uniform(0, 0.1 * delay)
    return delay + jitter
```

## Testing Rate Limits

### Load Testing Configuration

```yaml
# k6 load test configuration
scenarios:
  normal_usage:
    executor: constant-vus
    vus: 50
    duration: 5m
    
  spike_test:
    executor: ramping-vus
    stages:
      - duration: 2m
        target: 100
      - duration: 5m
        target: 100
      - duration: 2m
        target: 0
```

---

*Last updated: January 2025*
