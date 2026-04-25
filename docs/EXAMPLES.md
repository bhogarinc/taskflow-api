# TaskFlow API Usage Examples

This document provides practical examples for common TaskFlow API operations.

---

## Authentication Examples

### User Registration

```bash
curl -X POST https://api.taskflow.io/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePass123!",
    "full_name": "John Doe"
  }'
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "email": "john.doe@example.com",
  "full_name": "John Doe",
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T10:30:00Z"
}
```

### Login

```bash
curl -X POST https://api.taskflow.io/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "email": "john.doe@example.com",
    "password": "SecurePass123!"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

### Using Access Token

```bash
export TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."

curl -X GET https://api.taskflow.io/v1/users/me \
  -H "Authorization: Bearer $TOKEN"
```

### Refresh Token

```bash
curl -X POST https://api.taskflow.io/v1/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{
    "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4..."
  }'
```

---

## Task Management Examples

### Create a Task

```bash
curl -X POST https://api.taskflow.io/v1/tasks \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement user authentication",
    "description": "Add JWT-based authentication to the API endpoints",
    "board_id": "550e8400-e29b-41d4-a716-446655440001",
    "priority": "high",
    "due_date": "2025-02-15",
    "labels": ["backend", "auth", "sprint-1"],
    "estimated_hours": 8
  }'
```

**Response:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440002",
  "title": "Implement user authentication",
  "description": "Add JWT-based authentication to the API endpoints",
  "status": "backlog",
  "priority": "high",
  "board_id": "550e8400-e29b-41d4-a716-446655440001",
  "assignee_id": null,
  "due_date": "2025-02-15",
  "labels": ["backend", "auth", "sprint-1"],
  "estimated_hours": 8,
  "created_by": "550e8400-e29b-41d4-a716-446655440000",
  "created_at": "2025-01-15T10:35:00Z",
  "updated_at": "2025-01-15T10:35:00Z"
}
```

### List Tasks with Filters

```bash
# Get all high priority tasks
curl -X GET "https://api.taskflow.io/v1/tasks?priority=high&status=todo" \
  -H "Authorization: Bearer $TOKEN"

# Search tasks
curl -X GET "https://api.taskflow.io/v1/tasks?search=authentication" \
  -H "Authorization: Bearer $TOKEN"

# Tasks due this week
curl -X GET "https://api.taskflow.io/v1/tasks?due_before=2025-01-22&due_after=2025-01-15" \
  -H "Authorization: Bearer $TOKEN"

# Paginated results
curl -X GET "https://api.taskflow.io/v1/tasks?page=2&per_page=50&sort_by=due_date&sort_order=asc" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "items": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440002",
      "title": "Implement user authentication",
      "status": "todo",
      "priority": "high",
      "assignee": {
        "id": "550e8400-e29b-41d4-a716-446655440003",
        "full_name": "Jane Smith"
      },
      "due_date": "2025-02-15",
      "created_at": "2025-01-15T10:35:00Z",
      "updated_at": "2025-01-15T11:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "per_page": 20,
    "total": 45,
    "total_pages": 3,
    "has_next": true,
    "has_prev": false
  }
}
```

### Update Task Status

```bash
# Quick status update
curl -X PATCH https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002/status \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "in_progress"
  }'

# Full task update
curl -X PATCH https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002 \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Implement user authentication (updated)",
    "description": "Updated description",
    "priority": "medium",
    "due_date": "2025-02-20",
    "labels": ["backend", "auth"]
  }'
```

### Assign Task

```bash
curl -X POST https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002/assign \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "assignee_id": "550e8400-e29b-41d4-a716-446655440003"
  }'

# Unassign task
curl -X POST https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002/assign \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "assignee_id": null
  }'
```

---

## Board Management Examples

### Create a Board

```bash
curl -X POST https://api.taskflow.io/v1/boards \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Q1 2025 Sprint",
    "description": "First quarter development sprint",
    "team_id": "550e8400-e29b-41d4-a716-446655440004",
    "is_public": false
  }'
```

