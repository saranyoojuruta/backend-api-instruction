# Alumni Directory API Specification

**Version**: 1.0  
**Base URL**: `http://localhost:5050/api/v1`  
**Date**: September 23, 2025

## Standard Response Format

All endpoints return responses in the following envelope format:

```json
{
  "success": boolean,
  "data": object | null,
  "error": string | null
}
```

## Error Codes

Common error codes used across endpoints:
- `PROFILE_NOT_FOUND` - Profile does not exist
- `AUTH_INVALID` - Invalid authentication credentials
- `AUTH_RATE_LIMITED` - Too many authentication attempts
- `OTP_INVALID` - Invalid or expired OTP
- `USER_NOT_APPROVED` - User registration pending approval
- `CLAIM_ALREADY_PENDING` - Profile claim already exists
- `VALIDATION_ERROR` - Request validation failed

---

## 🔐 Authentication Endpoints

### POST /auth/register
Register a new user account.

**Request:**
```json
{
  "email": "user@example.com",  // Optional if phone provided
  "phone": "+1234567890",       // Optional if email provided
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "userId": 123,
    "requiresApproval": true
  },
  "error": null
}
```

### POST /auth/login
Authenticate user and get access token.

**Request:**
```json
{
  "email": "user@example.com",  // Optional if phone provided
  "phone": "+1234567890",       // Optional if email provided
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "accessToken": "jwt_token_here",
    "expiresIn": 3600
  },
  "error": null
}
```

### POST /auth/oauth/{provider}/start
Start OAuth authentication flow.

**Parameters:**
- `provider`: `google` | `facebook`

**Response:**
```json
{
  "success": true,
  "data": {
    "redirectUrl": "https://accounts.google.com/oauth/authorize?..."
  },
  "error": null
}
```

### POST /auth/otp/request
Request OTP verification code.

**Request:**
```json
{
  "channel": "email",           // "email" | "sms"
  "target": "user@example.com"  // Email or phone number
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "ttlSeconds": 300
  },
  "error": null
}
```

### POST /auth/otp/verify
Verify OTP code.

**Request:**
```json
{
  "channel": "email",
  "target": "user@example.com",
  "code": "123456"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "verified": true
  },
  "error": null
}
```

### POST /auth/password/forgot
Request password reset.

**Request:**
```json
{
  "email": "user@example.com"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "ok": true
  },
  "error": null
}
```

### POST /auth/password/reset
Reset password with token.

**Request:**
```json
{
  "token": "reset_token_here",
  "newPassword": "newSecurePassword123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "ok": true
  },
  "error": null
}
```

---

## 👤 Profile Endpoints

### GET /profiles/{id}
Get profile by ID.

