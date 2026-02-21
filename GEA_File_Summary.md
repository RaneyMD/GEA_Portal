# GEA System — File-by-File Summary

## GOOGLE APPS SCRIPT FILES (.gs)

---

## 1. Config.gs
**Purpose:** Single source of truth for all system constants, IDs, and settings.

**Spreadsheet IDs**
- MEMBER_DIRECTORY_ID, RESERVATIONS_ID, SYSTEM_BACKEND_ID, PAYMENT_TRACKING_ID

**Tab Names**
- TAB_HOUSEHOLDS, TAB_INDIVIDUALS, TAB_MEMBERSHIP_LEVELS
- TAB_RESERVATIONS, TAB_GUEST_LISTS, TAB_USAGE_TRACKING
- TAB_CONFIGURATION, TAB_EMAIL_TEMPLATES, TAB_AUDIT_LOG, TAB_PENDING_APPROVALS, TAB_HOLIDAY_CALENDAR, TAB_SESSIONS
- TAB_PAYMENTS

**Google Drive Folder IDs**
- FOLDER_PHOTOS_PENDING, FOLDER_PHOTOS_APPROVED, FOLDER_DOCUMENTS, FOLDER_MEMBERSHIP_APPLICATIONS, FOLDER_BRAND_ASSETS, FOLDER_PAYMENT_CONFIRMATIONS, FOLDER_PASSPORT_SCANS

