# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**GEA Management System** is a Google Apps Script web application for the Gaborone Employee Association. It manages:
- Member directory and household records
- Facility reservations (tennis courts, Leobo venue)
- Payment tracking and membership activation
- Email notifications and automated tasks
- Board/admin operations

The system is built entirely on **Google Sheets** (4 spreadsheets) as the database, with a **Google Apps Script** backend serving a web portal (Portal.html) for members and an admin interface (Admin.html) for board members.

## Architecture Overview

### Core Components

1. **Config.gs** — Single source of truth for all system constants:
   - Spreadsheet/folder IDs
   - Brand colors, logos, email addresses
   - Business rules (age thresholds, facility limits, membership categories)
   - Payment methods and account details
   - Error messages and audit log action types
   - All configurable values should go here

2. **Code.gs** — Web app entry point and routing:
   - `doGet(e)` — Receives all HTTP requests, routes by `action` parameter
   - ~13 handler functions for different API endpoints (login, dashboard, book reservation, etc.)
   - Routes split into: PUBLIC (login/serve), MEMBER (dashboard/profile/reservations), and BOARD (admin operations)
   - Image serving: `_handleImageProxy()` proxies Drive files as binary images, `_handleImageDiagnostic()` tests logo URLs
   - Returns JSON responses via `ContentService`

3. **AuthService.gs** — Authentication & session management:
   - `login(email, password)` — Verifies credentials, creates session token
   - `validateSession(token)` — Checks if token is valid and not expired (sliding window)
   - Role checking: `isBoard()`, `isMember()`, `requireAuth(token, role)`
   - Session tokens stored in "Sessions" tab with 24-hour expiry

4. **MemberService.gs** — Member data CRUD (only file that touches Member Directory sheet):
   - Member lookups: `getMemberByEmail()`, `getMemberById()`, `getHouseholdMembers()`
   - Access checks: `isActiveMember()`, `canAccessUnaccompanied()`, `isFitnessEligible()`
   - Nightly checks: `checkBirthdays()`, `checkExpiringMemberships()`, `checkExpiringDocuments()`

5. **ReservationService.gs** — Reservations & facility limits:
   - Limit checking: `getTennisHoursThisWeek()`, `checkReservationLimits()`
   - Lifecycle: `createReservation()`, `approveReservation()`, `denyReservation()`, `cancelReservation()`
   - Nightly tasks: `processBumpWindowExpirations()`, `sendGuestListReminders()`, `sendRsoDailySummary()`

6. **EmailService.gs** — All email sending:
   - `sendEmail(templateId, recipient, variables)` — Fetches template, replaces {{PLACEHOLDERS}}, sends HTML email
   - `replacePlaceholders()` — Handles conditional blocks ({{IF_VAR}}...{{END_IF}})
   - `buildHtmlEmail()` — Wraps content in GEA master template with header/footer/branding

7. **NotificationService.gs** — Scheduled trigger entry points:
   - `runNightlyTasks()` — Master function (2:00 AM): membership checks, guest reminders, bump windows, photo reminders
   - `triggerRsoDailySummary()` — 6:00 AM: sends reservation summary to RSO
   - `syncHouseholdPhonesFromPrimary()` — Nightly: syncs Primary member phone to Household fields

8. **Utilities.gs** — General helpers (date, phone, ID, validation, audit logging):
   - Date/business day logic: `calculateBusinessDayDeadline()`, `isBusinessDay()`, `isHoliday()`
   - Phone formatting: `formatPhoneNumber()`, `isValidPhoneNumber()`, `countryCodeToDialCode()`
   - ID generation: `generateId(prefix)` → "PREFIX-YEAR-RANDOMNUMBER"
   - Audit logging: `logAuditEntry()` (failures logged to console, never thrown)

9. **Tests.gs** — Test suite (no test runner, run manually):
   - `runAllTests()` — Runs 10 test functions sequentially
   - Tests config, utilities, holidays, email templates, member lookups, access checks, limits
   - Helper: `_assert(label, condition, actual)` — Logs PASS/FAIL

10. **Portal.html** — Member-facing interface:
    - Login overlay, authenticated content below
    - Pages: Dashboard, Reservations, Profile, Membership Card, Payment
    - Uses sessionStorage for token persistence
    - `apiCall(action, params)` — Generic fetch wrapper to backend
    - **Known issues**: Phone number integration incomplete (initPhoneInputs not called), Guest list not implemented

