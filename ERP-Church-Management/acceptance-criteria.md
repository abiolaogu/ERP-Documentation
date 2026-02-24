# Acceptance Criteria -- ERP-Church-Management
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Acceptance Criteria Overview

This document defines testable acceptance criteria for each feature of ERP-Church-Management. Each criterion follows the Given-When-Then format and maps to specific user stories and use cases.

---

## 2. Gateway Acceptance Criteria

### AC-GW-001: Health Check
- **Given** the gateway is running
- **When** a GET request is sent to `/healthz`
- **Then** the response status is 200 and body contains `{"status":"healthy","module":"ERP-Church-Management"}`

### AC-GW-002: Capabilities Endpoint
- **Given** the gateway is running with a capabilities config
- **When** a GET request is sent to `/v1/capabilities`
- **Then** the response contains the module name, capabilities list, and integration mode

### AC-GW-003: Tenant Enforcement
- **Given** a valid JWT is provided
- **When** a business route request is sent WITHOUT `X-Tenant-ID` header
- **Then** the response status is 400 with error "missing X-Tenant-ID"

### AC-GW-004: JWT Enforcement
- **Given** no Authorization header is provided
- **When** a business route request is sent
- **Then** the response status is 401 with error "missing/invalid bearer token"

### AC-GW-005: Service Routing
- **Given** a valid JWT and X-Tenant-ID are provided
- **When** a request is sent to `/v1/member/any-path`
- **Then** the request is proxied to `http://member-service:8080/any-path`

---

## 3. Member Service Acceptance Criteria

### AC-MEM-001: Create Member
- **Given** an admin user is authenticated
- **When** a POST request is sent to `/v1/member/members` with valid member data
- **Then** a member record is created with an auto-generated membership ID (format: MEM000XXX)
- **And** the response status is 201

### AC-MEM-002: Search Members
- **Given** members exist with name "John Smith"
- **When** a GET request is sent to `/v1/member/members/search?query=John`
- **Then** the response includes members matching "John" in first name, last name, or membership ID
- **And** results are limited to the requesting tenant's data

### AC-MEM-003: Absentee Detection
- **Given** a member has not attended any event in the past 3 weeks
- **When** the absentee checker runs
- **Then** the member appears in the absentee list
- **And** a follow-up activity of type "Absentee Check" is created

### AC-MEM-004: Natural Group Assignment
- **Given** a member exists
- **When** their natural group is updated to "Youth"
- **Then** the member appears in Youth natural group queries
- **And** does not appear in previous natural group queries

### AC-MEM-005: Tenant Isolation
- **Given** members exist in Tenant A and Tenant B
- **When** Tenant A queries all members
- **Then** only Tenant A members are returned (zero Tenant B members)

---

## 4. Visitor Service Acceptance Criteria

### AC-VIS-001: Register First-Timer
- **Given** a visitor has never been registered
- **When** a POST request creates the visitor with `isFirstTime: true`
- **Then** the visitor record is created with status "New"
- **And** a `visitor.created` event is published

### AC-VIS-002: 72-Hour Follow-up Trigger
- **Given** a visitor was registered 2 hours ago with no contact
- **When** the hourly cron job runs
- **Then** the visitor is identified as needing follow-up
- **And** multi-channel outreach is initiated

### AC-VIS-003: 72-Hour Contact Recording
- **Given** a visitor exists within the 72-hour window
- **When** an Account Officer records a 72-hour contact
- **Then** `contactedWithin72Hours` is set to true
- **And** the 72-hour KPI is updated

### AC-VIS-004: Visitor-to-Member Conversion
- **Given** a visitor with complete information exists
- **When** the convert-to-member endpoint is called
- **Then** a new member record is created with all visitor data
- **And** the visitor status is updated to "Converted to Member"
- **And** the membership ID follows the MEM000XXX format
- **And** the visitor conversion KPI is updated

### AC-VIS-005: Welcome Gift Tracking
- **Given** a visitor exists
- **When** a welcome gift is recorded
- **Then** `welcomeGiftGiven` is true and `welcomeGiftDate` is set

