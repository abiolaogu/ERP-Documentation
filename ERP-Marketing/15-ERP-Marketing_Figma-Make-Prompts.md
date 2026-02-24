# Figma & Make Prompts -- ERP-Marketing (Marketing Automation)
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make (Integromat) automation prompts for the ERP-Marketing module. ERP-Marketing is the Marketing Automation vertical of the consolidated ERP platform, covering campaign management, email marketing, social media management, digital advertising, audience segmentation, customer journey orchestration, multi-touch attribution, marketing analytics, and content management.

All prompts reference real services (campaign-service, email-marketing-service, social-service, ads-service, segment-service, journey-service, attribution-service, analytics-service, content-service) and align with the AIDD guardrails defined in `erp/aidd.guardrails.yaml`. The platform uses Apache Pulsar as the event backbone (topics: command, event, audit, observability), Quickwit for observability/search, and integrates with ERP-Platform via NATS for suite-wide coordination.

The frontend stack is **Refine + Ant Design** (React) with support for both standalone and suite-integrated deployment.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance; color contrast >= 4.5:1 text, >= 3:1 UI components
- Minimum 44x44px touch targets on all interactive elements
- Clear focus-visible outlines (2px solid) for keyboard navigation
- Plain-language marketing labels (e.g., "Send to Segment" not "Execute Audience Dispatch Pipeline")
- Semantic HTML landmarks reflected in Figma layer naming
- Screen reader annotations for campaign performance charts, funnel visualizations, and attribution graphs
- Data visualizations: always include data table alternative for charts

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading; initial route bundle < 220 KB gzipped
- Skeleton UIs shown within 200ms; full content within 1s on 4G
- Virtualized tables for contact lists > 100 entries, campaign logs > 500 entries
- Email template editor: iframe-sandboxed preview, < 300KB template render
- Real-time campaign metrics: WebSocket updates for live send progress
- Chart rendering: < 500ms for datasets up to 100K data points

### 2.3 Reliability, Trust, And Safety
- Every bulk send operation (email blast, SMS campaign, ad budget change) requires confirmation with audience count and cost estimate
- CAN-SPAM / GDPR compliance: unsubscribe link mandatory in all email templates, consent verification before send
- Trust signals: send preview, test email delivery, estimated inbox rate, spam score
- Campaign scheduling: timezone-aware preview, confirmation of send time in recipient local time
- Budget controls: daily/lifetime budget caps with alert thresholds, requires approval for spends > threshold
- A/B test configuration with statistical significance calculator

### 2.4 Observability And Testability
- Every marketing action emits: `{event, campaign_id, channel, action, segment_id, user_id, tenant_id, timestamp}`
- Campaign events published to Pulsar topics: `persistent://billyronks/extract-marketing/event`
- Error boundaries with correlation_id referencing Quickwit logs
- Performance instrumentation: email delivery latency, open tracking lag, click-through attribution delay
- Feature flag integration: "Visible when `ai_content_generation` = true"

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens -- Marketing Theme

```
Create a Figma page titled "Marketing Design Tokens" containing the complete
token specification for the Marketing Automation module.

COLOR PALETTE (Light Theme):
- Primary: #7C3AED (Violet -- creative, bold, marketing energy)
- Primary Hover: #6D28D9
- Primary Light: #EDE9FE
- Secondary: #0EA5E9 (Sky Blue -- analytics, data, clarity)
- Accent: #F43F5E (Rose -- CTAs, urgency, conversion focus)
- Success: #059669 (Delivered, Active, Converted, Goal Met)
- Warning: #D97706 (Pending, Low engagement, Budget warning)
- Danger: #DC2626 (Failed, Bounced, Over budget, Spam flagged)
- Info: #2563EB (Scheduled, In progress, Informational)
- Social-Facebook: #1877F2
- Social-Instagram: #E4405F
- Social-Twitter: #1DA1F2
- Social-LinkedIn: #0A66C2
- Social-TikTok: #000000
- Neutral-50: #FAFAFA (Page background)
- Neutral-100: #F5F5F5 (Card background)
- Neutral-200: #E5E5E5 (Borders, dividers)
- Neutral-500: #737373 (Secondary text)
- Neutral-900: #171717 (Primary text)
- Surface: #FFFFFF

COLOR PALETTE (Dark Theme):
- Background: #0A0A0A
- Surface: #171717
- Surface Elevated: #262626
- Primary: #A78BFA (Light Violet)
- Secondary: #38BDF8
- Accent: #FB7185
- Text Primary: #FAFAFA
- Text Secondary: #A3A3A3
- Border: #404040

CHANNEL COLORS (consistent across all views):
- Email: #7C3AED (Violet)
- Social: #0EA5E9 (Sky Blue)
- Ads (Paid): #F43F5E (Rose)
- SMS: #059669 (Green)
- Push: #D97706 (Amber)
- Web: #6366F1 (Indigo)

CONVERSION FUNNEL GRADIENT:
- Top of Funnel: #7C3AED (Awareness)
- Middle of Funnel: #0EA5E9 (Consideration)
- Bottom of Funnel: #059669 (Conversion)
- Post-Conversion: #D97706 (Retention)

TYPOGRAPHY (Inter font family):
- Display: 30px/36px, semibold 600 -- page titles ("Campaign Performance")
- Heading 1: 24px/32px, semibold 600 -- section headers ("Email Analytics")
- Heading 2: 20px/28px, medium 500 -- card titles ("Open Rate Trend")
- Heading 3: 16px/24px, medium 500 -- subsection headers
- Body: 14px/20px, regular 400 -- table cells, descriptions
- Body Small: 12px/16px, regular 400 -- timestamps, metadata
- Label: 14px/20px, medium 500 -- form labels, column headers
- Mono: 13px/20px, JetBrains Mono -- campaign IDs, tracking codes, revenue amounts
- Metric Large: 36px/44px, bold 700 -- hero KPI numbers (Open Rate: 24.8%)

SPACING: 4px base (4, 8, 12, 16, 24, 32, 48px)
Card padding: 24px. Section gap: 32px. Page margin: 32px desktop, 16px mobile.

ELEVATION:
- Level 0: none
- Level 1: 0 1px 3px rgba(0,0,0,0.08) -- cards
- Level 2: 0 4px 12px rgba(0,0,0,0.1) -- modals, email preview panels
- Level 3: 0 8px 24px rgba(0,0,0,0.12) -- full-screen editors

BORDER RADIUS: 4px (badges), 8px (cards, inputs), 12px (modals), 9999px (avatars, channel dots)

GRID:
- Desktop (1440px): 12-column, 72px col, 24px gutter, 260px sidebar
- Tablet (1024px): 12-column, 56px col, 16px gutter, collapsed sidebar
- Mobile (390px): 4-column, fluid, 16px gutter, bottom nav

Include both light and dark themes. Light is default; dark available for
marketing ops late-night campaign monitoring.
```

#### Prompt F-002: Component Library -- Marketing Components

