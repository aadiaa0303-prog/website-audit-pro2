# Website Audit Pro - API Documentation

## Base URL
```
http://localhost:3000/api
```

---

## Endpoints

### 1. POST /audit
**Audit a website for security, performance, SEO, and accessibility**

#### Request
```json
{
    "url": "https://example.com"
}
```

#### Parameters
- `url` (string, required): Valid HTTP/HTTPS URL to audit
  - Must be a valid URL format
  - Cannot be localhost or private IP (127.0.0.1, 192.168.*, 10.0.*)
  - Max length: 2048 characters

#### Response (200 OK)
```json
{
    "success": true,
    "id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "url": "https://example.com",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "security": {
        "score": 85,
        "ssl": {
            "valid": true,
            "protocol": "TLS 1.3",
            "expiryDays": 120,
            "certificateIssuer": "Let's Encrypt",
            "publicKeyAlgorithm": "RSA-4096"
        },
        "headers": {
            "hsts": true,
            "xFrame": true,
            "xss": true,
            "contentType": true,
            "csp": true,
            "referrerPolicy": true
        },
        "cookies": {
            "secure": true,
            "httpOnly": true,
            "sameSite": true
        },
        "vulnerabilities": [
            {
                "name": "XSS",
                "severity": "high"
            }
        ]
    },
    "performance": {
        "score": 92,
        "loadTime": 1.45,
        "firstContentfulPaint": 0.8,
        "largestContentfulPaint": 1.2,
        "resources": {
            "optimized": true,
            "minified": true,
            "compressed": true,
            "caching": true
        },
        "images": {
            "optimized": true,
            "formats": ["WebP", "AVIF"],
            "avgSize": 125
        }
    },
    "seo": {
        "score": 78,
        "metaTags": {
            "title": true,
            "description": true,
            "keywords": true,
            "viewport": true,
            "canonical": true
        },
        "links": {
            "internal": 45,
            "external": 12,
            "broken": 0
        },
        "mobileFriendly": true,
        "crawlability": true
    },
    "accessibility": {
        "score": 88,
        "screenReader": true,
        "contrast": true,
        "keyboardNav": true,
        "semanticHTML": true,
        "altText": true,
        "ariaLabels": true
    }
}
```

#### Error Response (400 Bad Request)
```json
{
    "error": "Validation failed",
    "details": [
        {
            "field": "url",
            "message": "Invalid URL format. Must be a valid HTTP/HTTPS URL"
        }
    ]
}
```

#### Rate Limiting
- **Limit:** 5 requests per 1 minute per IP
- **Headers:** X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset

#### Security Notes
- ✅ URL is validated on both frontend and backend
- ✅ Private IPs are blocked
- ✅ Request timeout: 30 seconds (frontend)
- ✅ Response validated before display

---

### 2. GET /results/:id
**Retrieve audit results by ID**

#### Parameters
- `id` (string, required, path): Valid audit result ID (32 hex characters)

#### Response (200 OK)
```json
{
    "success": true,
    "id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
    "url": "https://example.com",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "security": { /* ... */ },
    "performance": { /* ... */ },
    "seo": { /* ... */ },
    "accessibility": { /* ... */ }
}
```

#### Error Response (404 Not Found)
```json
{
    "error": "Audit result not found",
    "id": "invalid-id"
}
```

#### Rate Limiting
- **Limit:** 100 requests per 15 minutes per IP

#### Security Notes
- ✅ ID format validated (32 hex characters only)
- ✅ Results cannot be enumerated (secure random IDs)
- ✅ No sensitive information in responses

---

### 3. GET /history
**Get paginated audit history**

#### Query Parameters
- `page` (integer, optional, default: 1): Page number (1-1000)
- `limit` (integer, optional, default: 20): Results per page (1-100)

#### Response (200 OK)
```json
{
    "success": true,
    "data": [
        {
            "id": "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6",
            "url": "https://example.com",
            "timestamp": "2024-01-15T10:30:00.000Z",
            "scores": {
                "security": 85,
                "performance": 92,
                "seo": 78,
                "accessibility": 88
            },
            "totalScore": 86
        }
    ],
    "pagination": {
        "page": 1,
        "limit": 20,
        "total": 156,
        "pages": 8
    }
}
```

#### Rate Limiting
- **Limit:** 100 requests per 15 minutes per IP

#### Security Notes
- ✅ Pagination prevents listing abuse
- ✅ Results sorted by timestamp (newest first)
- ✅ Max 100 results per page limit

