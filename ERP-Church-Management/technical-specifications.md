# Technical Specifications -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. API Reference

### 1.1 Global Contracts

**Base URL**: `https://{host}/v1/{service}/`

**Authentication**: Bearer JWT in `Authorization` header

**Multi-tenancy**: `X-Tenant-ID` header required on all business routes

**Correlation**: `X-Correlation-ID` header (auto-generated if absent)

### 1.2 Standard Response Format

```json
{
  "success": true,
  "message": "string",
  "data": {},
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 50,
    "totalPages": 2
  }
}
```

---

## 2. Member Service API

### 2.1 Endpoints

| Method | Path | Description | Roles |
|---|---|---|---|
| POST | /members | Create member | admin, pastor |
| GET | /members | List members (paginated) | admin, pastor, minister, HOD |
| GET | /members/:id | Get member by ID | admin, pastor, account_officer (assigned) |
| PUT | /members/:id | Update member | admin, pastor |
| DELETE | /members/:id | Delete member | admin |
| GET | /members/search?query= | Search members | admin, pastor, account_officer |
| GET | /members/statistics | Get member statistics | admin, pastor, directorate_head |
| GET | /members/absentees?weeks=3 | Get absentee members | admin, pastor, directorate_head |
| GET | /members/:id/attendance | Get member attendance | admin, pastor, account_officer |
| GET | /members/:id/donations | Get member giving history | admin, member (self) |
| GET | /members/:id/groups | Get member groups | admin, member (self) |
| PUT | /members/:id/natural-group | Update natural group | admin |
| POST | /members/:id/account-officer | Assign account officer | admin, directorate_head |
| PUT | /members/:id/status | Update member status | admin, pastor |

### 2.2 Member Object Schema

```json
{
  "id": "uuid",
  "tenantId": "uuid",
  "membershipId": "MEM000001",
  "title": "Mr|Mrs|Dr|Pastor|Rev|Elder",
  "firstName": "string (required)",
  "lastName": "string (required)",
  "middleName": "string",
  "email": "string (email format)",
  "phone": "string (required, E.164)",
  "alternativePhone": "string",
  "whatsappNumber": "string",
  "telegramUsername": "string",
  "facebookMessengerId": "string",
  "dateOfBirth": "date (YYYY-MM-DD)",
  "gender": "Male|Female",
  "maritalStatus": "Single|Married|Widowed|Divorced",
  "address": "string",
  "city": "string",
  "state": "string",
  "country": "string",
  "postalCode": "string",
  "memberType": "New Believer|Member|Prospect|Worker|Minister|Pastor",
  "memberStatus": "Active|Inactive|Transferred|Deceased",
  "naturalGroup": "Youth|Men|Women|Elders|Teens|Children",
  "joinDate": "date",
  "salvationDate": "date",
  "baptismDate": "date",
  "householdId": "uuid",
  "communicationPreferences": {},
  "engagementScore": "decimal (0-100)",
  "createdAt": "timestamp",
  "updatedAt": "timestamp"
}
```

---

## 3. Visitor Service API

| Method | Path | Description |
|---|---|---|
| POST | /visitors | Create visitor |
| GET | /visitors | List visitors (paginated) |
| GET | /visitors/:id | Get visitor by ID |
| PUT | /visitors/:id | Update visitor |
| DELETE | /visitors/:id | Delete visitor |
| GET | /visitors/first-timers | Get first-time visitors |
| GET | /visitors/pending-followup | Get visitors pending follow-up |
| GET | /visitors/72-hour-status | Get 72-hour follow-up status |
| POST | /visitors/:id/follow-up | Record follow-up activity |
| GET | /visitors/:id/follow-ups | Get visitor follow-up history |
| POST | /visitors/:id/convert | Convert visitor to member |
| POST | /visitors/:id/account-officer | Assign account officer |
| PUT | /visitors/:id/status | Update visitor status |
| POST | /visitors/:id/welcome-gift | Record welcome gift |
| POST | /visitors/:id/72-hour-contact | Record 72-hour contact |
| GET | /visitors/:id/72-hour-report | Get 72-hour completion report |

---

## 4. Follow-up Service API

| Method | Path | Description |
|---|---|---|
| POST | /followups | Create follow-up activity |
| GET | /followups | List follow-ups (filtered) |
| GET | /followups/:id | Get follow-up by ID |
| PUT | /followups/:id | Update follow-up |
| GET | /followups/by-officer/:officerId | Get officer's follow-ups |
| GET | /followups/by-directorate/:directorate | Get directorate follow-ups |
| POST | /followups/assign-officer | Assign account officer |
| PUT | /followups/:id/complete | Mark follow-up complete |
| GET | /followups/pending | Get all pending follow-ups |
| POST | /followups/route | Route to directorate |

---

## 5. Giving Service API