```
Create a Figma page titled "Marketing Component Library" with the following
reusable components. Show all states and light/dark variants.

CAMPAIGN CARD:
- 340px wide. Campaign name, channel icon (email/social/ads), status badge,
  date range, audience size, performance mini-metrics (sent, opened, clicked)
- Status: Draft (gray), Scheduled (blue), Active (green pulse), Paused (amber),
  Completed (green), Failed (red)
- Quick actions: View, Edit, Duplicate, Pause/Resume, Delete
- Example: "Spring Sale 2026 | Email | Active | Feb 15 - Mar 15 |
  Segment: High-Value Customers (12,450) | Sent: 12,450 | Opened: 3,112 (25%)"

EMAIL PERFORMANCE CARD:
- Horizontal funnel visualization:
  Sent (100%) -> Delivered (97.2%) -> Opened (24.8%) -> Clicked (4.2%) -> Converted (1.8%)
- Each stage: count and percentage, colored bar section
- Bounce rate, unsubscribe rate, spam complaint rate below
- Example: "Sent: 12,450 | Delivered: 12,102 | Opened: 3,002 | Clicked: 508 | Converted: 218"

SOCIAL POST CARD:
- Platform icon (Facebook/Instagram/Twitter/LinkedIn/TikTok)
- Post preview: thumbnail image (120x120), caption (truncated 2 lines)
- Engagement metrics row: Likes, Comments, Shares, Impressions
- Scheduled time or posted time
- Status: Draft, Scheduled, Published, Failed
- Example: "Instagram | 'Discover our Spring Collection...' | Likes: 2,340 |
  Comments: 89 | Shares: 156 | Impressions: 45,200 | Published Feb 20, 14:00"

AD CAMPAIGN CARD:
- Platform badge (Google Ads, Meta Ads, LinkedIn Ads)
- Campaign name, objective (Awareness, Consideration, Conversion)
- Budget: Spent / Total (progress bar): "NGN 450,000 / NGN 1,000,000 (45%)"
- Performance: Impressions, Clicks, CTR, CPC, Conversions, ROAS
- Status: Active (green), Paused (amber), Ended (gray), Over Budget (red)
- Example: "Google Ads | Spring Sale SEM | Conversion | NGN 450K / NGN 1M |
  Impressions: 125,000 | Clicks: 3,450 | CTR: 2.76% | CPC: NGN 130 | Conv: 89 | ROAS: 4.2x"

SEGMENT CARD:
- Segment name, audience size, refresh status (real-time/scheduled)
- Rule summary preview: "Purchased in last 30 days AND email_opt_in = true AND region = Lagos"
- Trend: audience size change over last 30 days (+8% / -3%)
- Quick actions: View Members, Edit Rules, Use in Campaign, Export
- Example: "High-Value Lagos Customers | 12,450 contacts | Real-time sync |
  Rules: purchase_count > 3 AND region = 'Lagos' AND email_opt_in = true | +8% (30d)"

JOURNEY MAP NODE:
- Trigger node (green circle): "Contact enters segment"
- Action nodes (colored rectangles): Send Email, Wait, SMS, Update Field
- Decision nodes (diamond): "Opened email?" Yes/No branches
- Exit node (red circle): "Journey complete"
- Connection lines with arrows, animated flow indicator
- Node hover: shows node configuration summary

ATTRIBUTION MODEL CARD:
- Model name (First Touch, Last Touch, Linear, U-Shaped, Algorithmic)
- Visualization: horizontal stacked bar showing touchpoint credit distribution
- Touchpoints: Email, Social, Ads, Direct, Organic Search, Referral
- Example: "U-Shaped Attribution for Spring Sale"
  First Touch (Email): 40% | Mid Touch (Social Ad): 10% | Mid Touch (Search): 10% | Last Touch (Direct): 40%

CONTENT TEMPLATE CARD:
- Thumbnail preview (200x150px) of email/landing page template
- Template name, type (Email, Landing Page, Social Post, Ad Creative)
- Category tags: "Promotional", "Newsletter", "Transactional"
- Performance badge: "Top performer" (if used in successful campaigns)
- "Use Template" button, "Preview" button, "Edit" button
- Example: "Spring Sale Hero Email | Email Template | Promotional |
  Used: 8 campaigns | Avg Open Rate: 26.4% (above average)"

METRIC CHIP:
- Compact pill-shaped component for inline metrics
- Format: Label + Value + Trend arrow
- Examples: "Open Rate 24.8% (up-arrow green)", "CTR 4.2% (down-arrow red)",
  "Revenue NGN 8.4M (up-arrow green)", "Unsubscribes 0.3% (flat)"

SIDEBAR NAVIGATION:
- Sections: Dashboard, Campaigns, Email, Social, Ads, Journeys, Segments,
  Attribution, Analytics, Content, Settings
- Active state: Primary background + white text + left accent
- Badge counts: Active Campaigns (8), Scheduled Posts (12), Draft Emails (5)
- Workspace selector at top (for multi-brand)

CHART COMPONENTS:
- Line chart with multi-series (opens, clicks, conversions over time)
- Bar chart (campaign comparison)
- Funnel chart (conversion funnel)
- Donut chart (channel mix)
- Heatmap (send time optimization -- hour x day grid)
- All with tooltips, legends, data table toggle (accessibility)
```

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Marketing Command Center Dashboard (1440px)

```
Design a desktop Marketing Dashboard at 1440x900px for the ERP-Marketing module.
Target user: Marketing Manager / CMO.

HEADER:
- "Marketing Command Center" title
- Date range picker: "Last 30 days" with presets (7d, 30d, 90d, YTD, Custom)
- "Create Campaign" primary button (dropdown: Email, Social, Ad, Journey)
- Notification bell, dark/light toggle, user avatar "Amina Ogundimu, Marketing Director"

SIDEBAR (260px, collapsible): Per component library spec.

CONTENT -- ROW 1 (KPI Cards, 5 across):
- Card 1: "Total Contacts" -- 148,230 -- +2,340 this month (green)
- Card 2: "Active Campaigns" -- 8 -- across 3 channels
- Card 3: "Email Open Rate (30d)" -- 24.8% -- +1.2% vs last period (green)
- Card 4: "Total Conversions (30d)" -- 1,892 -- NGN 28.4M revenue attributed
- Card 5: "Marketing Spend (MTD)" -- NGN 4.2M -- 68% of monthly budget

CONTENT -- ROW 2 (Two-column, 60/40):
- Left (60%): "Performance Over Time" -- Multi-line chart (last 30 days)
  X-axis: daily dates
  Three series:
  -- Emails Sent (violet line): ranges 400-800/day
  -- Opens (blue line): ranges 100-200/day
  -- Conversions (green line): ranges 15-65/day
  Annotation on Feb 15: "Spring Sale launched" (spike in all metrics)
  Annotation on Feb 22: "Retargeting campaign started"

- Right (40%): "Channel Mix" donut chart:
  Email: 52% of conversions (violet)
  Social: 23% (sky blue)
  Paid Ads: 18% (rose)
  Organic: 7% (green)
  Center: "1,892 total conversions"

CONTENT -- ROW 3 (Three-column, equal):
- Left: "Top Performing Campaigns" list (last 30d):
  1. "Spring Sale Email Blast" -- 25.4% open, 4.8% CTR, 218 conversions
  2. "Instagram Spring Collection" -- 45.2K reach, 2.3K engagements
  3. "Google SEM - Spring Keywords" -- 3.45K clicks, ROAS 4.2x
  4. "LinkedIn Thought Leadership" -- 12.1K impressions, 234 leads
  Each with channel icon and mini-performance bar

- Center: "Upcoming Scheduled" card:
  -- "Weekly Newsletter" -- Tomorrow, 09:00 WAT -- 48,230 recipients -- [Preview]
  -- "Social: Product Launch Teaser" -- Feb 25, 12:00 -- Instagram + Twitter -- [Edit]
  -- "Retargeting Ad: Cart Abandonment" -- Continuous -- Meta Ads -- [Pause]
  -- "Journey: Welcome Series" -- Ongoing -- 320 active contacts -- [View]

- Right: "Quick Stats" stacked cards:
  -- Email Health: Deliverability 97.2% (green), Bounce 1.8% (green), Spam 0.04% (green)
  -- List Growth: +2,340 new contacts, -120 unsubscribes = net +2,220
  -- Social Followers: 45.2K total, +1,200 this month
  -- Ad Budget: 68% spent, 32% remaining, projected to exhaust by month end

CONTENT -- ROW 4 (Full width):
- "Send Time Optimization" heatmap:
  Rows: Days (Mon-Sun), Columns: Hours (06:00-22:00)
  Cell color intensity: green (high engagement) to light (low engagement)
  Best time highlighted: "Tuesday 10:00 WAT -- 31.2% avg open rate"
  Based on analytics-service historical data

CONTENT -- ROW 5 (Full width):
- "Attribution Summary" -- Horizontal bar showing multi-touch attribution
  across all campaigns for the period:
  Email first-touch: 35% | Social assist: 20% | Paid last-click: 25% |
  Organic: 12% | Direct: 8%
  "View Full Attribution Report" link (attribution-service)

Both light and dark themes. Skeleton loading for all data cards.
```

