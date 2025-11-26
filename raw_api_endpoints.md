## 3. API Endpoints

### Error Response Format (RFC 7807-based)

All error responses follow a structured format based on [RFC 7807 Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807).

**Standard Error Response:**
```json
{
  "error": {
    "type": "validation_error",
    "title": "Invalid request parameters",
    "status": 400,
    "detail": "Request validation failed",
    "trace_id": "trace_abc123",
    "errors": [
      {
        "field": "name",
        "message": "Name is required and must not be empty",
        "code": "REQUIRED_FIELD"
      },
      {
        "field": "templates",
        "message": "At least one template is required",
        "code": "MIN_LENGTH"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

**Error Object Fields:**
- `type` (required): Error type identifier (snake_case)
- `title` (required): Short, human-readable summary
- `status` (required): HTTP status code
- `detail` (required): Human-readable explanation specific to this occurrence
- `trace_id` (optional): Distributed tracing ID for debugging
- `errors` (optional): Array of field-specific validation errors

**Error Detail Fields (for validation errors):**
- `field` (required): The field name that failed validation
- `message` (required): Human-readable error message
- `code` (required): Machine-readable error code (e.g., `REQUIRED_FIELD`, `INVALID_FORMAT`)

**Metadata Fields:**
- `request_id` (required): Unique identifier for this request
- `timestamp` (required): ISO 8601 timestamp when the error occurred

**Common Error Types:**
- `unauthorized` - 401 Unauthorized
- `forbidden` - 403 Forbidden
- `not_found` - 404 Not Found
- `validation_error` - 400 Bad Request
- `conflict` - 409 Conflict
- `unprocessable_entity` - 422 Unprocessable Entity
- `internal_server_error` - 500 Internal Server Error
- `rate_limit_exceeded` - 429 Too Many Requests

### 3.1 Mailbox Management

#### List Mailbox Integrations

```
GET /integrations/mailboxes
```

Get all mailbox integrations for the current user with their settings and status.

**Query Parameters:** None

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 42,
      "type": "gmail",
      "account_identifier": "me@acme.com",
      "status": "connected",
      "is_default": true,
      "per_day": 200,
      "per_hour": 20,
      "min_delay_seconds": 600,
      "timezone": "Africa/Cairo",
      "authorized_gmail": true,
      "token_expiry_date": "2025-11-30T10:00:00Z"
    },
    {
      "id": 43,
      "type": "gmail",
      "account_identifier": "sales@acme.com",
      "status": "connected",
      "is_default": false,
      "per_day": 150,
      "per_hour": 15,
      "min_delay_seconds": 900,
      "timezone": "America/New_York",
      "authorized_gmail": true,
      "token_expiry_date": "2025-12-15T10:00:00Z"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 2
  }
}
```

**Error Responses:**

