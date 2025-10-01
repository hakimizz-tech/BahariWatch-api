# BahariWatch API Documentation

Community-driven illegal fishing monitoring system for Kenya's coastal regions.

## Overview

BahariWatch integrates crowdsourced reports with AIS/VMS vessel tracking to detect and prevent IUU (Illegal, Unreported, and Unregulated) fishing in Kenya's coastal waters.

### Key Features

- 🌍 **Offline-first mobile reporting** with automatic sync
- 🔒 **Anonymous community reporting** with privacy safeguards
- 📡 **Real-time vessel tracking** (AIS/VMS/PDS integration)
- 🔍 **Spatial-temporal matching engine**
- 🚨 **Enforcement alert system** with action tracking
- 📊 **Comprehensive audit trails** for research ethics compliance

##  Quick Start

### View Documentation


 * For detailed documentation, please visit: https://bahariwatch-api.vercel.app
 

### API Base URLs

- **Development:** `http://localhost:8000/v1`
- **Production:** `https://api.bahariwatch.ke/v1`



## 🏗️ API Design Standards

This API follows **Microsoft REST API Guidelines** for consistency and best practices.

### Naming Conventions

- **Resource names:** Plural nouns (e.g., `/reports`, `/vessels`)
- **Properties:** camelCase (e.g., `userId`, `createdAt`, `photoUrl`)
- **Query parameters:** Use `$` prefix for OData operators (`$filter`, `$orderby`, `$count`)
- **Headers:** Standard HTTP headers; custom headers prefixed with `X-`
- **Date/Time:** ISO 8601 format (e.g., `2025-05-15T14:30:00Z`)
- **IDs:** String format with semantic prefixes (e.g., `rpt-20250515-001`)

### Collection Responses

All collection endpoints return an object with a `value` array for extensibility:

```json
{
  "value": [...],
  "@odata.nextLink": "https://api.bahariwatch.ke/v1/reports?cursor=...",
  "@odata.count": 1250,
  "nextCursor": "eyJpZCI6InJwdC0wMDIifQ",
  "count": 1250
}
```

### Error Handling

Errors follow RFC 7807 problem details format:

```json
{
  "error": {
    "code": "ValidationError",
    "message": "Request validation failed",
    "target": "location",
    "details": [
      {
        "code": "RequiredField",
        "message": "location is required",
        "target": "location"
      }
    ],
    "innerError": {
      "code": "DB_CONNECTION_TIMEOUT",
      "trace": ["..."]
    }
  }
}
```

## 🔑 Authentication

The API uses JWT bearer tokens:

```bash
# Register a new user
curl -X POST https://api.bahariwatch.ke/v1/users/register \
  -H "Content-Type: application/json" \
  -d '{"deviceId": "device-123"}'

# Login
curl -X POST https://api.bahariwatch.ke/v1/users/login \
  -H "Content-Type: application/json" \
  -d '{"deviceId": "device-123"}'

# Use the token
curl -X GET https://api.bahariwatch.ke/v1/reports \
  -H "Authorization: Bearer <your-token>"
```

## 📝 Common Operations

### Submit a Report

```bash
curl -X POST https://api.bahariwatch.ke/v1/reports \
  -H "Authorization: Bearer <token>" \
  -H "Idempotency-Key: $(uuidgen)" \
  -F "photo=@fishing_vessel.jpg" \
  -F 'location={"latitude": -4.0435, "longitude": 39.6682, "accuracy": 15.5}' \
  -F "notes=Large trawler using illegal nets"
```

### List Reports

```bash
# Basic list
curl -X GET 'https://api.bahariwatch.ke/v1/reports?limit=20' \
  -H "Authorization: Bearer <token>"

# With filters
curl -X GET 'https://api.bahariwatch.ke/v1/reports?$filter=status eq "verified" and county eq "Kilifi"&$orderby=timestamp desc&$count=true' \
  -H "Authorization: Bearer <token>"
```