#### Prompt F-004: Campaign Builder and Manager (1440px)

```
Design two connected desktop screens at 1440x900px for the campaign-service.

SCREEN A -- CAMPAIGN LIBRARY:
- Top: "Campaigns" title, "Create Campaign" primary button (dropdown: Email, Social, Ad, Multi-Channel)
- View toggle: Grid (default) | Table
- Filters: Channel (Email, Social, Ads, Multi), Status (Draft, Scheduled, Active, Paused, Completed), Date Range
- Search: by campaign name, tag

GRID VIEW (3 cards per row):
Campaign Card 1: "Spring Sale 2026 -- Email Campaign"
  Status: Active (green badge), Date: Feb 15 - Mar 15
  Audience: High-Value Customers (12,450)
  Performance: Open 24.8% | Click 4.2% | Conv 218
  [View] [Pause] [Duplicate]

Campaign Card 2: "Product Launch -- Multi-Channel"
  Status: Scheduled (blue badge), Launch: Mar 1
  Channels: Email + Social + Ads
  Audience: All Active Contacts (148,230)
  [Edit] [Preview]

Campaign Card 3: "Abandoned Cart Recovery -- Journey"
  Status: Active (green), Ongoing
  Active contacts: 320, Converted: 89 (27.8%)
  Revenue attributed: NGN 4.2M
  [View] [Pause]

Campaign Card 4: "February Newsletter"
  Status: Completed (green check), Sent Feb 15
  Audience: Newsletter Subscribers (48,230)
  Performance: Open 22.1% | Click 3.8%
  [View Report] [Duplicate]

Campaign Card 5: "LinkedIn B2B Campaign"
  Status: Draft (gray), Not yet scheduled
  Target: B2B Decision Makers (3,400)
  Budget: NGN 500,000
  [Edit] [Delete]

TABLE VIEW:
| Campaign Name | Channel | Status | Audience | Sent/Reach | Open/Engage | Click | Conv | Revenue | Actions |
|---------------|---------|--------|----------|-----------|-------------|-------|------|---------|---------|
| Spring Sale 2026 | Email | Active | 12,450 | 12,450 | 24.8% | 4.2% | 218 | NGN 6.5M | ... |
| Product Launch | Multi | Scheduled | 148,230 | -- | -- | -- | -- | -- | ... |
| Abandoned Cart | Journey | Active | 320 active | 1,840 total | 28.4% | 8.1% | 89 | NGN 4.2M | ... |

SCREEN B -- EMAIL CAMPAIGN BUILDER (multi-step wizard):
Step 1 "Setup": Campaign name, description, tags, A/B test toggle
Step 2 "Audience": Select segment(s) from segment-service, estimated reach,
  suppression list (recent purchasers, unsubscribed), de-duplication preview
Step 3 "Content": Email editor (see next prompt for detail)
  Subject line: "Spring into savings -- 25% off everything!"
  Preview text: "Shop our biggest sale of the season..."
  From name: "Amina at ShopRight" | From email: "hello@shopright.com"
  Template selection or drag-and-drop builder
Step 4 "Schedule":
  Send now / Schedule for later
  Date picker + time picker with timezone selector
  Send time optimization toggle: "AI will choose optimal send time per recipient"
  Sending speed: Normal / Throttled (X emails per hour)
Step 5 "Review":
  Full campaign summary
  Estimated metrics (based on segment + template history)
  Compliance checklist: Unsubscribe link (check), Postal address (check), Subject not spammy (check)
  Spam score: 2.1/10 (green)
  "Send Test Email" button
  "Schedule Campaign" / "Send Now" primary button

Progress bar showing wizard steps. Back/next navigation. Save draft at any step.
```

#### Prompt F-005: Email Template Editor (1440px)

```
Design a desktop Email Template Editor at 1440x900px for the email-marketing-service
and content-service.

LAYOUT (Three-panel):
LEFT PANEL (260px) -- "Content Blocks":
- Draggable content blocks:
  -- Header (logo + nav links)
  -- Hero Image (full width with text overlay)
  -- Text Block (rich text)
  -- Image + Text (two-column)
  -- Button (CTA)
  -- Product Grid (2x2 or 3-column product cards)
  -- Social Links (icon row)
  -- Divider
  -- Spacer
  -- Footer (unsubscribe, address, social links)
  -- Dynamic Content (personalization block)
  -- Countdown Timer
- Each block: icon + name, drag handle, hover preview

CENTER PANEL (660px) -- "Email Canvas":
- Device preview toggle: Desktop (600px) | Mobile (375px)
- Email canvas with actual rendered preview
- Content blocks are drag-reorderable with drop indicators
- Click block to select: shows blue outline with drag handle, delete icon, duplicate icon
- Selected block: inline editing (double-click text to edit)
- Personalization tags: dropdown insert {{first_name}}, {{company}}, {{product_name}}
- Live preview updates as content changes

Example email content:
  [Header: ShopRight Logo | Shop | New Arrivals | Sale]
  [Hero: Full-width image of spring collection, "Spring Sale -- 25% Off Everything"]
  [Text: "Dear {{first_name}}, Spring is here and so are the savings..."]
  [Product Grid:
    Product 1: "Ankara Maxi Dress" | NGN 15,000 -> NGN 11,250 | [Shop Now]
    Product 2: "Leather Crossbody Bag" | NGN 22,000 -> NGN 16,500 | [Shop Now]
    Product 3: "Gold Statement Earrings" | NGN 8,500 -> NGN 6,375 | [Shop Now]
    Product 4: "Printed Silk Scarf" | NGN 12,000 -> NGN 9,000 | [Shop Now] ]
  [CTA Button: "Shop the Sale" -- large, rose/accent color]
  [Footer: Unsubscribe | View in browser | ShopRight, 15 Victoria Island, Lagos]

RIGHT PANEL (360px) -- "Block Settings":
When a block is selected, shows configuration:
- For Hero Image: image upload, alt text, overlay text, link URL, background color
- For Text: rich text editor (bold, italic, link, heading, list, alignment)
- For Button: text, URL, color, size, border radius, alignment
- For Product Grid: product source (manual / from catalog / dynamic), columns, CTA text
- For Dynamic Content: condition builder (e.g., "Show if customer_tier = Gold")

BOTTOM BAR (sticky):
- "Preview & Test" button (opens modal with rendered preview + send test email)
- "Save Template" button
- "Subject Line Tester" (AI-powered from content-service, shows predicted open rate)
- "Spam Score": 2.1/10 (green indicator)
- "HTML Source" toggle (for advanced users)

A/B TEST MODE:
- Split canvas: Variant A | Variant B side by side
- Variant controls: traffic split slider (50/50, 70/30, etc.)
- Test variable selector: Subject Line, From Name, Content, Send Time
- Winner criteria: Open Rate, Click Rate, Conversion Rate
- Sample size calculator: "Send to 20% of audience, winner to remaining 80%"
```