```json
// 200 OK - Empty list if no integrations
// (Returns empty array, not 404)

// 401 Unauthorized - Invalid or missing auth token
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### OAuth Authorization

```
GET /auth/google
```

Exchange OAuth authorization code for access tokens and create/update integration.

**Query Parameters:**
- `authorizeGmail` (required): `true`
- `code` (required): OAuth authorization code from Google
- `redirect_uri` (required): OAuth redirect URI

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 42,
    "type": "gmail",
    "account_identifier": "me@acme.com",
    "status": "connected",
    "authorized_gmail": true,
    "per_day": 200,
    "per_hour": 20,
    "min_delay_seconds": 600,
    "timezone": "Africa/Cairo"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 400 Bad Request - Invalid authorization code
{
  "error": {
    "type": "validation_error",
    "title": "Invalid Authorization Code",
    "status": 400,
    "detail": "The authorization code is invalid or has expired",
    "errors": [
      {
        "field": "code",
        "message": "Authorization code is invalid or expired",
        "code": "INVALID_AUTH_CODE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Missing required parameters
{
  "error": {
    "type": "validation_error",
    "title": "Missing Required Parameters",
    "status": 400,
    "detail": "Required query parameters are missing",
    "errors": [
      {
        "field": "code",
        "message": "Authorization code is required",
        "code": "REQUIRED_FIELD"
      },
      {
        "field": "redirect_uri",
        "message": "Redirect URI is required",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 500 Internal Server Error - OAuth provider error
{
  "error": {
    "type": "internal_server_error",
    "title": "OAuth Provider Error",
    "status": 500,
    "detail": "Failed to exchange authorization code with Google"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Integration Settings

```
PATCH /integrations/:id
```

Update rate limits, timezone, or status for a mailbox integration.

**Path Parameters:**
- `id` (required): Integration ID

**Request Body:**
```json
{
  "per_day": 200,
  "per_hour": 20,
  "min_delay_seconds": 600,
  "timezone": "Africa/Cairo",
  "status": "paused"
}
```

All fields are optional.

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 42,
    "type": "gmail",
    "account_identifier": "user@company.com",
    "status": "paused",
    "authorized_gmail": true,
    "per_day": 200,
    "per_hour": 20,
    "min_delay_seconds": 600,
    "timezone": "Africa/Cairo"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found - Integration doesn't exist
{
  "error": {
    "type": "not_found",
    "title": "Integration Not Found",
    "status": 404,
    "detail": "Integration with ID 42 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden - Not the owner
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this integration"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid values
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid parameter values",
    "errors": [
      {
        "field": "per_day",
        "message": "Must be between 1 and 1000",
        "code": "OUT_OF_RANGE"
      },
      {
        "field": "timezone",
        "message": "Invalid timezone identifier",
        "code": "INVALID_FORMAT"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.2 Sequence Management

#### List Sequences

```
GET /sequences
```

List all sequences for the authenticated user with statistics.

**Query Parameters:**
- `status` (optional): Filter by status (`draft`, `active`, `paused`, `archived`)

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 101,
      "name": "Outbound – Cold V1",
      "status": "active",
      "total_leads": 250,
      "open_rate": 0.47,
      "reply_rate": 0.093,
      "updated_at": "2025-11-08T10:00:00Z"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 1
  }
}
```

**Error Responses:**

