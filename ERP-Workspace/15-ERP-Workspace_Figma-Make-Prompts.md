# Figma & Make Prompts -- ERP-Workspace
> Version: 1.0 | Last Updated: 2026-02-23 | Status: Draft
> Classification: Internal | Author: AIDD System

---

## 1. Purpose

This document provides production-ready Figma design prompts and Make.com automation prompts for ERP-Workspace, the enterprise collaboration suite competing with Google Workspace and Microsoft 365. It covers all seven services (email, calendar, chat, meet, docs, drive, contacts) across desktop, tablet, and mobile breakpoints. Each prompt is copy-paste ready for Figma Make Mode or direct handoff to a design team.

ERP-Workspace integrates with ERP-IAM for authentication (OIDC/JWT), ERP-Platform for entitlements, and communicates over a Redpanda/Kafka event backbone using CloudEvents envelopes.

---

## 2. AIDD Guardrails (Apply To All Prompts)

### 2.1 User Experience And Accessibility
- WCAG 2.1 AA compliance on all screens; contrast ratio >= 4.5:1 for body text, >= 3:1 for large text
- Minimum 44x44px touch targets on all interactive elements (buttons, links, toggles, list items)
- Visible focus indicators (2px blue outline offset 2px) for keyboard navigation on every interactive element
- Screen reader landmarks: `<nav>`, `<main>`, `<aside>`, `<header>` with `aria-label` on every region
- Plain-language labels; no jargon in user-facing microcopy; action verbs for buttons ("Send", "Save Draft", "Join Meeting")
- Clear state communication: loading (skeleton), empty (illustration + CTA), error (inline message + retry), success (toast + green check)
- Reduced motion: honor `prefers-reduced-motion`; disable all animation when set
- Color is never the sole indicator of state; always pair with icon or text

### 2.2 Performance And Frontend Efficiency
- Route-level lazy loading via React.lazy + Suspense for each workspace module (email, calendar, chat, meet, docs, drive, contacts)
- Initial route bundle < 220KB gzip; critical CSS inlined; non-critical CSS loaded asynchronously
- Skeleton UIs for every data-dependent component; shimmer animation with 1.5s cycle
- Images: WebP with AVIF fallback; srcset for 1x, 2x, 3x densities; lazy loading with IntersectionObserver
- Virtual scrolling for lists exceeding 50 items (email inbox, chat messages, contact lists, drive files)
- Service worker for offline read access to cached emails and calendar; background sync for draft saves

### 2.3 Reliability, Trust, And Safety
- Optimistic UI for safe mutations (star, archive, label) with instant visual feedback and server reconciliation
- Recovery paths: undo toast (5s) for destructive actions (delete email, remove contact, leave meeting)
- Autosave for compose windows and document edits (debounced 2s); "All changes saved" indicator with timestamp
- Trust signals: end-to-end encryption badge on confidential emails, lock icon on private channels, "Recording" red dot on meetings
- Audit trail: all user actions emitted as CloudEvents to `erp.workspace.<entity>.<action>` topics
- DLP (Data Loss Prevention) warning banners when PII detected in outbound messages

### 2.4 Observability And Testability
- Analytics events: `workspace.email.compose`, `workspace.calendar.event_created`, `workspace.meet.joined`, `workspace.chat.message_sent`, `workspace.drive.file_uploaded`, `workspace.docs.document_saved`
- Error events: `workspace.error.network`, `workspace.error.auth_expired`, `workspace.error.upload_failed`
- Performance instrumentation: LCP, FID, CLS tracked per route; Lighthouse >= 95 for Performance and Accessibility
- Feature flags: all new features behind LaunchDarkly flags; instrumented with exposure events

---

## 3. Figma Design Prompts

### 3.1 Design System Foundation

#### Prompt F-001: Core Design Tokens

Design a comprehensive design token page for the ERP-Workspace design system. Use an 8px spatial grid throughout. Display the following token categories in organized sections with visual swatches and code references:

**Color Palette:**
- Primary: Blue-600 (#2563EB) with shades from Blue-50 (#EFF6FF) to Blue-900 (#1E3A5F)
- Secondary: Indigo-600 (#4F46E5) for accents and secondary actions
- Success: Emerald-600 (#059669) for confirmations, online status, sent indicators
- Warning: Amber-500 (#F59E0B) for caution states, pending actions, snoozed items
- Error: Red-600 (#DC2626) for errors, delete actions, failed states, leave meeting
- Neutral: Slate-50 (#F8FAFC) background, White (#FFFFFF) surface, Slate-900 (#0F172A) text primary, Slate-500 (#64748B) text secondary, Slate-200 (#E2E8F0) borders
- Dark Mode: Slate-950 (#020617) background, Slate-900 (#0F172A) surface, Slate-100 (#F1F5F9) text primary
- Semantic Colors: Email indicator Blue-500, Chat indicator Green-500, Calendar indicator Purple-500, Document indicator Orange-500, Meet indicator Teal-500, Drive indicator Yellow-500

**Typography:**
- Font family: Inter for UI text, JetBrains Mono for code and technical content
- Scale: Display 36px/44px 700, H1 30px/36px 700, H2 24px/32px 600, H3 20px/28px 600, H4 16px/24px 600, Body-L 16px/24px 400, Body-M 14px/20px 400, Body-S 12px/16px 400, Caption 11px/14px 500, Overline 10px/14px 700 uppercase tracking 0.1em

**Spacing:** 4px, 8px, 12px, 16px, 20px, 24px, 32px, 40px, 48px, 64px, 80px

**Border Radius:** 4px inputs, 6px buttons, 8px cards, 12px modals, 16px popovers, 9999px pills/avatars

**Elevation:** Level-0 none, Level-1 0 1px 3px rgba(0,0,0,0.1), Level-2 0 4px 6px rgba(0,0,0,0.1), Level-3 0 10px 15px rgba(0,0,0,0.1), Level-4 0 20px 25px rgba(0,0,0,0.15)

**Breakpoints:** Mobile 390px, Tablet 768px, Desktop 1024px, Wide 1440px

Show both light and dark mode variants side by side.

---

#### Prompt F-002: Component Library

Design a comprehensive component library page showing all reusable UI components for ERP-Workspace. Each component must show all states (default, hover, active, focus, disabled, loading, error) in both light and dark mode:

**Navigation Components:**
- App Switcher: 48px icon grid (3x3) showing Email (envelope), Calendar (calendar), Chat (message-circle), Meet (video), Docs (file-text), Drive (hard-drive), Contacts (users). Active app highlighted with Blue-600 background. Hover shows tooltip with app name.
- Sidebar Navigation: 240px width, collapsible to 64px (icon-only). Section headers (12px overline, Slate-500), nav items (14px, 40px height, 8px left padding, hover Slate-100 background, active Blue-50 background with Blue-600 left border 3px). Badge counts (red pill, white text, max "99+"). Collapse/expand chevron at bottom.
- Breadcrumb: Slash-separated path with links (Slate-500) and current page (Slate-900, font-weight 600)
- Tab Bar: Underline tabs (Blue-600 active indicator 2px), pill tabs (Blue-50 background active), icon tabs

**Content Components:**
- Message List Item: 72px height, left avatar (36px circle), sender name (14px 600), subject (14px 400), preview text (12px Slate-500 single line ellipsis), timestamp (12px Slate-400 right-aligned), unread indicator (3px Blue-600 left border + bold text). Star toggle icon. Checkbox on hover (left of avatar). Swipe actions (mobile): archive (green left), delete (red right).
- Calendar Event Card: Rounded rectangle with 3px left color border (calendar color), title (14px 600), time (12px Slate-500), optional location pin icon + text (12px). States: upcoming (full opacity), past (60% opacity), cancelled (strikethrough), tentative (dashed border).
- Chat Message Bubble: Avatar (32px) + sender name (12px 600) + timestamp (12px Slate-400) header. Message body (14px). Supports markdown rendering, code blocks (JetBrains Mono, Slate-50 background), image previews (max 400px wide, rounded 8px), file attachments (icon + name + size pill). Hover shows reaction bar and thread reply icon. Own messages right-aligned with Blue-50 background.
- Contact Card: Avatar (48px), name (16px 600), title and company (14px Slate-500), quick action icon buttons (email, chat, call, video) in row.
- File Card (Grid View): 200x180px, file type preview thumbnail (120px height), filename (14px truncated), modified date (12px Slate-500), owner avatar (24px). Hover overlay with actions (share, download, more).
- File Row (List View): 48px height, file type icon (24px), filename, owner, modified date, size columns. Hover highlight Slate-50.

**Action Components:**
- Compose FAB: 56px circle, Blue-600, white pencil icon, Level-3 shadow. Hover scale 1.05. Position: bottom-right 24px offset.
- Button: Primary (Blue-600 fill white text), Secondary (white fill Blue-600 border and text), Ghost (no border, Blue-600 text), Danger (Red-600 fill white text). Sizes: sm 32px, md 40px, lg 48px. Loading state: spinner replacing text.
- Icon Button: 40px square, transparent background, Slate-600 icon, hover Slate-100 background. Variants: with badge dot, with tooltip.
- Search Bar: 40px height, Slate-100 background, search icon left, Cmd+K hint badge right, focus ring Blue-600. Expanded: dropdown with recent searches and category filters.

**Feedback Components:**
- Toast Notification: 360px width, Level-3 shadow, icon (success green check, error red x, info blue info, warning amber triangle) + message (14px) + optional action link + dismiss X. Auto-dismiss 5s. Position: top-center or bottom-center.
- Skeleton Loader: Slate-200 shapes matching content layout, shimmer animation left-to-right 1.5s ease infinite.
- Empty State: Centered illustration (200x200px), heading (20px 600), description (14px Slate-500), CTA button.
- Typing Indicator: Three animated dots (8px circles, Slate-400) with staggered bounce animation.
- Online Status Dot: 10px circle, Green-500 (online), Amber-500 (away), Slate-300 (offline), Red-500 (do not disturb).
- Notification Badge: Red-500 pill, white text (11px 700), min-width 18px, height 18px, positioned top-right -4px offset.

**Collaboration Components:**
- Presence Avatars: Stacked avatar row (overlapping 8px), max 3 visible + "+N" pill. Each avatar has colored ring matching their cursor color.
- Collaborative Cursor: Colored caret (2px wide, 20px tall) with name label (10px white text on colored pill, positioned above cursor).
- Real-Time Badge: "Live" text in Green-500 pill with animated pulse dot, shown when real-time collaboration is active.

---

### 3.2 Desktop Pages (1440px)

#### Prompt F-003: Unified Inbox (1440px Desktop)

Design a unified inbox interface at 1440px viewport width that consolidates email, chat notifications, calendar alerts, document activity, and drive notifications into a single prioritized feed.

**Layout:** Three-column structure filling the full viewport height below a 56px top navigation bar.

**Top Navigation Bar (56px, full width):**
- Left: App switcher hamburger icon (24px), ERP-Workspace wordmark logo
- Center: Global search bar (480px width, 40px height) with magnifying glass icon, placeholder "Search mail, chats, files, events... (Cmd+K)", search scope dropdown (All / This Mailbox)
- Right: AI Focus toggle (sparkle icon + "Focus" label, toggle switch), notification bell with red badge count "12", settings gear icon, user avatar (32px circle) with online status dot, tenant/organization dropdown

**Left Sidebar (240px width, Slate-50 background, 1px right border Slate-200):**
- Top: Blue-600 "Compose" button (full width minus 16px padding, 40px height, white text, envelope-plus icon)
- Navigation sections with icon (20px, Slate-500) + label (14px) + optional count badge:
  - Inbox (bold, Blue-600 icon if active) with unread count "24"
  - Starred
  - Snoozed (with snooze clock icon)
  - Sent
  - Drafts with count "3"
  - Scheduled with count "1"
  - Spam with count "2"
  - Trash
- Divider line
- "Labels" section header with "+" icon to create:
  - Color dot (8px circle) + label name for each: "Work" (blue), "Personal" (green), "Finance" (red), "Travel" (purple)
- Divider line
- "Channels" collapsible section (Chat shortcut):
  - Hash icon + channel name: "#general" (unread dot), "#engineering" with count "5", "#design"
- Divider line
- "Direct Messages" collapsible section:
  - Avatar (24px) + name + online status dot: "Sarah Chen" (green), "James Wilson" (gray), "Priya Patel" (green)
- Bottom: Storage indicator "7.2 GB of 30 GB" with progress bar

**Center Column (400px width, white background, 1px right border Slate-200):**
- Column header (48px): "Inbox" title (16px 600), filter chips row: "All" (active, Blue-50 background Blue-600 text), "Email" (with envelope icon), "Chat" (message-circle icon), "Calendar" (calendar icon), "Files" (paperclip icon). Right side: "Sort: Date" dropdown, "Unread first" toggle.
- Message list grouped by date sections:
  - Date divider: "Today -- February 23, 2026" (12px overline, Slate-400, centered line)
  - Each message item (80px height):
    - Left: Checkbox (on hover) or sender avatar (36px), for email show sender initial/photo
    - Type indicator: colored dot (8px) top-left of avatar -- Blue for email, Green for chat, Purple for calendar, Orange for document, Yellow for drive
    - Content area: Sender name (14px 600) + timestamp right-aligned (12px Slate-400)
    - Subject/title (14px 400, truncated single line)
    - Preview text (12px Slate-500, single line truncated)
    - Right edge: Star toggle, attachment paperclip icon if applicable
    - Unread state: 3px Blue-600 left border, sender name and subject in font-weight 700
    - AI summary available: small sparkle icon (12px, Indigo-400) after subject
  - Sample data (12 items visible):
    1. [Email, Unread] "Amara Okafor" -- "Q1 Revenue Report Review" -- "Please review the attached Q1 revenue report before..." -- 10:42 AM -- has attachment
    2. [Calendar, Unread] "Calendar" -- "Product Sync in 30 minutes" -- "Meeting with Engineering team at 11:00 AM in Room 4B" -- 10:30 AM
    3. [Chat, Unread] "#engineering" -- "David Kim" -- "The deployment pipeline is green, merging the feature..." -- 10:15 AM
    4. [Email] "Newsletter Digest" -- "Weekly Industry Roundup" -- "Top stories in enterprise software this week..." -- 9:55 AM
    5. [Document] "Google Docs" -- "Priya Patel edited Product Roadmap 2026" -- "Added section on AI integration milestones..." -- 9:30 AM
    6. [Email, Starred] "Ben Torres" -- "Re: Partnership Agreement Draft" -- "Looks good to me. I've added a few comments on..." -- 9:12 AM
    7. [Drive] "Drive" -- "Sarah Chen shared 'Brand Assets v3' folder" -- "You now have edit access to the Brand Assets..." -- 8:45 AM
  - Date divider: "Yesterday -- February 22, 2026"
    8. [Email] "Compliance Team" -- "Annual Security Training Reminder" -- "Please complete your security awareness training..." -- 5:30 PM
    9. [Chat] "Sarah Chen (DM)" -- "Can you review the mockups?" -- "I've uploaded the latest designs to the shared..." -- 4:15 PM
    10. [Calendar] "Calendar" -- "Sprint Retrospective Summary" -- "Action items from yesterday's retro have been..." -- 3:00 PM

**Right Column (remaining width ~800px, white background):**
- Shows selected message content. Default state (no selection): centered empty state with envelope illustration and "Select a message to read" text.
- When email selected:
  - Header: Subject (20px 600), star toggle, labels (colored pills)
  - Metadata bar: Sender avatar (40px) + name (14px 600) + email (12px Slate-500), "to me" expandable, date "Feb 23, 2026, 10:42 AM", reply/forward/more dropdown icons
  - Message body: rendered HTML email content with proper typography
  - Attachment bar: file icon + name + size + download button for each
  - Thread: previous messages in collapsed accordion, expand icon
  - Action bar (bottom, sticky): Reply button (primary), Reply All, Forward, Archive, Delete (red), Move to, Label dropdown, Snooze clock dropdown, More (...)
- When chat selected: shows chat thread with reply input at bottom
- When calendar selected: shows event details with RSVP buttons (Accept/Tentative/Decline)

**States to design:** Default loaded, empty inbox, search active with results, bulk selection mode (checkboxes visible with bulk action bar), network error with retry.

---

#### Prompt F-004: Calendar Week View (1440px Desktop)

Design a calendar week view at 1440px viewport filling the full viewport below the 56px workspace nav bar.

**Top Navigation Bar:** Same as F-003 with Calendar app highlighted in app switcher.

**Calendar Toolbar (56px height, white background, bottom border):**
- Left: Back/forward arrow buttons (icon buttons), "Today" button (secondary, 36px), date range "Feb 23 -- Mar 1, 2026" (18px 600)
- Center: View switcher segmented control: "Day" | "Week" (active) | "Month" | "Schedule"
- Right: "Timezone: Africa/Lagos (WAT)" dropdown, settings gear, "+" Create Event button (Blue-600 primary, calendar-plus icon)

**Left Sidebar (240px, Slate-50 background):**
- Mini month grid (February 2026): 7 columns (S M T W T F S), days as small clickable numbers, today circled in Blue-600, selected week highlighted in Blue-50 strip. Arrow navigation for months.
- Divider
- "My Calendars" section (14px 600 header):
  - Checkbox + color swatch (12px circle) + calendar name: "Personal" (Blue-500 checked), "Work" (Green-500 checked), "Team Syncs" (Purple-500 checked), "Holidays" (Red-400 checked), "Birthdays" (Amber-500 unchecked)
- "Other Calendars" section:
  - "Engineering Team" (Teal-500 checked), "Company Events" (Indigo-500 checked)
- "Add calendar" link (Blue-600, plus icon)
- Divider
- "Upcoming Events" quick list:
  - Next 3 events with color dot + time + title: "11:00 AM Product Sync", "2:00 PM Design Review", "4:30 PM 1:1 with Manager"

**Main Grid (remaining width):**
- All-day events row (48px height): Horizontal bars spanning across days for multi-day events. Sample: "Company Offsite" spanning Wed-Fri (Purple-500 bar, white text). "Amara's Birthday" on Thursday (Amber-500 bar).
- Day column headers (48px each): Day abbreviation + date number. "Mon 23" (today: date number in Blue-600 circle, 28px). "Tue 24", "Wed 25", "Thu 26", "Fri 27", "Sat 28", "Sun 1"
- Time grid: Left axis showing hours 6:00 AM to 10:00 PM (24px per hour label, Slate-400 12px). Horizontal gridlines every hour (Slate-100) and half-hour (Slate-50 dashed). 7 day columns separated by Slate-200 vertical lines.
- Current time indicator: Horizontal red line (#DC2626) spanning the full grid width at 10:45 AM position. Small red circle (8px) at the left edge.
- Calendar events as rounded-corner cards (8px radius) placed on the grid:
  - 9:00-10:00 Mon: "Sprint Planning" (Green-500 left border 3px, Green-50 fill), "Conf Room A" location text, video icon
  - 11:00-12:00 Mon: "Product Sync" (Purple-500 border, Purple-50 fill), "3 attendees" avatar stack
  - 2:00-3:30 Mon: "Design Review" (Blue-500 border, Blue-50 fill), "Figma" link
  - 10:00-10:30 Tue: "1:1 Sarah" (Teal-500 border, Teal-50 fill)
  - 3:00-4:00 Tue: "Engineering All-Hands" (Green-500 border), video icon, "42 attendees"
  - 9:30-10:00 Wed: "Daily Standup" (Green-500 border), recurring icon
  - 1:00-2:30 Wed: "Client Demo" (Red-500 border, Red-50 fill), "External" badge
  - Overlapping events (Thu 2:00-3:00): "Budget Review" and "Vendor Call" displayed side-by-side within the same time slot, each taking 50% width
- Minimap (bottom-right corner, 120x80px, semi-transparent Slate-900/80 background): shows the full day range with a viewport indicator rectangle

**Quick Event Creation:** Clicking and dragging on the grid creates a temporary blue overlay with an inline form: title input, time display, calendar color selector, "More options" link, "Save" button.

**Event Popover (on click):** 320px wide card with event title (16px 600), date/time, location with map link, video meeting "Join" button (Blue-600), attendees avatar list with RSVP status (accepted=green check, declined=red x, tentative=yellow question), description preview, "Edit" and "Delete" buttons.

**States:** Default loaded week, event being dragged to reschedule (ghost + drop indicator), event creation popover, event detail popover, empty day state.

---

#### Prompt F-005: Chat Workspace (1440px Desktop)

Design a team messaging interface at 1440px viewport. Three-column layout below the 56px workspace nav bar.

**Left Sidebar (260px, Slate-900 dark background, white text -- Slack-like dark theme for chat sidebar):**
- Top: Workspace name "Acme Corp" dropdown (16px 600) with online status selector (green dot + "Active")
- Search bar (36px, Slate-800 background, Slate-400 placeholder text, magnifying glass icon)
- Divider (Slate-700)
- "Channels" section (12px overline, Slate-400):
  - Create channel "+" icon next to header
  - Items: Hash icon (16px, Slate-400) + channel name (14px, Slate-200) + optional unread count badge (Blue-500 pill):
    - "#general" (bold white if unread)
    - "#engineering" with badge "5" (Blue-500)
    - "#design" with badge "2"
    - "#product"
    - "#random"
    - "#announcements" (lock icon for restricted)
  - "Browse all channels" link (Slate-400, 12px)
- Divider
- "Direct Messages" section (12px overline):
  - Items: Avatar (24px circle) + name (14px) + status dot:
    - "Sarah Chen" (Green-500 online dot), bold = unread
    - "James Wilson" (Slate-400 offline dot)
    - "Priya Patel" (Green-500), has unread badge "3"
    - "David Kim" (Amber-500 away dot)
    - "Amara Okafor" (Red-500 DND dot)
  - "Invite people" link

**Center Column (flex, min 500px, white background):**
- Channel Header (56px, bottom border):
  - Hash icon + "#engineering" (16px 600), topic text "Frontend architecture and code reviews" (12px Slate-500, truncated)
  - Right: Member count "18" with users icon (click opens member list), pin icon (click shows pinned messages), search icon, phone icon (start huddle), video icon (start meeting), channel settings kebab menu
- Message Feed (scrollable, flex-grow):
  - Date divider: "Monday, February 23, 2026" centered, Slate-400 text with horizontal lines
  - Messages with rich content:
    1. **Sarah Chen** (avatar 36px, name 14px 600, timestamp "9:12 AM" Slate-400):
       "Good morning team! The new CI pipeline is ready for review. Here's the summary:"
       - Attached code block (JetBrains Mono, Slate-50 background, 12px, with language label "yaml" and copy button):
         ```
         stages:
           - build
           - test
           - deploy
         ```
       - Reactions row: thumbs-up (3), rocket (2), eyes (1) -- each as a pill with emoji + count
       - Thread indicator: "4 replies" with 3 stacked mini avatars, "Last reply 10 min ago" -- Blue-600 link
    2. **David Kim** (timestamp "9:25 AM"):
       "Looks great @Sarah! One question -- are we gating deploys on coverage thresholds?"
       - @mention highlighted in Blue-100 background Blue-700 text
    3. **Bot: CI Pipeline** (bot badge, gray avatar with gear icon, timestamp "9:30 AM"):
       Embedded card: Green-500 left border, "Build #247 passed" title, "main branch - 3m 42s" subtitle, "View Details" link
    4. **Priya Patel** (timestamp "9:45 AM"):
       Message with inline image: screenshot preview (max 400px wide, 8px radius, click to expand)
       "Here are the updated mockups for the notification center. Let me know your thoughts!"
    5. **James Wilson** (timestamp "10:02 AM"):
       "Shared a file:" -- File attachment card: PDF icon, "Q1_Architecture_Review.pdf", "2.4 MB", download button
  - Unread divider: "New messages" red line with label (Red-500)
  - Typing indicator at bottom: "Sarah Chen is typing..." with animated three dots
- Compose Area (bottom, 120px min height, top border):
  - Rich text input with placeholder "Message #engineering" (14px)
  - Formatting toolbar (32px height): Bold, Italic, Strikethrough, Code, Link, Ordered List, Bullet List, Blockquote, Code Block
  - Bottom row: Attachment button (paperclip), Emoji picker button (smiley), Mention button (@), Slash command button (/), AI assist button (sparkle icon, "Help me write"), Schedule send button (clock). Right side: Send button (Blue-600, arrow-up icon, 36px circle)

**Right Panel (320px, collapsible, white background, left border):**
- Default: collapsed (not visible, center column takes full remaining width)
- Thread view (when "4 replies" clicked):
  - Header: "Thread in #engineering" with close X button
  - Original message displayed in full
  - Reply messages below
  - Reply input at bottom with "Reply..." placeholder
  - "Also send to #engineering" checkbox
- Channel details view (when settings clicked):
  - Channel name, topic, description (editable)
  - "Members" section with avatar list and "Add people" button
  - "Pinned Messages" section
  - "Files" section showing shared files
  - "Integrations" section showing connected bots/webhooks
  - "Notification preferences" dropdown: All messages, Mentions only, Nothing

**States:** Channel loaded with messages, empty channel ("No messages yet, start a conversation!"), thread open, member list open, file upload in progress (progress bar on compose area), message edit mode (yellow background on edited message), deleted message ("This message was deleted" gray italic text).

---

#### Prompt F-006: Video Meeting Room (1440px Desktop)

Design a video conferencing interface at 1440px viewport in full-screen mode (no workspace nav bar).

**Video Grid Area (full viewport minus bottom controls bar):**
- Gallery View (default): Responsive grid of participant video tiles. For 4 participants: 2x2 grid with 16px gaps. Each tile:
  - Video feed fills the tile with 12px border radius
  - Bottom-left overlay: participant name (14px white, text-shadow for readability) + role badge "Host" (if applicable, small Slate-600/80 pill)
  - Bottom-right overlay: microphone icon (muted = red microphone-off icon with red circle background, unmuted = white microphone icon)
  - Top-right overlay (on hover): Pin button, more options kebab
  - Active speaker: 3px Blue-500 animated glow border
  - Video off state: Slate-800 background with large centered avatar (80px) and name below
  - Poor connection: amber wifi-off icon in top-left corner
- Sample participants: "You (Abiola Ogunsakin)" (self-view, mirrored, small "You" badge), "Sarah Chen" (active speaker, blue glow), "David Kim" (muted indicator), "Priya Patel"
- For 9+ participants: 3x3 grid with pagination arrows left/right and "Page 1 of 2" indicator
- Speaker View: 1 large tile (75% width) + vertical filmstrip (25% width) of other participants
- Presentation View: shared screen large (80% width) + sidebar filmstrip. Presenter name and "Presenting" badge overlay on shared content.

**Top Bar (transparent overlay, appears on mouse movement):**
- Left: Meeting title "Product Sync - Weekly" (16px white 600), elapsed time "00:23:15" (14px Slate-300), lock icon if end-to-end encrypted
- Center: Recording indicator (when active): pulsing red dot (8px) + "Recording" text (14px Red-400)
- Right: Layout switcher (gallery/speaker/presentation icons), full-screen toggle, closed captions "CC" toggle, AI meeting notes button (sparkle + "Notes"), clock showing current time "10:23 AM"

**Bottom Control Bar (80px height, centered, rounded pill 600px width, Slate-900/90 background, backdrop-blur, Level-4 shadow):**
- Buttons (48px circle each, 12px gap):
  - Microphone toggle: White icon (unmuted) / Red background with microphone-off icon (muted). Small dropdown caret for device selection.
  - Camera toggle: White icon (on) / Red background with camera-off icon (off). Dropdown for device selection.
  - Screen Share: Monitor-arrow-up icon, white. Active state: Blue-500 background. Dropdown: "Your entire screen", "Application window", "Browser tab".
  - More: Ellipsis icon, white. Menu: "Virtual backgrounds", "Noise cancellation", "Speaker stats", "Settings", "Report a problem"
  - Reactions: Smiley icon, white. Popover shows: thumbs-up, clap, heart, laugh, surprised, raised-hand as 48px emoji buttons. Raised hand shows count badge when others have hands raised. Floating reaction animations appear above the video grid for 3s when someone reacts.
  - Chat: Message-circle icon, white. Badge count "3" for unread. Opens right side panel with chat messages.
  - Participants: Users icon, white. Count "4" displayed. Opens right side panel with attendee list: each row has avatar, name, host/co-host badge, mic/camera status icons, "Mute" and "Remove" actions for host.
  - Breakout Rooms: Grid icon (host only). Opens modal with room creation, participant assignment drag-and-drop.
  - Record: Circle icon, white. Active: red filled circle with pulse animation. Requires host permission confirmation dialog.
- Far right (separated by 24px gap): "Leave" button (Red-600 pill, 44px height, "Leave" text + phone-off icon). Host sees dropdown: "Leave meeting" / "End meeting for all" (red, confirmation required).

**AI Captions Bar (when CC enabled):** Translucent Slate-900/70 bar (64px height) just above controls, full width, with caption text (16px white) showing speaker name in bold + spoken text. Auto-scroll. Option to select caption language from dropdown.

**AI Meeting Notes Panel (when notes enabled, right side, 360px):** Real-time AI-generated meeting summary. Sections: "Key Points" (bullet list auto-updating), "Action Items" (checkbox list with assignee avatars), "Decisions" (numbered list). "Copy Notes" button at bottom.

**Waiting Room Overlay (before admission):** Full-screen overlay on top of blurred/dimmed video. Centered card (400px): meeting title, "Waiting for the host to let you in" message, self-preview video (240px, rounded), microphone/camera pre-check toggles, "You'll join as Abiola Ogunsakin" text, "Leave" button.

**States:** Gallery view 4 participants, speaker view, presentation sharing, waiting room, meeting ended summary screen ("Meeting ended -- Duration 45:23" with links to recording and AI notes).

---

#### Prompt F-007: Document Editor (1440px Desktop)

Design a collaborative document editor at 1440px viewport using ONLYOFFICE-style editing.

**Top Bar (48px height, white, bottom border):**
- Left: Back arrow to Drive, document icon, editable title "Q1 Product Roadmap" (16px 600, click to rename, shows edit cursor), star toggle, move/copy dropdown
- Center: Auto-save indicator "All changes saved" (12px Slate-500, green check icon) or "Saving..." (12px Slate-400, spinner) -- updates on every edit
- Right: Share button (Blue-600, "Share" text + users-plus icon), collaborator presence avatars (3 stacked, 28px, colored rings: Sarah=Teal, David=Orange, each with colored cursor matching ring), "Editing" / "Suggesting" / "Viewing" mode dropdown, comments panel toggle (message-circle with badge "5"), version history clock icon, more (print, download, page setup)

**Toolbar (44px height, white, bottom border, scrollable on smaller viewports):**
- Group 1: Undo/Redo arrow buttons
- Separator
- Group 2: Font family dropdown "Inter" (120px), Font size dropdown "11" (48px), Bold (B), Italic (I), Underline (U), Strikethrough (S), Text color (A with colored underline + dropdown), Highlight color (paint bucket + dropdown)
- Separator
- Group 3: Heading style dropdown "Normal text" (shows H1, H2, H3, Normal, Title, Subtitle)
- Separator
- Group 4: Align left/center/right/justify, line spacing dropdown
- Separator
- Group 5: Bullet list, Numbered list, Checklist, Indent/outdent
- Separator
- Group 6: Insert link, Insert image, Insert table (grid selector), Insert horizontal rule
- Separator
- Group 7: Comment (message-square-plus), Suggest edit (pencil with plus)
- Separator
- Group 8: AI Assistant dropdown (sparkle icon, Blue-600): "Summarize", "Expand", "Rewrite", "Translate", "Fix grammar", "Generate outline"

**Document Canvas (centered, remaining height):**
- Light gray background (Slate-100)
- White page area (816px wide representing A4 letter proportions), Level-1 shadow
- Visible margins (1 inch = 72px, shown with rulers if rulers enabled)
- Top ruler (horizontal, px markings) and left ruler (vertical) in Slate-200
- Document content with realistic formatted text:
  - Title: "Q1 2026 Product Roadmap" (24px 700)
  - Subtitle: "ERP-Workspace Engineering" (16px 400 Slate-500)
  - Heading 2: "1. Executive Summary" (20px 600)
  - Body paragraph with lorem-equivalent business text
  - Heading 2: "2. Key Milestones"
  - Table (4 columns: Milestone, Owner, Date, Status): 5 rows of sample data
  - Heading 2: "3. AI Integration Roadmap"
  - Bulleted list with 4 items
  - An inserted image placeholder (300x200px, Slate-100 with image icon)
- Collaborative cursors: Sarah's teal cursor in paragraph 2 with "Sarah Chen" label above, David's orange cursor in the table with "David Kim" label
- Text selection by another user: highlighted in their color (teal highlight for Sarah's selection)
- A comment anchor: yellow highlighted text "AI Integration Roadmap" with a small comment icon in the right margin, thread indicator showing "3"

**Right Panel (320px, collapsible):**
- Comments Tab:
  - Thread 1: Priya Patel avatar + "Great progress on the milestones! Can we add Q2 targets?" + timestamp "2h ago". Reply from Abiola: "Added in section 4." + "Resolved" button
  - Thread 2: Sarah Chen avatar + "Should we include the NLP pipeline here?" + timestamp "45m ago". Unresolved, highlighted yellow.
  - Thread 3: David Kim avatar + "Typo in row 3 of the table" + timestamp "10m ago"
  - "Add comment" button at bottom
- Version History Tab:
  - Timeline list: "Current version" (bold), "Feb 23, 10:30 AM - Abiola Ogunsakin (5 edits)", "Feb 22, 4:15 PM - Sarah Chen (12 edits)", "Feb 22, 11:00 AM - David Kim (3 edits)", "Feb 21, 9:00 AM - Created by Abiola Ogunsakin"
  - Each version: "Restore" button and "Name this version" link

**Bottom Status Bar (28px, Slate-50, top border):**
- Left: Page "1 of 3", Word count "1,247 words", Character count "7,832"
- Center: Language "English (US)" dropdown
- Right: Zoom "100%" with +/- buttons

**States:** Editing normally, suggesting mode (green highlight on insertions, red strikethrough on deletions), viewing mode (read-only, toolbar grayed out), comment panel open, version history comparison (split diff view), offline mode ("You're offline. Changes will sync when reconnected." banner).

---

#### Prompt F-008: File Browser / Drive (1440px Desktop)

Design a cloud file storage and management interface at 1440px viewport.

**Top Navigation Bar:** Same as F-003 with Drive app highlighted.

**Left Sidebar (240px, Slate-50 background):**
- "New" dropdown button (Blue-600 primary, plus icon, full width minus padding):
  - Menu: "Upload file" (upload-cloud icon), "Upload folder", divider, "New folder" (folder-plus), "New document" (file-text), "New spreadsheet" (table), "New presentation" (layout), "New form" (clipboard)
- Navigation:
  - "My Drive" (folder icon, active = Blue-600 text + Blue-50 background)
  - "Shared with me" (users icon) with badge "4 new"
  - "Team Drives" (expandable): "Engineering" (folder icon), "Design" (folder icon), "Marketing" (folder icon)
  - "Recent" (clock icon)
  - "Starred" (star icon)
  - "Trash" (trash icon)
- Divider
- Storage usage: "12.4 GB of 50 GB used" (14px), progress bar (24% filled, Blue-500), "Manage storage" link (12px Blue-600)

**Main Content Area (remaining width):**
- Toolbar (48px):
  - Breadcrumb: "My Drive" > "Projects" > "2026" (each segment is a link, Slate-500, current bold Slate-900)
  - Right side: Search input (240px), View toggle: Grid icon (active) / List icon, Sort dropdown "Modified" with arrow, Filter funnel icon dropdown (Type: Documents/Spreadsheets/Presentations/PDFs/Images/Videos/Folders; Owner: Anyone/Me/Specific people; Modified: Today/Week/Month/Year)
- Quick Access Row (when in My Drive root, 180px height):
  - "Quick Access" header (14px 600)
  - Horizontal scroll of recent/suggested file cards (160x120px each): Large file type icon or preview thumbnail, filename below (12px truncated), modified time (11px Slate-400). 6 cards visible. Sample: "Q1 Report.docx" (doc icon blue), "Budget 2026.xlsx" (sheet icon green), "Brand Guide.pdf" (PDF red), "Team Photo.jpg" (actual thumbnail), "Sprint Planning.pptx" (presentation orange)
- File Grid (default view):
  - Folder cards (200x160px, Slate-50 background, 8px radius, 1px Slate-200 border):
    - Large folder icon (48px, Slate-400 or colored for shared)
    - Folder name (14px 600, truncated)
    - Item count (12px Slate-500) "12 items"
    - Shared indicator: avatar stack if shared
    - Sample folders: "Designs" (shared, 2 avatars), "Documentation" (24 items), "Assets" (shared, 3 avatars), "Archive"
  - File cards (200x200px, white background, 8px radius, Level-1 shadow on hover):
    - Preview area (200x140px): actual image preview for images, first page for PDFs, type-specific large icon for documents. Slate-50 background with centered icon for unsupported previews.
    - Info area (60px): Filename (14px, truncated), modified date (12px Slate-400)
    - Hover state: overlay with action icons (share, download, more), checkbox top-left
    - Sample files: "Architecture Diagram.png" (image preview), "Q1 Revenue.xlsx" (spreadsheet icon green, "Jan 15"), "Meeting Notes.docx" (doc icon blue, "Feb 22"), "Product Demo.mp4" (video icon red, play button overlay, "Feb 20"), "API Spec.pdf" (PDF icon red, "Feb 18")
- File List (alternate view):
  - Table columns: Checkbox | File icon + Name | Owner (avatar + name) | Last modified | File size | Shared with (avatar stack)
  - Sortable column headers (click to sort, arrow indicator)
  - Row height 48px, hover highlight Slate-50
  - Selected row: Blue-50 background, checkbox checked

**Selection Mode (when files selected):**
- Top action bar replaces breadcrumb: "3 selected" count, "Share" button, "Download" button, "Move" button, "Star" toggle, "Delete" button (red), "More..." dropdown, "Clear selection" X button

**Right Details Panel (320px, appears when info icon clicked or keyboard shortcut):**
- File preview (large, top area)
- Metadata: Name, Type, Size ("2.4 MB"), Owner (avatar + name), Location (breadcrumb), Created "Feb 10, 2026", Modified "Feb 22, 2026"
- "Sharing" section: List of people with access (avatar + name + permission level dropdown: Viewer/Commenter/Editor)
- "Activity" tab: Timeline of actions ("Abiola edited - 2h ago", "Sarah viewed - yesterday", "Created - Feb 10")
- "Version History" section: Version list with "Restore" and "Download" actions per version

**Context Menu (right-click):** Open, Open with, Share, Get link (copy icon), Move to, Add to starred, Rename, Make a copy, Download, Organize (add to folder), File information, Version history, Download, Delete.

**Drag and Drop:** Files being dragged show a blue outline drop zone on target folders. "Upload" drop zone appears at top when dragging from desktop.

**States:** My Drive with files, empty folder ("This folder is empty. Drag files here or click New."), search results, upload in progress (progress bar per file in bottom-right notification area), storage quota exceeded warning.

---

### 3.3 Mobile Pages (390px)

#### Prompt F-009: Mobile Email Inbox (390px)

Design the email inbox for a 390px mobile viewport (iPhone 15 Pro proportions).

**Status Bar (44px, system):** Time, signal, battery -- standard iOS/Android status bar.

**App Header (56px):**
- Left: Hamburger menu icon (opens sidebar drawer overlay)
- Center: "Inbox" title (18px 600) with unread count in parentheses "(24)"
- Right: Search icon (magnifying glass), compose FAB (see below)

**Filter Bar (44px, horizontal scroll):**
- Pill-shaped filter chips: "All" (active, Blue-600 fill white text), "Unread" (Slate-200 fill), "Starred", "Attachments", "From me"

**Email List (remaining height minus bottom nav):**
- Each email item (88px height, full width):
  - Left: Sender avatar (40px circle, left margin 16px)
  - Content (flex, left margin 12px, right margin 16px):
    - Row 1: Sender name (14px 600, truncated) + timestamp right-aligned (12px Slate-400): "Amara Okafor" -- "10:42 AM"
    - Row 2: Subject (14px 400, single line truncated): "Q1 Revenue Report Review"
    - Row 3: Preview (12px Slate-500, single line truncated): "Please review the attached Q1 revenue report before..."
  - Right edge: Star icon (20px, Slate-300 or Amber-400 if starred)
  - Unread state: sender name font-weight 700, Blue-600 dot (8px) to left of avatar
  - Attachment indicator: small paperclip icon after preview text
  - Divider: 1px Slate-100 line, 68px left indent (aligned with text, not avatar)
- Swipe gestures (shown with partial swipe state):
  - Swipe right: Green background reveals Archive icon + "Archive" text
  - Swipe left: Red background reveals Trash icon + "Delete" text
- Pull-to-refresh indicator at top (spinner)

**Compose FAB (56px):**
- Blue-600 circle, bottom-right corner, 16px margin from edges
- White pencil (edit) icon, Level-3 shadow
- Scrolls with content but always visible above bottom nav

**Bottom Navigation Bar (80px with safe area):**
- 5 tabs equally spaced: Email (envelope, active Blue-600), Calendar (calendar), Chat (message-circle, badge "5"), Meet (video), More (grid)
- Each: 24px icon + 10px label, inactive Slate-400, active Blue-600

**Sidebar Drawer (overlays from left, 300px width, Slate-900/50 scrim on remaining area):**
- Account switcher at top: avatar + email + expand arrow
- Navigation items matching desktop sidebar: Inbox (24), Starred, Snoozed, Sent, Drafts (3), Spam, Trash
- Labels section with color dots
- Settings link at bottom

**States:** Loaded inbox, empty inbox (illustration + "No emails yet"), pull-to-refresh active, swipe action in progress, search overlay (full-screen with recent searches and results), network offline banner (Amber-500 bar "You're offline" at top).

---

#### Prompt F-010: Mobile Calendar Day View (390px)

Design the calendar day view for 390px mobile.

**App Header (56px):**
- Left: Hamburger menu icon
- Center: "Mon, Feb 23" (16px 600)
- Right: Today button (pill, 32px), "+" create event icon button

**Day Selector Strip (80px):**
- Horizontal scrollable row of 7 day columns (each 56px wide, centered):
  - Day abbreviation (12px Slate-400): "SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"
  - Date number (20px): "22", "23" (today: Blue-600 circle background, white text), "24", "25", "26", "27", "28"
  - Event dot indicators below date (3 dots max, colored to match calendars)
- Selected day: column has Blue-50 background

**Time Grid (scrollable, remaining height minus bottom nav):**
- Left time axis (48px): Hours "6 AM", "7 AM"... "10 PM" in 12px Slate-400
- Horizontal gridlines every hour (Slate-100)
- Current time: red line with dot at left edge
- Events as full-width-minus-padding cards (16px horizontal margin):
  - 9:00-10:00: "Sprint Planning" card (Green-500 left border 3px, Green-50 fill, 14px 600 title, 12px "Conf Room A" location, 12px "9:00 - 10:00 AM" time)
  - 11:00-12:00: "Product Sync" card (Purple-500 border, Purple-50 fill, video camera icon, "3 attendees")
  - 2:00-3:30: "Design Review" (Blue-500 border, Blue-50 fill, "Figma link")
  - Overlapping events stack vertically with "and 1 more" link
- Tap on event opens detail sheet (bottom sheet, 70vh):
  - Event title (20px 600), color dot
  - Date and time with calendar icon
  - Location with map pin icon
  - "Join video call" button (Blue-600 primary, full width) if video meeting
  - Attendees list with RSVP status
  - Description text
  - "Edit" and "Delete" action buttons
  - Drag handle at top of sheet for dismiss

**Quick Create:** Tapping on an empty time slot shows a minimal inline form with title input, time display, and "Save" / "More options" buttons.

**Bottom Navigation Bar:** Same as F-009 with Calendar tab active.

**States:** Day with events, empty day ("No events today" with illustration), event detail bottom sheet, quick create form, all-day events expandable section at top.

---

#### Prompt F-011: Mobile Chat (390px)

Design the chat conversation view for 390px mobile.

**App Header (56px):**
- Left: Back arrow (returns to channel/DM list)
- Center: Channel "#engineering" or DM contact name with avatar (24px) + online status dot
- Right: Phone icon (start call), video icon (start meeting), info icon (channel details)

**Message Feed (scrollable, remaining height):**
- Messages with compact mobile layout:
  - Each message: Avatar (32px) left-aligned, name (12px 600) + timestamp (11px Slate-400) row, message body below (14px, full width minus 60px left margin)
  - Own messages: no avatar, right-aligned, Blue-50 background pill
  - Code blocks: horizontal scroll, JetBrains Mono 12px, Slate-50 background
  - Images: full width minus 32px padding, 8px radius, tap to full-screen
  - File attachments: card with icon + name + size + download button
  - Link previews: compact card with site favicon + title + description
  - Reactions: small pills below message (emoji + count, 24px height)
  - Thread indicator: "4 replies" link, Blue-600, opens thread view
- Date dividers: centered, Slate-400 text
- Typing indicator: "Sarah is typing..." with animated dots
- "Scroll to bottom" floating button (40px circle, arrow-down, Blue-600) when scrolled up, badge with new message count

**Compose Bar (sticky bottom, above keyboard):**
- Row 1: Attachment icon (paperclip), text input (flex, 40px min height, auto-grow, placeholder "Message #engineering"), Send button (Blue-600 circle, 36px, arrow-up icon) -- send button visible only when text is entered
- Text input supports: @mentions (shows autocomplete overlay list above input), emoji (opens system emoji keyboard), markdown formatting
- Long-press send button: schedule send option
- Attachment button opens sheet: "Photo & Video", "File", "Camera"

**Channel/DM List (back navigation destination):**
- Search bar at top (40px)
- "Channels" section with hash + name + unread badge
- "Direct Messages" section with avatar + name + status + preview text + time
- "+" FAB for new channel/DM (Blue-600, bottom-right)

**States:** Conversation loaded, empty channel, long message with "read more" truncation, image gallery (horizontal swipe in full-screen), voice message recording (hold to record, red waveform UI), thread view (slides in from right), channel details (slides in from right with member list, pinned, files, settings).

---

#### Prompt F-012: Mobile Meet Pre-Join and In-Call (390px)

Design the video meeting experience for 390px mobile.

**Pre-Join Screen (full screen):**
- Top: "Product Sync -- Weekly" meeting title (18px 600 centered), "11:00 AM - 12:00 PM" (14px Slate-500)
- Center: Self-preview video (full width minus 32px padding, 280px height, 16px radius). If camera off: Slate-800 background with your avatar (80px) centered.
- Below preview: Name display "You'll join as Abiola Ogunsakin" (14px)
- Toggle row (centered, 48px height): Microphone toggle (44px circle, white if on, Red-500 if muted), Camera toggle (44px circle, same pattern), Audio output picker (speaker icon with dropdown)
- Bottom: "Join now" button (Blue-600, full width minus 32px, 48px height, 16px radius) + "Back" text link below

**In-Call (full screen):**
- Video area (full screen minus 80px bottom bar):
  - Active speaker full-screen view (primary)
  - Self-view: small picture-in-picture (120x90px, 8px radius, draggable, bottom-right corner, Level-3 shadow)
  - Tap to toggle: gallery view (2x2 grid with 8px gaps) vs speaker view
  - Participant name overlay: bottom-left, 12px white text with dark shadow
  - Mute indicators: small red mic-off icon on muted participants
  - Double-tap: zoom on a participant
  - Swipe left/right: page through participants in gallery view (dots indicator at bottom)
- Bottom Control Bar (80px, safe area padding, Slate-900/90 background, rounded top corners 16px):
  - 5 icon buttons (44px circle each, white icons):
    - Mute/unmute microphone (red background when muted)
    - Camera on/off (red background when off)
    - More (ellipsis): opens bottom sheet with "Share screen", "Raise hand", "Reactions", "Backgrounds", "Settings"
    - Chat (message-circle, opens overlay chat): badge count for unread
    - Leave (Red-600 background, phone-off icon)
- Raised hand indicator: hand emoji floats above participant name for 5s
- AI captions: translucent bar (48px) above controls with speaker name + caption text (14px white), scrolling

**Portrait Presentation View:** Shared screen content fills top 60% of viewport, participant filmstrip (horizontal scroll, 80px thumbnails) in middle, controls at bottom.

**States:** Pre-join, in-call 1:1, in-call 4 participants gallery, in-call presentation sharing, chat overlay (half-screen slide-up panel), participant list (full screen slide-up), poor connection warning banner (amber), reconnecting overlay, meeting ended summary.

---

#### Prompt F-013: Mobile Drive / Files (390px)

Design the file browser for 390px mobile.

**App Header (56px):**
- Left: Hamburger menu
- Center: "My Drive" or current folder name (16px 600)
- Right: Search icon, view toggle (grid/list), more (sort, select multiple)

**Quick Access (horizontal scroll, 120px section):**
- "Recent" header (12px overline) with "See all" link
- Horizontal cards (100x100px, 8px gap): file preview thumbnail or type icon, filename below (11px truncated). 4-5 cards visible.

**File List (default mobile view, scrollable):**
- Section: "Folders" header (12px overline, Slate-400)
  - Folder items (56px height): Folder icon (24px) + folder name (14px) + chevron right, item count (12px Slate-400) right-aligned. Shared folders show tiny avatar stack.
- Section: "Files" header
  - File items (64px height): File type icon (32px) + content area:
    - Row 1: Filename (14px, truncated)
    - Row 2: Size + "  " + Modified date (12px Slate-400)
  - Right: More (kebab) icon button
  - Long-press: enters multi-select mode with checkboxes
- Swipe actions:
  - Swipe right: Share (Blue-500 background)
  - Swipe left: Delete (Red-500 background)

**File Grid (alternate view):**
- 2-column grid (8px gap, 16px horizontal padding)
- Cards (full column width x 160px): preview area (120px) + info (40px with filename and date)

**FAB:** Blue-600 "+" circle, bottom-right. Tap opens bottom sheet: "Upload file", "Upload photo", "Scan document" (camera icon), "New folder", "New document", "New spreadsheet"

**File Detail Sheet (bottom sheet, 80vh):**
- Drag handle at top
- Large preview (or file type icon for non-previewable)
- Filename (18px 600), file type, size
- Action buttons row: Share, Download, Move, Star, Delete
- "Details" section: Owner, Location, Created, Modified
- "Sharing" section: People with access
- "Activity" section: recent activity timeline

**Bottom Navigation Bar:** Same base with Drive/Files tab active (custom icon: hard-drive or folder).

**States:** My Drive with files, empty folder, upload progress (notification-style at top with progress bar), multi-select mode (checkboxes + floating action bar at bottom with Share/Move/Delete), offline available indicator (green check on synced files).

---

#### Prompt F-014: Mobile Contacts (390px)

Design the contacts view for 390px mobile.

**App Header (56px):**
- Left: Hamburger menu
- Center: "Contacts" (16px 600)
- Right: Search icon, "+" add contact icon button

**Search Bar (when search icon tapped, full-width overlay):**
- Full-width input (40px) with back arrow and clear X. Results appear as the user types -- showing matching contacts with avatar + name + email highlighted.

**Contacts List (scrollable, remaining height):**
- "Favorites" section (if any favorites exist):
  - Horizontal avatar strip (64px per avatar, horizontal scroll): avatar (48px) + name below (11px). "Sarah Chen", "David Kim", "Priya Patel"
- Alphabet section headers: Sticky (32px height, Slate-50 background, 14px 600 Slate-600): "A", "B", "C"...
- Contact rows (64px height):
  - Avatar (40px circle, left margin 16px) with online status dot (if from organization directory)
  - Name (14px 600) + job title or email (12px Slate-500) below
  - Right: Phone icon (direct call) or chevron right
- Right edge: Alphabet quick-scroll strip (A-Z, 12px each, Slate-400, tap or drag to jump)
- Pull-to-refresh to sync contacts

**Contact Detail (pushed view, slides in from right):**
- Large avatar (80px centered) with edit icon overlay
- Name (20px 600 centered), title and company (14px Slate-500 centered)
- Action buttons row (centered, 4 icons): Email (envelope), Chat (message-circle), Call (phone), Video (video) -- each 44px circle with label below
- Detail sections:
  - "Phone" section: phone number(s) with type label (Mobile, Work), tap to call
  - "Email" section: email address(es) with type label, tap to compose
  - "Address" section: postal address, tap to open maps
  - "Company" section: company name, department, title
  - "Notes" section: editable text area
- Bottom actions: "Edit Contact", "Share Contact" (vCard), "Block", "Delete" (red)

**Add/Edit Contact (pushed view):**
- Photo placeholder (80px circle with camera icon, tap to select photo)
- Form fields: First name, Last name, Company, Job title, Email (+ add more), Phone (+ add more), Address, Birthday, Notes
- Top right: "Save" button (Blue-600)

**Bottom Navigation:** Accessed via "More" tab in bottom nav, or dedicated tab if contacts is in bottom bar.

**States:** Contact list loaded, empty contacts ("Add contacts to get started" with illustration), contact detail, edit mode, search with results, search with no results.

---

### 3.4 Tablet/Responsive (1024px)

#### Prompt F-015: Tablet Email (1024px)

Design the email interface for 1024px tablet viewport (iPad landscape proportion).

**Layout:** Two-column layout (sidebar is hidden behind hamburger, accessible via swipe from left edge or hamburger tap).

**Left Column (380px, message list):**
- Header (56px): Hamburger menu, "Inbox" title with count "(24)", search icon, compose button (Blue-600 pill "Compose" with pencil icon)
- Filter chips row (horizontal scroll): All, Unread, Starred, Attachments
- Message list identical to mobile but with wider items (more text visible in preview)
- Pull-to-refresh

**Right Column (remaining 640px, message detail):**
- Full message view when a message is selected
- Identical to desktop right column content
- Action bar at bottom: Reply (primary), Reply All, Forward, Archive, Delete, More

**Landscape Split:** 40/60 split with resizable divider (drag handle, 4px gray line)

**Compose:** Opens as a modal (80% width, 90vh height) rather than overlay like mobile or side panel like desktop.

**Sidebar Drawer:** 300px overlay from left with scrim, same content as desktop sidebar.

**States:** Two-column with message selected, two-column with no selection (right column shows "Select a message"), compose modal open, sidebar drawer open.

---

#### Prompt F-016: Tablet Calendar Week View (1024px)

Design the calendar week view at 1024px tablet viewport.

**Layout:** Full-width calendar grid with collapsible sidebar (accessed via hamburger).

**Toolbar (56px):** Navigation arrows, "Today" button, "Feb 23 - Mar 1, 2026" date, view switcher (Day/Week/Month), "+" create event.

**Week Grid:** 7 day columns filling the full width. Hour labels on left axis (40px). Events displayed as cards. Smaller text (12px) compared to desktop (14px) due to narrower columns. Touch-optimized: tap event opens bottom sheet detail rather than popover.

**Event Creation:** Tap and hold on time slot opens a bottom sheet form (not inline popover like desktop). Fields: Title, Date/Time pickers (native), Calendar selector, Location, Video meeting toggle, Description, Attendees with search.

**Mini Month + Calendars:** Hidden in sidebar drawer. Sidebar overlay from left: mini month picker, "My Calendars" checkboxes, "Other Calendars".

**States:** Week view loaded, event bottom sheet open, create event sheet, sidebar drawer open with calendars.

---

### 3.5 Notification Center (All Breakpoints)

#### Prompt F-017: Notification Center

Design a notification center accessible from the bell icon in the top navigation bar.

**Desktop (1440px) -- Dropdown Panel (400px wide, max 80vh, anchored to bell icon):**
- Header (48px): "Notifications" (16px 600), "Mark all as read" link (12px Blue-600), settings gear icon, close X
- Filter tabs: "All" | "Unread" | "Mentions" | "@You"
- Notification list (scrollable):
  - Each notification (72px, full width):
    - Left: App icon (24px, colored by type: email=blue envelope, calendar=purple calendar, chat=green message, docs=orange file, drive=yellow folder, meet=teal video)
    - Content: Title (14px 600, single line), description (12px Slate-500, 2 lines max), timestamp (12px Slate-400, relative "5m ago", "2h ago", "Yesterday")
    - Right: Unread dot (Blue-600, 8px) or action button where relevant
    - Hover: Slate-50 background, "Mark as read" and "Dismiss" icons appear
  - Sample notifications:
    1. [Email] "New email from Amara Okafor" -- "Q1 Revenue Report Review -- Please review the attached..." -- "5m ago" (unread dot)
    2. [Calendar] "Meeting starting soon" -- "Product Sync at 11:00 AM -- Join video call" -- "15m ago" -- "Join" button (Blue-600 small)
    3. [Chat] "Mentioned in #engineering" -- "David Kim: @you Can you review the deployment config?" -- "30m ago" (unread dot)
    4. [Docs] "Comment on Q1 Roadmap" -- "Sarah Chen commented: 'Should we include NLP pipeline?'" -- "1h ago"
    5. [Drive] "Shared with you" -- "Sarah Chen shared 'Brand Assets v3' folder with you" -- "2h ago"
    6. [Meet] "Recording available" -- "Product Sync recording is ready to view" -- "Yesterday"
  - "See all notifications" link at bottom

**Mobile (390px) -- Full-Screen Overlay:**
- Header (56px): Back arrow, "Notifications" title, "Mark all read" button
- Same notification list but full-screen scrollable
- Swipe right to mark as read (Blue-500 check icon background)
- Swipe left to dismiss (Slate-500 X icon background)

**Notification Settings (accessible from gear icon):**
- Per-app toggle table:
  - Rows: Email, Calendar, Chat, Meet, Docs, Drive, Contacts
  - Columns: Push, Email Digest, In-App, Sound
  - Each cell: toggle switch
- "Do Not Disturb" schedule: start time, end time, days of week
- "Priority Only" mode: toggle to only show mentions and direct messages

---

## 4. Make Automation Prompts

### Prompt M-001: New User Workspace Provisioning

**Trigger:** Webhook -- ERP-IAM emits `user.created` CloudEvent when new user is provisioned in the organization.

**Actions:**
1. Parse the CloudEvent payload extracting `user_id`, `email`, `display_name`, `tenant_id`, `role`, `department`.
2. HTTP POST to `{WORKSPACE_API}/v1/email/accounts` to create the user's email mailbox with default quota (15 GB), creating aliases for firstname.lastname and f.lastname formats.
3. HTTP POST to `{WORKSPACE_API}/v1/calendar/accounts` to create default calendar with timezone from user's locale preference, subscribing to "Company Holidays" and department shared calendars.
4. HTTP POST to `{WORKSPACE_API}/v1/drive/accounts` to create the user's personal drive folder structure with default "My Documents", "Shared with Me", and department team drive membership.
5. HTTP POST to `{WORKSPACE_API}/v1/chat/memberships` to auto-join the user to default channels: "#general", "#announcements", and their department channel (e.g., "#engineering" for Engineering department).
6. HTTP POST to `{WORKSPACE_API}/v1/contacts/import` to seed the user's contact directory with organizational directory entries (colleagues in same department, direct reports, manager).
7. Send a welcome email via `{WORKSPACE_API}/v1/email/send` from "workspace@{tenant_domain}" with subject "Welcome to {org_name} Workspace" containing onboarding links, keyboard shortcuts cheat sheet, and mobile app download links.
8. Emit `workspace.user.provisioned` CloudEvent to Redpanda topic `erp.workspace.provisioning` with user_id, timestamp, and list of provisioned services.
9. POST to `{OBSERVABILITY}/v1/events` logging the provisioning duration and success/failure status for SLO tracking.

**Error Handling:** If any step fails, retry 3 times with exponential backoff (5s, 15s, 45s). On final failure, send alert to `#workspace-ops` Slack channel and create a Jira ticket with failed step details. Partial provisioning state is recorded so the flow can be resumed manually.

---

### Prompt M-002: Meeting Recording and AI Summary Pipeline

**Trigger:** Webhook -- `erp.workspace.meet.recording.completed` CloudEvent emitted when a meeting recording finishes processing.

**Actions:**
1. Parse payload: `meeting_id`, `recording_url`, `duration_seconds`, `participant_ids[]`, `tenant_id`, `organizer_id`.
2. HTTP GET to `{DRIVE_API}/v1/recordings/{recording_id}` to fetch the recording metadata and verify storage location in tenant's MinIO bucket.
3. HTTP POST to `{AI_SERVICE}/v1/transcribe` with the recording URL to generate a text transcript. Poll for completion (max 10 minutes, 30s intervals).
4. HTTP POST to `{AI_SERVICE}/v1/summarize` with the transcript to generate: meeting summary (3-5 bullet points), action items (with detected assignees), key decisions, follow-up topics. Apply PII redaction guards from `erp/aidd.guardrails.yaml`.
5. HTTP POST to `{DOCS_API}/v1/docs` to create a new document titled "{meeting_title} - Notes - {date}" in the organizer's Drive under "Meeting Notes" folder, containing the AI-generated summary, action items as a checkbox list, full transcript in a collapsible section, and a link to the recording.
6. HTTP POST to `{CALENDAR_API}/v1/events/{event_id}/attachments` to link the notes document and recording to the original calendar event.
7. For each participant in `participant_ids[]`: HTTP POST to `{EMAIL_API}/v1/email/send` delivering an email with subject "Meeting Notes: {meeting_title}" containing a summary preview and link to the full document.
8. HTTP POST to `{CHAT_API}/v1/channels/{meeting_channel_id}/messages` posting a bot message: "Meeting notes for '{meeting_title}' are ready: [View Notes]({doc_url}) | [Watch Recording]({recording_url})".
9. Emit `workspace.meet.notes.generated` CloudEvent with meeting_id, doc_id, summary word count, action item count for analytics.

**Error Handling:** Transcription timeout triggers a retry with an alternative AI model. Summarization PII detection failure halts pipeline and alerts the compliance channel. Partial delivery (some participants not emailed) is logged and retried individually.

---

### Prompt M-003: Cross-Service Activity Digest

**Trigger:** Scheduled -- Daily at 7:00 AM in user's local timezone (Africa/Lagos default).

**Actions:**
1. HTTP GET to `{WORKSPACE_API}/v1/users?active=true&tenant_id={tenant_id}` to fetch all active users with their timezone and notification preferences (skip users with digest disabled).
2. For each user, in parallel batches of 50:
   a. HTTP GET `{EMAIL_API}/v1/email/inbox?user_id={id}&since=yesterday&unread=true` -- fetch unread email count and top 3 senders.
   b. HTTP GET `{CALENDAR_API}/v1/events?user_id={id}&date=today` -- fetch today's schedule with meeting titles, times, and video links.
   c. HTTP GET `{CHAT_API}/v1/mentions?user_id={id}&since=yesterday&unread=true` -- fetch unread mentions with channel name and snippet.
   d. HTTP GET `{DOCS_API}/v1/activity?user_id={id}&since=yesterday` -- fetch documents edited by collaborators that the user owns or is shared on.
   e. HTTP GET `{DRIVE_API}/v1/shared?user_id={id}&since=yesterday` -- fetch newly shared files.
3. Compile digest HTML email using the workspace email template:
   - "Good morning, {first_name}!" header
   - "Today's Schedule" section: chronological event list with "Join" buttons for video meetings
   - "Unread Emails" section: "{count} unread emails from {top_senders}" with "Open Inbox" button
   - "Mentions & Messages" section: unread chat mentions with channel links
   - "Document Activity" section: "3 documents updated by collaborators" with file names and links
   - "Newly Shared" section: files shared with user yesterday
   - Footer: "Manage digest preferences" link
4. HTTP POST `{EMAIL_API}/v1/email/send` to deliver the digest email from "digest@{tenant_domain}".
5. Emit `workspace.digest.sent` CloudEvent with user_id, item counts per category, and delivery timestamp.

**Error Handling:** Individual user API failures do not block other users. Failed digests are queued for retry at 8:00 AM. Users with > 3 consecutive delivery failures are flagged for notification preference review.

---

### Prompt M-004: Real-Time Collaboration Presence Sync

**Trigger:** Webhook -- `erp.workspace.docs.session.started` CloudEvent when a user opens a document for editing.

**Actions:**
1. Parse payload: `user_id`, `document_id`, `session_id`, `tenant_id`, `cursor_position`.
2. HTTP GET `{DOCS_API}/v1/docs/{document_id}/sessions` to fetch all active editing sessions for this document.
3. For each active collaborator (excluding the triggering user):
   a. HTTP POST `{CHAT_API}/v1/notifications/push` to send a real-time WebSocket notification: `{ type: "collaborator_joined", document_id, user_name, user_avatar, cursor_color }`.
4. HTTP POST `{DOCS_API}/v1/docs/{document_id}/presence` to register the user's presence with assigned cursor color (sequential from palette: Teal, Orange, Purple, Pink, Emerald, Amber).
5. If the document has not been edited in > 7 days and has comments pending resolution, send a nudge notification to the document owner: "Your document '{title}' has {count} unresolved comments and {user_name} just started editing."
6. Emit `workspace.docs.collaboration.active` CloudEvent with document_id, active_session_count, and user_ids for analytics dashboards.

**Error Handling:** Presence registration failure triggers a client-side fallback where the user appears as "Anonymous Collaborator". WebSocket notification failures are logged but do not block the editing session.

---

### Prompt M-005: Storage Quota Warning and Cleanup

**Trigger:** Scheduled -- Every 6 hours.

**Actions:**
1. HTTP GET `{DRIVE_API}/v1/storage/usage?tenant_id={tenant_id}` to fetch per-user storage consumption data.
2. Filter users exceeding 80% quota threshold.
3. For each user at 80-95% usage:
   a. HTTP GET `{DRIVE_API}/v1/files?user_id={id}&sort=size_desc&in_trash=true` to identify large files in trash that can be permanently deleted.
   b. HTTP GET `{DRIVE_API}/v1/files?user_id={id}&sort=last_accessed_asc&limit=10` to identify files not accessed in 90+ days.
   c. Send email notification: "Storage Alert: You're using {percent}% of your {quota} GB. We found {trash_size} in your trash and {stale_count} files you haven't accessed in 90+ days. [Review Storage]({drive_url}/storage)"
4. For users at 95-100% usage:
   a. Send urgent email with subject "Action Required: Storage Almost Full" and a prominent "Free Up Space" CTA button.
   b. Send push notification via `{CHAT_API}/v1/notifications/push`.
   c. HTTP POST `{ADMIN_API}/v1/users/{id}/flags` to set `storage_warning_critical=true` which triggers an in-app banner.
5. For users at 100%:
   a. HTTP POST `{ADMIN_API}/v1/users/{id}/restrictions` to enable read-only mode on Drive uploads (existing files remain accessible).
   b. Send email explaining the restriction with steps to resolve.
   c. Alert IT admin via `{CHAT_API}/v1/channels/#it-admin/messages`.
6. Emit `workspace.storage.quota.check` CloudEvent with tenant-level stats: total users checked, users at warning, users at critical, users restricted.

**Error Handling:** API failures for individual users are logged and retried in the next 6-hour cycle. Admin alerts for critical users are sent regardless of individual notification failures.

---

### Prompt M-006: Calendar Conflict Detection and Smart Scheduling

**Trigger:** Webhook -- `erp.workspace.calendar.event.creating` CloudEvent (pre-commit hook, before event is saved).

**Actions:**
1. Parse payload: `organizer_id`, `attendee_ids[]`, `proposed_start`, `proposed_end`, `title`, `is_recurring`, `tenant_id`.
2. For each attendee in `attendee_ids[]`, HTTP GET `{CALENDAR_API}/v1/availability?user_id={id}&start={proposed_start}&end={proposed_end}` to check for conflicts.
3. If conflicts detected:
   a. HTTP POST `{AI_SERVICE}/v1/scheduling/suggest` with all attendees' free/busy data for the same day and the next 5 business days to generate 3 alternative time slots where all attendees are free.
   b. Return conflict response to the organizer's client: `{ conflicts: [{user, event_title, overlap_minutes}], suggestions: [{start, end, score}] }`.
   c. The client displays: "2 attendees have conflicts. Suggested times: [Tue 2:00 PM] [Wed 10:00 AM] [Thu 3:30 PM]" with one-click reschedule buttons.
4. If no conflicts:
   a. Return clear response: `{ conflicts: [], suggestions: [] }`.
5. HTTP POST `{CALENDAR_API}/v1/rooms/availability?capacity>={attendee_count}&start={proposed_start}&end={proposed_end}` to suggest available meeting rooms if the event is in-person.
6. Emit `workspace.calendar.conflict_check` CloudEvent with organizer_id, attendee_count, conflicts_found (boolean), suggestions_offered count.

**Error Handling:** Availability API timeout for an individual attendee defaults to "available" with a warning. AI suggestion service failure returns only the conflict data without suggestions. Room availability failure is non-blocking.

---

## 5. Prompt Usage Guidelines

### 5.1 How To Use Figma Prompts
1. Open Figma and create a new design file named "ERP-Workspace-{ScreenName}-{Breakpoint}".
2. Set the frame dimensions to the target breakpoint (1440px, 1024px, or 390px width).
3. Copy the relevant prompt text and paste it into Figma Make Mode or provide it to the design team.
4. Design all stated states (default, loading, empty, error, hover, active, disabled) as separate frames within the same page.
5. Apply the design tokens from F-001 consistently across all screens.
6. Use Auto Layout in Figma for all layouts to ensure responsive behavior.
7. Export as separate pages: "Desktop", "Tablet", "Mobile" with artboards per screen and state.

### 5.2 How To Use Make Prompts
1. Import each workflow into Make.com as a new scenario.
2. Configure the webhook trigger URL and register it in the ERP-Workspace event system.
3. Replace `{WORKSPACE_API}`, `{EMAIL_API}`, `{CALENDAR_API}`, etc. with actual service endpoints.
4. Set up error handling modules with the specified retry policies.
5. Test each workflow with sample CloudEvent payloads before activating.
6. Monitor execution logs for the first 48 hours after activation.

### 5.3 Prompt Modification Rules
- All prompts are additive: extend, do not reduce scope.
- Color values must match the design tokens in F-001 exactly.
- Any new component should be added to the component library (F-002) before use.
- Mobile prompts must maintain WCAG 2.1 AA touch target minimums (44x44px).
- Make workflow prompts must include error handling for every HTTP action.

---

## 6. Output Packaging Convention

| Artifact | Format | Naming Convention |
|----------|--------|-------------------|
| Figma design file | .fig | `ERP-Workspace-{Screen}-v{version}.fig` |
| Design tokens | JSON | `workspace-design-tokens-v{version}.json` |
| Component library | .fig | `ERP-Workspace-Components-v{version}.fig` |
| Icon set | SVG sprite | `workspace-icons-v{version}.svg` |
| Prototype | Figma link | `ERP-Workspace-Prototype-{flow}-v{version}` |
| Make scenario export | JSON | `make-workspace-{workflow}-v{version}.json` |
| Handoff spec | PDF + Zeplin/Figma dev mode | `ERP-Workspace-Handoff-{Sprint}.pdf` |

---

## 7. Performance Acceptance Targets

| Metric | Target | Measurement Method |
|--------|--------|--------------------|
| Lighthouse Performance | >= 95 | Chrome DevTools audit on each route |
| Lighthouse Accessibility | >= 95 | Chrome DevTools audit on each route |
| First Contentful Paint (FCP) | < 1.2s | WebPageTest, field data via CrUX |
| Largest Contentful Paint (LCP) | < 2.5s | WebPageTest, field data via CrUX |
| First Input Delay (FID) | < 100ms | Field data via CrUX |
| Cumulative Layout Shift (CLS) | < 0.1 | Field data via CrUX |
| Time to Interactive (TTI) | < 3.5s | Lighthouse audit |
| Initial route JS bundle (gzip) | < 220KB | Webpack bundle analyzer |
| Image optimization | WebP/AVIF, srcset | Automated CI check |
| Email inbox render (1000 items) | < 500ms | Virtual scroll perf test |
| Calendar week render | < 300ms | Paint timing API |
| Chat message append | < 50ms | Performance.mark/measure |
| Document save (autosave) | < 200ms | Network timing API |
| File upload start | < 1s | User-perceived timing |
| Video meeting join | < 3s | End-to-end timing from click to connected |
| API response (read p99) | < 200ms | Backend SLO monitoring |
| API response (write p99) | < 500ms | Backend SLO monitoring |

---

## 8. AIDD Handoff Gate Template

Before any screen is considered ready for development, it must pass the following gate:

| Gate Item | Requirement | Verified By |
|-----------|-------------|-------------|
| Visual fidelity | All states designed (default, loading, empty, error, hover, active, disabled) | Design Lead |
| Responsive coverage | Desktop (1440px), Tablet (1024px), Mobile (390px) artboards present | Design Lead |
| Dark mode | All screens have dark mode variants | Design Lead |
| Accessibility | WCAG 2.1 AA color contrast verified, focus states designed, screen reader annotations added | Accessibility QA |
| Touch targets | All interactive elements >= 44x44px on mobile and tablet | Design Lead |
| Design tokens | All colors, typography, spacing, and elevation use tokens from F-001 | Design System Owner |
| Component reuse | All UI elements reference components from F-002 library | Design System Owner |
| Interaction spec | Animations, transitions, and micro-interactions documented with timing and easing | Design Lead |
| Content | Real data, not lorem ipsum; localization-friendly string lengths tested | Content Strategist |
| Performance annotations | Lazy loading boundaries, skeleton placements, and virtual scroll zones annotated | Frontend Lead |
| Analytics annotations | All tracking events (click, view, error) annotated on the design | Product Manager |
| Handoff artifacts | Figma dev mode enabled, spacing/sizing tokens exported, assets exported as SVG/WebP | Design Lead |
| Prototype | Interactive prototype covering primary user flow linked from design file | Design Lead |
| Approval | Sign-off from Product Manager, Engineering Lead, and Design Lead | All three |

---

## 9. Revision History

| Version | Date | Author | Changes |
|---------|------|--------|---------|
| 1.0 | 2026-02-23 | AIDD System | Initial comprehensive Figma & Make prompts covering all 7 workspace services across desktop, tablet, and mobile breakpoints |

---

*Generated by AIDD System on 2026-02-23*