#### Prompt F-006: Customer Journey Builder (1440px)

```
Design a desktop Customer Journey Builder at 1440x900px for the journey-service.
Target user: Marketing Automation Specialist.

HEADER:
- "Journey: Welcome Series for New Subscribers" (editable name)
- Status: Draft | Active | Paused
- "Activate Journey" primary button (or "Pause" if active)
- "Settings" gear, "Analytics" chart icon

CANVAS (Full width, zoomable, pannable):
Visual node-based journey builder with drag-and-drop:

TRIGGER (Entry Point) -- Green rounded rectangle:
  "Contact Added to Segment: New Subscribers"
  Config: segment_id from segment-service, continuous entry

-> WAIT NODE -- Clock icon, gray:
  "Wait 1 hour"

-> ACTION: SEND EMAIL -- Violet envelope icon:
  "Welcome Email"
  Template: "welcome_series_01"
  Subject: "Welcome to ShopRight, {{first_name}}!"

-> DECISION NODE -- Blue diamond:
  "Opened welcome email?"
  Yes branch (right) -> No branch (down)

YES BRANCH:
-> WAIT: "Wait 2 days"
-> ACTION: SEND EMAIL:
  "Product Recommendations"
  Template: "welcome_series_02_engaged"
  Subject: "Picks just for you, {{first_name}}"

-> DECISION: "Clicked any product?"
  Yes -> ACTION: TAG CONTACT: "Engaged Subscriber"
       -> EXIT: "Journey Complete -- Engaged" (green)
  No -> WAIT: "Wait 3 days"
     -> ACTION: SEND EMAIL: "Exclusive Offer"
     -> EXIT: "Journey Complete -- Offered" (amber)

NO BRANCH (didn't open welcome):
-> WAIT: "Wait 3 days"
-> ACTION: SEND EMAIL:
  "We miss you! Here's 10% off"
  Template: "welcome_series_02_reengagement"
  Subject: "Don't miss out -- 10% off your first order"

-> DECISION: "Opened re-engagement email?"
  Yes -> merge to Product Recommendations path
  No -> ACTION: TAG CONTACT: "Low Engagement"
     -> EXIT: "Journey Complete -- Unengaged" (red)

NODE CONFIGURATION PANEL (right side, 360px, appears on node click):
- For Email nodes: template selector, subject line, from, preview
- For Wait nodes: duration (hours/days/weeks) or until specific date/time or until event
- For Decision nodes: condition builder (opened email, clicked link, visited page, purchased, custom event)
- For Action nodes: send email, send SMS, tag contact, update field, webhook, add to segment, remove from segment

JOURNEY ANALYTICS OVERLAY (toggle):
- Each node shows real-time metrics:
  -- Send Email "Welcome": 8,420 entered, 8,180 delivered, 2,045 opened (25%)
  -- Decision "Opened?": Yes: 2,045 (25%) | No: 6,135 (75%)
- Conversion through journey: funnel visualization
- Drop-off points highlighted in red

BOTTOM TOOLBAR:
- Zoom controls (+, -, fit to screen)
- Undo/Redo
- "Add Node" floating action button with node type selector
- Minimap (bottom-right corner) showing full journey zoom-out
```

#### Prompt F-007: Audience Segmentation Builder (1440px)

```
Design a desktop Audience Segmentation page at 1440x900px for the segment-service.

HEADER:
- "Segments" title
- "Create Segment" primary button
- Search segments, filter by type (Static, Dynamic, AI-Predicted)

SEGMENT LIBRARY (Grid view, 3 per row):
Segment Card 1: "High-Value Lagos Customers"
  Size: 12,450 contacts | Type: Dynamic (auto-refresh)
  Rules preview: "purchase_count > 3 AND region = 'Lagos' AND email_opt_in = true"
  Last synced: 30 min ago | Growth: +8% (30d)
  Used in: 3 active campaigns
  [View] [Edit] [Use in Campaign]

Segment Card 2: "Newsletter Subscribers"
  Size: 48,230 contacts | Type: Dynamic
  Rules: "subscribed_to = 'newsletter' AND status = 'active'"
  Growth: +2,340 this month

Segment Card 3: "Cart Abandoners (7d)"
  Size: 890 contacts | Type: Dynamic (real-time)
  Rules: "cart_created_at > 7_days_ago AND purchased = false"
  Size fluctuation: 850-950 daily

Segment Card 4: "AI: Likely to Churn"
  Size: 2,340 contacts | Type: AI-Predicted (ml-service)
  Confidence: 85% | Refresh: daily
  Prediction basis: "No engagement in 30d, declining purchase frequency"

SEGMENT BUILDER (Full page, opened on "Create" or "Edit"):

TOP: Segment name input, type selector (Dynamic/Static), description

RULES BUILDER (Visual query builder):
- "Contacts who match [ALL / ANY] of these conditions:"

Rule Row 1: [Property dropdown: "Purchase Count"] [Operator: "is greater than"] [Value: "3"]  [AND]
Rule Row 2: [Property: "Region"] [Operator: "equals"] [Value: "Lagos"]  [AND]
Rule Row 3: [Property: "Email Opt-In"] [Operator: "is"] [Value: "True"]  [AND]
Rule Row 4: [Property: "Last Purchase Date"] [Operator: "is after"] [Value: "2025-12-01"]

- "Add Condition" button, "Add Group" button (nested AND/OR groups)
- Property categories: Contact Info, Purchase History, Email Engagement,
  Social Engagement, Website Activity, Custom Fields, AI Predictions

PREVIEW PANEL (right side, 400px):
- Real-time audience size: "12,450 contacts match these rules"
- Audience size chart: trend over last 30 days
- Sample contacts list (first 10):
  | Name | Email | Purchase Count | Region | Last Purchase |
  | Chioma A. | chioma@... | 12 | Lagos | Feb 20 |
  | Adamu B. | adamu@... | 8 | Lagos | Feb 18 |
- Demographics breakdown: Gender split, Age distribution, Region distribution
- Overlap analysis: "8,200 of these contacts are also in 'Newsletter Subscribers'"

BOTTOM BAR:
- "Save Segment" button
- "Estimate Size" button (if not auto-updating)
- "Export to CSV" button
- "Use in Campaign" button
```

#### Prompt F-008: Marketing Analytics Deep Dive (1440px)