11. **Admin.html** — Board-facing interface:
    - Similar structure to Portal.html but for board operations
    - Pages: Dashboard (stub), Pending Reservations, Members, Photos, Payments
    - **Known issues**: 80% of UI is structural but non-functional (shows placeholders), PORTAL_API_URL hardcoded instead of injected

### Data Model

The system uses 4 Google Sheets:
1. **GEA Member Directory** — Households, Individuals, Membership Levels
2. **GEA Reservations** — Reservations, Guest Lists, Usage Tracking
3. **GEA Payment Tracking** — Payments
4. **GEA System Backend** — Configuration, Sessions, Audit Log, Holiday Calendar, Email Templates, Pending Approvals

Key schema: Households (1:N) → Individuals, Households (1:N) → Reservations, Households (1:N) → Payments

## Development Commands

### Running Tests

In the **Google Apps Script editor**:
```javascript
runAllTests()  // Runs all 10 tests
testConfig()   // Tests spreadsheet connectivity and constants
testMemberLookup()  // Tests member/household lookups
testReservationLimits()  // Tests weekly tennis and monthly leobo limits
testSendEmail()  // Sends real test email (requires TEST_RECIPIENT config)
```

### Deploying Changes

1. Edit code in the Apps Script editor
2. Save (Ctrl+S / Cmd+S)
3. No build step required — changes take effect immediately for new executions
4. For deployed web app, redeploy after major changes: **Deploy > Deploy New Version**

### Viewing Logs

In the **Google Apps Script editor** → **Executions** tab to view:
- Function call history
- Errors and logs (from `Logger.log()`)
- Execution duration

### Manually Triggering Nightly Tasks

For testing/debugging:
```javascript
runNightlyTasks()  // Runs all nightly checks at once
testAllTriggers()  // Runs all trigger functions manually
testRsoSummary()   // Sends RSO summary for today
```

## Code Patterns & Conventions

### Error Handling

- **No try-catch for expected errors** — Use validation checks instead:
  ```javascript
  if (!member) return errorResponse(ERR_NOT_MEMBER, "NOT_FOUND");
  ```
- **Try-catch only for unexpected failures** (I/O, Google API calls):
  ```javascript
  try {
    var sheet = SpreadsheetApp.openById(MEMBER_DIRECTORY_ID).getSheetByName(TAB_HOUSEHOLDS);
  } catch (err) {
    Logger.log("ERROR: " + err);
    return errorResponse("Could not load members.", "READ_FAILED");
  }
  ```
- **Audit log failures never thrown** — logged to console only:
  ```javascript
  logAuditEntry(email, action, type, id, details);  // Failures logged, never thrown
  ```

### Response Format

All API endpoints return JSON via `successResponse()` or `errorResponse()`:
```javascript
successResponse({ key: value }, "Optional message")
// Returns: { success: true, message: "...", data: {...} }

errorResponse("User-facing message", "ERROR_CODE")
// Returns: { success: false, message: "...", code: "ERROR_CODE" }
```

### Row-to-Object Conversion

All sheet data read as 2D arrays and converted via helper:
```javascript
var row = rowToObject(headers, data[i]);  // Returns { column_name: value, ... }
```

### Phone Number Storage

Three-field model (not formatted string):
- `country_code` — ISO code (BW, US, GB, etc.)
- `phone_number` — Numeric only (71234567)
- `phone_whatsapp` — Boolean

Retrieved via `formatPhoneNumber(countryCode, phone)` → "+267 71234567"

### Audit Logging

Log all user actions (required for compliance):
```javascript
logAuditEntry(auth.session.email, AUDIT_MEMBER_UPDATED, "Individual", memberId,
              "Updated phone numbers: +267 71234567");
```

## Important Configuration Points

### Editing Business Rules

Most business rules are in **Config.gs**. Examples:
- Tennis limits: `TENNIS_WEEKLY_LIMIT_HOURS`, `TENNIS_SESSION_MAX_HOURS`
- Age thresholds: `AGE_UNACCOMPANIED_ACCESS`, `AGE_VOTING`
- Facility approval requirements: `FACILITIES_REQUIRING_APPROVAL`

Changes take effect immediately; no code recompilation needed.

### Adding New API Endpoints

1. Add handler function `_handleNewAction(p)` in Code.gs
2. Add case in `_routeAction()` switch statement
3. Verify auth via `requireAuth(p.token)` if protected
4. Return `successResponse()` or `errorResponse()`

