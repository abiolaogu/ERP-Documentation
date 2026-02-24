# API Reference -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. API Overview

All API endpoints are accessed through the Go API gateway at `https://{host}:8090`. Every request (except `/healthz` and `/v1/capabilities`) requires:
- `Authorization: Bearer {jwt_token}` header
- `X-Tenant-ID: {tenant_uuid}` header

The gateway routes requests to the appropriate microservice based on the URL path prefix: `/v1/{service}/{resource}`.

---

## 2. Gateway Endpoints

### 2.1 Health Check

```
GET /healthz
```

**Authentication**: None required

**Response 200**:
```json
{
  "status": "healthy",
  "module": "ERP-Church-Management"
}
```

### 2.2 Capabilities

```
GET /v1/capabilities
```

**Authentication**: None required

**Response 200**:
```json
{
  "module": "ERP-Church-Management",
  "version": "1.0.0",
  "capabilities": [
    "member-management",
    "visitor-assimilation",
    "followup-ministry",
    "giving-management",
    "event-management",
    "group-management",
    "discipleship-tracking",
    "welfare-management",
    "multi-channel-communication",
    "kpi-dashboard",
    "volunteer-management",
    "facility-management"
  ],
  "integration_mode": "standalone_plus_suite",
  "aidd_governance": "enforced"
}
```

---

## 3. Member Service Endpoints

### 3.1 Create Member

```
POST /v1/member/members
```

**Request Body**:
```json
{
  "title": "Mr",
  "firstName": "John",
  "lastName": "Adewale",
  "middleName": "Oluwaseun",
  "email": "john.adewale@email.com",
  "phone": "+2348012345678",
  "whatsappNumber": "+2348012345678",
  "telegramUsername": "john_adewale",
  "dateOfBirth": "1990-05-15",
  "gender": "Male",
  "maritalStatus": "Married",
  "address": "123 Church Street",
  "city": "Lagos",
  "state": "Lagos",
  "country": "Nigeria",
  "memberType": "New Believer",
  "naturalGroup": "Men",
  "salvationDate": "2026-01-15"
}
```

**Response 201**:
```json
{
  "success": true,
  "message": "Member created successfully",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "membershipId": "MEM000042",
    "firstName": "John",
    "lastName": "Adewale",
    "memberType": "New Believer",
    "memberStatus": "Active",
    "naturalGroup": "Men",
    "createdAt": "2026-02-23T10:00:00.000Z"
  }
}
```

### 3.2 List Members

```
GET /v1/member/members?page=1&limit=50&memberType=Active&naturalGroup=Men&search=John
```

**Query Parameters**:

| Parameter | Type | Default | Description |
|---|---|---|---|
| page | integer | 1 | Page number |
| limit | integer | 50 | Items per page (max 200) |
| memberType | string | - | Filter by member type |
| memberStatus | string | - | Filter by status |
| naturalGroup | string | - | Filter by natural group |
| search | string | - | Search across name, ID, email, phone |

**Response 200**:
```json
{
  "success": true,
  "data": [...],
  "pagination": {
    "total": 2450,
    "page": 1,
    "limit": 50,
    "totalPages": 49
  }
}
```

### 3.3 Get Member by ID

```
GET /v1/member/members/:id
```

**Response 200**: Full member object with relations (user, accountOfficers, smallGroups, ministries, mentor)

### 3.4 Update Member

```
PUT /v1/member/members/:id
```

**Request Body**: Partial member fields to update

### 3.5 Delete Member

```
DELETE /v1/member/members/:id
```

### 3.6 Member Statistics

```
GET /v1/member/members/statistics
```

**Response 200**:
```json
{
  "success": true,
  "data": {
    "totalMembers": 2450,
    "activeMembers": 2100,
    "inactiveMembers": 350,
    "byNaturalGroup": [
      {"naturalGroup": "Men", "count": 650},
      {"naturalGroup": "Women", "count": 720},
      {"naturalGroup": "Youth", "count": 480},
      {"naturalGroup": "Teens", "count": 200},
      {"naturalGroup": "Children", "count": 300},
      {"naturalGroup": "Elders", "count": 100}
    ],
    "byMemberType": [
      {"memberType": "Member", "count": 1800},
      {"memberType": "Worker", "count": 400},
      {"memberType": "New Believer", "count": 150},
      {"memberType": "Minister", "count": 80},
      {"memberType": "Pastor", "count": 20}
    ]
  }
}
```

### 3.7 Absentee Members

```
GET /v1/member/members/absentees?weeks=3
```