```
Design a desktop Marketing Analytics page at 1440x900px for the analytics-service
and attribution-service.

HEADER:
- "Marketing Analytics" title
- Date range: "Feb 1 - Feb 23, 2026" with comparison toggle: "vs Jan 1 - Jan 23"
- Channel filter: All | Email | Social | Ads | Organic
- "Export Report" button (PDF, CSV)

ROW 1 (KPI Cards with comparison, 5 across):
- Total Revenue Attributed: NGN 28.4M (+12% vs prior period, green)
- Cost Per Acquisition: NGN 2,340 (-8%, green -- lower is better)
- Return on Ad Spend: 4.2x (+0.3x, green)
- Customer Lifetime Value: NGN 145,000 (+5%, green)
- Marketing ROI: 6.8x (+1.2x, green)

ROW 2 (Full width -- Revenue Attribution by Channel):
- Stacked area chart over time (daily):
  X-axis: dates in February
  Stacked areas: Email (violet), Social (blue), Paid Ads (rose), Organic (green), Direct (gray)
  Total line on top
- Toggle between: Revenue, Conversions, Leads

ROW 3 (Two-column, 50/50):
- Left: "Multi-Touch Attribution" -- Sankey diagram or horizontal stacked bar:
  Showing customer journey touchpoints leading to conversion:
  Email (first touch): 35% credit
  Social Ad (mid): 15%
  Google Search (mid): 20%
  Retargeting Ad (mid): 15%
  Direct Visit (last touch): 15%
  Model selector: First Touch | Last Touch | Linear | U-Shaped | Algorithmic

- Right: "Conversion Funnel" -- Vertical funnel:
  Visitors: 245,000 (100%)
  Leads: 14,700 (6%)
  MQLs: 5,880 (2.4%)
  SQLs: 2,352 (0.96%)
  Opportunities: 941 (0.38%)
  Customers: 376 (0.15%)
  Conversion rates between each stage labeled

ROW 4 (Three-column, equal):
- Left: "Email Performance Summary":
  Campaigns sent: 12
  Total emails: 148,450
  Avg Open Rate: 24.8% (industry avg: 21.5% -- green)
  Avg Click Rate: 4.2% (industry avg: 2.6% -- green)
  Avg Unsubscribe: 0.3% (green, below 0.5% threshold)
  Deliverability: 97.2%
  Top campaign: "Spring Sale" (25.4% open, 4.8% click)

- Center: "Social Performance Summary":
  Total Posts: 45
  Total Reach: 456,000
  Total Engagements: 23,400
  Engagement Rate: 5.1%
  Followers gained: 1,200
  Best platform: Instagram (8.2% engagement)
  Best post: "Spring Collection Launch" (2,340 likes, 156 shares)

- Right: "Paid Advertising Summary":
  Campaigns active: 5
  Total Spend: NGN 4.2M
  Impressions: 1,250,000
  Clicks: 34,500
  CTR: 2.76%
  Conversions: 892
  CPA: NGN 4,709
  ROAS: 4.2x
  Top campaign: "Spring Sale SEM" (ROAS 5.8x)

ROW 5 (Full width):
- "Cohort Analysis" heatmap table:
  Rows: Acquisition cohort (month), Columns: Month 1, Month 2, Month 3...
  Values: Retention rate (%)
  Nov 2025: 100% | 68% | 52% | 41%
  Dec 2025: 100% | 71% | 55%
  Jan 2026: 100% | 73%
  Feb 2026: 100%
  Color: dark green (high retention) -> light green -> amber -> red (low)

Both light and dark themes. All charts include "View as Table" accessibility toggle.
```

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Marketing Dashboard (390px)

```
Design a mobile Marketing Dashboard at 390x844px for marketing managers on-the-go.

NAVIGATION:
- Bottom tab bar: Home, Campaigns, Analytics, Content, Profile
- Top bar: Brand logo, search icon, notification bell

HOME TAB:
- Greeting: "Good morning, Amina" + "Monday, Feb 23, 2026"
- Quick Actions (horizontal scroll of circular icons):
  New Campaign, Schedule Post, View Reports, Check Emails, Create Segment

- "Today's Snapshot" card:
  -- Emails sent today: 4,200
  -- Opens today: 1,040 (24.8%)
  -- Social engagements: 890
  -- Ad spend today: NGN 140,000

- "Active Campaigns" (scrollable list):
  Each card (compact):
  -- Channel icon + Campaign name
  -- Key metric (open rate for email, engagement for social, ROAS for ads)
  -- Status badge + mini trend line
  -- Example: "[Email] Spring Sale | 24.8% open | Active"
  -- Example: "[IG] Spring Collection | 5.1% engage | Active"
  -- Example: "[Ads] SEM Spring | ROAS 4.2x | Active"
  "View All (8 active)" link

- "Alerts" card:
  -- "Email bounce rate above 3% for campaign 'Flash Friday'" -- amber
  -- "Ad budget 85% spent for 'SEM Spring' -- 7 days remaining" -- amber
  -- "Social post scheduled for 12:00 ready for approval" -- info

- "Recent Performance" mini chart:
  Line chart (last 7 days): daily conversions
  Below: "Total conversions: 542 this week (+8%)"

Touch targets >= 44px. Pull-to-refresh. Skeleton loading.
```

#### Prompt F-010: Mobile Campaign Manager (390px)

```
Design a mobile Campaign Management flow at 390x844px.

SCREEN 1 -- "Campaigns" tab:
- Segment control: "Active (8)" | "Scheduled (3)" | "Drafts (5)" | "Completed"
- Filter chips: All Channels, Email, Social, Ads

Campaign list (full-width cards):
Card 1: "[Email] Spring Sale 2026"
  Active | Feb 15 - Mar 15 | 12,450 recipients
  Open: 24.8% | Click: 4.2% | Conv: 218
  [View] [...more]

Card 2: "[Multi] Product Launch"
  Scheduled | Mar 1 | Email + Social + Ads
  Audience: 148,230
  [Edit] [Preview]

Card 3: "[Journey] Welcome Series"
  Active | Ongoing | 320 contacts in journey
  Converted: 89 (27.8%) | Revenue: NGN 4.2M
  [View] [Pause]

SCREEN 2 -- "Campaign Detail" (tapped from list):
- Campaign name + status header
- Performance cards (horizontal scroll):
  [Sent: 12,450] [Delivered: 97.2%] [Opened: 24.8%] [Clicked: 4.2%] [Converted: 218]
- Performance chart (line, last 7 days of campaign)
- Engagement timeline: hourly opens/clicks for recent send
- Quick actions: Pause, Duplicate, Share Report
- "Audience" section: segment name, size
- "Content" section: subject line, preview thumbnail

SCREEN 3 -- "Quick Social Post" (bottom sheet, 85% height):
- Platform selector (toggle chips): Instagram, Twitter, LinkedIn, Facebook
- Media upload: camera/gallery picker (grid of recent photos + camera button)
- Caption textarea with character count per platform
  (Twitter: 248/280, Instagram: 1,200/2,200)
- Hashtag suggestions (from content-service AI)
- Schedule: Post Now | Schedule (date/time picker)
- "Post" full-width button

SCREEN 4 -- "Campaign Quick Create" (full page wizard):
- Step 1: Channel selection (large tappable cards)
- Step 2: Audience (select segment from list)
- Step 3: Content (select template, edit subject/body inline)
- Step 4: Schedule (date/time/timezone)
- Step 5: Review and confirm
- Compact wizard with back/next navigation, save draft any step
```

#### Prompt F-011: Mobile Analytics View (390px)