```json
// 401 Unauthorized
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid status filter
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid status value",
    "errors": [
      {
        "field": "status",
        "message": "Must be one of: draft, active, paused, archived",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Create Sequence

```
POST /sequences
```

Create a new sequence with instructions, templates, and context.

**Request Body:**
```json
{
  "name": "Outbound – Cold V1",
  "instruction": "High-level rules...",
  "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
  "context": "...",
  "templates": [
    {
      "name": "Step 1 - Intro",
      "subject": "Quick intro",
      "body": "<p>...</p>"
    }
  ],
  "notes": "internal notes"
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "id": 201,
    "name": "Outbound – Cold V1",
    "status": "draft",
    "instruction": "High-level rules...",
    "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
    "context": "...",
    "templates": [
      {
        "name": "Step 1 - Intro",
        "subject": "Quick intro",
        "body": "<p>...</p>"
      }
    ],
    "notes": "internal notes",
    "created_at": "2025-11-25T10:00:00Z",
    "updated_at": "2025-11-25T10:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 400 Bad Request - Validation error
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid request body",
    "errors": [
      {
        "field": "name",
        "message": "Name is required",
        "code": "REQUIRED_FIELD"
      },
      {
        "field": "instruction",
        "message": "Instruction is required",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 401 Unauthorized
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 422 Unprocessable Entity - Invalid template format
{
  "error": {
    "type": "unprocessable_entity",
    "title": "Invalid Template Format",
    "status": 422,
    "detail": "Email templates contain invalid data",
    "errors": [
      {
        "field": "templates[0].body",
        "message": "Invalid HTML content",
        "code": "INVALID_HTML"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Get Sequence

```
GET /sequences/:id
```

Load a specific sequence with all configuration details.

**Path Parameters:**
- `id` (required): Sequence ID

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 201,
    "name": "Outbound – Cold V1",
    "status": "draft",
    "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
    "context": "...",
    "instruction": "High-level rules...",
    "templates": [
      {
        "name": "Step 1 - Intro",
        "subject": "Quick intro",
        "body": "<p>...</p>"
      }
    ],
    "notes": "internal notes",
    "created_at": "2025-11-25T10:00:00Z",
    "updated_at": "2025-11-25T10:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 201 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 401 Unauthorized
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Sequence

```
PUT /sequences/:id
```

Update all fields of a sequence (full object replacement).

**Path Parameters:**
- `id` (required): Sequence ID

**Request Body:**
```json
{
  "name": "Outbound – Cold V2",
  "instruction": "Updated rules...",
  "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
  "context": "...",
  "templates": [
    {
      "name": "Step 1 - Intro",
      "subject": "New subject",
      "body": "<p>...</p>"
    }
  ],
  "notes": "updated notes"
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 201,
    "name": "Outbound – Cold V2",
    "status": "draft",
    "instruction": "Updated rules...",
    "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
    "context": "...",
    "templates": [
      {
        "name": "Step 1 - Intro",
        "subject": "New subject",
        "body": "<p>...</p>"
      }
    ],
    "notes": "updated notes",
    "created_at": "2025-11-25T10:00:00Z",
    "updated_at": "2025-11-25T12:30:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 201 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Validation error
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid request body",
    "errors": [
      {
        "field": "name",
        "message": "Name is required",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Cannot modify active sequence
{
  "error": {
    "type": "conflict",
    "title": "Sequence Is Active",
    "status": 409,
    "detail": "Cannot modify an active sequence. Please pause it first.",
    "errors": [
      {
        "field": "status",
        "message": "Sequence must be paused before modification",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Sequence Status

```
PATCH /sequences/:id
```

Update only the status of a sequence (activate/pause/archive).

**Path Parameters:**
- `id` (required): Sequence ID

**Request Body:**
```json
{
  "status": "active"
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 201,
    "name": "Outbound – Cold V1",
    "status": "active",
    "instruction": "High-level rules...",
    "lead_context": "<your_profile> Name: Abdallah Absi Work Experience: 1. CEO at Village (village.ai), Jan 2022 - Present Village helps founders find warm intros to companies",
    "context": "...",
    "templates": [
      {
        "name": "Step 1 - Intro",
        "subject": "Quick intro",
        "body": "<p>...</p>"
      }
    ],
    "notes": "internal notes",
    "created_at": "2025-11-25T10:00:00Z",
    "updated_at": "2025-11-25T14:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 400 Bad Request - Invalid status
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid status value",
    "errors": [
      {
        "field": "status",
        "message": "Must be one of: draft, active, paused, archived",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 201 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 422 Unprocessable Entity - Cannot activate incomplete sequence
{
  "error": {
    "type": "unprocessable_entity",
    "title": "Incomplete Sequence",
    "status": 422,
    "detail": "Cannot activate sequence without leads and templates",
    "errors": [
      {
        "field": "templates",
        "message": "At least one template is required",
        "code": "REQUIRED_FIELD"
      },
      {
        "field": "leads",
        "message": "At least one lead must be enrolled",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.3 Email Templates

#### Get System Templates

```
GET /templates
```

Get system-provided email templates for the template library picker.

**Query Parameters:** None

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 1,
      "name": "Follow-up",
      "subject": "Quick update",
      "body": "<p>...</p>",
      "category": "follow_up"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 1
  }
}
```

**Error Responses:**

```json
// 401 Unauthorized
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.4 Lead Management

#### List Sequence Leads

```
GET /sequences/:id/leads
```

List all leads enrolled in a sequence with their status and statistics.

**Path Parameters:**
- `id` (required): Sequence ID

**Query Parameters:**
- `status` (optional): Filter by lead status (`not_started`, `active`, `paused`, `finished`)

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 1001,
      "lead_id": 501,
      "lead_name": "Jane Doe",
      "company": "Acme",
      "status": "active",
      "latest_update": "email_sent",
      "introducers": [
        {
          "lead_id": 742,
          "rank": 1
        },
        {
          "lead_id": 889,
          "rank": 2
        }
      ],
      "next_followup_at": "2025-11-15T10:00:00Z",
      "last_sent_at": "2025-11-10T09:00:00Z",
      "last_inbound_at": null
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 1
  }
}
```

**Error Responses:**

```json
// 404 Not Found - Sequence doesn't exist
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 101 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid status filter
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid status value",
    "errors": [
      {
        "field": "status",
        "message": "Must be one of: not_started, active, paused, finished",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Enroll Leads in Sequence

```
POST /sequences/:id/leads
```

Bulk enroll leads into a sequence.

**Path Parameters:**
- `id` (required): Sequence ID

**Request Body:**
```json
{
  "lead_ids": [501, 502, 503],
  "integration_id": 42
}
```

**Success Response (201 Created):**
```json
{
  "data": {
    "sequence_id": "101",
    "enrolled_count": 3,
    "items": [
      {
        "id": 1001,
        "lead_id": 501,
        "lead_name": "Jane Doe",
        "company": "Acme",
        "status": "not_started",
        "created_at": "2025-11-25T10:00:00Z"
      },
      {
        "id": 1002,
        "lead_id": 502,
        "lead_name": "John Smith",
        "company": "TechCorp",
        "status": "not_started",
        "created_at": "2025-11-25T10:00:00Z"
      }
    ]
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found - Sequence doesn't exist
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 101 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Validation error
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid request body",
    "errors": [
      {
        "field": "lead_ids",
        "message": "Lead IDs are required and must be a non-empty array",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Leads already enrolled
{
  "error": {
    "type": "conflict",
    "title": "Leads Already Enrolled",
    "status": 409,
    "detail": "Some leads are already enrolled in this sequence",
    "errors": [
      {
        "field": "lead_ids[0]",
        "message": "Lead 501 is already enrolled",
        "code": "DUPLICATE_ENTRY"
      },
      {
        "field": "lead_ids[2]",
        "message": "Lead 503 is already enrolled",
        "code": "DUPLICATE_ENTRY"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 404 Not Found - Invalid lead IDs
{
  "error": {
    "type": "not_found",
    "title": "Leads Not Found",
    "status": 404,
    "detail": "Some lead IDs do not exist",
    "errors": [
      {
        "field": "lead_ids[1]",
        "message": "Lead 502 does not exist",
        "code": "NOT_FOUND"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Sequence Lead

```
PATCH /sequences/:sequenceId/leads/:leadId
```

Update a lead's introducers, status, or next follow-up time.

**Path Parameters:**
- `sequenceId` (required): Sequence ID
- `leadId` (required): Lead ID

**Request Body:**
```json
{
  "introducers": [
    {
      "lead_id": 742,
      "rank": 1
    },
    {
      "lead_id": 889,
      "rank": 2
    }
  ],
  "status": "active",
  "next_followup_at": "2025-11-15T10:00:00Z"
}
```

All fields are optional.

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 1001,
    "lead_id": 501,
    "lead_name": "Jane Doe",
    "company": "Acme",
    "status": "active",
    "latest_update": "email_sent",
    "introducers": [
      {
        "lead_id": 742,
        "rank": 1
      },
      {
        "lead_id": 889,
        "rank": 2
      }
    ],
    "next_followup_at": "2025-11-15T10:00:00Z",
    "updated_at": "2025-11-25T11:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found - Lead not in sequence
{
  "error": {
    "type": "not_found",
    "title": "Sequence Lead Not Found",
    "status": 404,
    "detail": "Lead 501 is not enrolled in sequence 101"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this sequence lead"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid introducer IDs
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid introducer data",
    "errors": [
      {
        "field": "introducers[0].lead_id",
        "message": "Lead ID 742 does not exist",
        "code": "NOT_FOUND"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Cannot modify finished lead
{
  "error": {
    "type": "conflict",
    "title": "Lead Already Finished",
    "status": 409,
    "detail": "Cannot modify a finished sequence lead",
    "errors": [
      {
        "field": "status",
        "message": "Lead is in finished state and cannot be modified",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Remove Lead from Sequence

```
DELETE /sequences/:sequenceId/leads/:leadId
```

Remove a lead from a sequence (soft delete).

**Path Parameters:**
- `sequenceId` (required): Sequence ID
- `leadId` (required): Lead ID

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 1001,
    "lead_id": 501,
    "lead_name": "Jane Doe",
    "company": "Acme",
    "status": "finished",
    "detailed_status": "removed",
    "introducers": [
      {
        "lead_id": 742,
        "rank": 1
      }
    ],
    "removed_at": "2025-11-25T12:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Lead Not Found",
    "status": 404,
    "detail": "Lead 501 is not enrolled in sequence 101"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Get Lead Activities

```
GET /sequences/:sequenceId/leads/:leadId/activities
```

Get all activity history for a specific lead in a sequence.

**Path Parameters:**
- `sequenceId` (required): Sequence ID
- `leadId` (required): Lead ID

**Query Parameters:** None

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 321,
      "sequence_lead_id": 1001,
      "activity_type": "outgoing_email",
      "status": "approved",
      "subject": "Quick intro",
      "engagement_status": "opened",
      "executed_at": "2025-11-10T09:00:00Z",
      "created_at": "2025-11-10T08:55:00Z"
    },
    {
      "id": 322,
      "sequence_lead_id": 1001,
      "activity_type": "incoming_email",
      "status": null,
      "engagement_status": "replied",
      "executed_at": "2025-11-10T09:05:00Z",
      "created_at": "2025-11-10T09:05:00Z"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 2
  }
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Lead Not Found",
    "status": 404,
    "detail": "Lead 501 is not enrolled in sequence 101"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.5 Email Operations

#### List Sequence Emails

```
GET /sequences/:id/emails
```

List all sent and scheduled emails for a sequence.

**Path Parameters:**
- `id` (required): Sequence ID

**Query Parameters:**
- `status` (optional): Filter by email status (`scheduled`, `sent`, `failed`)

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 9001,
      "sequence_lead_id": 1001,
      "integration_id": 42,
      "to_email": "jane@acme.com",
      "subject": "Quick intro",
      "status": "sent",
      "scheduled_at": null,
      "sent_at": "2025-11-10T09:00:00Z"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 1
  }
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Not Found",
    "status": 404,
    "detail": "Sequence with ID 101 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Get Lead Email History

```
GET /sequences/:sequenceId/leads/:leadId/emails
```

Get complete email thread history (sent + received) for a specific lead.

**Path Parameters:**
- `sequenceId` (required): Sequence ID
- `leadId` (required): Lead ID

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "lead": {
      "lead_id": 501,
      "lead_name": "Jane Doe",
      "email": "jane@acme.com"
    },
    "email_threads": [
      {
        "thread_id": "xx1",
        "emails": [
          {
            "type": "email_outgoing",
            "id": 9001,
            "subject": "Quick intro",
            "from_email": "user@company.com",
            "to_email": "jane@acme.com",
            "status": "sent",
            "sent_at": "2025-11-10T09:00:00Z",
            "integration_thread_id": "thr-123"
          }
        ]
      },
      {
        "thread_id": "xx2",
        "emails": [
          {
            "type": "email_outgoing",
            "id": 9002,
            "subject": "Follow-up",
            "from_email": "user@company.com",
            "to_email": "jane@acme.com",
            "status": "sent",
            "sent_at": "2025-11-10T09:00:00Z",
            "integration_thread_id": "thr-456"
          },
          {
            "type": "email_inbox",
            "id": 7001,
            "from_email": "jane@acme.com",
            "received_at": "2025-11-10T09:05:00Z",
            "body_preview": "Thanks for reaching out..."
          }
        ]
      }
    ]
  },
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 2
  }
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Sequence Lead Not Found",
    "status": 404,
    "detail": "Lead 501 is not enrolled in sequence 101"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this sequence"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Scheduled Email

```
PATCH /emails/:id
```

Edit a scheduled email's subject, body, or send time (only if not yet sent).

**Path Parameters:**
- `id` (required): Email ID from `email_outgoing` table

**Request Body:**
```json
{
  "subject": "Updated subject",
  "body_html": "<p>Updated...</p>",
  "due_at": "2025-11-12T09:30:00Z"
}
```

All fields are optional.

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 9002,
    "sequence_lead_id": 1001,
    "subject": "Updated subject",
    "body_html": "<p>Updated...</p>",
    "status": "approved",
    "due_at": "2025-11-12T09:30:00Z",
    "updated_at": "2025-11-11T15:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Email Not Found",
    "status": 404,
    "detail": "Email with ID 9002 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Email already sent
{
  "error": {
    "type": "conflict",
    "title": "Email Already Sent",
    "status": 409,
    "detail": "Cannot modify an email that has already been sent",
    "errors": [
      {
        "field": "status",
        "message": "Email has already been sent and cannot be modified",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this email"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid due_at
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid due_at timestamp",
    "errors": [
      {
        "field": "due_at",
        "message": "Cannot schedule emails in the past",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Cancel Scheduled Email

```
DELETE /emails/:id
```

Cancel a scheduled email (sets status to `canceled`).

**Path Parameters:**
- `id` (required): Email ID from `email_outgoing` table

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 9002,
    "sequence_lead_id": 1001,
    "subject": "Updated subject",
    "status": "canceled",
    "due_at": "2025-11-12T09:30:00Z",
    "canceled_at": "2025-11-11T16:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Email Not Found",
    "status": 404,
    "detail": "Email with ID 9002 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Email already sent
{
  "error": {
    "type": "conflict",
    "title": "Email Already Sent",
    "status": 409,
    "detail": "Cannot cancel an email that has already been sent",
    "errors": [
      {
        "field": "status",
        "message": "Email has already been sent and cannot be canceled",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to cancel this email"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Get Email Engagement Activity

```
GET /emails/:id/activities
```

Get engagement timeline for a sent email (delivered, opened, clicked, replied, bounced).

**Path Parameters:**
- `id` (required): Email ID from `email_outgoing` table

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "events": [
      {
        "id": 1,
        "type": "delivered",
        "created_at": "2025-11-10T09:00:20Z"
      },
      {
        "id": 2,
        "type": "opened",
        "created_at": "2025-11-10T09:03:11Z",
        "user_agent": "Apple Mail"
      }
    ],
    "replies": [
      {
        "id": 7001,
        "from_email": "jane@acme.com",
        "received_at": "2025-11-10T09:05:00Z",
        "body_preview": "Thanks for reaching out..."
      }
    ]
  },
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 3
  }
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Email Not Found",
    "status": 404,
    "detail": "Email with ID 9001 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this email"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Email not sent yet
{
  "error": {
    "type": "validation_error",
    "title": "Email Not Sent",
    "status": 400,
    "detail": "Cannot retrieve activity for unsent email",
    "errors": [
      {
        "field": "status",
        "message": "Email must be sent before activity can be retrieved",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.6 Activity & Approval Workflow

#### List Pending Activities

```
GET /activities
```

List pending draft emails awaiting user approval.

**Query Parameters:**
- `activity_type` (optional): Filter by activity type (e.g., `draft_email`)
- `status` (optional): Filter by status (e.g., `pending`)

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": [
    {
      "id": 321,
      "sequence_lead_id": 1001,
      "sequence_id": 101,
      "sequence_name": "Outbound – Cold V1",
      "lead_name": "Jane Doe",
      "activity_type": "draft_email",
      "status": "pending",
      "subject": "Hi Jane — quick intro",
      "body_html": "<p>...</p>",
      "due_at": "2025-11-11T08:00:00Z",
      "ai_confidence": 0.82,
      "ai_reason": "No prior response, sending initial outreach",
      "created_at": "2025-11-10T15:00:00Z"
    }
  ],
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 1
  }
}
```

**Error Responses:**

```json
// 401 Unauthorized
{
  "error": {
    "type": "unauthorized",
    "title": "Unauthorized",
    "status": 401,
    "detail": "Authentication required"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Invalid filter
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid query parameters",
    "errors": [
      {
        "field": "activity_type",
        "message": "Must be one of: incoming_email, outgoing_email, draft_email, approval_decision, planner_decision, system_note",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Update Draft Activity

```
PATCH /activities/:id
```

Update a draft email (subject, body, or due time) without approving it.

**Path Parameters:**
- `id` (required): Activity ID from `sequence_lead_activity` table

**Request Body:**
```json
{
  "subject": "Updated intro",
  "body_html": "<p>Updated...</p>",
  "due_at": "2025-11-11T08:15:00Z"
}
```

All fields are optional.

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 321,
    "sequence_lead_id": 1001,
    "activity_type": "draft_email",
    "status": "pending",
    "subject": "Updated intro",
    "body_html": "<p>Updated...</p>",
    "due_at": "2025-11-11T08:15:00Z",
    "updated_at": "2025-11-10T16:00:00Z"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Activity Not Found",
    "status": 404,
    "detail": "Activity with ID 321 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Activity already approved
{
  "error": {
    "type": "conflict",
    "title": "Activity Already Processed",
    "status": 409,
    "detail": "Cannot modify an activity that has already been approved or rejected",
    "errors": [
      {
        "field": "status",
        "message": "Activity has already been processed and cannot be modified",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to modify this activity"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid request body",
    "errors": [
      {
        "field": "due_at",
        "message": "Cannot schedule in the past",
        "code": "INVALID_VALUE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Approve Draft Email

```
POST /activities/:id/approve
```

Approve a draft email and queue it for sending.

**Path Parameters:**
- `id` (required): Activity ID from `sequence_lead_activity` table

**Request Body:**
```json
{
  "subject": "Final subject to send",
  "body_html": "<p>Final...</p>",
  "due_at": "2025-11-11T08:30:00Z"
}
```

All fields are optional (uses draft values if not provided).

**Success Response (200 OK):**
```json
{
  "data": {
    "activity": {
      "id": 321,
      "status": "approved",
      "is_approved": true,
      "approved_at": "2025-11-10T17:00:00Z"
    },
    "email": {
      "id": 9100,
      "sequence_lead_id": 1001,
      "status": "approved",
      "subject": "Final subject to send",
      "due_at": "2025-11-11T08:30:00Z",
      "created_at": "2025-11-10T17:00:00Z"
    }
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Activity Not Found",
    "status": 404,
    "detail": "Activity with ID 321 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Already processed
{
  "error": {
    "type": "conflict",
    "title": "Activity Already Processed",
    "status": 409,
    "detail": "This activity has already been approved or rejected",
    "errors": [
      {
        "field": "status",
        "message": "Activity was already processed and cannot be approved again",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Not a draft
{
  "error": {
    "type": "validation_error",
    "title": "Invalid Activity Type",
    "status": 400,
    "detail": "Only draft_email activities can be approved",
    "errors": [
      {
        "field": "activity_type",
        "message": "Activity type must be draft_email to be approved",
        "code": "INVALID_TYPE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to approve this activity"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Reject Draft Email

```
POST /activities/:id/reject
```

Reject a draft email and trigger AI to regenerate.

**Path Parameters:**
- `id` (required): Activity ID from `sequence_lead_activity` table

**Request Body:**
```json
{
  "reason": "tone_not_right"
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "id": 321,
    "sequence_lead_id": 1001,
    "activity_type": "draft_email",
    "status": "rejected",
    "subject": "Hi Jane — quick intro",
    "due_at": "2025-11-11T08:00:00Z",
    "rejected_at": "2025-11-10T17:30:00Z",
    "rejection_reason": "tone_not_right"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Activity Not Found",
    "status": 404,
    "detail": "Activity with ID 321 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Already processed
{
  "error": {
    "type": "conflict",
    "title": "Activity Already Processed",
    "status": 409,
    "detail": "This activity has already been approved or rejected",
    "errors": [
      {
        "field": "status",
        "message": "Activity was already processed and cannot be rejected again",
        "code": "INVALID_STATE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 400 Bad Request - Not a draft
{
  "error": {
    "type": "validation_error",
    "title": "Invalid Activity Type",
    "status": 400,
    "detail": "Only draft_email activities can be rejected",
    "errors": [
      {
        "field": "activity_type",
        "message": "Activity type must be draft_email to be rejected",
        "code": "INVALID_TYPE"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to reject this activity"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

#### Get Activity Thread

```
GET /activities/:id/thread
```

Get related email thread history for a specific activity.

**Path Parameters:**
- `id` (required): Activity ID from `sequence_lead_activity` table

**Request Body:** None

**Success Response (200 OK):**
```json
{
  "data": {
    "items": [
      {
        "type": "email_outgoing",
        "id": 9001,
        "subject": "Intro",
        "sent_at": "2025-11-10T09:00:00Z"
      },
      {
        "type": "email_inbox",
        "id": 7001,
        "received_at": "2025-11-10T09:05:00Z",
        "body_preview": "Thanks..."
      }
    ]
  },
  "metadata": {},
  "pagination": {
    "cursor": null,
    "has_more": false,
    "count": 2
  }
}
```

**Error Responses:**

```json
// 404 Not Found
{
  "error": {
    "type": "not_found",
    "title": "Activity Not Found",
    "status": 404,
    "detail": "Activity with ID 321 not found"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 403 Forbidden
{
  "error": {
    "type": "forbidden",
    "title": "Forbidden",
    "status": 403,
    "detail": "You do not have permission to access this activity"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```

---

### 3.7 Webhooks

#### Incoming Email Webhook

```
POST /webhooks/emails/incoming
```

Receive normalized inbound email events from Gmail (replies, OOF, bounces, opens, clicks).

**Request Body:**
```json
{
  "integration_id": 42,
  "integration_message_id": "msg-abc123",
  "integration_thread_id": "thr-xyz789",
  "in_reply_to_integration_message_id": "msg-def456",
  "from_email": "jane@acme.com",
  "to_email": "user@company.com",
  "subject": "Re: Quick intro",
  "body_html": "<p>Thanks for reaching out...</p>",
  "received_at": "2025-11-10T09:05:00Z",
  "type": "replied"
}
```

**Success Response (200 OK):**
```json
{
  "data": {
    "email_inbox_id": 7001,
    "sequence_lead_id": 1001,
    "matched": true,
    "processing_status": "queued_for_ai_planner"
  },
  "metadata": {},
  "pagination": null
}
```

**Error Responses:**

```json
// 400 Bad Request - Invalid payload
{
  "error": {
    "type": "validation_error",
    "title": "Validation Error",
    "status": 400,
    "detail": "Invalid webhook payload",
    "errors": [
      {
        "field": "integration_message_id",
        "message": "Integration message ID is required",
        "code": "REQUIRED_FIELD"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 409 Conflict - Duplicate message
{
  "error": {
    "type": "conflict",
    "title": "Duplicate Message",
    "status": 409,
    "detail": "This message has already been processed",
    "errors": [
      {
        "field": "integration_message_id",
        "message": "Message msg-abc123 was already processed",
        "code": "DUPLICATE_ENTRY"
      }
    ]
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}

// 500 Internal Server Error
{
  "error": {
    "type": "internal_server_error",
    "title": "Webhook Processing Error",
    "status": 500,
    "detail": "Failed to process incoming email"
  },
  "metadata": {
    "request_id": "req_xyz789",
    "timestamp": "2024-01-06T12:00:00Z"
  }
}
```
