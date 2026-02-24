# Figma & Make Prompts -- ERP-IAM (Identity & Access Management)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-IAM module. ERP-IAM is the Identity and Access Management vertical of the consolidated ERP platform, covering identity lifecycle management, directory services, session management, device trust assessment, mobile device management (MDM), credential vault, automated provisioning/deprovisioning, and comprehensive audit logging.

All prompts reference real services (identity-service, directory-service, session-service, device-trust-service, mdm-service, credential-vault-service, provisioning-service, audit-service) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`. The module integrates with ERP-Platform as the centralized identity provider (`ERP-Directory`) for all other ERP modules, using NATS as the event backbone.

The platform supports standalone and suite-integrated deployment with Keycloak-based OAuth2/OIDC, OAuth2-Proxy, and a Flutter-based MFA authenticator app.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance; color contrast >= 4.5:1 text, >= 3:1 UI components
- Minimum 44x44px touch targets on all interactive elements
- Clear focus-visible outlines (2px solid) for keyboard navigation
- Plain-language security labels (e.g., "Sign In" not "Authenticate", "Trusted Device" not "Device Trust Score >= 0.8")
- Semantic HTML landmarks reflected in Figma layer naming
- Screen reader annotations for security status charts, access graphs, and audit timelines
- Security-sensitive forms: password strength meters, MFA setup wizards with clear step indicators

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for user directories > 100 entries, audit logs > 500 entries
- Login page: < 100 KB total (critical path optimization)
- Session status checks: < 50ms (cached, no UI blocking)

### 2.3 Reliability, Trust, And Safety
- Every privilege escalation (role change, admin grant, policy modification) requires confirmation with re-authentication
- MFA enrollment flows: clear recovery path if device is lost (backup codes, admin override)
- Trust signals: session activity indicator, last login time, device fingerprint verification
- Emergency access: "Break glass" procedure for locked-out administrators with full audit
- All credential operations (create, rotate, revoke) require human-in-the-loop per AIDD guardrails
- No cross-tenant data access; tenant isolation enforced at identity-service level

### 2.4 Observability And Testability
- Every authentication event emits: `{event, identity_id, method, device_id, ip, geo, tenant_id, timestamp, success}`
- Every authorization decision emits: `{event, identity_id, resource, action, policy_id, decision, timestamp}`
- Error boundaries with correlation_id for support escalation
- Performance instrumentation: login flow timing, MFA completion rate, session establishment time
- Feature flag integration: "Visible when `passwordless_enabled` = true"

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens -- IAM Security Theme

```
Create a Figma page titled "IAM Design Tokens" containing the complete token
specification for the Identity & Access Management module.

COLOR PALETTE (Light Theme):
- Primary: #1E3A5F (Deep Navy -- authority, security, trust)
- Primary Hover: #162D4A
- Primary Light: #E8EEF4
- Accent: #0EA5E9 (Sky Blue -- interactive elements, links, active states)
- Success: #059669 (Authenticated, Active, Verified, Compliant)
- Warning: #D97706 (Expiring session, Pending MFA, Weak password)
- Danger: #DC2626 (Failed auth, Revoked, Compromised, Policy violation)
- Info: #6366F1 (Indigo -- informational badges, policy details)
- Secure-Green: #047857 (Encrypted, Trusted device, Strong authentication)
- Neutral-50: #F8FAFC (Page background)
- Neutral-100: #F1F5F9 (Card background)
- Neutral-200: #E2E8F0 (Borders)
- Neutral-500: #64748B (Secondary text)
- Neutral-900: #0F172A (Primary text)
- Surface: #FFFFFF

COLOR PALETTE (Dark Theme):
- Background: #0B1120 (Near-black -- security operations center)
- Surface: #111827
- Surface Elevated: #1F2937
- Primary: #60A5FA (Light Blue for visibility)
- Accent: #38BDF8
- Danger: #F87171
- Success: #34D399
- Warning: #FBBF24
- Text Primary: #F1F5F9
- Text Secondary: #9CA3AF
- Border: #374151

SECURITY STATUS COLORS (always paired with icon + label):
- Authenticated / Verified: #059669 + shield-check icon
- MFA Active: #047857 + lock icon
- Session Active: #0EA5E9 + circle-dot icon
- Warning / Expiring: #D97706 + alert-triangle icon
- Revoked / Blocked: #DC2626 + ban icon
- Inactive / Pending: #64748B + clock icon

TYPOGRAPHY (Inter font family):
- Display: 30px/36px, semibold 600 -- page titles ("Identity Directory")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Active Sessions")
- Heading 2: 20px/28px, medium 500 -- card titles ("Device Trust")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- table cells, descriptions
- Body Small: 12px/16px, regular 400 -- timestamps, metadata, helper text
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- UUIDs, IP addresses, tokens, audit IDs, device fingerprints

SPACING: 4px base (4, 8, 12, 16, 24, 32, 48px)
Card padding: 24px. Section gap: 32px. Page margin: 32px desktop, 16px mobile.

ELEVATION:
- Level 0: none
- Level 1: 0 1px 3px rgba(0,0,0,0.1) -- cards
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, dropdown panels
- Level 3: 0 8px 24px rgba(0,0,0,0.15) -- security alert overlays

BORDER RADIUS: 4px (badges), 8px (cards, inputs), 12px (modals), 9999px (avatars, status dots)

GRID:
- Desktop (1440px): 12-column, 72px col, 24px gutter, 260px sidebar
- Tablet (1024px): 12-column, 56px col, 16px gutter, collapsed sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include both themes side-by-side. Light is default for admin console; dark is
available for security operations center (SOC) monitoring.
```

#### Prompt F-002: Component Library -- IAM Components

```
Create a Figma page titled "IAM Component Library" with the following reusable
components. Show all states and light/dark variants.

IDENTITY CARD:
- 320px wide. Avatar (initials or photo), full name, email, identity ID (UUID truncated),
  status badge (Active/Suspended/Locked/Deprovisioned), MFA status icon,
  last login timestamp, tenant membership badges
- Quick actions: View Profile, Reset Password, Manage Sessions, Disable Account
- Example: "Ngozi Adeyemi | ngozi@company.com | IDN-a3f8...c721 | Active |
  MFA: TOTP enabled | Last login: 2 hours ago | Tenant: TechCorp"

SESSION CARD:
- Device icon (laptop/phone/tablet), OS + Browser/App,
  IP address (192.168.1.45), geo location (Lagos, Nigeria),
  session start time, last activity time, status (Active/Expired/Revoked)
- "Revoke" button (red, requires confirmation)
- Current session indicator: "This device" green badge
- Example: "MacBook Pro -- Chrome 120 -- 192.168.1.45 -- Lagos, NG --
  Started: Feb 23, 09:00 -- Last active: 2 min ago -- Active"

DEVICE TRUST BADGE:
- Trust score visualization (0-100 gauge or horizontal bar)
- Trust level: High (green, 80-100), Medium (amber, 50-79), Low (red, 0-49)
- Factors: OS updated (check/x), Encryption enabled (check/x),
  Antivirus active (check/x), Biometric available (check/x),
  MDM enrolled (check/x), Jailbreak detected (check/x)
- Example: Score 87/100, High Trust, 5 of 6 factors passing