| Method | Path | Description |
|---|---|---|
| POST | /giving/donations | Record donation |
| GET | /giving/donations | List donations (paginated) |
| GET | /giving/donations/:id | Get donation by ID |
| GET | /giving/donations/by-member/:memberId | Member giving history |
| GET | /giving/donations/summary | Giving summary (by type, period) |
| POST | /giving/pledges | Create pledge |
| GET | /giving/pledges | List pledges |
| GET | /giving/pledges/:id | Get pledge by ID |
| PUT | /giving/pledges/:id/payment | Record pledge payment |
| POST | /giving/campaigns | Create pledge campaign |
| GET | /giving/campaigns | List campaigns |
| GET | /giving/statements/:memberId/:year | Generate giving statement |

---

## 6. Event Service API

| Method | Path | Description |
|---|---|---|
| POST | /events | Create event |
| GET | /events | List events (filtered) |
| GET | /events/:id | Get event by ID |
| PUT | /events/:id | Update event |
| DELETE | /events/:id | Delete event |
| POST | /events/:id/checkin | Check in to event |
| GET | /events/:id/attendance | Get event attendance |
| GET | /events/:id/attendance/count | Get attendance count |
| POST | /events/:id/checkout | Check out of event |

---

## 7. Communication Service API

| Method | Path | Description |
|---|---|---|
| POST | /communications/send | Send message |
| POST | /communications/broadcast | Broadcast to group |
| GET | /communications | List communications |
| GET | /communications/:id | Get communication by ID |
| GET | /communications/:id/status | Get delivery status |
| POST | /communications/templates | Create message template |
| GET | /communications/templates | List templates |

---

## 8. KPI Service API

| Method | Path | Description |
|---|---|---|
| GET | /kpis | Get all KPIs |
| GET | /kpis/dashboard | Get dashboard summary |
| GET | /kpis/by-category/:category | Get KPIs by category |
| GET | /kpis/by-period?start=&end= | Get KPIs for period |
| GET | /kpis/directorate/:directorate | Get directorate KPIs |
| POST | /kpis/calculate | Trigger KPI recalculation |
| GET | /kpis/trends | Get KPI trends |

---

## 9. Domain Event Specifications

### 9.1 Event Envelope

```json
{
  "event_id": "550e8400-e29b-41d4-a716-446655440000",
  "event_type": "visitor.created",
  "version": "1.0",
  "tenant_id": "tenant-uuid",
  "timestamp": "2026-02-23T10:00:00.000Z",
  "correlation_id": "corr-uuid",
  "source": "visitor-service",
  "payload": {}
}
```

### 9.2 Event Catalog

| Event Type | Payload Fields | Topic |
|---|---|---|
| `visitor.created` | id, firstName, lastName, phone, email, visitDate, isFirstTime | church.visitor.events |
| `visitor.72hr.contact.recorded` | visitorId, contactMethod, contactedBy, contactDate | church.visitor.events |
| `visitor.converted` | visitorId, memberId, membershipId, conversionDate | church.visitor.events |
| `member.created` | id, membershipId, firstName, lastName, memberType, naturalGroup | church.member.events |
| `member.absentee.detected` | memberId, lastAttendanceDate, weeksAbsent | church.member.events |
| `followup.assigned` | followupId, accountOfficerId, targetId, targetType | church.followup.events |
| `giving.recorded` | donationId, memberId, givingType, amount, currency | church.giving.events |
| `event.checkin` | eventId, memberId, checkInTime, method | church.event.events |
| `welfare.case.created` | caseId, memberId, category, urgency | church.welfare.events |
| `kpi.calculated` | kpiId, name, category, target, actual, status | church.kpi.events |

---

## 10. Rate Limits

| Endpoint Category | Rate Limit | Window |
|---|---|---|
| Read (GET) | 100 requests | Per minute per tenant |
| Write (POST/PUT/DELETE) | 30 requests | Per minute per tenant |
| Search | 20 requests | Per minute per user |
| Communication send | 10 requests | Per minute per user |
| Bulk operations | 5 requests | Per minute per tenant |

---

## 11. Pagination Specification

All list endpoints support cursor-based or offset-based pagination:

**Query Parameters**:
- `page` (integer, default: 1) -- Page number
- `limit` (integer, default: 50, max: 200) -- Items per page
- `sort` (string) -- Sort field
- `order` (string, "ASC"|"DESC", default: "DESC") -- Sort direction

**Response Pagination Object**:
```json
{
  "total": 1500,
  "page": 3,
  "limit": 50,
  "totalPages": 30
}
```

---

## 12. Error Codes

| HTTP Status | Error Code | Description |
|---|---|---|
| 400 | BAD_REQUEST | Invalid request body or parameters |
| 400 | MISSING_TENANT | X-Tenant-ID header not provided |
| 401 | UNAUTHORIZED | Missing or invalid JWT token |
| 403 | FORBIDDEN | Insufficient role permissions |
| 403 | ENTITLEMENT_DENIED | Module not entitled for tenant |
| 404 | NOT_FOUND | Resource does not exist |
| 409 | CONFLICT | Duplicate resource (e.g., membership ID) |
| 422 | UNPROCESSABLE | Business rule violation |
| 429 | RATE_LIMITED | Too many requests |
| 500 | INTERNAL_ERROR | Unexpected server error |
| 503 | SERVICE_UNAVAILABLE | Upstream service unreachable |