### Get Board with Tasks

```bash
# Get board details
curl -X GET https://api.taskflow.io/v1/boards/550e8400-e29b-41d4-a716-446655440001 \
  -H "Authorization: Bearer $TOKEN"

# Get tasks grouped by status
curl -X GET "https://api.taskflow.io/v1/boards/550e8400-e29b-41d4-a716-446655440001/tasks?group_by=status" \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "groups": [
    {
      "key": "backlog",
      "tasks": [
        { "id": "...", "title": "Task 1", "status": "backlog" },
        { "id": "...", "title": "Task 2", "status": "backlog" }
      ]
    },
    {
      "key": "in_progress",
      "tasks": [
        { "id": "...", "title": "Task 3", "status": "in_progress" }
      ]
    },
    {
      "key": "done",
      "tasks": []
    }
  ]
}
```

---

## Team Management Examples

### Create a Team

```bash
curl -X POST https://api.taskflow.io/v1/teams \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Engineering Team",
    "description": "Core engineering team"
  }'
```

### Add Team Member

```bash
curl -X POST https://api.taskflow.io/v1/teams/550e8400-e29b-41d4-a716-446655440004/members \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "jane.smith@example.com",
    "role": "member"
  }'
```

### List Team Members

```bash
curl -X GET https://api.taskflow.io/v1/teams/550e8400-e29b-41d4-a716-446655440004/members \
  -H "Authorization: Bearer $TOKEN"
```

**Response:**
```json
{
  "items": [
    {
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "user": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "full_name": "John Doe"
      },
      "role": "owner",
      "joined_at": "2025-01-15T10:30:00Z"
    },
    {
      "user_id": "550e8400-e29b-41d4-a716-446655440003",
      "user": {
        "id": "550e8400-e29b-41d4-a716-446655440003",
        "full_name": "Jane Smith"
      },
      "role": "member",
      "joined_at": "2025-01-15T11:00:00Z"
    }
  ]
}
```

---

## Comments Examples

### Add Comment

```bash
curl -X POST https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002/comments \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "This looks good, but we should consider edge cases for token expiration."
  }'
```

### List Comments

```bash
curl -X GET https://api.taskflow.io/v1/tasks/550e8400-e29b-41d4-a716-446655440002/comments \
  -H "Authorization: Bearer $TOKEN"
```

---

## Python SDK Example

```python
import requests
from datetime import datetime

class TaskFlowClient:
    def __init__(self, base_url: str, access_token: str = None):
        self.base_url = base_url.rstrip('/')
        self.session = requests.Session()
        if access_token:
            self.session.headers['Authorization'] = f'Bearer {access_token}'
    
    def login(self, email: str, password: str):
        response = self.session.post(
            f"{self.base_url}/auth/login",
            json={"email": email, "password": password}
        )
        response.raise_for_status()
        data = response.json()
        self.session.headers['Authorization'] = f'Bearer {data["access_token"]}'
        return data
    
    def create_task(self, title: str, board_id: str, **kwargs):
        payload = {
            "title": title,
            "board_id": board_id,
            **kwargs
        }
        response = self.session.post(
            f"{self.base_url}/tasks",
            json=payload
        )
        response.raise_for_status()
        return response.json()
    
    def list_tasks(self, **filters):
        response = self.session.get(
            f"{self.base_url}/tasks",
            params=filters
        )
        response.raise_for_status()
        return response.json()
    
    def update_task_status(self, task_id: str, status: str):
        response = self.session.patch(
            f"{self.base_url}/tasks/{task_id}/status",
            json={"status": status}
        )
        response.raise_for_status()
        return response.json()

# Usage
client = TaskFlowClient("https://api.taskflow.io/v1")
client.login("john.doe@example.com", "SecurePass123!")

# Create a task
task = client.create_task(
    title="Implement API documentation",
    board_id="550e8400-e29b-41d4-a716-446655440001",
    priority="high",
    due_date="2025-02-01",
    labels=["documentation", "api"]
)
print(f"Created task: {task['id']}")

# List tasks
tasks = client.list_tasks(
    status="todo",
    priority="high",
    sort_by="due_date",
    sort_order="asc"
)
print(f"Found {tasks['pagination']['total']} tasks")

# Update status
client.update_task_status(task['id'], "in_progress")
```