CREDENTIAL CARD:
- Credential type icon: Password, API Key, OAuth Token, Certificate, SSH Key
- Name/identifier, created date, expiry date, last used date
- Status: Active (green), Expiring Soon (amber), Expired (red), Revoked (gray)
- Actions: Rotate, Revoke, Copy (for API keys, with masked display)
- Example: "API Key -- prod-analytics-key -- Created: Jan 15, 2026 --
  Expires: Jul 15, 2026 -- Last used: 1 hour ago -- Active"

POLICY RULE CARD:
- Policy name, description, target (role/group/user), effect (Allow/Deny)
- Conditions summary (IP range, time window, device trust level)
- Priority number, enabled/disabled toggle
- "Edit" and "Test" actions
- Example: "MFA Required for Admin Access | Require TOTP/WebAuthn for any
  user with admin role | Target: admin-group | Effect: Deny if MFA not completed"

AUDIT LOG ENTRY:
- Timestamp (mono font), actor (avatar + name or "System"), action verb,
  target resource, result (Success/Failure), IP address, correlation ID
- Expandable detail section with full request/response metadata
- Example: "Feb 23, 10:30:12 UTC | Ngozi Adeyemi | Logged in |
  Method: Password + TOTP | IP: 105.112.34.56 | Lagos, NG | Success | CID: req-abc123"

MFA SETUP WIZARD:
- Step progress: 1. Choose Method -> 2. Setup -> 3. Verify -> 4. Backup Codes
- Method options: TOTP Authenticator App, WebAuthn/FIDO2 Security Key, SMS (with security warning)
- QR code display for TOTP enrollment (256x256px)
- Verification code input (6-digit, spaced OTP input boxes)
- Backup codes display (8 codes, mono font, print/download)
- Example: Step 2 showing QR code with "Scan with Google Authenticator or ERP MFA app"

PASSWORD STRENGTH METER:
- Horizontal bar: Red (Weak) -> Amber (Fair) -> Yellow-Green (Good) -> Green (Strong)
- Requirements checklist below: min 12 chars, uppercase, lowercase, number, special char
- Real-time validation as user types
- Breach check indicator: "This password has not been found in known data breaches" (green check)

LOGIN PAGE COMPONENTS:
- Logo + tenant branding area
- Email/username input
- Password input with show/hide toggle
- "Sign In" primary button (full width)
- "Forgot Password" link
- "Sign in with SSO" button (for SAML/OIDC providers)
- MFA prompt screen (6-digit code input + "Use backup code" link)
- Error states: invalid credentials, account locked, MFA required

SIDEBAR NAVIGATION (IAM):
- Sections: Dashboard, Identities, Groups & Roles, Sessions, Device Trust,
  Credentials Vault, Provisioning, Policies, Audit Log, Settings
- Active state: Primary background + white text + left accent
- Badge counts: Active Sessions (342), Pending Provisions (8), Alerts (3)
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: IAM Admin Dashboard (1440px)

```
Design a desktop IAM Admin Dashboard at 1440x900px.
Target user: Security Administrator / IT Admin.

HEADER:
- "Identity & Access Management" title
- Global search (Cmd+K): search users, groups, sessions, audit logs
- Notification bell (3 security alerts), dark/light toggle, admin avatar
  "Ngozi Adeyemi | Security Administrator"

SIDEBAR (260px, collapsible): Per component library spec.

CONTENT -- ROW 1 (KPI Cards, 5 across):
- Card 1: "Total Identities" -- 4,832 -- +24 this month
- Card 2: "Active Sessions" -- 1,247 -- across 892 unique users
- Card 3: "MFA Adoption" -- 94.2% -- +2.1% this quarter (green trend)
- Card 4: "Failed Logins (24h)" -- 47 -- 12 from same IP (red flag icon)
- Card 5: "Pending Provisions" -- 8 -- 3 urgent (access requests)

CONTENT -- ROW 2 (Two-column, 55/45):
- Left (55%): "Authentication Activity" -- Area chart (last 7 days)
  X-axis: Days (Mon-Sun), Y-axis: Auth events
  Two series: Successful (green area) and Failed (red area, stacked)
  Mon: 4200/32, Tue: 4350/28, Wed: 4100/45, Thu: 4500/31,
  Fri: 4380/47, Sat: 890/12, Sun: 650/8
  Annotation: spike in failures on Wed flagged with "Brute force attempt detected"

- Right (45%): "Security Alerts" card (scrollable list):
  -- "Brute force: 45 failed attempts from 105.112.xx.xx targeting admin accounts" -- 2h ago (red)
  -- "Device trust: 12 users logged in from low-trust devices" -- 4h ago (amber)
  -- "Credential expiry: 23 API keys expire within 7 days" -- Today (amber)
  -- "Unusual location: Ngozi Adeyemi login from Nairobi (usually Lagos)" -- Yesterday (amber)
  -- "Compliance: 6% of users have not enabled MFA" -- Policy check (info)
  Each alert: severity icon, description, timestamp, "Investigate" link

CONTENT -- ROW 3 (Two-column, 50/50):
- Left: "Identity Distribution" -- Donut chart
  Employees: 3,200 (66%), Contractors: 450 (9%), Service Accounts: 892 (18%),
  External Partners: 290 (6%)
  Center text: "4,832 total"

- Right: "Top Authentication Methods" -- Horizontal bar chart
  Password + TOTP: 62%
  WebAuthn/FIDO2: 18%
  SSO (SAML): 12%
  Password only: 6% (red bar -- policy non-compliant)
  Passwordless: 2%

CONTENT -- ROW 4 (Full width):
- "Recent Audit Trail" (last 20 entries, compact table):
  | Timestamp | Actor | Action | Target | Method | IP | Location | Result |
  | Feb 23, 10:30:12 | Ngozi Adeyemi | Login | -- | Password+TOTP | 105.112.34.56 | Lagos | Success |
  | Feb 23, 10:28:45 | System | Provision | Blessing Eze | Auto | -- | -- | Success |
  | Feb 23, 10:25:01 | Unknown | Login | admin@company.com | Password | 203.45.67.89 | Unknown | Failed |
  | Feb 23, 10:22:18 | Chidi Nnamdi | Password Change | Self | -- | 105.112.35.12 | Lagos | Success |
  | Feb 23, 10:20:05 | Emeka Okafor | Role Grant | David Adeleke | Manual | 105.112.34.78 | Lagos | Success |
  "View Full Audit Log" link

CONTENT -- ROW 5 (Full width):
- "Device Trust Overview" -- Summary tiles:
  High Trust (80-100): 2,340 devices (green) | Medium Trust (50-79): 980 devices (amber) |
  Low Trust (0-49): 180 devices (red) | Unknown: 45 devices (gray)
  "12 devices flagged for jailbreak/root detection" -- investigate link

Both light and dark themes. Dark theme optimized for SOC monitoring displays.
```

#### Prompt F-004: Identity Directory Management (1440px)