**Brand & Logo URLs** (Google Cloud Storage public bucket: gea-public-assets)
- COLOR_OLD_GLORY_BLUE (#0A3161), COLOR_OLD_GLORY_RED (#B31942), etc.
- LOGO_ROUND_* (80, 120, 160, 200, 240 px sizes) — GCS URLs for email/card/web use
- LOGO_TYPE_LIGHT_* (560, 800, 1120 px) — Light variant for white/light backgrounds
- LOGO_TYPE_DARK_* (560, 800, 1120 px) — Dark variant for dark backgrounds
- FAVICON_URL — 32px favicon for browser tab
- **Format**: https://storage.googleapis.com/gea-public-assets/gea-logo-*.png
- **Note**: Files in gea-public-assets bucket are public (allUsers has Storage Object Viewer role)

**Email Addresses**
- EMAIL_CHAIR, EMAIL_TREASURER, EMAIL_SECRETARY, EMAIL_BOARD, EMAIL_MEMBERS, EMAIL_RSO, EMAIL_MGT, EMAIL_LEGACY

**Association Information**
- ASSOCIATION_NAME, ASSOCIATION_SHORT_NAME, ASSOCIATION_WEBSITE, ASSOCIATION_LOCATION

**Membership Configuration**
- CATEGORY_* (Full, Affiliate, Associate, Diplomatic, Community, Temporary)
- HOUSEHOLD_INDIVIDUAL, HOUSEHOLD_FAMILY
- RELATIONSHIP_* (Primary, Spouse, Child, Staff)
- CATEGORIES_REQUIRING_SPONSOR, MAX_TEMPORARY_MONTHS

**Age Thresholds**
- AGE_UNACCOMPANIED_ACCESS (15), AGE_FITNESS_CENTER (15), AGE_VOTING (16), AGE_OFFICE_ELIGIBLE (16), AGE_DOCUMENT_REQUIRED (16)

**Facility Reservation Rules**
- TENNIS_WEEKLY_LIMIT_HOURS (3), TENNIS_SESSION_MAX_HOURS (2), TENNIS_WALKIN_AVAILABLE (true), TENNIS_BUMP_WINDOW_DAYS (1)
- LEOBO_MONTHLY_LIMIT (1), LEOBO_MAX_HOURS (6), LEOBO_BUMP_WINDOW_DAYS (5)
- GUEST_LIST_DEADLINE_DAYS (3)
- BOARD_APPROVAL_HOURS (24)
- STATUS_* (Pending, Approved, Tentative, Confirmed, Cancelled, Completed, Waitlisted)
- FACILITY_* (Tennis, Leobo, Whole)

**Staff Member Rules**
- STAFF_MAX_CHILDREN, STAFF_GUEST_CHILDREN, STAFF_RESERVATION_ALLOWED, STAFF_LEOBO_ACCESS, STAFF_EMAIL_REQUIRED

**Document Verification**
- DOCS_BOTSWANA_CITIZENS, DOCS_OTHER_CITIZENS, DOCS_FULL_MEMBERS
- DOC_TYPE_* (Passport, Omang, National, None)
- PASSPORT_WARNING_MONTHS (6), YOUTH_DOCUMENT_REQUIRED (false)

**Payment Configuration**
- PAYMENT_SDFCU, PAYMENT_PAYPAL, PAYMENT_ZELLE, PAYMENT_ABSA
- SDFCU_*, PAYPAL_*, ZELLE_*, ABSA_* (bank account details)

**Notification & Scheduling**
- RENEWAL_REMINDER_DAYS_1 (30), RENEWAL_REMINDER_DAYS_2 (7)
- RSO_SUMMARY_HOUR (6), RSO_SUMMARY_TIMEZONE
- BIRTHDAY_CHECK_HOUR (7), PHOTO_REMINDER_DAYS_AFTER (7)
- WAITLIST_HOLD_HOURS (24), SESSION_TIMEOUT_HOURS (24)

**Holiday Calendar**
- HOLIDAY_TYPE_US, HOLIDAY_TYPE_BOTSWANA, HOLIDAY_TYPE_ONEOFF

**Photo Management**
- PHOTO_STATUS_* (none, pending, approved, rejected)
- PHOTO_MAX_SIZE_MB (2), PHOTO_ACCEPTED_TYPES (jpeg, png)
- PHOTO_REJECTION_REASONS (list of standard reasons)

**URLs**
- URL_HOME, URL_MEMBER_PORTAL, URL_ADMIN_PORTAL, URL_RESERVATION, URL_MEMBERSHIP_CARD, URL_PAYMENT, URL_GUEST_LIST, URL_VERIFY

**Audit Log Action Types**
- AUDIT_MEMBER_* (Created, Updated, Deactivated)
- AUDIT_PAYMENT_* (Recorded, Verified, Submitted)
- AUDIT_RESERVATION_* (Created, Approved, Denied, Cancelled, Bumped)
- AUDIT_PHOTO_* (Submitted, Approved, Rejected)
- AUDIT_LOGIN, AUDIT_LOGIN_FAILED, AUDIT_PASSWORD_*, AUDIT_APPLICATION_*, AUDIT_CONFIG_*, AUDIT_HOLIDAY_*

**Error Messages**
- ERR_NOT_MEMBER, ERR_MEMBERSHIP_EXPIRED, ERR_MEMBERSHIP_PENDING, ERR_NOT_AUTHORIZED
- ERR_TENNIS_LIMIT, ERR_LEOBO_LIMIT, ERR_GUEST_LIST_LATE, ERR_CONFLICT
- ERR_INDIVIDUAL_FAMILY, ERR_SPOUSE_EXISTS, ERR_STAFF_EXISTS, ERR_EMERGENCY_CONTACT, ERR_INVALID_DURATION, ERR_YOUTH_UNACCOMPANIED

**Password Security**
- PASSWORD_MIN_LENGTH (12), PASSWORD_HASH_ALGORITHM ("SHA256")

**Phone Number System**
- SUPPORTED_COUNTRIES (BW, US, GB, ZA, AU, IE, TZ)
- COUNTRY_CODE_TO_DIAL_CODE (mapping)
- DIAL_CODE_TO_COUNTRY_CODE (reverse mapping)
- PHONE_LENGTH_CONSTRAINTS (min/max by country)
- PREFERRED_COUNTRIES (BW, US, GB)
- HOUSEHOLD_PHONE_SYNC_ENABLED (true), HOUSEHOLD_PHONE_SYNC_HOUR (2)

**System Metadata**
- SYSTEM_NAME, SYSTEM_VERSION (1.15.0), SYSTEM_BUILD_DATE, SYSTEM_DEVELOPER, SYSTEM_CONTACT

---

## 2. Code.gs
**Purpose:** Web app entry point. Routes all HTTP requests to appropriate handlers.

**Main Entry Point**
- `doGet(e)` — Receives all GET requests from the web app. Routes by `action` parameter.

**Public Routes (no auth required)**
- `_handleLogin(p)` — Processes email + password login. Calls AuthService.login().
- `_handleLogout(p)` — Invalidates session token.
- `serve` — Returns Portal.html (member interface)
- `serve_admin` — Returns Admin.html (board interface)
- `image_diagnostic` — Returns diagnostic page showing all image assets from Config.gs
- `img` (with `id` param) — Image proxy handler. Returns Google Drive file as binary image response.

**Member Routes (valid token required)**
- `_handleDashboard(p)` — Dashboard data: member info, household members, upcoming reservations, membership status
- `_handleProfile(p)` — View/edit profile. Supports method=update for field changes.
- `_handleReservations(p)` — List all member's reservations (past and future)
- `_handleBook(p)` — Create new reservation. Validates facility, date, time, duration. Calls createReservation().
- `_handleCancel(p)` — Cancel a reservation. Verifies ownership.
- `_handleCard(p)` — Digital membership card data for all household members
- `_handlePaymentSubmit(p)` — Submit payment confirmation. Stores payment record, notifies board.
- `_handleUpdatePhoneNumbers(p)` — Update primary/secondary phone numbers with country codes. Processes intl-tel-input data.

**Board Routes (board role required)**
- `_handleAdminPending(p)` — List all pending reservations awaiting board approval
- `_handleAdminApprove(p)` — Approve a pending reservation. Changes status, sends email.
- `_handleAdminDeny(p)` — Deny a reservation with optional reason. Sends email.
- `_handleAdminMembers(p)` — List all member households
- `_handleAdminPhoto(p)` — Approve/reject member photos with reason
- `_handleAdminPayment(p)` — Verify/confirm payments. Routes to _confirmPayment() or _markPaymentNotFound()

**Payment Verification (Board)**
- `_confirmPayment(paymentId, verifiedBy)` — Mark payment verified. Activates household. Sets expiration date.
- `_markPaymentNotFound(paymentId, markedBy)` — Mark payment as not found in account.

**Image Handling**
- `_handleImageDiagnostic(params)` — Serves HTML diagnostic page with live image loading tests. Tests all LOGO_* and FAVICON URLs from Config.gs.
- `_handleImageProxy(p)` — Proxies Google Drive files as binary image responses. Allows embedding Drive images in iframes without hotlink issues. Usage: `?action=img&id=<DRIVE_FILE_ID>`
- `getImageConstants()` — Collects all image URL constants for diagnostic page

**Helper Functions**
- `_routeAction(action, params)` — Routes action strings to handler functions
- `_setupTestPasswords()` — TEMPORARY: Sets initial test passwords. DELETE AFTER INITIAL ACTIVATION.
- `_getMemberUpcomingReservations(householdId)` — Returns reservations >= today (not cancelled)
- `_getMemberAllReservations(householdId)` — Returns all reservations, sorted newest first
- `_safePublicHousehold(hh)` — Returns subset of household fields safe to send to browser
- `_mimeTypeEnumFromContentType_(contentType)` — Maps content-type string to ContentService.MimeType enum

---

## 3. AuthService.gs
**Purpose:** Authentication, session management, and role checking.

**Login & Password**
- `login(email, password)` — Main login function. Verifies email exists, password hash matches, household is active. Creates session.
- `hashPassword(plaintext)` — SHA256 hashing for password storage and verification
- `setPassword(individualId, plainPassword, boardEmail)` — Board-initiated password setting. Sends tpl_032 confirmation email.

**Session Management**
- `validateSession(token)` — Checks if token is valid and not expired. Refreshes expiry on activity (sliding window).
- `logout(token)` — Invalidates a session token immediately.
- `purgeExpiredSessions()` — Deletes all expired sessions from Sessions tab. Run nightly.

**Role Checking**
- `isBoard(token)` — Returns true if token belongs to board member
- `isApprover(token)` — Returns true if token belongs to board or MGT (can approve reservations)
- `isMember(token)` — Returns true if token is any valid member session
- `requireAuth(token, requiredRole)` — Middleware. Returns {ok: true, session: ...} or {ok: false, response: errorJSON}

**Helpers**
- `_createSession(email, role)` — Creates session record in Sessions tab. Returns token.
- `_generateToken()` — Cryptographically random 64-character hex token
- `_sessionExpiry()` — Returns Date 24 hours from now
- `_getRoleForEmail(email)` — Determines role (board/mgt/member) from email address
- `_sendFirstLoginWelcome(member)` — Sends tpl_021 on first portal login
- `_safePublicMember(member)` — Returns subset of member fields safe for browser

---

## 4. MemberService.gs
**Purpose:** All member data CRUD operations. Only file that directly touches Member Directory sheet.

**Member Lookup (Read)**
- `getMemberByEmail(email)` — Finds active individual by email. Primary lookup for auth.
- `getMemberById(individualId)` — Finds individual by ID
- `getHouseholdById(householdId)` — Finds household by ID
- `getHouseholdMembers(householdId)` — Returns all active members of household, sorted (Primary → Spouse → Children → Staff)
- `getHouseholdByMemberEmail(email)` — One-call lookup: email → member → household
- `getMembershipLevel(levelId)` — Finds membership level config (dues, voting rights, etc.)

**Access Checks**
- `isActiveMember(email)` — Comprehensive check: exists, household is active, not expired, not denied. Returns {isActive, status, message}.
- `canAccessUnaccompanied(individualId)` — Age >= AGE_UNACCOMPANIED_ACCESS
- `isFitnessEligible(individualId)` — Age >= AGE_FITNESS_CENTER
- `isVotingEligible(individualId)` — Age >= AGE_VOTING AND Full membership household

**Write Functions**
- `updateMemberField(individualId, fieldName, value, updatedBy)` — Updates single field. Logs change.
- `updateHouseholdField(householdId, fieldName, value, updatedBy)` — Updates household field. Logs change.
- `_updateField(...)` — Internal: shared update logic for any sheet
- `createHouseholdRecord(householdData, createdBy)` — Creates new household. Returns household_id.
- `createMemberRecord(householdId, individualData, createdBy)` — Creates new individual. Calculates age-based fields. Returns individual_id.

**Photo Management**
- `updatePhotoStatus(individualId, status, decidedBy, rejectionReason)` — Approves/rejects photo. Sends appropriate email.

**Nightly Checks (Scheduled Triggers)**
- `checkBirthdays()` — Sends birthday emails (tpl_022, tpl_023, tpl_024 for age milestones)
- `checkExpiringDocuments()` — Sends passport expiration warnings (tpl_015) 6 months before expiry
- `checkExpiringMemberships()` — Sends renewal reminders at 30 days, 7 days, then deactivates expired households

**Helpers**
- `_getPrimaryEmail(householdId)` — Returns email of Primary member
- `_getPrimaryFirstName(householdId)` — Returns first name of Primary member
- `_getPrimaryFullName(householdId)` — Returns full name of Primary member

---

## 5. ReservationService.gs
**Purpose:** Reservation creation, approval, cancellation, limit checking, and RSO operations.

**Usage Checking**
- `getTennisHoursThisWeek(householdId, forDate)` — Returns total tennis hours booked in the week containing forDate
- `getLeoboReservationsThisMonth(householdId, forDate)` — Returns count of leobo/whole-facility reservations this month
- `getLeoboHoursThisMonth(householdId, forDate)` — Returns total leobo hours this month
- `checkReservationLimits(householdId, facility, eventDate, durationHours)` — Comprehensive limit check. Returns {allowed, isExcess, reason, hoursUsed, hoursLimit}.

**Guest List Deadline**
- `getGuestListDeadline(eventDate)` — Calculates 4 business days before event (skips weekends + holidays)
- `isGuestListDeadlineMet(eventDate)` — Returns true if deadline hasn't passed yet

**Reservation Lifecycle**
- `createReservation(params)` — Creates new reservation record. Handles standard vs. excess flow. Sends appropriate emails. Returns {success, reservationId, status, message}.
- `approveReservation(reservationId, approvedBy, notes)` — Approves pending reservation. Sets status to TENTATIVE (excess) or CONFIRMED (standard). Sends tpl_010 email.
- `denyReservation(reservationId, deniedBy, reason)` — Denies reservation. Sends tpl_011 email.
- `cancelReservation(reservationId, cancelledBy, reason)` — Cancels reservation (member or board). Sends tpl_012 email.

**Scheduled Nightly Tasks**
- `processBumpWindowExpirations()` — Promotes TENTATIVE → CONFIRMED once bump window deadline passes
- `sendGuestListReminders()` — Sends tpl_013 reminder to members whose deadline is tomorrow
- `sendRsoDailySummary()` — Sends tpl_014 to RSO with all today's reservations (members, guests, vendors attending)

**Read Helpers**
- `getReservationById(reservationId)` — Finds reservation by ID
- `hasConflict(facility, startTime, endTime)` — Checks if facility is already booked in that time window

**Internal Helpers**
- `_updateReservationField(...)` — Updates single reservation field with logging
- `_sumReservationHours(householdId, facility, fromDate, toDate, statuses)` — Sums reservation hours matching criteria
- `_countReservations(householdId, facilities, fromDate, toDate, statuses)` — Counts matching reservations
- `_sendReservationNotifications(params, row, hh, limitCheck)` — Sends member + board emails for new reservation

---

## 6. EmailService.gs
**Purpose:** All email sending. Templates → placeholders → HTML → send.

**Main Entry Point**
- `sendEmail(templateId, recipient, variables)` — Fetches template, replaces {{PLACEHOLDERS}}, builds HTML, sends via GmailApp. Returns boolean.

**Template Loading**
- `getEmailTemplate(templateId)` — Fetches template from Email Templates tab. Cached per execution. Strips leading quotes from subject/body.

**Placeholder Replacement**
- `replacePlaceholders(text, variables)` — Replaces {{VAR}} with values. Handles {{IF_VAR}}...{{END_IF}} conditional blocks.

**HTML Email Building**
- `buildHtmlEmail(subject, plainText)` — Wraps plain text in GEA master HTML template with header (logo + title), footer (contact info), and GEA branding.
- `plainTextToHtml(plainText)` — Converts plain text body to styled HTML. Auto-detects:
  - ALL CAPS lines → section headers
  - "Key: Value" format → styled key-value rows
  - "- item" format → bullet lists
  - "*** WARNING" lines → warning box
  - Regular text → paragraphs

**Utilities**
- `escapeHtml(text)` — Escapes <>&"' to prevent HTML injection

---

## 7. NotificationService.gs
**Purpose:** Entry points for scheduled triggers and automated tasks.

**Primary Trigger (Nightly at 2:00 AM)**
- `runNightlyTasks()` — Master function. Calls in sequence:
  - checkExpiringMemberships()
  - checkExpiringDocuments()
  - checkBirthdays()
  - sendGuestListReminders()
  - processBumpWindowExpirations()
  - sendPhotoReminders()
  - purgeExpiredSessions()

**RSO Daily Summary (6:00 AM)**
- `triggerRsoDailySummary()` — Wrapper that calls sendRsoDailySummary()

**Holiday Calendar Reminder (Yearly Nov 1)**
- `sendHolidayCalReminder()` — Sends tpl_027 to board reminding to update holidays for next year

**Household Phone Sync (Nightly 2:00 AM)**
- `syncHouseholdPhonesFromPrimary()` — NEW: Nightly sync of Primary individual's phone to Household fields. Copies country_code_primary + phone_primary if Household phone is blank. Maintains audit trail.

**Photo Reminders**
- `sendPhotoReminders()` — Sends tpl_016 to members who activated 7+ days ago but haven't submitted photos

**Manual Test Helpers**
- `testAllTriggers()` — Runs all trigger functions manually (useful for testing)
- `testRsoSummary()` — Manually sends RSO summary for today

---

## 8. Utilities.gs
**Purpose:** General-purpose helpers used throughout the system.

**Business Day Calculations**
- `calculateBusinessDayDeadline(eventDate, daysBack)` — Counts backward N business days from event date (skips weekends + holidays)
- `isBusinessDay(date)` — Returns true if not weekend and not holiday
- `isHoliday(date)` — Checks if date is in Holiday Calendar
- `getHolidays(year)` — Returns all active holidays for a year (cached per execution)

**Phone Number System** (NEW COMPREHENSIVE SUITE)
- `formatPhoneNumber(countryCode, phoneNumber)` — Formats as +[dialcode] [phone] (e.g., +267 71234567)
- `isValidPhoneNumber(countryCode, phoneNumber)` — Validates country exists, phone is numeric, length fits country constraints
- `countryCodeToDialCode(countryCode)` — Converts BW → 267, US → 1, etc.
- `dialCodeToCountryCode(dialCode)` — Reverse: 267 → BW, 1 → US, etc.

**ID Generation**
- `generateId(prefix)` — Generates PREFIX-YEAR-RANDOMNUMBER (e.g., RES-2026-145238491)

**Date/Time Formatting**
- `formatDate(date, forStorage)` — Default: "Saturday, March 15, 2026". forStorage=true: "2026-03-15"
- `formatTime(date)` — Returns "9:00 AM"
- `formatCurrency(amount, currency)` — "$100 USD" or "P1,400 BWP"

**Date Arithmetic**
- `addDays(date, days)` — Returns new Date N days later (negative = earlier)
- `getWeekStart(date)` — Returns Monday of the week containing date (time zeroed)
- `getMonthStart(date)` — Returns 1st of month for date (time zeroed)

**Age & Birthdays**
- `calculateAge(dateOfBirth)` — Returns age in whole years as of today
- `isBirthdayToday(dateOfBirth)` — Returns true if today is their birthday

**Configuration Lookup**
- `getConfigValue(key)` — Reads setting from Configuration tab (cached per execution)

**Audit Logging**
- `logAuditEntry(userEmail, actionType, targetType, targetId, details)` — Appends row to Audit Log. Failures logged only to console, never thrown.

**Validation**
- `isValidEmail(email)` — Basic email format check
- `isValidPhone(phone)` — Checks for 7+ digits
- `sanitizeInput(input)` — Strips leading =+-@ to prevent formula injection

**Web App Response Helpers**
- `successResponse(data, message)` — Returns {success: true, message, data}
- `errorResponse(message, code)` — Returns {success: false, message, code}

**Data Helper**
- `rowToObject(headers, row)` — Converts spreadsheet row array to named object using headers

---

## 9. Tests.gs
**Purpose:** Comprehensive test suite. Run manually from Apps Script editor.

**Test Runner**
- `runAllTests()` — Executes all 10 tests in sequence

**Individual Tests**
- `testConfig()` — Verifies all constants are set, all 4 spreadsheets are reachable
- `testUtilities()` — Tests date formatting, age calc, ID generation, currency formatting
- `testHolidayCalendar()` — Checks holiday lookups and business day logic
- `testBusinessDayCalculator()` — Tests 4-business-day deadline math across weekends
- `testEmailTemplateLoad()` — Verifies all 31 templates are present and active
- `testMemberLookup()` — Tests getMemberByEmail(), getMemberById(), getHouseholdById() (requires TEST_MEMBER_EMAIL config)
- `testAccessChecks()` — Tests age-based eligibility logic
- `testReservationLimits()` — Tests weekly tennis and monthly leobo limits
- `testGuestListDeadline()` — Tests deadline calculation and isGuestListDeadlineMet()
- `testAuditLog()` — Writes test entry and verifies it was written

**Manual Tests (Run separately)**
- `testSendEmail()` — Sends real test email using tpl_001 (requires TEST_RECIPIENT config)
- `testRsoEmailManual()` — Sends real RSO summary email (requires EMAIL_RSO config)

**Helper**
- `_assert(label, condition, actual)` — Logs PASS or FAIL

---

---

## HTML FILES

---

## 10. Portal.html
**Purpose:** Member-facing interface. Login, dashboard, reservations, profile, card, payments.

**Overall Structure**
- Single-page app (SPA) with login screen overlay and authenticated content below
- Uses sessionStorage for token persistence
- API calls via `apiCall()` function to backend Code.gs
- Page navigation via `showPage()` function

**Authentication Section**
- `submitLogin(event)` — Form submission. Sends email + password to `_handleLogin`. Stores token in sessionStorage.
- `logout()` — Clears token, hides app, shows login screen
- `showAuthenticatedUI()` — Makes header and content visible
- Global: `currentToken`, `currentUser`

**API Communication**
- `apiCall(action, params)` — Generic fetch wrapper. Adds action + token to params. Returns promise that resolves to JSON response.

**Page Navigation**
- `showPage(pageName)` — Switches between dashboard/reservations/profile/card/payment pages. Loads fresh data.
- Pages toggle active/inactive CSS classes

**Dashboard Page**
- `loadDashboard()` — Calls `dashboard` API. Displays:
  - Membership status (household name, status badge, expiration)
  - Household members table (name, relationship, photo status)
  - Upcoming reservations (date, facility, status)

**Reservations Page**
- `loadReservations()` — Calls `reservations` API. Shows all member's reservations (past + future)
- `submitReservation(event)` — Form submission. Validates facility/date/time/duration. Calls `book` API. Shows confirmation.
- `cancelReservation(resId)` — Confirmation dialog. Calls `cancel` API.
- Event listener: Shows/hides guest count field based on "I will have guests" checkbox

**Profile Page**
- `loadProfile()` — Calls `profile` API. Shows:
  - Personal info (name, email, phone)
  - Emergency contact
  - **TODO:** Initialize intl-tel-input here
- **Phone Number Section (INCOMPLETE):**
  - `initPhoneInputs()` — Initializes intl-tel-input on primary + secondary phone fields (NOT CALLED YET)
  - `updatePhoneFields(fieldType)` — Extracts country code + phone from intl-tel-input, stores in hidden fields
  - `savePhoneNumbers()` — Calls `updatePhoneNumbers` API with country_code + phone_number + whatsapp flags
  - `showPhoneMessage(message, type)` — Displays success/error feedback
  - **Issue:** initPhoneInputs() must be called at end of loadProfile() but currently isn't

**Card Page (Digital Membership Card)**
- `loadCard()` — Calls `card` API. Displays card for each household member with status, type, expiration, relationship

**Payment Page**
- `submitPayment(event)` — Form submission. Collects payment method, reference, date, amount, notes. Calls `payment` API.

**Utilities**
- Global constants: PORTAL_API_URL (injected by Apps Script), currentToken (from sessionStorage), currentUser

**Key Issues**
- Phone number integration incomplete (initPhoneInputs not called)
- No real-time validation feedback on reservation form
- Payment confirmation lacks visual polish
- Guest list submission not implemented
- Photo upload not implemented

---

## 11. Admin.html
**Purpose:** Board member interface. Dashboard, pending reservations, members, photos, payments.

**Overall Structure**
- Similar to Portal.html: login overlay + authenticated content
- Uses sessionStorage for token persistence
- API calls via `apiCall()` to backend Code.gs
- All page UI is structured but many load functions are **stubs**

**Authentication Section**
- `submitLogin(event)` — Form submission with email + password
- **ISSUE:** Login form only has email field, missing password input
- `logout()` — Clears token, shows login screen
- `showAuthenticatedUI()` — Shows header + sidebar + content

**API Communication**
- `apiCall(action, params)` — Generic fetch wrapper (same as Portal.html)
- **ISSUE:** PORTAL_API_URL is hardcoded instead of injected via Apps Script

**Sidebar Navigation**
- `showPage(pageName)` — Switches between dashboard/reservations/members/photos/payments

**Dashboard Page** ⚠️ **PLACEHOLDER STUBS**
- `loadDashboard()` — Calls `admin_pending` API but:
  - Dashboard counts are **hardcoded** (`dashPendingCount = 3`, etc.)
  - Photo and payment counts not fetched from backend
  - Upcoming reservations list shows placeholder text "Loading..."
  - Recent actions section is empty placeholder
- **TODO:** Connect to real backend APIs for counts

**Pending Reservations Page** ⚠️ **PARTIAL**
- `loadReservations()` — Calls `admin_pending` API and populates table
  - Reservation list works ✅
  - Search/filter works ✅
- `openReservationDetail(resId)` — Opens modal but shows **hardcoded data**
  - "John Smith", "Tennis Courts", "February 14", etc. are fake
  - **TODO:** Fetch real reservation data from backend
- `approveReservation()` — Calls `admin_approve` API. Works ✅
- `denyReservation()` — Calls `admin_deny` API. Works ✅
- `setupReservationSearch()` — Client-side filter by text

**Members List Page** ⚠️ **PARTIAL**
- `loadMembers()` — Calls `admin_members` API and populates table
  - List display works ✅
  - Search/filter works ✅
- `openMemberDetail(householdId)` — Opens modal but shows **hardcoded data**
  - "Smith Household", household info, individual members all fake
  - **TODO:** Fetch real household + members data from backend
- `setupMemberSearch()` — Client-side filter by text

**Photo Review Page** ⚠️ **PARTIAL**
- `loadPhotos()` — Calls `admin_photo` API with action='list'
  - Grid structure present ✅
  - Uses 📸 emoji placeholder instead of actual photos
  - **TODO:** Convert photo_file_id to Google Drive URL
  - **TODO:** Display actual photos from Drive
- `approvePhoto(individualId)` — Calls `admin_photo` API with action='approve'. Works ✅
- `openPhotoRejectModal(individualId)` — Opens reason selection modal
- `confirmRejectPhoto()` — Calls `admin_photo` API with action='reject'. Works ✅

**Payment Verification Page** ✅ **COMPLETE**
- `loadPayments()` — Calls `admin_payment` API and populates table
  - Payment list display works ✅
  - Search/filter works ✅
- `confirmPayment(paymentId)` — Calls `admin_payment` API with action='confirm'. Marks verified. ✅
- `markPaymentNotFound(paymentId)` — Calls `admin_payment` API with action='not_found'. ✅
- `setupPaymentSearch()` — Client-side filter by text

**Modal Management**
- `closeModal(modalId)` — Removes 'show' class from modal overlay
- Global click handler: Closes modals when clicking outside

**Key Issues**
- 80% of page UI is **structural but non-functional** (just showing placeholders)
- No password field in login form
- PORTAL_API_URL hardcoded instead of injected
- Photo file display not implemented
- Member/reservation detail modals show fake data

---

## SUMMARY TABLE: File Purposes & Key Functions

| File | Purpose | Key Functions |
|------|---------|----------------|
| **Config.gs** | All constants & settings | 200+ constants (IDs, colors, limits, messages) |
| **Code.gs** | Web app routing & entry point | doGet(), image proxy, diagnostic page, 13 route handlers (_handleLogin, _handleBook, _handleImageProxy, etc.) |
| **AuthService.gs** | Login, sessions, roles | login(), validateSession(), requireAuth(), _createSession() |
| **MemberService.gs** | Member data CRUD | getMemberByEmail(), getHouseholdMembers(), createMemberRecord(), checkBirthdays(), checkExpiringMemberships() |
| **ReservationService.gs** | Reservations & limits | createReservation(), approveReservation(), getTennisHoursThisWeek(), checkReservationLimits(), sendRsoDailySummary() |
| **EmailService.gs** | Email sending | sendEmail(), getEmailTemplate(), replacePlaceholders(), buildHtmlEmail() |
| **NotificationService.gs** | Scheduled triggers | runNightlyTasks(), triggerRsoDailySummary(), syncHouseholdPhonesFromPrimary() |
| **Utilities.gs** | General helpers | calculateBusinessDayDeadline(), formatPhoneNumber(), generateId(), formatDate(), logAuditEntry(), sanitizeInput() |
| **Tests.gs** | Test suite | testConfig(), testMemberLookup(), testReservationLimits(), runAllTests() |
| **Portal.html** | Member interface | Login, dashboard, reservations, profile (phone incomplete), card, payments |
| **Admin.html** | Board interface | Login, dashboard (stub), reservations (partial), members (partial), photos (partial), payments (complete) |