### Adding New Email Templates

1. Add row to "Email Templates" tab in GEA System Backend
2. Use template ID `tpl_NNN`
3. Subject and body support {{VARIABLE}} placeholders
4. Call: `sendEmail("tpl_NNN", recipient, { VAR: value, ... })`

### Updating Spreadsheet IDs

After copying a spreadsheet or moving data:
1. Get new ID from spreadsheet URL
2. Update in **Config.gs** (MEMBER_DIRECTORY_ID, RESERVATIONS_ID, etc.)
3. Save and run `runAllTests()` to verify connectivity

### Image Asset Storage & Updates

**Public assets** (logos, favicon) are stored in **Google Cloud Storage** (`gea-public-assets` bucket):
- All LOGO_* and FAVICON_URL constants in Config.gs point to GCS URLs
- Format: `https://storage.googleapis.com/gea-public-assets/gea-logo-*.png`
- Bucket is publicly readable (no authentication needed)
- To update images: Upload new files to bucket, update filenames in Config.gs if needed

**Member photos & documents** remain in Google Drive:
- Private folders for photos (pending, approved), documents, passports
- Access controlled via Drive folder sharing
- File IDs stored in Individuals sheet (`photo_file_id`, `document_scan_file_id`)

**Image proxy**: If you need to serve Drive files as images in the web app:
- Use `?action=img&id=<DRIVE_FILE_ID>` to fetch the image
- Currently unauthenticated for public logos, but can add `requireAuth()` for private images

## Testing Workflow

1. **Unit tests** — Run individual test functions for isolated checks
2. **Integration tests** — `runAllTests()` checks end-to-end flows with real data
3. **Manual testing** — Use Portal.html / Admin.html locally to test UI
4. **Production data** — Test data exists (HSH-2026-TEST01) but should be deleted before production go-live

Test data is in Households/Individuals sheets with IDs starting with `-TEST`.

## Common Debugging Tasks

### Check if a Member Can Login

```javascript
var member = getMemberByEmail("jane@example.com");
var active = isActiveMember("jane@example.com");  // Returns { isActive, status, message }
```

### Check Reservation Limits

```javascript
var limits = checkReservationLimits("HSH-2026-001", "Tennis Court", new Date(), 1.5);
// Returns { allowed, isExcess, reason, hoursUsed, hoursLimit }
```

### View Session Data

In "Sessions" tab of GEA System Backend: check `active` status and `expires_at` timestamp

### Check Audit Trail

In "Audit Log" tab: filter by `user_email` or `target_id` to see all actions for a member or reservation

## File Size & Performance Notes

- No external dependencies (jQuery, Bootstrap, etc.) — uses vanilla JavaScript and Google APIs
- Sheets API calls are the bottleneck — use caching where possible (`getConfigValue()` caches per execution)
- Email sending via GmailApp (no rate limits for domain accounts)
- Session tokens: simple 64-character hex strings (no JWT complexity)

## Known Issues & TODOs

From Admin.html code comments:
- Dashboard counts are hardcoded instead of fetched from backend
- Reservation/member detail modals show fake data (hardcoded)
- Photo display not yet implemented (no Google Drive URL conversion)
- Member/reservation search is client-side only

From Portal.html code comments:
- Phone number integration incomplete (`initPhoneInputs()` not called)
- No real-time validation feedback on forms
- Guest list submission not implemented
- Photo upload not implemented

## Deployment & Go-Live Checklist

- [ ] Delete test data (HSH-2026-TEST*, IND-2026-TEST*)
- [ ] Update email addresses in Config.gs (board, treasurer, RSO)
- [ ] Update logo/favicon URLs in Config.gs (must be shareable)
- [ ] Set up Google Apps Script time-driven triggers for nightly tasks and RSO summary
- [ ] Configure web app deployment: **Deploy > New Deployment > Web App > Execute as [service account]**
- [ ] Embed deployed URL in Google Sites
- [ ] Test login, reservations, and payment flows end-to-end
- [ ] Verify all email templates are active in "Email Templates" tab
- [ ] Remove `_setupTestPasswords()` function from Code.gs

## Support & Questions

For issues with the system:
- Check **Audit Log** tab for what changed and when
- Review **Google Apps Script Executions** tab for errors
- Run `testConfig()` to verify all spreadsheet IDs are correct
- Contact: treasurer@geabotswana.org