**Parameters:**
- `id`: Profile ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 100,
    "firstName": "Alice",
    "lastName": "Wong",
    "nickname": "Al",
    "email": "alice@example.com",
    "phone": "+1234567890",
    "dateOfBirth": "1995-03-15T00:00:00Z",
    "address": "123 Main St, Bangkok",
    "institution": "ABC University",
    "faculty": "Engineering",
    "cohort": "2015",
    "group": "Group A",
    "workplace": "Tech Co",
    "jobTitle": "Engineer",
    "socialMediaLinks": "{\"facebook\":\"alice.wong\"}",
    "familyStatus": "Single",
    "pictureUrl": "https://example.com/photos/alice.jpg",
    "additionalPhotos": "[\"photo1.jpg\",\"photo2.jpg\"]",
    "isClaimed": false,
    "visibilitySettings": "{\"phone\":false,\"email\":true}"
  },
  "error": null
}
```

### POST /profiles/search
Search profiles with filters. *(Changed from GET to POST)*

**Request:**
```json
{
  "q": "alice",                 // Search term (optional)
  "faculty": "Engineering",     // Faculty filter (optional)
  "cohort": "2015",            // Cohort filter (optional)
  "isClaimed": true,           // Claim status filter (optional)
  "page": 1,                   // Page number (default: 1)
  "pageSize": 20               // Items per page (default: 20, max: 100)
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 100,
        "firstName": "Alice",
        "lastName": "Wong",
        "nickname": "Al",
        "email": "alice@example.com",
        "faculty": "Engineering",
        "cohort": "2015",
        "workplace": "Tech Co",
        "jobTitle": "Engineer",
        "isClaimed": false
      }
    ],
    "total": 1,
    "page": 1,
    "pageSize": 20
  },
  "error": null
}
```

### POST /profiles
Create a new profile.

**Request:**
```json
{
  "firstName": "John",
  "lastName": "Doe",
  "nickname": "Johnny",
  "email": "john@example.com",
  "phone": "+1234567890",
  "dateOfBirth": "1995-01-01T00:00:00Z",
  "address": "456 Oak St",
  "institution": "ABC University",
  "faculty": "Business",
  "cohort": "2018",
  "group": "Group B",
  "workplace": "Business Corp",
  "jobTitle": "Manager",
  "socialMediaLinks": "{\"linkedin\":\"john.doe\"}",
  "familyStatus": "Married"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 102
  },
  "error": null
}
```

### PUT /profiles/{id}
Update existing profile.

**Parameters:**
- `id`: Profile ID (integer)

**Request:** Same as POST /profiles

**Response:**
```json
{
  "success": true,
  "data": {
    "updated": true
  },
  "error": null
}
```

### DELETE /profiles/{id}
Soft delete profile.

**Parameters:**
- `id`: Profile ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "deleted": true
  },
  "error": null
}
```

### POST /profiles/{id}/recover
Recover soft-deleted profile.

**Parameters:**
- `id`: Profile ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "recovered": true
  },
  "error": null
}
```

### POST /profiles/{id}/photos
Upload profile photos.

**Parameters:**
- `id`: Profile ID (integer)

**Request:** Multipart form data
- `profilePhoto`: Single image file (optional)
- `additionalPhotos`: Multiple image files (optional)

**Response:**
```json
{
  "success": true,
  "data": {
    "uploaded": {
      "profilePhoto": 1,
      "additionalPhotos": 3
    }
  },
  "error": null
}
```

---

## 🎯 Claims Endpoints

### POST /claims/link
Generate claim link for profile.

**Request:**
```json
{
  "profileId": 100
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "claimLink": "https://alumni.example.com/claim/abc123token"
  },
  "error": null
}
```

### POST /claims
Create profile claim request.

**Request:**
```json
{
  "profileId": 100,
  "evidence": "I am Alice Wong, graduated in 2015"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "claimId": 1,
    "status": "Pending"
  },
  "error": null
}
```

### GET /claims/{id}
Get claim request details.

**Parameters:**
- `id`: Claim ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "profileId": 100,
    "evidence": "I am Alice Wong, graduated in 2015",
    "status": "Pending",
    "rejectionReason": null,
    "createdAt": "2025-09-23T10:00:00Z",
    "approvedAt": null,
    "rejectedAt": null
  },
  "error": null
}
```

### POST /claims/{id}/approve
Approve claim request (Admin only).

**Parameters:**
- `id`: Claim ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "status": "Approved"
  },
  "error": null
}
```

### POST /claims/{id}/reject
Reject claim request (Admin only).

**Parameters:**
- `id`: Claim ID (integer)

**Request:**
```json
{
  "reason": "Insufficient evidence provided"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "status": "Rejected"
  },
  "error": null
}
```

### POST /claims/profiles/{id}/unclaim
Unclaim a profile (Admin only).

**Parameters:**
- `id`: Profile ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "status": "Unclaimed"
  },
  "error": null
}
```

---

## 🔔 Notifications Endpoints