```
Design two connected desktop screens at 1440x900px for the identity-service
and directory-service.

SCREEN A -- IDENTITY DIRECTORY:
- Top: "Identities" title, "Create Identity" primary button, search bar
  (search by name, email, UUID), filter chips
- Filters: Type (Employee/Contractor/Service Account/External), Status
  (Active/Suspended/Locked/Deprovisioned), MFA Status, Department, Tenant
- View toggle: Table (default) | Grid

TABLE:
| Avatar | Name / Email            | Identity ID (truncated) | Type         | Status   | MFA    | Last Login       | Groups | Actions |
|--------|-------------------------|-------------------------|--------------|----------|--------|------------------|--------|---------|
| [NG]   | Ngozi Adeyemi           | IDN-a3f8...c721        | Employee     | Active   | TOTP   | 2 hours ago      | 5      | ...     |
|        | ngozi@company.com       |                         |              |          |        |                  |        |         |
| [CD]   | Chidi Nnamdi            | IDN-b7e2...d934        | Employee     | Active   | WebAuthn| 30 min ago      | 3      | ...     |
|        | chidi@company.com       |                         |              |          |        |                  |        |         |
| [SVC]  | ci-pipeline-bot         | IDN-0f1a...e567        | Service Acct | Active   | N/A    | 5 min ago       | 2      | ...     |
|        | ci-bot@system.internal  |                         |              |          |        |                  |        |         |
| [FB]   | Fatima Bello            | IDN-c4d9...a123        | Employee     | Suspended| TOTP   | 14 days ago     | 4      | ...     |
|        | fatima@company.com      |                         |              |          |        |                  |        |         |
| [EXT]  | John Smith              | IDN-e8f5...b890        | External     | Active   | SMS    | Yesterday       | 1      | ...     |
|        | john@partner.co.uk      |                         |              |          |        |                  |        |         |

- Pagination: "Showing 1-50 of 4,832 identities"
- Bulk actions: "12 selected -- Suspend | Enable MFA | Deprovision | Export"

SCREEN B -- IDENTITY PROFILE DETAIL:
Back breadcrumb: "Identities > Ngozi Adeyemi"

LEFT PANEL (320px):
- Large avatar (96px), name, email, Identity ID (full UUID, mono font, copy button)
- Type: Employee | Status: Active (green badge)
- Department: IT Security | Manager: Emeka Okafor
- Created: Jan 15, 2024 | Last Modified: Feb 20, 2026
- Quick actions: Reset Password, Force MFA Re-enrollment, Suspend, Deprovision

CENTER PANEL (Tabbed):
Tab 1 "Profile": Personal info, contact details, organizational hierarchy,
  custom attributes (key-value pairs from directory-service)

Tab 2 "Authentication":
- Current methods: Password (last changed: Feb 1, 2026), TOTP (enrolled: Jan 2024)
- Password policy compliance: all checks passing (green)
- Login history (last 20 logins):
  | Date | Method | IP | Location | Device | Result |
  | Feb 23, 10:30 | Password+TOTP | 105.112.34.56 | Lagos | MacBook Chrome | Success |
  | Feb 23, 08:15 | Password+TOTP | 105.112.34.56 | Lagos | iPhone Safari | Success |
  | Feb 22, 16:45 | SSO | 105.112.34.56 | Lagos | MacBook Chrome | Success |
- "Force Password Reset" button, "Revoke All Sessions" button

Tab 3 "Groups & Roles":
- Group memberships: IT-Admin, Security-Team, All-Employees, VPN-Users, Cloud-Admin
- Role assignments: security_admin, identity_manager, audit_viewer
- Effective permissions matrix (computed from all groups/roles)
- "Add to Group" button, "Assign Role" button

Tab 4 "Sessions":
- Active sessions list (session-service):
  Each session card per component library spec
  "Revoke All" button, "Revoke Selected" button

Tab 5 "Devices":
- Registered devices (device-trust-service):
  | Device | OS | Trust Score | MDM Status | Last Seen | Actions |
  | MacBook Pro 16" | macOS 14.3 | 92/100 High | Managed | 2 min ago | Trust Details |
  | iPhone 15 Pro | iOS 17.3 | 88/100 High | Managed | 1 hour ago | Trust Details |
  | Unknown Linux | Ubuntu 22 | 34/100 Low | Not enrolled | 3 days ago | Block, Investigate |

Tab 6 "Credentials" (credential-vault-service):
- API Keys, OAuth tokens, certificates issued to this identity
- Each with status, expiry, last used, rotate/revoke actions

Tab 7 "Audit":
- Filtered audit log for this identity only (from audit-service)
- All authentication, authorization, and administrative events
- Exportable for compliance reporting

RIGHT PANEL (240px):
- "Risk Assessment" card: Low/Medium/High risk score based on:
  -- Login patterns, device trust, MFA status, access patterns
- "Provisioning Status": list of applications/resources provisioned
- "Compliance": MFA enabled (check), password policy (check), last training (check)
```

#### Prompt F-005: Session Management Console (1440px)

```
Design a desktop Session Management page at 1440x900px for the session-service.
Target user: Security Administrator monitoring active sessions.

HEADER:
- "Active Sessions" title with live count badge (1,247 active)
- "Revoke All Sessions" danger button (requires re-auth confirmation)
- Search: by user, IP, device, location
- Filter: Session age (< 1h, 1-4h, 4-8h, > 8h), Device type, Location, Trust level

REAL-TIME SESSION MAP (Top section, full width, 400px tall):
- World map (or focused on Africa) with dots for active session locations
- Dot size = number of sessions at location
- Dot color: green (normal), amber (unusual location), red (suspicious)
- Legend: "1,100 Lagos | 45 Abuja | 30 London | 22 Nairobi | 50 Other"
- Animated: new sessions appear with pulse effect
- Click on dot cluster: zoom to show individual sessions

SESSION TABLE (below map):
| User | Device / Browser | IP Address | Location | Started | Last Active | Duration | Trust | Status | Actions |
|------|-------------------|------------|----------|---------|-------------|----------|-------|--------|---------|
| Ngozi Adeyemi | MacBook / Chrome 120 | 105.112.34.56 | Lagos, NG | 09:00 | Now | 1h 30m | High (92) | Active | Revoke |
| Chidi Nnamdi | iPhone / Safari 17 | 105.112.35.12 | Lagos, NG | 08:45 | 5m ago | 1h 45m | High (88) | Active | Revoke |
| ci-pipeline-bot | API Client | 10.0.1.50 | Internal | 00:00 | Now | 10h 30m | N/A | Active | Revoke |
| John Smith | Windows / Edge 120 | 51.89.45.67 | London, UK | 08:00 | 15m ago | 2h 30m | Medium (65) | Active | Revoke |
| SUSPICIOUS | Unknown / curl | 203.45.67.89 | Unknown | 10:25 | 10:25 | 5m | Low (12) | Blocked | Investigate |

- Suspicious rows: red highlight with investigation icon
- Sortable by all columns
- Real-time updates: new rows animate in, expired rows fade out
- Bulk select + revoke for mass session termination

SESSION DETAIL PANEL (480px slide-in on click):
- Full session metadata: session ID (UUID), creation time, last refresh,
  expiry time, authentication method used, MFA status
- Device fingerprint details from device-trust-service
- Activity timeline: pages visited, actions performed, API calls made
- Geographic map pin (single session location)
- "Revoke Session" button with confirmation
- "Flag as Suspicious" button
- "Block IP" button (creates firewall rule)

ANOMALY DETECTION SECTION (collapsible, bottom):
- "Session Anomalies Detected" card:
  -- "3 sessions from previously unseen countries" -- Investigate
  -- "ci-pipeline-bot session running > 10h (usual < 2h)" -- Review
  -- "5 concurrent sessions for single user (Chidi Nnamdi) across 3 locations" -- Alert

Both light and dark SOC themes.
```