```
Design a mobile Analytics view at 390x844px for quick performance checks.

SCREEN 1 -- "Analytics" tab:
- Period selector (horizontal scroll): Today | 7d | 30d | 90d | Custom
- Channel filter chips: All | Email | Social | Ads

- Hero metric card (full width, 120px):
  "Total Revenue (30d)"
  NGN 28.4M
  +12% vs prior period (green arrow + comparison bar)

- Metric grid (2x2):
  Conversions: 1,892 (+8%)
  CPA: NGN 2,340 (-8%)
  ROAS: 4.2x (+0.3x)
  ROI: 6.8x (+1.2x)

- "Revenue by Channel" horizontal bar chart (compact):
  Email: NGN 14.8M (52%)
  Social: NGN 6.5M (23%)
  Ads: NGN 5.1M (18%)
  Organic: NGN 2.0M (7%)

- "Trend" line chart (full width, 200px):
  Daily conversions for selected period
  Touch to reveal data point tooltips

- "Top Campaigns" ranked list:
  1. Spring Sale Email | 218 conv | NGN 6.5M revenue
  2. Cart Abandon Journey | 89 conv | NGN 4.2M
  3. SEM Spring | 67 conv | NGN 3.8M
  4. IG Spring Collection | 45 conv | NGN 2.1M
  Each with channel icon and performance bar

SCREEN 2 -- "Campaign Report" (tapped from top campaigns):
- Campaign header: name, channel, date range, status
- Funnel visualization (vertical, mobile-optimized):
  Sent: 12,450 (100%)
  -> Delivered: 12,102 (97.2%)
  -> Opened: 3,002 (24.8%)
  -> Clicked: 508 (4.2%)
  -> Converted: 218 (1.8%)
- "Share Report" button (generates PNG summary)
- "Download CSV" button

SCREEN 3 -- "Real-Time" (live send monitoring):
- Live counter: "Sending: 8,420 of 12,450" with progress ring
- Real-time open counter with rolling sparkline
- Delivery status: Delivered 97.2% | Bounced 1.8% | Deferred 1.0%
- Auto-refreshes via WebSocket
```

#### Prompt F-012: Mobile Content Library (390px)

```
Design a mobile Content Library view at 390x844px for the content-service.

SCREEN 1 -- "Content" tab:
- Segment control: "Templates" | "Assets" | "Drafts"
- Filter chips: Email | Social | Ads | Landing Page

TEMPLATES VIEW:
- Grid (2 per row): Template thumbnail (170x120px), name, type badge, performance badge
  -- "Spring Sale Hero" | Email | Top performer
  -- "Product Showcase" | Email | --
  -- "Story Template" | Instagram | --
  -- "Ad Creative 1" | Meta Ads | Top performer
  -- Each: tap to preview, long-press for quick actions

ASSETS VIEW:
- Grid (3 per row): Image thumbnails (110x110px)
- Upload FAB button (bottom-right, camera + gallery)
- Asset types: Images, Videos, Documents, Brand Assets
- Search by tag or filename

SCREEN 2 -- "Template Preview" (full screen):
- Large preview (scrollable, rendered at mobile width)
- Bottom action bar: [Use in Campaign] [Edit] [Duplicate] [Share]
- "Performance" section: used in X campaigns, avg open rate, avg click rate
- "Tags": Promotional, Spring, Sale

SCREEN 3 -- "AI Content Assistant" (bottom sheet):
- "Generate content" prompt input
- Suggestions: "Write a subject line for spring sale", "Create social post for product launch"
- AI response with multiple variants to choose from
- "Use This" button copies to clipboard or inserts into active editor
- Powered by content-service AI capabilities
```

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-013: Tablet Marketing Dashboard (1024px)

```
Design a tablet-adapted Marketing Dashboard at 1024x768px.

ADAPTATIONS FROM DESKTOP (1440px):
- Sidebar collapsed to icon-only (72px) with tooltip labels
- KPI cards: 3+2 row layout
- Performance chart and Channel Mix: stacked vertically
- Three-column section becomes two-column (top campaigns + scheduled stacked on left, stats on right)
- Send time heatmap: scrollable horizontally, shows 12-hour range
- Attribution summary: simplified horizontal bar (no Sankey)

SPECIFIC LAYOUT:
Row 1: 3 KPI cards (Contacts, Active Campaigns, Open Rate)
Row 2: 2 KPI cards (Conversions, Spend)
Row 3: Performance Over Time chart (full width, 300px)
Row 4: Channel Mix donut (full width, 280px)
Row 5: Two-column -- Top Campaigns (left) | Upcoming Scheduled (right)
Row 6: Quick Stats (full width, horizontal scroll of stat cards)

Touch: >= 44px targets. Support landscape and portrait.
Portrait (768x1024): all sections single column, stacked.
```

#### Prompt F-014: Tablet Email Editor (1024px)

```
Design a tablet-adapted Email Template Editor at 1024x768px.

ADAPTATIONS:
- Three-panel layout becomes two-panel:
  -- Left panel: Email canvas (full width minus settings panel)
  -- Content blocks: accessible via bottom sheet FAB (floating action button)
  -- Settings panel: slides in from right (360px) when block selected,
     overlays canvas
- Canvas width: 600px centered (email standard width fits within tablet)
- Block drag-and-drop: supported with touch gestures
- Block settings: bottom sheet in portrait mode
- Preview toggle (desktop/mobile): in-canvas switch

PORTRAIT (768x1024):
- Single panel: canvas only, full width
- Content blocks: bottom sheet triggered by + FAB
- Settings: full-width bottom sheet on block selection
- Preview: full-screen overlay

JOURNEY BUILDER (tablet):
- Canvas with touch-optimized node dragging
- Nodes: larger touch targets (56x56px minimum)
- Node configuration: right panel in landscape, bottom sheet in portrait
- Zoom: pinch-to-zoom gesture
- Pan: two-finger drag

All interactive elements >= 44px. Support iPadOS split view.
```

---

## 4. Make Automation Prompts

### Prompt M-001: Email Campaign Send Orchestration

```
Create a Make scenario titled "Marketing -- Email Campaign Send Pipeline".

TRIGGER: Webhook from campaign-service when campaign status changes to "sending".
Payload: { campaign_id, campaign_name, segment_id, template_id, subject, from_name,
           from_email, scheduled_time, ab_test_config, send_settings }

STEP 1 -- Resolve Audience:
- Call segment-service: GET /v1/segments/{id}/contacts
- Apply suppression rules:
  -- Remove unsubscribed contacts
  -- Remove bounced contacts (hard bounce)
  -- Remove contacts who received email in last 24h (send frequency cap)
  -- Remove contacts in other active A/B tests
- Calculate final audience size
- If audience < 10: warn and pause for review

STEP 2 -- Prepare Send:
- Call content-service: GET /v1/templates/{id}/render
- Personalize template with merge tags per contact
- Generate tracking pixels and link wrapping (attribution-service UTM params)
- If A/B test: split audience per ab_test_config (50/50 or custom split)
- Pre-send spam check: validate content against spam rules

STEP 3 -- Execute Send:
- Call email-marketing-service: POST /v1/sends/bulk
  { campaign_id, contacts: [...], template_rendered, tracking_config }
- Throttle: respect sending limits (e.g., 500/min for warm-up, full speed for established)
- Publish Pulsar event: "persistent://billyronks/extract-marketing/event"
  { type: "campaign.send_started", campaign_id, contact_count }

STEP 4 -- Monitor Delivery:
- Poll email-marketing-service for delivery stats every 60 seconds
- Track: delivered, bounced (soft/hard), deferred
- If bounce rate > 5%: auto-pause send and alert marketing team
- If spam complaints > 0.1%: auto-pause and escalate

STEP 5 -- A/B Test Evaluation (if applicable):
- After sample period (e.g., 4 hours):
  -- Compare variant metrics (open rate, click rate)
  -- Statistical significance check (p < 0.05)
  -- If significant winner: send winner to remaining audience
  -- If no winner: send variant A (control) to remaining
- Notify marketing team with test results

STEP 6 -- Post-Send:
- Generate campaign report with full metrics
- Update analytics-service with final send data
- Publish Pulsar audit event with send summary
- If campaign is part of a journey: notify journey-service of completion

ERROR HANDLING: Delivery failures retry 3x per email. Full send pause if
infrastructure error. Alert marketing ops on Slack.
VOLUME: Up to 200,000 emails per campaign.
```

### Prompt M-002: Social Media Publishing Automation