### Search Vessels

```bash
curl -X POST https://api.bahariwatch.ke/v1/vessels/search \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "location": {"latitude": -4.0435, "longitude": 39.6682},
    "radius": 10,
    "startTime": "2025-05-15T14:00:00Z",
    "endTime": "2025-05-15T15:00:00Z"
  }'
```

## 🔍 Query Parameters

### Filtering ($filter)

Uses OData-style expressions:

```bash
# Equal
status eq 'verified'

# Comparison
location/latitude gt -4.5

# Logical operators
status eq 'verified' and timestamp gt '2025-01-01'

# String functions
contains(notes, 'trawler')
```

### Sorting ($orderby)

```bash
timestamp desc
county asc, timestamp desc
```

### Pagination

```bash
# First page
/reports?limit=20

# Next page
/reports?limit=20&cursor=eyJpZCI6InJwdC0wMDIifQ

# With count
/reports?limit=20&$count=true
```

## 🌐 Supported Languages

Use the `Accept-Language` header for localization:

```bash
# English (default)
curl -H "Accept-Language: en" ...

# Swahili
curl -H "Accept-Language: sw" ...
```

## 🔄 Async Operations

Use the `Prefer` header for long-running operations:

```bash
# Return immediately with 202 Accepted
curl -X POST https://api.bahariwatch.ke/v1/sync/reports/batch \
  -H "Prefer: respond-async" \
  -H "Authorization: Bearer <token>" \
  -d '@batch_reports.json'

# Wait up to 30 seconds before returning 202
curl -X POST https://api.bahariwatch.ke/v1/matches/trigger \
  -H "Prefer: wait=30" \
  -H "Authorization: Bearer <token>"
```

## 🔐 Webhook Security

All webhook payloads include an `X-Webhook-Signature` header:

```python
import hmac
import hashlib

def verify_webhook(request, webhook_secret):
    signature = hmac.new(
        webhook_secret.encode(),
        request.body,
        hashlib.sha256
    ).hexdigest()
    
    expected = f"sha256={signature}"
    received = request.headers.get('X-Webhook-Signature')
    
    return hmac.compare_digest(expected, received)
```

## 📊 Rate Limiting

- **Community reporters:** 10 reports per hour
- **BMU officers:** 100 requests per minute
- **Enforcement agents:** 200 requests per minute

Check rate limit headers in responses:

```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 7
X-RateLimit-Reset: 1684156800
```

## 🏷️ API Endpoints

### Core Resources

- 📋 **Reports** - Community incident reporting
- 🚢 **Vessels** - AIS/VMS vessel tracking
- 🔗 **Matches** - Report-vessel correlations
- 🚨 **Alerts** - Enforcement notifications
- 👮 **Enforcement** - Patrol logs and actions

### Supporting Resources

- 👥 **Users** - Authentication and profiles
- 📍 **Locations** - Hotspots and geographic data
- 📈 **Analytics** - Dashboard metrics
- 🔔 **Webhooks** - Real-time notifications
- 📜 **Audit** - Compliance trails
- 🔍 **Search** - Cross-entity queries
- 📦 **Batch** - Bulk operations

## 🧪 Testing

```bash
# Health check
curl https://api.bahariwatch.ke/v1/health

# Readiness check
curl https://api.bahariwatch.ke/v1/health/ready

# Version info
curl https://api.bahariwatch.ke/v1/version
```

## 📄 License

MIT License - See LICENSE file for details

## 👥 Contact

- **Name:** Joshua Kimathi
- **Email:** kimathijosh286@gmail.com

## 🤝 Contributing

This API is part of a research project on community-driven IUU fishing prevention in Kenya's coastal waters. For inquiries about collaboration or data access, please contact the team.

---

**Version:** 1.0.0  
**Last Updated:** October 1, 2025  
**Compliance:** Microsoft REST API Guidelines, RFC 7807, OData