#### Prompt F-006: Credential Vault Management (1440px)

```
Design a desktop Credential Vault page at 1440x900px for the credential-vault-service.
Target user: DevOps Engineer / Security Admin managing secrets and API keys.

HEADER:
- "Credential Vault" title with lock icon
- "Create Credential" primary button (dropdown: API Key, OAuth Token, Certificate, SSH Key, Secret)
- Search and filter bar

SUMMARY CARDS (4 across):
- Total Credentials: 234
- Expiring Soon (30 days): 23 (amber badge)
- Expired: 7 (red badge)
- Last Rotation: "2 days ago"

CREDENTIAL TABLE:
| Type | Name | Owner | Created | Expires | Last Used | Last Rotated | Status | Actions |
|------|------|-------|---------|---------|-----------|--------------|--------|---------|
| API Key | prod-analytics-key | analytics-svc | Jan 15, 2026 | Jul 15, 2026 | 1 hour ago | Mar 15 | Active | Rotate, Revoke, Copy |
| Certificate | tls-api-gateway | infra-team | Dec 1, 2025 | Dec 1, 2026 | Always | Dec 1 | Active | Renew, Revoke |
| OAuth Token | github-ci-token | ci-pipeline-bot | Feb 1, 2026 | Aug 1, 2026 | 30 min ago | Feb 1 | Active | Rotate, Revoke |
| SSH Key | deploy-key-prod | deploy-svc | Nov 10, 2025 | None | 3 days ago | Jan 10 | Active | Rotate, Revoke |
| API Key | staging-test-key | dev-team | Jun 1, 2025 | Mar 1, 2026 | 2 weeks ago | Never | Expiring | Rotate NOW |
| Secret | db-password-prod | app-backend | Sep 1, 2025 | Sep 1, 2026 | Always | Feb 15 | Active | Rotate |
| API Key | old-integration | legacy-svc | Jan 1, 2024 | Jan 1, 2025 | Never | Never | EXPIRED | Delete, Reissue |

- Expired rows: red background, strikethrough name
- Expiring rows: amber background
- Never-used credentials: gray italic "Never" with "Clean up?" suggestion
- Type column: icons for each credential type (key, certificate, lock, terminal)

CREATE CREDENTIAL MODAL (600px):
- Step 1: Type selection (visual cards with icons)
- Step 2: Configuration
  -- Name (required), Description
  -- Owner (identity search), Authorized scopes/permissions
  -- Expiry: custom date or preset (30d, 90d, 180d, 1yr, no-expiry with warning)
  -- Rotation policy: manual, auto-rotate every N days
  -- IP restrictions: allowed CIDR ranges
- Step 3: Review and Create
- Step 4: Display Secret (ONCE -- masked by default, click to reveal, copy button)
  WARNING banner: "This is the only time this secret will be shown. Copy it now."
  "I have saved this credential" checkbox required to dismiss

ROTATION HISTORY PANEL (expandable per credential):
- Timeline of all rotations: date, initiated by (auto/manual), old key prefix, new key prefix
- "Force Rotate Now" button
- Auto-rotation status and next scheduled rotation

AUDIT SECTION:
- "Vault Access Log" -- who accessed what credential, when, from where
- Filtered view from audit-service for credential-vault operations only
```

#### Prompt F-007: Provisioning Workflows (1440px)

```
Design a desktop Provisioning page at 1440x900px for the provisioning-service.
Target user: IT Admin managing user access provisioning and deprovisioning.

HEADER:
- "Provisioning" title
- "New Provision Request" button
- Tabs: "Pending (8)" | "In Progress (3)" | "Completed" | "Failed"

PENDING REQUESTS TABLE:
| Request ID | Requester | Target User | Type | Resources | Priority | Submitted | SLA | Actions |
|-----------|-----------|-------------|------|-----------|----------|-----------|-----|---------|
| PRV-2026-0234 | Emeka Okafor (Manager) | David Adeleke | New Hire | Email, Slack, GitHub, AWS, Jira | Normal | Feb 22 | 24h | Approve, Reject |
| PRV-2026-0235 | HR System (Auto) | Blessing Eze | Role Change | Add: Admin Portal, Remove: Intern tools | Normal | Feb 22 | 24h | Approve, Reject |
| PRV-2026-0236 | Ngozi Adeyemi | ci-deploy-bot | Service Account | Kubernetes, Docker Registry, ArgoCD | High | Feb 23 | 4h | Approve, Reject |
| PRV-2026-0237 | HR System (Auto) | Fatima Bello | Offboarding | REVOKE ALL | Urgent | Feb 23 | 1h | EXECUTE, Hold |

- Offboarding rows: red accent border, "REVOKE ALL" in red badge
- SLA countdown timer for each request (turns red when < 1h remaining)

PROVISION DETAIL PANEL (480px slide-in):
- Request metadata: ID, requester, target user, type, submitted date
- Resource access matrix:
  | Resource | Current Access | Requested | Policy Match | Status |
  | Email (Google) | None | Read/Write | Auto-approve | Ready |
  | Slack | None | Member | Auto-approve | Ready |
  | GitHub | None | Developer | Requires approval | Pending |
  | AWS Console | None | PowerUser | Requires approval + MFA | Pending |
  | Jira | None | Developer | Auto-approve | Ready |
- "Approve All" button, "Approve Selected" button, "Reject with Reason" button
- Post-approval: progress tracker per resource (Queued -> Provisioning -> Verified)

DEPROVISIONING WORKFLOW (for offboarding):
- Checklist of all resources to be revoked:
  -- [ ] Revoke all active sessions (session-service)
  -- [ ] Disable identity (identity-service)
  -- [ ] Revoke all credentials (credential-vault-service)
  -- [ ] Remove from all groups (directory-service)
  -- [ ] Wipe MDM devices (mdm-service)
  -- [ ] Archive mailbox
  -- [ ] Transfer file ownership
  -- [ ] Generate compliance report (audit-service)
- Progress bar: 0% -> executing -> 100% complete
- "Execute Deprovisioning" button (supervised action, requires admin re-auth)

PROVISIONING TEMPLATES:
- Template cards for common access profiles:
  "New Engineering Hire" -- Email, Slack, GitHub, AWS, Jira, Confluence
  "New Sales Rep" -- Email, Slack, Salesforce, HubSpot
  "Contractor (Limited)" -- Email (external), Slack (guest), Project-specific only
  "Admin Escalation" -- Adds admin roles, requires 2-person approval
```

#### Prompt F-008: Audit Log Explorer (1440px)