```
Create a Make scenario titled "Marketing -- Social Media Auto-Publisher".

TRIGGER: Webhook from content-service when social post status = "approved_for_publish"
OR scheduled cron for posts with future publish_at time.

STEP 1 -- Validate Post:
- Parse: { post_id, platforms: ["instagram", "twitter", "linkedin"],
           content, media_urls[], hashtags, publish_at, campaign_id }
- Verify media meets platform requirements:
  -- Instagram: image ratio 1:1 or 4:5, < 10MB
  -- Twitter: < 280 chars, image < 5MB
  -- LinkedIn: < 3000 chars, image < 5MB
- Adapt content per platform (character limits, hashtag limits)

STEP 2 -- Publish to Platforms:
- For each platform:
  -- Instagram: publish via Meta Business API
  -- Twitter/X: publish via Twitter API v2
  -- LinkedIn: publish via LinkedIn Marketing API
  -- Facebook: publish via Meta Graph API
- Capture: post_id per platform, publish_url, initial metrics

STEP 3 -- Track Engagement:
- Schedule metric collection: 1h, 4h, 24h, 48h, 7d after publish
- Collect: impressions, reach, likes, comments, shares, saves, link clicks
- Store in analytics-service

STEP 4 -- Cross-Post Intelligence:
- Compare same content across platforms
- Identify best-performing platform for content type
- Feed back to content-service for future recommendations

STEP 5 -- Notify:
- Slack notification to social team: "Posted to {platforms}: {preview}"
- If engagement rate below threshold after 4h: suggest boost/promotion
- If negative comments detected (sentiment analysis from ml-service): alert for review

STEP 6 -- Attribution:
- Update attribution-service with social touchpoint data
- Track link clicks through to conversions
- Publish Pulsar event: social.post_published

ERROR HANDLING: Platform API failures retry 3x. Failed posts queue for manual review.
Rate limit handling with exponential backoff per platform.
```

### Prompt M-003: Lead Scoring and Segment Sync

```
Create a Make scenario titled "Marketing -- Real-Time Lead Scoring".

TRIGGER: Pulsar consumer on "persistent://billyronks/extract-marketing/event"
for contact activity events (email open, click, page visit, form submit, purchase).

STEP 1 -- Calculate Score Delta:
- Parse event: { contact_id, event_type, properties, timestamp }
- Score rules:
  -- Email open: +5 points
  -- Email click: +10 points
  -- Page visit (pricing page): +20 points
  -- Form submit (demo request): +50 points
  -- Purchase: +100 points
  -- Email unsubscribe: -30 points
  -- 30 days inactive: -20 points (daily decay)

STEP 2 -- Update Contact Score:
- Call segment-service: PUT /v1/contacts/{id}/score
  { score_delta, new_total_score, scoring_event }
- Recalculate segment membership based on new score

STEP 3 -- Threshold Actions:
- If score crosses MQL threshold (>= 100 points):
  -- Tag contact as "MQL" in segment-service
  -- Notify sales team (via CRM integration or email)
  -- Add to segment "Marketing Qualified Leads"
  -- Publish event: "lead.qualified"

- If score crosses SQL threshold (>= 200 points):
  -- Tag as "SQL"
  -- Create opportunity in CRM
  -- Assign to sales rep based on territory/round-robin
  -- High-priority notification to assigned rep

- If score drops below re-engagement threshold (<= 30 points):
  -- Add to segment "Re-engagement Needed"
  -- Trigger re-engagement journey (journey-service)

STEP 4 -- Segment Sync:
- Real-time segment recalculation when scores change
- Update ads-service custom audiences (Meta/Google/LinkedIn) for retargeting
- Sync with email-marketing-service suppression/inclusion lists

VOLUME: ~50,000 events/day. Processing latency target: < 2s per event.
```

### Prompt M-004: Campaign Performance Report Generator

```
Create a Make scenario titled "Marketing -- Weekly Performance Report".

TRIGGER: Scheduled every Monday at 07:00 WAT.

STEP 1 -- Gather Metrics:
- Call analytics-service: GET /v1/analytics/summary?period=last_7_days
- Call campaign-service: GET /v1/campaigns?status=active&period=last_7_days
- Call email-marketing-service: GET /v1/metrics/aggregate?period=last_7_days
- Call social-service: GET /v1/metrics/aggregate?period=last_7_days
- Call ads-service: GET /v1/metrics/aggregate?period=last_7_days
- Call attribution-service: GET /v1/attribution/summary?period=last_7_days

STEP 2 -- Generate Report:
Section A: "Executive Summary"
  -- Total revenue attributed: NGN XX.XM (+/-% vs prior week)
  -- Total conversions: X,XXX
  -- Marketing spend: NGN X.XM
  -- ROAS: X.Xx
  -- Key wins and concerns (auto-generated from metric deltas)

Section B: "Email Performance"
  -- Campaigns sent, total emails, avg open rate, avg CTR
  -- Top performing campaign with metrics
  -- Deliverability health (bounce, spam complaint rates)

Section C: "Social Performance"
  -- Posts published, total reach, total engagements
  -- Best performing post with preview and metrics
  -- Follower growth across platforms

Section D: "Paid Advertising"
  -- Active campaigns, total spend, impressions, clicks, conversions
  -- ROAS per campaign
  -- Budget utilization and pacing

Section E: "Attribution Insights"
  -- Channel contribution to conversions
  -- Top converting touchpoint paths
  -- Recommendations for budget reallocation

Section F: "Audience Growth"
  -- New contacts acquired
  -- Segment size changes
  -- List health (engagement rate, churn)

STEP 3 -- Format and Distribute:
- Generate PDF report with charts and tables
- Generate email-friendly HTML summary (top metrics only)
- Send to: Marketing Director, CMO, Team leads
- Post summary to #marketing-reports Slack channel
- Store in analytics-service for historical comparison
- Publish Pulsar audit event: "report.weekly_generated"

STEP 4 -- Action Items:
- If any metric declined > 10%: flag for team discussion
- If budget > 90% spent with > 3 days remaining: recommend budget adjustment
- If email bounce > 3%: recommend list cleaning
- Generate suggested next-week priorities based on this week's data
```

### Prompt M-005: Journey Trigger and Event Routing

```
Create a Make scenario titled "Marketing -- Journey Event Router".

TRIGGER: Pulsar consumer on "persistent://billyronks/extract-marketing/event"
for all contact lifecycle events.

STEP 1 -- Parse Event:
- Event types: contact.created, contact.purchased, contact.abandoned_cart,
  contact.subscribed, contact.unsubscribed, contact.page_viewed, contact.form_submitted,
  email.opened, email.clicked, email.bounced

STEP 2 -- Match to Active Journeys:
- Call journey-service: GET /v1/journeys/active
- For each active journey, check if event matches any trigger or decision node:
  -- "Welcome Series": triggered by contact.subscribed
  -- "Cart Abandonment": triggered by contact.abandoned_cart
  -- "Post-Purchase Follow-up": triggered by contact.purchased
  -- "Re-engagement": triggered by 30_days_inactive (calculated)
  -- Decision nodes: email.opened / email.clicked to route contacts through journey

STEP 3 -- Route Contact Through Journey:
- For trigger matches: Call journey-service: POST /v1/journeys/{id}/enter
  { contact_id, trigger_event, entry_time }
- For decision node matches: Call journey-service: PUT /v1/journeys/{id}/contacts/{contact_id}/advance
  { decision_node_id, outcome: true/false, event_data }

STEP 4 -- Execute Journey Actions:
- If next node is email: queue to email-marketing-service
- If next node is SMS: queue to notification provider
- If next node is wait: schedule wake-up event
- If next node is tag/update: call segment-service
- If next node is webhook: call external URL

STEP 5 -- Track Journey Metrics:
- Update journey-service with contact progression
- Track conversion events attributed to journey
- Update analytics-service with journey performance

VOLUME: ~20,000 events/day. Routing latency: < 1s per event.
FAULT TOLERANCE: Dead letter queue for failed routings. Retry with backoff.
```