### GET /notifications
Get user notifications.

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1,
        "type": "OTP",
        "title": "OTP Verification",
        "message": "Your OTP code has been sent",
        "isRead": false,
        "createdAt": "2025-09-23T10:00:00Z"
      }
    ],
    "unreadCount": 1
  },
  "error": null
}
```

### POST /notifications/read
Mark notifications as read.

**Request:**
```json
{
  "ids": [1, 2, 3]
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "updated": 3
  },
  "error": null
}
```

---

## 🛡️ Admin Endpoints

### GET /admin/scopes
Get admin scope hierarchy.

**Response:**
```json
{
  "success": true,
  "data": {
    "tree": [
      {
        "id": 1,
        "type": "System",
        "key": "system",
        "displayName": "System Admin",
        "children": [
          {
            "id": 2,
            "type": "Faculty",
            "key": "engineering",
            "displayName": "Engineering Faculty"
          }
        ]
      }
    ]
  },
  "error": null
}
```

### POST /admin/scopes
Create admin scope.

**Request:**
```json
{
  "type": "Faculty",
  "key": "business",
  "parentId": 1
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": 3
  },
  "error": null
}
```

### GET /admin/registrations
Get user registrations.

**Query Parameters:**
- `status`: `Pending` | `Approved` | `Rejected` (default: Pending)

**Response:**
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1,
        "name": "John Doe",
        "email": "john@example.com",
        "status": "Pending",
        "createdAt": "2025-09-23T10:00:00Z"
      }
    ]
  },
  "error": null
}
```

### POST /admin/registrations/{id}/approve
Approve user registration.

**Parameters:**
- `id`: User ID (integer)

**Response:**
```json
{
  "success": true,
  "data": {
    "status": "Approved"
  },
  "error": null
}
```

### POST /admin/registrations/{id}/reject
Reject user registration.

**Parameters:**
- `id`: User ID (integer)

**Request:**
```json
{
  "reason": "Invalid information provided"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "status": "Rejected"
  },
  "error": null
}
```

### POST /admin/import
Import alumni data from file.

**Request:** Multipart form data
- `file`: Excel file (.xlsx)

**Response:**
```json
{
  "success": true,
  "data": {
    "jobId": "import-job-uuid-123"
  },
  "error": null
}
```

### GET /admin/import/{id}
Get import job status.

**Parameters:**
- `id`: Job ID (string)

**Response:**
```json
{
  "success": true,
  "data": {
    "report": {
      "status": "Completed",
      "errors": [
        {
          "row": 5,
          "message": "Invalid email format"
        }
      ]
    },
    "stats": {
      "total": 100,
      "successful": 95,
      "failed": 5
    }
  },
  "error": null
}
```

### POST /admin/export
Export alumni data.

**Request:**
```json
{
  "filters": {
    "faculty": "Engineering",
    "cohort": "2015"
  }
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "jobId": "export-job-uuid-456"
  },
  "error": null
}
```

### GET /admin/export/{id}
Get export job status.

**Parameters:**
- `id`: Job ID (string)

**Response:**
```json
{
  "success": true,
  "data": {
    "report": {
      "status": "Completed"
    },
    "url": "https://alumni.example.com/downloads/export.xlsx"
  },
  "error": null
}
```

---

## 🏥 Health Endpoint

### GET /health
Check API health status.

**Response:**
```json
{
  "status": "ok"
}
```

---

## 🔒 Authentication & Authorization

- Most endpoints require JWT authentication via `Authorization: Bearer <token>` header
- Admin endpoints require admin role permissions
- Profile endpoints respect visibility settings for non-owners
- Rate limiting applies to OTP and authentication endpoints

## 📊 Pagination

Search endpoints support pagination with:
- `page`: Page number (1-based, default: 1)
- `pageSize`: Items per page (1-100, default: 20)

Response includes:
- `items`: Array of results
- `total`: Total count of all items
- `page`: Current page number
- `pageSize`: Items per page returned