```
Design a desktop Audit Log Explorer at 1440x900px for the audit-service.
Target user: Compliance Officer / Security Investigator.

HEADER:
- "Audit Log" title with total events count
- "Export" button (CSV, JSON, PDF report)
- Date range picker: "Last 24 hours" | "Last 7 days" | "Last 30 days" | Custom
- "Create Report" button

SEARCH AND FILTER BAR (full width, expandable):
- Search: free text across all fields
- Filters (collapsible advanced panel):
  -- Actor: identity search
  -- Action: Login, Logout, Create, Update, Delete, Grant, Revoke, Provision, Deprovision
  -- Resource Type: Identity, Group, Role, Session, Credential, Policy, Device
  -- Result: Success, Failure
  -- IP Address / CIDR range
  -- Location (country/city)
  -- Correlation ID (for tracing related events)
  -- Tenant ID

AUDIT LOG TABLE:
| Timestamp | Correlation ID | Actor | Action | Resource | Detail | IP | Location | Result | |
|-----------|---------------|-------|--------|----------|--------|-----|----------|--------|--|
| Feb 23, 10:30:12.456 | req-abc123 | Ngozi Adeyemi | Login | identity-service | Password + TOTP | 105.112.34.56 | Lagos, NG | Success | > |
| Feb 23, 10:28:45.123 | req-def456 | System | Provision | David Adeleke | Auto-provision: Email, Slack | -- | Internal | Success | > |
| Feb 23, 10:25:01.789 | req-ghi789 | Unknown | Login | admin@company.com | Password only (MFA not completed) | 203.45.67.89 | Unknown | Failed | > |
| Feb 23, 10:22:18.234 | req-jkl012 | Chidi Nnamdi | Password Change | Self | Voluntary rotation | 105.112.35.12 | Lagos, NG | Success | > |
| Feb 23, 10:20:05.567 | req-mno345 | Emeka Okafor | Role Grant | David Adeleke | Added to: engineering-team, github-devs | 105.112.34.78 | Lagos, NG | Success | > |

- Failed events: red text, sortable by result
- Expandable row (>) shows full JSON payload:
  { event_id, timestamp, actor_id, actor_name, action, resource_type,
    resource_id, detail, ip, user_agent, geo, tenant_id, correlation_id,
    request_headers (sanitized), response_code, duration_ms }
- Pagination: "Showing 1-100 of 1,245,678 events" (virtualized scroll)

TIMELINE VISUALIZATION (toggle view):
- Horizontal timeline with event density heatmap
- Color-coded by result (green=success, red=failure)
- Zoomable: day -> hour -> minute -> second
- Spike detection: anomalous event clusters highlighted

CORRELATION VIEW:
- Enter correlation ID -> see all related events in sequence
- Visual flow: Event 1 -> Event 2 -> Event 3 (with timing between)
- Useful for tracing a full authentication + provisioning + access flow

COMPLIANCE REPORT GENERATOR:
- Template: "SOC 2 Access Review", "GDPR Data Access", "ISO 27001 Audit", "Custom"
- Date range, scope (all or specific users/resources)
- Generate PDF report with charts and detailed event tables
- Schedule recurring reports (weekly, monthly)
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile IAM Dashboard (390px)

```
Design a mobile IAM Dashboard at 390x844px for security administrators on-the-go.

NAVIGATION:
- Bottom tab bar: Home, Identities, Sessions, Alerts (badge: 3), Profile
- Top bar: ERP logo, search icon, notification bell

HOME TAB:
- Greeting: "Security Overview" + current date
- Security health score: Large circular gauge (0-100), currently 87/100 (green)
  Breakdown: MFA 94%, Password Policy 91%, Device Trust 78%, Session Hygiene 85%

- Quick Stats (2x2 grid):
  -- Active Users: 892 of 4,832
  -- Active Sessions: 1,247
  -- Failed Logins (24h): 47
  -- Expiring Credentials: 23

- "Security Alerts" card (top 3):
  -- "Brute force attempt from 203.45.67.89" [Investigate]
  -- "12 low-trust device logins" [Review]
  -- "23 API keys expire within 7 days" [Rotate]
  "View All Alerts (3)" link

- "Quick Actions" horizontal scroll:
  [Search User] [Revoke Session] [View Audit] [Lock Account] [Reset Password]

- "Recent Events" mini-timeline (last 5):
  -- "Login: Ngozi Adeyemi via TOTP" -- 2m ago
  -- "Provision: David Adeleke -- Email, Slack" -- 30m ago
  -- "Failed Login: admin attempt from 203.45.67.89" -- 1h ago

All touch targets >= 44px. Pull-to-refresh. Critical alerts push to lock screen.
```

#### Prompt F-010: Mobile Session Management (390px)

```
Design a mobile Session Management view at 390x844px.