### Prompt M-006: Ad Budget Monitor and Auto-Optimization

```
Create a Make scenario titled "Marketing -- Ad Budget Guardian".

TRIGGER: Scheduled every 4 hours + webhook on budget threshold alerts from ads-service.

STEP 1 -- Gather Ad Spend Data:
- Call ads-service: GET /v1/campaigns/active/metrics
- For each active campaign:
  -- Current spend vs daily budget
  -- Current spend vs lifetime budget
  -- Pacing: on track, under-pacing, over-pacing
  -- Performance: ROAS, CPA, CTR, conversion rate

STEP 2 -- Budget Alerts:
- If daily spend > 90% and < 50% of day elapsed: ALERT "Over-pacing"
  -- Auto-reduce bid by 10% (ads-service: PUT /v1/campaigns/{id}/bid)
  -- Notify marketing team
- If daily spend < 30% and > 50% of day elapsed: ALERT "Under-pacing"
  -- Suggest bid increase or audience expansion
  -- Notify marketing team
- If lifetime budget > 85% spent: ALERT "Budget nearly exhausted"
  -- Request budget approval extension

STEP 3 -- Performance Optimization (supervised action):
- If ROAS < 1x for > 3 consecutive days:
  -- Auto-pause campaign
  -- Notify marketing team: "Campaign X paused due to poor ROAS"
  -- Recommend: audience adjustment, creative refresh, bid change
- If CPA > 2x target:
  -- Reduce daily budget by 25%
  -- Notify for review
- If ROAS > 5x:
  -- Recommend budget increase
  -- Notify marketing team of opportunity

STEP 4 -- Cross-Platform Budget Allocation:
- Compare ROAS across platforms (Google, Meta, LinkedIn)
- Generate reallocation recommendation:
  "Shift 20% of LinkedIn budget to Meta -- ROAS is 3.8x vs 1.2x"
- Require human approval for budget shifts > NGN 100,000

STEP 5 -- Report:
- Daily budget utilization report
- Weekly optimization summary
- Alert history and actions taken
- Publish Pulsar event: "budget.check_completed"

AIDD GUARDRAIL: Budget changes > NGN 100,000 are supervised actions requiring
human approval. All budget modifications logged to audit trail.
```

---

## 5. Prompt Usage Guidelines

| Step | Action | Tool |
|------|--------|------|
| 1 | Copy the prompt text from Section 3 or 4 | Clipboard |
| 2 | Open Figma and activate the Make Design / AI plugin | Figma |
| 3 | Paste the prompt into the plugin input field | Plugin UI |
| 4 | Review generated output; verify marketing metrics are realistic | Manual |
| 5 | Connect components to shared Marketing Design System library | Figma Libraries |
| 6 | Replace sample brand data (ShopRight) with actual client brand | Manual |
| 7 | Export developer-ready assets using naming convention below | Figma Export |

**Marketing data guidelines:**
- Replace "ShopRight" with actual brand name and colors when available
- Adjust currency (NGN) to match target market
- Verify metric ranges are realistic for the vertical (B2C vs B2B)
- Social platform icons must use official brand guidelines (exact colors, proportions)
- Email templates must include CAN-SPAM/GDPR-compliant footer

---

## 6. Output Packaging Convention

```
Deliverable naming:
  MKT-{PromptID}-{Breakpoint}-{Theme}-v{Version}.fig
  Example: MKT-F003-1440-light-v1.fig

Layer naming:
  {Page}/{Section}/{Component}-{State}
  Example: Dashboard/KPICards/OpenRate-Default
  Example: EmailEditor/Canvas/HeroBlock-Selected

Asset export:
  /exports/marketing/{breakpoint}/{page}/
  Example: /exports/marketing/1440/dashboard/channel-mix-donut.png

Handoff tokens:
  /tokens/marketing-tokens.json (Style Dictionary format)

Channel icons:
  /exports/marketing/icons/channels/
  Example: /exports/marketing/icons/channels/email.svg

Social platform icons:
  /exports/marketing/icons/social/
  Must follow official brand guidelines per platform.
```

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial route bundle size | < 220 KB gzipped | Webpack bundle analyzer |
| Largest Contentful Paint (LCP) | < 1.5s | Lighthouse CI |
| First Input Delay (FID) | < 100ms | Web Vitals |
| Cumulative Layout Shift (CLS) | < 0.1 | Web Vitals |
| Time to Interactive (TTI) | < 2.5s | Lighthouse CI |
| Email editor load time | < 2s | Custom perf mark |
| Email template render | < 300ms | Custom perf mark |
| Journey builder canvas (50 nodes) | < 500ms | Custom perf mark |
| Analytics chart render (100K points) | < 500ms | Custom perf mark |
| Contact table (10K rows, virtualized) | < 300ms | Custom perf mark |
| Real-time send progress update | < 1s | WebSocket latency |
| API read latency (p99) | < 200ms | perf/targets.yaml SLO |
| API write latency (p99) | < 500ms | perf/targets.yaml SLO |
| Availability | >= 99.95% | perf/targets.yaml SLO |

---

## 8. AIDD Handoff Gate Template

```markdown
## Design Handoff Checklist -- ERP-Marketing

### Visual Completeness
- [ ] All pages designed for 1440px, 1024px, and 390px breakpoints
- [ ] Light and dark theme variants complete
- [ ] All component states documented (default, hover, active, disabled, error, loading, empty)
- [ ] Realistic marketing data used (no "Lorem ipsum" or "Test Campaign")
- [ ] Social platform brand colors match official guidelines

### Marketing UX
- [ ] Email template editor supports drag-and-drop with clear visual feedback
- [ ] Campaign send confirmation shows audience count and cost estimate
- [ ] A/B test configuration includes statistical significance guidance
- [ ] Journey builder nodes are clearly distinguishable by type
- [ ] Funnel visualizations include both percentage and absolute numbers

### Compliance
- [ ] Email templates include unsubscribe link in all previews
- [ ] Consent verification visible before bulk send operations
- [ ] GDPR/CAN-SPAM compliance checklist integrated in campaign review step
- [ ] Budget approval gates for spends above threshold
- [ ] Audit trail for all campaign sends and budget changes

### Accessibility
- [ ] Color contrast ratios verified (>= 4.5:1 text, >= 3:1 UI)
- [ ] All charts have "View as Table" alternative
- [ ] Touch targets >= 44x44px
- [ ] Focus order annotated (especially email editor and journey builder)
- [ ] Screen reader annotations for funnel charts and attribution graphs

### Developer Handoff
- [ ] Design tokens exported (Style Dictionary JSON)
- [ ] API data mapping documented (which service populates which component)
- [ ] WebSocket endpoints annotated for real-time send monitoring
- [ ] Pulsar event topics mapped to UI update points
- [ ] Feature flag annotation points marked (AI content generation, send-time optimization)
- [ ] Refine + Ant Design component mapping documented
```

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial comprehensive prompt set covering marketing dashboard, campaign builder, email editor, journey builder, segmentation, analytics, mobile and tablet views, and 6 Make automation workflows (email send, social publishing, lead scoring, weekly report, journey routing, ad budget optimization) |