### 3.8 Assign Account Officer

```
POST /v1/member/members/:id/account-officer
```

**Request Body**:
```json
{
  "accountOfficerId": "uuid-of-account-officer"
}
```

---

## 4. Visitor Service Endpoints

### 4.1 Create Visitor

```
POST /v1/visitor/visitors
```

### 4.2 Get First-Timers

```
GET /v1/visitor/visitors/first-timers
```

### 4.3 72-Hour Status

```
GET /v1/visitor/visitors/72-hour-status
```

**Response 200**:
```json
{
  "success": true,
  "data": {
    "pending": [...],
    "completed": [...],
    "completionRate": 85.5
  }
}
```

### 4.4 Record 72-Hour Contact

```
POST /v1/visitor/visitors/:id/72-hour-contact
```

**Request Body**:
```json
{
  "contactMethod": "WhatsApp",
  "notes": "Visitor was very receptive. Plans to attend next Sunday."
}
```

### 4.5 Convert to Member

```
POST /v1/visitor/visitors/:id/convert
```

**Request Body**:
```json
{
  "naturalGroup": "Youth",
  "salvationDate": "2026-02-20"
}
```

---

## 5. Follow-up Service Endpoints

### 5.1 List Follow-ups

```
GET /v1/followup/followups?status=Pending&directorate=1st Timer&accountOfficerId=uuid
```

### 5.2 Create Follow-up Activity

```
POST /v1/followup/followups
```

**Request Body**:
```json
{
  "visitorId": "uuid",
  "activityType": "Home Visit",
  "directorate": "1st Timer",
  "scheduledDate": "2026-02-25",
  "notes": "Schedule first home visit"
}
```

### 5.3 Route to Directorate

```
POST /v1/followup/followups/route
```

**Request Body**:
```json
{
  "followupId": "uuid",
  "targetDirectorate": "Further Follow-up",
  "reason": "Visitor needs additional engagement"
}
```

---

## 6. Giving Service Endpoints

### 6.1 Record Donation

```
POST /v1/giving/donations
```

**Request Body**:
```json
{
  "memberId": "uuid",
  "givingType": "Tithe",
  "amount": 50000.00,
  "currency": "NGN",
  "paymentMethod": "Bank Transfer",
  "donationDate": "2026-02-23",
  "referenceNumber": "TRF-2026-001234"
}
```

### 6.2 Giving Statement

```
GET /v1/giving/statements/:memberId/2025
```

**Response 200**: PDF download or JSON giving summary for the fiscal year.

---

## 7. Event Service Endpoints

### 7.1 Event Check-In

```
POST /v1/event/events/:id/checkin
```

**Request Body**:
```json
{
  "memberId": "uuid",
  "checkInMethod": "QR Code"
}
```

---

## 8. Communication Service Endpoints

### 8.1 Send Message

```
POST /v1/communication/send
```

**Request Body**:
```json
{
  "recipients": ["member-uuid-1", "member-uuid-2"],
  "channels": ["sms", "whatsapp", "email"],
  "subject": "Service Reminder",
  "body": "Don't forget about our special service this Sunday at 9 AM!",
  "scheduledAt": null
}
```

---

## 9. KPI Service Endpoints

### 9.1 Dashboard Summary

```
GET /v1/kpi/dashboard
```

**Response 200**:
```json
{
  "success": true,
  "data": [
    {
      "name": "72-Hour Contact Completion Rate",
      "category": "72-Hour Contact",
      "target": 90,
      "actual": 85.5,
      "unit": "%",
      "status": "On Track",
      "period": "Weekly"
    },
    {
      "name": "Visitor Conversion Rate",
      "category": "Visitor Conversion",
      "target": 60,
      "actual": 52.3,
      "unit": "%",
      "status": "On Track",
      "period": "Monthly"
    }
  ]
}
```

---

## 10. Common Error Responses

```json
{
  "success": false,
  "message": "Member not found",
  "error": "No member with ID 550e8400-... exists in tenant abc123"
}
```

| Status | Meaning |
|---|---|
| 400 | Bad Request -- Invalid input or missing required headers |
| 401 | Unauthorized -- Missing or invalid JWT |
| 403 | Forbidden -- Insufficient permissions or entitlement denied |
| 404 | Not Found -- Resource does not exist |
| 409 | Conflict -- Duplicate resource |
| 422 | Unprocessable Entity -- Business rule violation |
| 429 | Too Many Requests -- Rate limit exceeded |
| 500 | Internal Server Error |
| 503 | Service Unavailable -- Upstream service down |