SCREEN 1 -- "Sessions" tab:
- "My Active Sessions" section (user's own sessions):
  Current session card (highlighted with "This Device" green badge):
  iPhone 15 Pro | Safari | Lagos, NG | Active 1h 30m

  Other sessions:
  MacBook Pro | Chrome | Lagos, NG | Active 2h
  [Revoke] button

  iPad Air | Safari | Lagos, NG | Last active 3h ago
  [Revoke] button

  "Revoke All Other Sessions" link (danger)

SCREEN 2 -- "All Sessions" (admin view, requires role):
- Search bar: "Search by user or IP"
- Session list (scrollable):
  Each card:
  - User avatar + name
  - Device + browser
  - IP + location
  - Duration + last active
  - Trust score badge (High/Medium/Low)
  - Swipe left: [Revoke] [Investigate]

- Sort: Most Recent | Longest Active | Lowest Trust
- Filter: Trust Level, Device Type

SCREEN 3 -- "Session Detail" (tapped from list):
- Full session information
- Device trust breakdown (checklist of factors)
- Activity log for this session
- "Revoke Session" full-width button
- "Block IP" button
- "Flag User" button

SCREEN 4 -- "My Devices" (from Profile tab):
- List of registered devices with trust scores
- Each device: name, OS, trust score gauge, MDM status
- "Remove Device" swipe action
- "Register New Device" button at bottom
```

#### Prompt F-011: Mobile MFA Management (390px)

```
Design a mobile MFA Management flow at 390x844px for end users managing their
multi-factor authentication.

SCREEN 1 -- "Security" (from Profile menu):
- "Multi-Factor Authentication" section:
  Status: "Enabled -- TOTP Authenticator" (green shield icon)
  Last verified: Feb 23, 2026

  Methods:
  - TOTP Authenticator App: Active (primary method) [Manage]
  - Backup Codes: 5 of 8 remaining [View / Regenerate]
  - Security Key (WebAuthn): Not configured [Set Up]
  - SMS (not recommended): Not configured [Set Up]

- "Password" section:
  Last changed: Feb 1, 2026 (22 days ago)
  Strength: Strong (green bar)
  [Change Password]

- "Active Sessions" link -> navigates to sessions
- "Login History" link -> last 10 logins

SCREEN 2 -- "Set Up TOTP" (bottom sheet, 90% height):
- Step indicator: 1 of 4
- Step 1: "Install an authenticator app" with app store links
  (Google Authenticator, ERP MFA App, Authy)
- Step 2: QR code (200x200px) with manual entry key below (mono, copyable)
  "Scan this code with your authenticator app"
- Step 3: "Enter the 6-digit code from your app"
  OTP input: 6 separate boxes, auto-advance, numeric keyboard
  [Verify] button
- Step 4: Backup codes display (8 codes, mono font)
  "Save these codes in a safe place. Each can only be used once."
  [Copy All] [Download as Text] [Print]
  Checkbox: "I have saved these codes" -> [Complete Setup]

SCREEN 3 -- "Change Password" (full page):
- Current password field (required)
- New password field with real-time strength meter
- Confirm new password field
- Requirements checklist (live validation):
  -- Min 12 characters
  -- At least one uppercase letter
  -- At least one number
  -- At least one special character
  -- Not a previously used password
  -- Not found in breach databases
- [Update Password] full-width button (disabled until all checks pass)
- Biometric confirmation required after submission

SCREEN 4 -- "Emergency: Lost MFA Device":
- "Can't access your authenticator?" heading
- Option 1: Use a backup code [Enter Backup Code]
- Option 2: Contact IT Support [Send Request] (creates ticket)
- Option 3: Use WebAuthn security key (if configured) [Use Security Key]
- Warning: "For security, recovery may take up to 24 hours with identity verification"
```

#### Prompt F-012: Mobile Audit Viewer (390px)

```
Design a mobile Audit Log viewer at 390x844px for quick compliance checks on mobile.

SCREEN 1 -- "Audit Log" (from main menu or alert deep link):
- Date picker (horizontal scroll): Today | Yesterday | Last 7 days | Custom
- Quick filters (chips): All | Logins | Failures | Admin Actions | Provisions

- Event list (scrollable):
  Each card (compact, 80px tall):
  - Left: result icon (green check / red x)
  - Center: Actor name, action summary, timestamp
  - Right: chevron

  Examples:
  - [check] Ngozi Adeyemi -- Logged in -- 10:30 AM
  - [x] Unknown -- Failed login (admin@...) -- 10:25 AM
  - [check] System -- Provisioned David Adeleke -- 10:28 AM
  - [check] Emeka Okafor -- Granted role to David A. -- 10:20 AM

SCREEN 2 -- "Event Detail" (tapped from list):
- Full event metadata in a structured card layout:
  Actor: Ngozi Adeyemi (with avatar)
  Action: Login
  Method: Password + TOTP
  IP: 105.112.34.56
  Location: Lagos, Nigeria
  Device: MacBook Pro / Chrome 120
  Timestamp: Feb 23, 2026 10:30:12 UTC
  Correlation ID: req-abc123 (copyable)
  Result: Success
  Duration: 1.2s

- "Related Events" section: other events with same correlation ID
- "Share" button (generates compliance-safe report snippet)
- "Flag for Investigation" button
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Tablet IAM Dashboard (1024px)

```
Design a tablet-adapted IAM Dashboard at 1024x768px.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) with tooltip labels
- KPI cards: 3+2 row layout (3 on first row, 2 on second)
- Auth activity chart and Security alerts: stacked vertically
- Identity distribution and Auth methods charts: stacked vertically
- Audit log table: 5 columns visible (Timestamp, Actor, Action, Result, >) with expand
- Session map: simplified to top-10 locations bar chart instead of world map

SPECIFIC LAYOUT:
Row 1: 3 KPI cards (Identities, Sessions, MFA Adoption)
Row 2: 2 KPI cards (Failed Logins, Pending Provisions)
Row 3: Auth Activity chart (full width, 300px tall)
Row 4: Security Alerts list (full width, scrollable)
Row 5: Charts side-by-side (50/50) -- Identity Distribution | Auth Methods
Row 6: Recent Audit Trail (full width table, simplified columns)

Touch: >= 44px targets. Support landscape and portrait.
Portrait (768x1024): single column, all sections stacked.
```

#### Prompt F-014: Tablet Identity Directory (1024px)

```
Design a tablet Identity Directory at 1024x768px.

ADAPTATIONS:
- Sidebar: icon-only (72px)
- Search bar: full width below header
- Table: 5 visible columns (Avatar+Name, Type, Status, MFA, Actions)
  -- Identity ID, Last Login, Groups accessible via row expansion
- Bulk action bar: bottom-fixed with selected count
- Identity profile detail: modal overlay instead of split-screen
  -- Two-column layout within modal: profile info (left), tabbed content (right)

PORTRAIT MODE (768x1024):
- Table: 3 visible columns (Avatar+Name, Status, Actions)
- All other details in row expansion
- Profile detail: full-screen modal with back button

SESSION MANAGEMENT (tablet):
- Session list replaces map view
- Session cards: 2 per row in landscape, 1 per row in portrait
- Device trust badges visible inline

All interactive elements >= 44px. Support iPadOS split view at 507px compact width.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Identity Lifecycle Automation (Joiner-Mover-Leaver)

```
Create a Make scenario titled "IAM -- Identity Lifecycle: Joiner-Mover-Leaver".

TRIGGER: Webhook from ERP-HCM employee-service on employee status change events:
  - employee.created (Joiner)
  - employee.department_changed or employee.role_changed (Mover)
  - employee.terminated or employee.contract_ended (Leaver)

JOINER FLOW:
Step 1 -- Create Identity:
- Call identity-service: POST /v1/identities
  { email, full_name, type: "employee", department, manager_id, tenant_id }
- Generate temporary password and store in credential-vault-service

Step 2 -- Provision Access:
- Call provisioning-service: POST /v1/provisions
  { identity_id, template: department_template, priority: "normal" }
- Auto-provision based on department access template (e.g., "engineering" -> Email, Slack, GitHub, AWS)

Step 3 -- MFA Enrollment Invitation:
- Send welcome email with login credentials and MFA setup link
- Set MFA grace period: must enroll within 7 days
- After 7 days: force MFA enrollment on next login

Step 4 -- Directory Update:
- Call directory-service: POST /v1/directory/entries
- Add to appropriate groups based on department and role

MOVER FLOW:
Step 1 -- Calculate Access Delta:
- Compare current access (directory-service groups, provisioning-service resources) with new role requirements
- Generate add/remove list

Step 2 -- Adjust Access:
- Provision new resources needed for new role
- Schedule removal of old resources (grace period: 7 days for file transfer)
- Update directory-service group memberships

Step 3 -- Notify:
- Email to identity: "Your access has been updated for your new role"
- Email to new manager: access review confirmation
- Audit log entry via audit-service

LEAVER FLOW:
Step 1 -- Immediate Actions:
- Call session-service: POST /v1/sessions/revoke-all?identity_id={id}
- Call identity-service: PUT /v1/identities/{id}/status { status: "deprovisioned" }
- Call credential-vault-service: POST /v1/credentials/revoke-all?owner_id={id}

Step 2 -- Scheduled Actions:
- Queue MDM device wipe (mdm-service) after 24h grace period
- Archive email/files (30-day retention)
- Remove from all groups (directory-service)

Step 3 -- Compliance:
- Generate offboarding audit report (audit-service)
- Confirm all access revoked with checklist
- Publish NATS event: "identity.deprovisioned"

ERROR HANDLING: Each step retries 3x. Leaver flow failures escalate immediately
to Security team (cannot leave access active).
```

### Prompt M-002: Suspicious Login Detection and Response

```
Create a Make scenario titled "IAM -- Suspicious Login Response".

TRIGGER: Webhook from identity-service on every login event.
Payload: { identity_id, method, ip, user_agent, geo, device_fingerprint, success, timestamp }

STEP 1 -- Evaluate Risk:
- Check against known patterns for this identity:
  -- Usual login hours (from audit-service historical data)
  -- Usual locations (from session-service)
  -- Usual devices (from device-trust-service)
- Calculate risk score (0-100):
  -- New location: +30 risk
  -- New device: +20 risk
  -- Outside usual hours: +15 risk
  -- Failed previous attempt: +10 risk
  -- Low device trust: +20 risk
  -- Impossible travel (logged in from Lagos, now London within 1 hour): +50 risk

STEP 2 -- Route by Risk:
- LOW (0-30): Log and continue (normal)
- MEDIUM (31-60): Send verification email to user, log alert
- HIGH (61-80): Force MFA re-verification, notify security team
- CRITICAL (81-100): Suspend session, lock account, alert security team immediately

STEP 3 -- High/Critical Response:
- Push notification to security admin mobile: "ALERT: Suspicious login detected for {user}"
- Email security team with full event details
- If impossible travel: auto-revoke newest session, send confirmation to user's known email
- Create incident in audit-service with correlation ID

STEP 4 -- User Communication:
- For MEDIUM: "Was this you? Login from {location} at {time}" email with [Yes] [No] links
  -- If user clicks [No]: auto-lock account, force password reset, revoke all sessions
  -- If user clicks [Yes]: whitelist the location/device, reduce future risk scoring

STEP 5 -- Learn and Adapt:
- Update user's normal patterns based on confirmed legitimate logins
- Feed false positives back to reduce future alert fatigue
- Publish NATS event: "security.suspicious_login" with risk details

VOLUME: ~5,000 login events/day. Risk evaluation must complete in < 500ms.
```

### Prompt M-003: Credential Rotation Scheduler

```
Create a Make scenario titled "IAM -- Credential Auto-Rotation".

TRIGGER: Scheduled daily at 02:00 UTC.

STEP 1 -- Identify Credentials Due for Rotation:
- Call credential-vault-service: GET /v1/credentials?rotation_due=true
- Categories:
  -- Auto-rotate: credentials with auto-rotation policy enabled
  -- Expiring: credentials expiring within 14 days
  -- Overdue: credentials past rotation schedule

STEP 2 -- Auto-Rotate Eligible Credentials:
- For each auto-rotate credential:
  -- Generate new credential value
  -- Update in credential-vault-service
  -- If integration exists: push new credential to target system (via provisioning-service)
  -- Mark old credential as deprecated (grace period: 24h for rolling deployment)
  -- After 24h: revoke old credential

STEP 3 -- Notify for Manual Rotation:
- For credentials requiring manual rotation:
  -- Email owner: "Your API key {name} expires on {date}. Rotate now: {link}"
  -- 7 days before: first notice
  -- 3 days before: urgent reminder
  -- 1 day before: critical alert to owner + security team
  -- On expiry: auto-disable with notification

STEP 4 -- Compliance Report:
- Generate daily rotation compliance report:
  -- Total credentials, % within policy, % overdue
  -- List of expired/overdue credentials with owners
  -- Rotation success/failure log
- Send to security team and update audit-service

STEP 5 -- Emergency Rotation:
- Webhook endpoint for emergency rotation trigger (e.g., suspected breach)
- Rotate all credentials of specified type/owner immediately
- Revoke old values immediately (no grace period)
- Alert all affected system owners

ERROR HANDLING: Rotation failures create high-priority tickets.
AIDD GUARDRAIL: Credential operations are supervised actions -- log all to audit-service.
```

### Prompt M-004: Device Trust Continuous Assessment

```
Create a Make scenario titled "IAM -- Device Trust Assessment".

TRIGGER: Webhook from device-trust-service on device check-in (every 4 hours)
or session-service on new session creation.

STEP 1 -- Assess Device Trust Factors:
- Parse device telemetry:
  { device_id, os_version, encryption_enabled, antivirus_status,
    biometric_available, mdm_enrolled, jailbreak_detected, last_patch_date,
    firewall_enabled, disk_encryption }
- Calculate trust score (0-100):
  -- OS current: +20 | OS outdated > 30 days: -10
  -- Encryption enabled: +20 | Not encrypted: -30
  -- MDM enrolled: +15 | Not enrolled: -5
  -- Antivirus active: +10 | No antivirus: -10
  -- Jailbreak/root detected: -50
  -- Biometric available: +10
  -- Firewall enabled: +5
  -- Recent security patch (< 14 days): +10

STEP 2 -- Enforce Policy:
- If trust score drops below threshold (< 50):
  -- If device has active sessions: force re-authentication with MFA
  -- Block access to sensitive resources (credential-vault-service, admin panels)
  -- Notify user: "Your device trust score is low. Actions needed: {remediation_list}"
- If jailbreak detected:
  -- Immediately revoke all sessions from this device
  -- Block device from future logins
  -- Alert security team
  -- If MDM enrolled: trigger mdm-service remote lock

STEP 3 -- MDM Compliance:
- For managed devices: verify MDM policies are applied (mdm-service)
- Flag non-compliant devices for IT team
- Auto-push configuration profiles for policy violations

STEP 4 -- Update and Report:
- Update device-trust-service with new trust score
- Update identity-service risk profile based on device portfolio
- Weekly device compliance report for IT Admin

VOLUME: ~10,000 device check-ins/day.
```

### Prompt M-005: Compliance Audit Report Generator

```
Create a Make scenario titled "IAM -- Automated Compliance Reporting".

TRIGGER: Scheduled monthly (1st of each month at 06:00 UTC) + on-demand webhook.

STEP 1 -- Gather Data:
- Call audit-service: GET /v1/audit/summary?period=last_month
- Call identity-service: GET /v1/identities/stats (active, suspended, MFA stats)
- Call session-service: GET /v1/sessions/stats (avg duration, anomalies)
- Call credential-vault-service: GET /v1/credentials/stats (rotation compliance)
- Call device-trust-service: GET /v1/devices/stats (trust distribution)
- Call provisioning-service: GET /v1/provisions/stats (avg fulfillment time)

STEP 2 -- Generate Reports:
Report A: "SOC 2 Access Review"
  -- User access matrix (who has access to what)
  -- Dormant accounts (not logged in > 90 days)
  -- Privileged access review (admin accounts)
  -- MFA compliance percentage
  -- Session management policy adherence

Report B: "Security Incident Summary"
  -- Failed login attempts by category (brute force, credential stuffing, etc.)
  -- Suspicious login events and resolution status
  -- Account lockouts
  -- Emergency access ("break glass") usage

Report C: "Credential Management"
  -- Active credentials by type
  -- Rotation compliance percentage
  -- Expired credentials
  -- Never-used credentials (cleanup candidates)

Report D: "Device Compliance"
  -- Trust score distribution
  -- Non-compliant devices
  -- MDM enrollment rate
  -- Jailbreak/root detections

STEP 3 -- Distribute:
- Email CISO / Security Manager with all reports (PDF)
- Store in audit-service for regulatory access
- Update compliance dashboard in IAM admin console
- Publish NATS event: "compliance.monthly_report_generated"

STEP 4 -- Action Items:
- Generate task list from report findings:
  -- Dormant accounts > 90 days: schedule deprovisioning
  -- Expired credentials: schedule revocation
  -- Non-compliant devices: schedule MDM enforcement
  -- Low MFA adoption teams: schedule enrollment campaign
```

### Prompt M-006: Emergency Access (Break Glass) Workflow

```
Create a Make scenario titled "IAM -- Emergency Break Glass Access".

TRIGGER: Webhook from identity-service when a "break glass" authentication is initiated.
This is triggered when an administrator is locked out and invokes emergency override.

STEP 1 -- Validate Emergency:
- Verify the requesting identity exists and has break-glass eligibility
- Check that normal authentication paths are truly unavailable
  (verify account status, MFA device status)
- If not a genuine lockout: reject and alert security team

STEP 2 -- Multi-Person Authorization:
- Send approval request to 2 of 3 designated emergency approvers
  (Security Director, CTO, CISO)
- Approvers notified via: SMS + Push + Email (all channels)
- Each approver must provide their own MFA confirmation to approve
- Timeout: 30 minutes -- if not approved, request expires

STEP 3 -- Grant Temporary Access:
- Once 2 approvals received:
  -- Create temporary session with 2-hour TTL (session-service)
  -- Grant minimum necessary permissions (not full admin)
  -- Enable enhanced logging: every action in this session logged with extra detail
  -- Display banner: "EMERGENCY ACCESS -- Session expires in 1:59:45"
- Set account recovery in motion: MFA re-enrollment link sent to verified backup email

STEP 4 -- Full Audit Trail:
- Log every detail to audit-service:
  -- Who requested, when, why (mandatory justification text)
  -- Who approved (both approvers)
  -- Session ID, duration, all actions taken
  -- When session expired/ended
- Generate incident report automatically after session ends

STEP 5 -- Post-Incident:
- Auto-expire emergency session after 2 hours (hard limit)
- Send incident summary to all security team members
- Require post-incident review within 48 hours
- Update break-glass procedure documentation if gaps found
- Publish NATS event: "security.break_glass_completed"

AIDD GUARDRAIL: This is a supervised action requiring human-in-the-loop.
All steps logged with decision_logging: true per erp/aidd.guardrails.yaml.
```

---

## 5. Prompt Usage Guidelines

| Step | Action | Tool |
|------|--------|------|
| 1 | Copy the prompt text from Section 3 or 4 | Clipboard |
| 2 | Open Figma and activate the Make Design / AI plugin | Figma |
| 3 | Paste the prompt into the plugin input field | Plugin UI |
| 4 | Review generated output; verify security status indicators use color + icon + label | Manual |
| 5 | Connect components to shared IAM Design System library | Figma Libraries |
| 6 | Verify all credential displays are properly masked | Manual |
| 7 | Export developer-ready assets using naming convention below | Figma Export |

**Security UI guidelines:**
- Never show full credentials, tokens, or secrets in mockups -- use masked values (e.g., `sk_live_...7f8a`)
- Password fields must always default to obscured state
- Sensitive UUID fields should show truncated format with copy button
- All admin action buttons must have confirmation patterns
- Dark/SOC theme must be validated for extended monitoring use cases

---

## 6. Output Packaging Convention

```
Deliverable naming:
  IAM-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: IAM-F003-1440-light-v1.fig
  Example: IAM-F005-1440-dark-soc-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/ActiveSessions-Default
  Example: Vault/CredentialTable/APIKeyRow-Expiring

Asset export:
  /exports/iam/{breakpoint}/{page}/
  Example: /exports/iam/1440/dashboard/auth-activity-chart.png

Handoff tokens:
  /tokens/iam-tokens.json (Style Dictionary format)

Security icons:
  /exports/iam/icons/{category}/
  Example: /exports/iam/icons/auth/mfa-totp.svg
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Login page total weight | < 100 KB gzipped | Network tab |
| Login flow completion | < 3s (password + MFA) | Custom perf mark |
| Initial route bundle size | < 220 KB gzipped | Webpack bundle analyzer |
| Largest Contentful Paint (LCP) | < 1.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Web Vitals |
| Time to Interactive (TTI) | < 2.5s | Lighthouse CI |
| Session validation latency | < 50ms | Cached check |
| Audit log query (last 24h) | < 500ms | Custom perf mark |
| Directory search (4,800 users) | < 300ms | Custom perf mark |
| API read latency (p99) | < 200ms | perf/targets.yaml SLO |
| API write latency (p99) | < 500ms | perf/targets.yaml SLO |
| Availability | >= 99.95% | perf/targets.yaml SLO |

---

## 8. AIDD Handoff Gate Template

```markdown
## Design Handoff Checklist -- ERP-IAM

### Visual Completeness
- [ ] All pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light admin theme and dark SOC/monitoring theme complete
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic identity data used (no "test@test.com" or "John Doe")

### Security UX
- [ ] All credential displays use masked values by default
- [ ] Password fields default to obscured with show/hide toggle
- [ ] Destructive actions (revoke, deprovision, lock) require confirmation dialog with re-auth
- [ ] Break-glass emergency access flow fully documented
- [ ] MFA enrollment flow includes recovery path for lost devices
- [ ] Login error messages do not reveal whether email exists (prevent enumeration)

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI)
- [ ] Security status indicators use color + icon + label (color-blind safe)
- [ ] Touch targets >= 44x44px
- [ ] Focus order annotated (especially login flow and MFA input)
- [ ] Screen reader annotations for trust score gauges and session maps

### Developer Handoff
- [ ] Design tokens exported (Style Dictionary JSON)
- [ ] API data mapping documented (which service populates which component)
- [ ] Real-time WebSocket endpoints annotated (session monitoring)
- [ ] Feature flag annotation points marked (passwordless, WebAuthn)
- [ ] Keycloak/OAuth2 integration points documented

### AIDD Guardrails
- [ ] No cross-tenant data access patterns in UI
- [ ] All supervised actions (credential ops, bulk operations) have confirmation dialogs
- [ ] Audit trail access points visible throughout
- [ ] Human-in-the-loop gates marked for high-risk operations
- [ ] Decision logging integration points annotated
- [ ] Rollback capability visible (session revocation, credential restoration)
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial comprehensive prompt set covering IAM dashboard, identity directory, session management, credential vault, provisioning, audit logs, mobile and tablet views, and 6 Make automation workflows (JML lifecycle, suspicious login, credential rotation, device trust, compliance reporting, break-glass) |

---

## 9. Round 02 Implementation Handoff (Web, Mobile, Desktop)

### Prompt F-021: IAM Token-to-Component Mapping

```text
Create a handoff board named "ERP-IAM Round 02 - Token Mapping".
Map IAM security tokens to implemented components:
- status colors -> session badges, trust indicators, risk alerts.
- spacing/radius -> cards, forms, tables, side panels.
- typography -> dashboard metrics, audit entries, policy detail panels.
Provide side-by-side light/dark previews and accessibility annotations.
```

### Prompt F-022: IAM Web Operations Frames

```text
Create responsive web frames (1440, 1024, 390) for:
Dashboard, Users, Groups, Roles, Sessions, Devices, Audit Log.
For each frame include loading, empty, error, and success states.
Add role variants for Security Admin, IT Admin, Auditor, Helpdesk.
```

### Prompt F-023: IAM Mobile Operations Flow

```text
Design mobile screens for ERP IAM:
- Dashboard summary
- Sessions tab
- Devices tab
- Audit tab
- Error/offline state
Focus on fast triage, clear trust/risk signals, and thumb-friendly controls.
```

### Prompt F-024: IAM Desktop SOC Console

```text
Design a 1366x900 desktop SOC console:
- Service health matrix
- Auth event trend panel
- Risk alerts panel
- High-risk session actions requiring confirmation
Include compact and comfortable density variants for analyst workflows.
```

### Prompt F-025: IAM Component Contract Export

```text
Export implementation contracts for:
- user card
- session row
- trust badge
- audit timeline entry
- credential vault card
- policy rule panel
Include states, accessibility requirements, and interaction latency budgets.
```