---

### 4. POST /validate
**Validate and normalize a URL**

#### Request
```json
{
    "url": "https://example.com/path?query=value"
}
```

#### Response (200 OK)
```json
{
    "success": true,
    "valid": true,
    "normalized": "https://example.com/path?query=value",
    "hostname": "example.com",
    "protocol": "https:"
}
```

#### Error Response (400 Bad Request)
```json
{
    "success": false,
    "valid": false,
    "error": "Invalid URL format"
}
```

#### Rate Limiting
- **Limit:** 100 requests per 15 minutes per IP

#### Security Notes
- ✅ Validates proper URL structure
- ✅ Normalizes URLs for consistency
- ✅ Extracts hostname separately (safe)

---

### 5. GET /status
**Server health check endpoint**

#### Response (200 OK)
```json
{
    "success": true,
    "status": "Online",
    "uptime": "2h 15m",
    "timestamp": "2024-01-15T10:30:00.000Z",
    "auditsCompleted": 45,
    "memoryUsage": "125MB"
}
```

#### Rate Limiting
- **Not rate limited** (health check endpoint)

#### Security Notes
- ✅ Minimal information exposure
- ✅ No sensitive system details
- ✅ Memory usage reported safely

---

## HTTP Headers

### Response Headers
```
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com; style-src 'self' 'unsafe-inline' https://cdnjs.cloudflare.com; img-src 'self' data: https:; font-src 'self' https://cdnjs.cloudflare.com; connect-src 'self'
```

### Rate Limit Headers
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1705328400
```

---

## Error Codes

| Code | Message | Description |
|------|---------|-------------|
| 200 | Success | Request completed successfully |
| 400 | Bad Request | Invalid input or validation error |
| 404 | Not Found | Resource not found (e.g., audit result ID) |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error (details hidden for security) |

---

## Authentication & Authorization

Currently, the API does not require authentication. For production deployment, consider implementing:

- ✅ API key authentication
- ✅ OAuth2 for user accounts
- ✅ JWT tokens for sessions
- ✅ CORS with specific origin whitelist

---

## CORS Policy

**Allowed Origins:**
- `http://localhost:3000`
- `http://localhost:5000`
- `https://example.com` (configurable via `.env`)

**Allowed Methods:** GET, POST, OPTIONS

**Allowed Headers:** Content-Type

**Credentials:** Allowed (with `true` setting)

---

## Rate Limiting

### General API Limits
- **Window:** 15 minutes
- **Limit:** 100 requests per IP
- **Exception:** `/api/status` (health check not limited)

### Audit-Specific Limits
- **Window:** 1 minute
- **Limit:** 5 audit requests per IP
- **Purpose:** Prevent resource exhaustion

### Headers
- `X-RateLimit-Limit`: Maximum requests allowed
- `X-RateLimit-Remaining`: Requests remaining in current window
- `X-RateLimit-Reset`: Unix timestamp when limit resets

---

## Example Usage

### cURL
```bash
# Audit a website
curl -X POST http://localhost:3000/api/audit \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}'

# Get audit results
curl http://localhost:3000/api/results/{AUDIT_ID}

# Get audit history
curl "http://localhost:3000/api/history?page=1&limit=10"

# Validate URL
curl -X POST http://localhost:3000/api/validate \
  -H "Content-Type: application/json" \
  -d '{"url":"https://example.com"}'

# Health check
curl http://localhost:3000/api/status
```

### JavaScript (Fetch)
```javascript
// Audit a website
const response = await fetch('http://localhost:3000/api/audit', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ url: 'https://example.com' })
});
const result = await response.json();
console.log(result.id);

// Get results
const resultResponse = await fetch(`http://localhost:3000/api/results/${result.id}`);
const fullResults = await resultResponse.json();
```

### Python
```python
import requests
import json

# Audit a website
url = 'http://localhost:3000/api/audit'
headers = {'Content-Type': 'application/json'}
data = {'url': 'https://example.com'}

response = requests.post(url, headers=headers, json=data)
result = response.json()
print(f"Audit ID: {result['id']}")
print(f"Security Score: {result['security']['score']}")
```

---

## Response Times

- **Audit Request:** ~2000ms (simulated scanning time)
- **Results Retrieval:** ~300ms
- **History Listing:** <100ms
- **URL Validation:** <50ms
- **Status Check:** <10ms

---

**API Version:** 2.0.0
**Last Updated:** 2024
**Status:** Production Ready with Security Enhancements