---

## 5. Follow-up Service Acceptance Criteria

### AC-FU-001: Auto-Assign Account Officer
- **Given** a `visitor.created` event is consumed
- **When** the follow-up service processes the event
- **Then** the Account Officer with the fewest active assignments is selected
- **And** an assignment record is created with status "Active"

### AC-FU-002: Directorate Routing
- **Given** a visitor has been contacted but needs further follow-up
- **When** the case is routed to "Further Follow-up" directorate
- **Then** the follow-up record's directorate field is updated
- **And** the directorate head is notified

### AC-FU-003: Follow-up Activity Logging
- **Given** an Account Officer performs a home visit
- **When** they record the activity with type "Home Visit" and notes
- **Then** the activity is stored with timestamp and performer ID
- **And** the status can be tracked (Pending, In Progress, Completed)

---

## 6. Giving Service Acceptance Criteria

### AC-GIV-001: Record Tithe
- **Given** a member exists
- **When** a tithe of $1,000 is recorded via cash
- **Then** a donation record is created with giving_type "Tithe"
- **And** a tax receipt is generated
- **And** a `giving.recorded` event is published

### AC-GIV-002: Pledge Fulfillment
- **Given** a member has a pledge of $10,000
- **When** a payment of $2,500 is recorded against the pledge
- **Then** `fulfilled_amount` increases by $2,500
- **And** fulfillment percentage is 25%

### AC-GIV-003: Annual Statement
- **Given** a member has giving records for fiscal year 2025
- **When** a statement is requested for 2025
- **Then** a PDF is generated with all tax-deductible giving grouped by type
- **And** the total matches the sum of individual records

---

## 7. Event Service Acceptance Criteria

### AC-EVT-001: Event Check-In
- **Given** a Sunday Service event exists today
- **When** a member scans their QR code
- **Then** an attendance record is created with check_in_method "QR Code"
- **And** the event's actual_attendance count increments

### AC-EVT-002: First-Timer Check-In
- **Given** a person checks in who is not a member or visitor
- **When** they are registered at the kiosk
- **Then** a visitor record is created
- **And** the attendance record links to the visitor
- **And** the 4Cs workflow is initiated

---

## 8. Communication Service Acceptance Criteria

### AC-COM-001: Multi-Channel Delivery
- **Given** a member has WhatsApp and Email preferences enabled
- **When** a message is sent
- **Then** the message is delivered via WhatsApp
- **And** the message is delivered via Email
- **And** delivery status is tracked per channel

### AC-COM-002: Channel Fallback
- **Given** WhatsApp delivery fails
- **When** fallback is configured
- **Then** the message is automatically retried via SMS
- **And** the failure is logged

---

## 9. KPI Service Acceptance Criteria

### AC-KPI-001: 72-Hour KPI Calculation
- **Given** 10 visitors registered this week, 8 contacted within 72 hours
- **When** the KPI calculator runs
- **Then** the 72-hour contact completion rate KPI shows 80%
- **And** status is "On Track" (between 70-89%)

### AC-KPI-002: Dashboard Display
- **Given** KPIs have been calculated
- **When** a pastor views the KPI dashboard
- **Then** all 5 quarterly shepherding KPIs are displayed
- **And** each shows target, actual, and traffic-light status

---

## 10. Cross-Cutting Acceptance Criteria

### AC-SEC-001: Role-Based Access
- **Given** a user with role "member"
- **When** they attempt to access `/v1/member/members` (list all)
- **Then** they receive a 403 Forbidden response

### AC-SEC-002: Data Privacy
- **Given** a member requests data export
- **When** the export is generated
- **Then** it contains all personal data held by the system
- **And** is delivered securely

### AC-PERF-001: Response Time
- **Given** the system is under normal load
- **When** any API endpoint is called
- **Then** the p95 response time is < 200ms

### AC-PERF-002: Sunday Peak
- **Given** the system is under Sunday peak load (10x baseline)
- **When** check-in endpoints are called concurrently by 500 users
- **Then** all requests complete successfully with p99 < 1s