---

## JavaScript/TypeScript Example

```typescript
interface Task {
  id: string;
  title: string;
  status: 'backlog' | 'todo' | 'in_progress' | 'in_review' | 'done' | 'cancelled';
  priority: 'lowest' | 'low' | 'medium' | 'high' | 'highest';
}

class TaskFlowAPI {
  private baseUrl: string;
  private accessToken: string | null = null;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl.replace(/\/$/, '');
  }

  setAccessToken(token: string) {
    this.accessToken = token;
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseUrl}${endpoint}`;
    const headers: Record<string, string> = {
      'Content-Type': 'application/json',
      ...options.headers as Record<string, string>
    };

    if (this.accessToken) {
      headers['Authorization'] = `Bearer ${this.accessToken}`;
    }

    const response = await fetch(url, {
      ...options,
      headers
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'API request failed');
    }

    return response.json();
  }

  async login(email: string, password: string) {
    const data = await this.request<{
      access_token: string;
      refresh_token: string;
    }>('/auth/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    
    this.setAccessToken(data.access_token);
    return data;
  }

  async createTask(taskData: {
    title: string;
    board_id: string;
    priority?: string;
    due_date?: string;
    labels?: string[];
  }): Promise<Task> {
    return this.request<Task>('/tasks', {
      method: 'POST',
      body: JSON.stringify(taskData)
    });
  }

  async listTasks(filters?: {
    status?: string;
    priority?: string;
    search?: string;
    page?: number;
    per_page?: number;
  }): Promise<{ items: Task[]; pagination: any }> {
    const params = new URLSearchParams();
    if (filters) {
      Object.entries(filters).forEach(([key, value]) => {
        if (value !== undefined) params.append(key, String(value));
      });
    }
    
    return this.request(`/tasks?${params}`);
  }

  async updateTaskStatus(taskId: string, status: Task['status']): Promise<Task> {
    return this.request<Task>(`/tasks/${taskId}/status`, {
      method: 'PATCH',
      body: JSON.stringify({ status })
    });
  }
}

// Usage
const api = new TaskFlowAPI('https://api.taskflow.io/v1');

async function main() {
  await api.login('john.doe@example.com', 'SecurePass123!');
  
  const task = await api.createTask({
    title: 'Implement frontend integration',
    board_id: '550e8400-e29b-41d4-a716-446655440001',
    priority: 'high',
    due_date: '2025-02-01',
    labels: ['frontend', 'integration']
  });
  
  console.log('Created task:', task.id);
  
  await api.updateTaskStatus(task.id, 'in_progress');
}

main().catch(console.error);
```

---

## Error Handling Examples

### Handling Validation Errors

```python
import requests

def create_task_safe(client, task_data):
    try:
        response = client.post('/tasks', json=task_data)
        response.raise_for_status()
        return response.json()
    except requests.exceptions.HTTPError as e:
        if e.response.status_code == 422:
            error_data = e.response.json()
            print("Validation errors:")
            for detail in error_data.get('details', []):
                print(f"  - {detail['field']}: {detail['message']}")
        raise
```

### Handling Rate Limits

```python
import time

def api_request_with_backoff(client, method, url, max_retries=3, **kwargs):
    for attempt in range(max_retries):
        response = client.request(method, url, **kwargs)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('Retry-After', 60))
            print(f"Rate limited. Waiting {retry_after} seconds...")
            time.sleep(retry_after)
            continue
        
        response.raise_for_status()
        return response
    
    raise Exception(f"Max retries ({max_retries}) exceeded")
```

---

*For more examples and advanced usage patterns, see the [API Guide](./API_GUIDE.md).*
