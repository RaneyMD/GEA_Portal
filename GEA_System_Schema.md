# GEA Management System - Complete Database Schema

**Last Updated:** February 16, 2026  
**System Version:** Core Backend (v1.0)

---

## Overview

The GEA Management System is built on Google Sheets with a Google Apps Script backend. The system manages member directory, reservations, payments, sessions, and administrative functions for the Gaborone Employee Association.

**Core Spreadsheets:**
- GEA Member Directory
- GEA Reservations
- GEA Payment Tracking
- GEA System Backend

---

## 1. MEMBER DIRECTORY

### 1.1 Households Sheet

Represents a household unit (could be individual or family). The primary organizational unit for membership, dues, and reservations.

| Field | Type | Notes |
|-------|------|-------|
| `household_id` | Text | Unique ID (format: HSH-2026-XXXXX) |
| `primary_member_id` | Text | FK to Individuals table |
| `household_name` | Text | Display name (usually family name) |
| `membership_type` | Enum | Full, Affiliate, Associate, Diplomatic, Community, Temporary |
| `membership_category` | Text | Derived from membership_level_id |
| `membership_level_id` | Text | FK to Membership Levels |
| `membership_duration_months` | Number | For Temporary memberships |
| `membership_start_date` | Date | When membership becomes effective |
| `membership_expiration_date` | Date | When membership expires (auto-renewal needed) |
| `dues_amount` | Number (USD) | Annual dues in USD |
| `dues_paid_amount` | Number (USD) | Total paid to date |
| `dues_last_payment_date` | Date | Last payment received date |
| `balance_due` | Number (USD) | Calculated: dues_amount - dues_paid_amount |
| `address_street` | Text | Street address |
| `address_city` | Text | City (usually Gaborone) |
| `address_country` | Text | Country (usually Botswana) |
| `country_code_primary` | Text | Country code for primary phone (e.g., BW, US) |
| `phone_primary` | Text | Primary phone number (no country code) |
| `phone_primary_whatsapp` | Boolean | Whether primary phone has WhatsApp |
| `active` | Boolean | Membership is current/active |
| `application_status` | Enum | Pending, Approved, Denied, Withdrawn |
| `application_date` | Date | When application was submitted |
| `approved_by` | Email | Board member who approved |
| `approved_date` | Date | Date of approval |
| `denial_reason` | Text | If status = Denied |
| `created_date` | Date | Record creation date |
| `last_modified_date` | Date | Last update date |
| `notes` | Text | Internal notes |
| `sponsor_name` | Text | Non-Full member sponsor name |
| `sponsor_email` | Email | Non-Full member sponsor email |
| `sponsor_verified` | Boolean | Sponsor eligibility confirmed |
| `sponsor_verified_date` | Date | Date sponsor was verified |
| `sponsor_verified_by` | Email | Board member who verified |
| `sponsor_notes` | Text | Sponsorship notes |

**Key Relationships:**
- 1:N with Individuals (one household has many individuals)
- 1:1 with Membership Levels (via membership_level_id)
- 1:N with Reservations (household can make many reservations)
- 1:N with Payments (household can make many payments)

---

### 1.2 Individuals Sheet

Represents a single person (adult or child) within a household.

| Field | Type | Notes |
|-------|------|-------|
| `individual_id` | Text | Unique ID (format: IND-2026-XXXXX) |
| `household_id` | Text | FK to Households |
| `first_name` | Text | First name |
| `last_name` | Text | Last name |
| `date_of_birth` | Date | DOB for age calculations |
| `age_category` | Enum | Adult, Youth (16-17), Child (under 16) — Calculated from DOB |
| `relationship_to_primary` | Enum | Primary, Spouse, Child, Other |
| `email` | Email | Personal email address |
| `citizenship_country` | Text | Country of citizenship |
| `us_citizen` | Boolean | Is US citizen or resident alien |
| `country_code_primary` | Text | Country code for primary phone |
| `phone_primary` | Text | Primary phone (no country code) |
| `phone_primary_whatsapp` | Boolean | Primary phone has WhatsApp |
| `country_code_secondary` | Text | Country code for secondary phone |
| `phone_secondary` | Text | Secondary phone (no country code) |
| `phone_secondary_whatsapp` | Boolean | Secondary phone has WhatsApp |
| `document_type` | Enum | Passport, Omang, Other, None |
| `document_number` | Text | Passport or ID number |
| `document_expiration_date` | Date | Document expiration date |
| `document_scan_file_id` | Text | Google Drive file ID for scan |
| `document_verified_by` | Email | Board member who verified |
| `document_verified_date` | Date | Verification date |
| `expiration_warning_sent` | Boolean | Expiration warning email sent |
| `expiration_warning_date` | Date | When warning was sent |
| `can_access_unaccompanied` | Boolean | Access to recreational facilities without adult |
| `voting_eligible` | Boolean | Can vote in association elections |
| `photo_file_id` | Text | Google Drive file ID for photo |
| `photo_status` | Enum | none, submitted, approved, rejected |
| `photo_submitted_date` | Date | When photo was uploaded |
| `photo_approved_by` | Email | Board member who approved photo |
| `photo_approved_date` | Date | Approval date |
| `employment_office` | Text | USG office where employed (e.g., "Embassy Gaborone") |
| `employment_verification_file_id` | Text | File ID for employment verification |
| `emergency_contact_name` | Text | Emergency contact name |
| `country_code_emergency` | Text | Country code for emergency phone |
| `phone_emergency` | Text | Emergency contact phone |
| `phone_emergency_whatsapp` | Boolean | Emergency contact has WhatsApp |
| `emergency_contact_relationship` | Text | Relationship to member |
| `emergency_contact_email` | Email | Emergency contact email |
| `arrival_date` | Date | When person arrived in Botswana |
| `departure_date` | Date | Expected or actual departure from Botswana |
| `active` | Boolean | Record is current/active |
| `created_date` | Date | Record creation date |
| `last_modified_date` | Date | Last update date |
| `staff_rso_cleared` | Boolean | RSO cleared for staff employment |
| `staff_rso_clearance_date` | Date | Date of RSO clearance |
| `fitness_center_eligible` | Boolean | Can use fitness center (based on age) |
| `office_eligible` | Boolean | Can hold board office (age 16+) |
| `photo_rejection_reason` | Text | If photo status = rejected |
| `password_hash` | Text | SHA256 hash of password for login |

**Key Relationships:**
- N:1 with Households (many individuals per household)
- 1:N with Sessions (one person can have multiple active sessions)
- 1:N with Audit Log entries

**Calculated/Derived Fields:**
- `age_category` - Derived from date_of_birth
- `fitness_center_eligible` - Derived from age (age >= configuration.age_fitness_center)
- `office_eligible` - Derived from age (age >= configuration.age_office_eligible)
- `voting_eligible` - Derived from age and membership type

---

### 1.3 Membership Levels Sheet

Reference table defining membership types and associated attributes.

| Field | Type | Notes |
|-------|------|-------|
| `level_id` | Text | Unique ID (e.g., full_indiv, full_family, affiliate_indiv, etc.) |
| `level_name` | Text | Display name |
| `membership_category` | Enum | Full, Affiliate, Associate, Diplomatic, Community, Temporary |
| `household_type` | Enum | Individual or Family |
| `annual_dues_usd` | Number | Annual dues in USD |
| `annual_dues_bwp` | Number | Annual dues in BWP (Pula) |
| `voting_rights` | Boolean | Can vote in elections (Full members only) |
| `office_eligible` | Boolean | Can hold board position (Full members only) |
| `max_duration_months` | Number | For Temporary: max 6 months |
| `eligibility_criteria` | Text | Description of who can join this category |
| `active` | Boolean | This membership level is available |
| `monthly_dues_usd` | Number | For future monthly billing (not currently used) |
| `monthly_dues_bwp` | Number | For future monthly billing (not currently used) |

**Current Membership Levels:**
- `full_indiv` - Full Membership (Individual)
- `full_family` - Full Membership (Family)
- `affiliate_indiv` - Affiliate (Individual)
- `affiliate_family` - Affiliate (Family)
- `associate_indiv` - Associate (Individual)
- `associate_family` - Associate (Family)
- `diplomatic_indiv` - Diplomatic (Individual)
- `diplomatic_family` - Diplomatic (Family)
- `community_indiv` - Community/Guest (Individual)
- `community_family` - Community/Guest (Family)
- `temporary` - Temporary (6 months max)

---

## 2. RESERVATIONS

### 2.1 Reservations Sheet

Main reservations table for facilities (tennis courts, Leobo venue, etc.).

| Field | Type | Notes |
|-------|------|-------|
| `reservation_id` | Text | Unique ID (auto-generated) |
| `household_id` | Text | FK to Households |
| `submitted_by_individual_id` | Text | FK to Individuals (who made the reservation) |
| `submitted_by_email` | Email | Email of person who submitted |
| `submission_timestamp` | DateTime | When reservation was created |
| `facility` | Enum | Tennis Court, Leobo, Entire Facility, etc. |
| `reservation_date` | Date | Date of the reservation |
| `start_time` | Time | Start time (e.g., "09:00") |
| `end_time` | Time | End time (e.g., "11:00") |
| `duration_hours` | Number | Calculated: hours between start and end |
| `event_name` | Text | Name of event/purpose |
| `guest_count` | Number | Number of non-member guests |
| `guest_list_submitted` | Boolean | Guest list has been provided |
| `guest_list_deadline` | Date | Deadline for submitting guest list (4 business days before) |
| `status` | Enum | Tentative, Confirmed, Cancelled, Bumped, Approved by Board |
| `board_approval_required` | Boolean | Requires board review (large events, etc.) |
| `board_approved_by` | Email | Board member who approved |
| `board_approval_timestamp` | DateTime | When board approved |
| `board_denial_reason` | Text | If board denied |
| `rso_notified_timestamp` | DateTime | When RSO was notified (for guest list) |
| `calendar_event_id` | Text | Google Calendar event ID (for integration) |
| `cancelled_by` | Email | Who cancelled |
| `cancellation_timestamp` | DateTime | When cancelled |
| `cancellation_reason` | Text | Reason for cancellation |
| `notes` | Text | Additional notes |
| `is_excess_reservation` | Boolean | Reservation that exceeds limits (pending approval) |
| `bump_window_deadline` | Date | Last date tentative reservation can be bumped |
| `bumped_by_household_id` | Text | Household that bumped this reservation |
| `bumped_date` | Date | Date bumped |

**Key Relationships:**
- N:1 with Households
- N:1 with Individuals (submitted_by)
- 1:N with Guest Lists

**Business Rules:**
- Tennis: Max 3 hours/week per household, 2 hours/session
- Leobo: Max 1 reservation/month per household, max 6 hours/reservation
- Guest lists due 4 business days before event
- Tentative reservations can be bumped within window (1 day for tennis, 5 days for Leobo)

---

### 2.2 Guest Lists Sheet

Guest list submissions for reservations (RSO security requirement).

| Field | Type | Notes |
|-------|------|-------|
| `guest_id` | Text | Unique ID (auto-generated) |
| `reservation_id` | Text | FK to Reservations |
| `guest_category` | Enum | Family Member, Colleague, Friend, Local, Other |
| `guest_name` | Text | Full name of guest |
| `age_category` | Enum | Adult, Youth, Child |
| `vendor` | Boolean | Is this person a vendor/contractor? |
| `vendor_company` | Text | If vendor, their company |
| `submission_timestamp` | DateTime | When guest was added to list |

---

### 2.3 Usage Tracking Sheet

Tracks weekly/monthly usage limits to enforce reservation policies.

| Field | Type | Notes |
|-------|------|-------|
| `household_id` | Text | FK to Households |
| `household_name` | Text | Display name |
| `week_start_date` | Date | Monday of the week |
| `tennis_hours_this_week` | Number | Cumulative tennis hours this week |
| `month_start_date` | Date | First day of month |
| `leobo_reservations_this_month` | Number | Count of Leobo reservations this month |
| `last_calculated` | DateTime | When usage was last recalculated |

**Calculated nightly at midnight GMT+2 to reset weekly/monthly counters.**

---

## 3. PAYMENT TRACKING

### 3.1 Payments Sheet

Records all payment transactions for membership dues.

| Field | Type | Notes |
|-------|------|-------|
| `payment_id` | Text | Unique ID (auto-generated) |
| `household_id` | Text | FK to Households |
| `household_name` | Text | Display name |
| `payment_date` | Date | Date payment was received |
| `payment_method` | Enum | Bank Transfer, Cash, Check, Credit Card |
| `currency` | Enum | USD, BWP |
| `amount` | Number | Amount in specified currency |
| `amount_usd` | Number | Normalized to USD |
| `amount_bwp` | Number | Normalized to BWP |
| `payment_type` | Enum | Dues Payment, Late Fee, Donation |
| `applied_to_period` | Text | Which membership period (e.g., "2026-01") |
| `recorded_by` | Email | Board member who recorded payment |
| `notes` | Text | Payment notes |
| `journal_entry_id` | Text | Link to accounting system (future) |
| `payment_reference` | Text | Bank reference number or check number |
| `payment_confirmation_file_id` | Text | Google Drive file ID for receipt/proof |
| `payment_submitted_date` | Date | When member submitted payment |
| `payment_verified_date` | Date | When board verified receipt |
| `payment_verified_by` | Email | Board member who verified |

**Key Relationships:**
- N:1 with Households

**Auto-Calculations:**
- `balance_due` in Households = dues_amount - dues_paid_amount (updated when payment recorded)
- `active` status in Households = balance_due <= 0 (payment required for active status)

---

## 4. SYSTEM BACKEND

### 4.1 Configuration Sheet

System-wide settings and business rule thresholds.

| Field | Type | Notes |
|-------|------|-------|
| `config_key` | Text | Setting name |
| `config_value` | Text | Setting value |
| `description` | Text | What this setting controls |
| `last_modified` | DateTime | When last changed |

**Current Configuration:**

| Key | Value | Purpose |
|-----|-------|---------|
| `youth_document_required` | FALSE | Require ID for members under 16 |
| `passport_warning_months` | 6 | Months before document expiration to warn |
| `document_types_botswana` | Omang, Passport | Accepted ID types for BW citizens |
| `max_temporary_months` | 6 | Max duration for temporary membership |
| `tennis_weekly_limit` | 3 | Max hours/week per household |
| `tennis_session_max_hours` | 2 | Max hours per single session |
| `tennis_walkin_available` | TRUE | Allow walk-ups when not reserved |
| `tennis_bump_window_days` | 1 | Days before event can bump tentative |
| `leobo_monthly_limit` | 1 | Max reservations/month per household |
| `leobo_max_hours` | 6 | Max hours per Leobo reservation |
| `leobo_bump_window_days` | 5 | Business days before event can bump |
| `guest_list_deadline_business_days` | 4 | Business days to submit guest list |
| `age_unaccompanied_access` | 15 | Min age for unaccompanied facility access |
| `age_fitness_center` | 15 | Min age for fitness center use |
| `age_voting` | 16 | Min age to vote |
| `age_office_eligible` | 16 | Min age for board position |
| `age_document_required` | 16 | Min age requiring document upload |

---

### 4.2 Sessions Sheet

Active user sessions for login management.

| Field | Type | Notes |
|-------|------|-------|
| `session_id` | Text | Unique ID (format: SES-2026-XXXXX) |
| `token` | Text | Secure session token (SHA256 hash) |
| `email` | Email | User's email address |
| `role` | Enum | member, board, admin |
| `created_at` | DateTime | When session started |
| `expires_at` | DateTime | When session expires (48 hours default) |
| `active` | Boolean | Session is still valid |

**Session Management:**
- Sessions expire after 48 hours
- Token is secure hash (SHA256)
- One session per user at a time (new login invalidates previous)
- Manual logout clears session

---

### 4.3 Audit Log Sheet

Complete audit trail of all system changes for compliance and debugging.

| Field | Type | Notes |
|-------|------|-------|
| `log_id` | Text | Unique ID (format: LOG-2026-XXXXX) |
| `timestamp` | DateTime | When action occurred |
| `user_email` | Email | Who performed the action |
| `action_type` | Enum | LOGIN, LOGIN_FAILED, LOGOUT, MEMBER_UPDATED, PASSWORD_SET, RESERVATION_CREATED, PAYMENT_RECORDED, etc. |
| `target_type` | Enum | Individual, Household, Reservation, Payment, etc. |
| `target_id` | Text | ID of the affected record |
| `details` | Text | Detailed description of what changed |
| `ip_address` | Text | IP address of request (for security) |

**Example Log Entries:**
- "LOGIN successful (role: member)"
- "Updated phone_primary → 71825225"
- "Password set by board member"
- "Reservation created for Tennis Court"

---

### 4.4 Holiday Calendar Sheet

US Federal and Botswana public holidays for business day calculations.

| Field | Type | Notes |
|-------|------|-------|
| `holiday_id` | Text | Unique ID (e.g., US-2026-01, BW-2026-02) |
| `holiday_date` | Date | Date of holiday |
| `holiday_name` | Text | Name (e.g., "New Year's Day") |
| `holiday_type` | Enum | US Federal, Botswana Public |
| `holiday_year` | Number | Year |
| `notes` | Text | Observance rules (e.g., "Sat→Fri observed") |
| `active` | Boolean | Holiday is recognized in system |
| `created_by` | Email | Who added this holiday |
| `created_date` | Date | Date added |

**2026 Holidays (Sample):**
- 01/01 - New Year's Day (US)
- 01/02 - New Year's Public Holiday (BW)
- 01/19 - MLK Day (US)
- 04/03 - Good Friday (BW)
- 05/25 - Memorial Day (US)
- 07/01 - Sir Seretse Khama Day (BW)
- 09/07 - Labor Day (US)
- 11/26 - Thanksgiving (US)

**Used for:**
- Calculating 4 business day deadline for guest lists
- Determining when facilities are closed
- Computing membership expiration dates

---

### 4.5 Email Templates Sheet

Templates for all automated system emails.

| Field | Type | Notes |
|-------|------|-------|
| `template_id` | Text | Unique ID (e.g., tpl_001, tpl_025, tpl_032) |
| `template_name` | Text | Display name |
| `subject` | Text | Email subject line (with variables) |
| `body` | Text | Email body (with variables) |
| `active` | Boolean | Template is in use |

**Current Templates:**

| ID | Name | Purpose |
|----|------|---------|
| tpl_001 | Application Received | Confirm membership application submitted |
| (others) | ... | Various notifications, approvals, etc. |

**Template Variables:**
- `{{FIRST_NAME}}` - First name
- `{{FULL_NAME}}` - Full name
- `{{MEMBERSHIP_LEVEL}}` - Level name
- `{{HOUSEHOLD_TYPE}}` - Individual or Family
- `{{FAMILY_MEMBERS_LIST}}` - Formatted list of family
- `{{DURATION_MONTHS}}` - For temporary memberships
- `{{DUES_AMOUNT}}` - Annual or period dues
- `{{SPONSOR_NAME}}` - Non-full member sponsor
- `{{IF_FAMILY}}...{{END_IF}}` - Conditional block for families
- `{{IF_TEMPORARY}}...{{END_IF}}` - Conditional for temporary members
- `{{IF_NON_FULL}}...{{END_IF}}` - Conditional for non-full members

---

### 4.6 Pending Approvals Sheet

Queue of items awaiting board review and action.

| Field | Type | Notes |
|-------|------|-------|
| `approval_id` | Text | Unique ID |
| `approval_type` | Enum | Membership Application, Reservation, Photo, Document Verification, etc. |
| `item_id` | Text | ID of the item needing approval |
| `submitted_by` | Email | Who submitted for approval |
| `submission_date` | Date | When submitted |
| `status` | Enum | Pending, Approved, Denied, Withdrawn |
| `assigned_to` | Email | Board member assigned to review |
| `notes` | Text | Reviewer notes |

---

## 5. DATA RELATIONSHIPS & REFERENTIAL INTEGRITY

### Primary Key-Foreign Key Relationships

```
Households (household_id)
├─ 1:N → Individuals (household_id FK)
├─ N:1 ← Membership Levels (membership_level_id FK)
├─ 1:N → Reservations (household_id FK)
└─ 1:N → Payments (household_id FK)

Individuals (individual_id)
├─ N:1 ← Households (household_id FK)
└─ 1:N → Sessions (email FK)

Reservations (reservation_id)
├─ N:1 ← Households (household_id FK)
├─ N:1 ← Individuals (submitted_by_individual_id FK)
└─ 1:N → Guest Lists (reservation_id FK)

Usage Tracking
└─ N:1 ← Households (household_id FK)

Sessions
└─ N:1 ← Individuals (email FK)

Audit Log
└─ N:1 ← Individuals (user_email FK)
```

---

## 6. CALCULATED/DERIVED FIELDS

| Field | Location | Calculated From | Recalculation |
|-------|----------|-----------------|--------------|
| `balance_due` | Households | dues_amount - dues_paid_amount | On payment recorded |
| `age_category` | Individuals | date_of_birth | Nightly |
| `fitness_center_eligible` | Individuals | DOB vs config.age_fitness_center | Nightly |
| `office_eligible` | Individuals | DOB vs config.age_office_eligible | Nightly |
| `voting_eligible` | Individuals | DOB vs config.age_voting | Nightly |
| `tennis_hours_this_week` | Usage Tracking | Sum of Tennis Court reservations | Nightly (Monday reset) |
| `leobo_reservations_this_month` | Usage Tracking | Count of Leobo reservations | Nightly (1st of month reset) |
| `duration_hours` | Reservations | end_time - start_time | On creation |
| `guest_list_deadline` | Reservations | reservation_date - 4 business days | On creation |
| `bump_window_deadline` | Reservations | Tennis: res_date - 1 day, Leobo: res_date - 5 days | On creation |

---

## 7. DATA VALIDATION RULES

### Individuals
- email: Must be valid email format
- date_of_birth: Cannot be in future
- phone_primary, phone_secondary: Numbers only, 7-12 digits
- document_expiration_date: Cannot be in past if document_type != None

### Households
- dues_amount: Must be > 0 and match membership level
- membership_start_date: Cannot be after membership_expiration_date
- balance_due: Calculated, not editable

### Reservations
- duration_hours: Max 2 (tennis) or 6 (Leobo)
- Tennis: Ensure tennis_hours_this_week + duration_hours <= 3 (config.tennis_weekly_limit)
- Leobo: Ensure leobo_reservations_this_month + 1 <= 1 (config.leobo_monthly_limit)
- start_time < end_time
- reservation_date >= today

### Payments
- amount: Must be > 0
- payment_date: Cannot be in future
- payment_method: Required

---

## 8. NOTES FOR DEVELOPERS

### Data Types Across Google Sheets
- **Text** - Any string content
- **Email** - Text field with email validation
- **Number** - Integer or decimal
- **Date** - YYYY-MM-DD format
- **DateTime** - YYYY-MM-DD HH:MM:SS format
- **Time** - HH:MM format (24-hour)
- **Boolean** - TRUE or FALSE
- **Enum** - Restricted set of values (enforced via data validation)

### Important Field Notes
- All IDs are auto-generated with format PREFIX-YEAR-RANDOMNUMBER
- Dates use ISO 8601 format (YYYY-MM-DD)
- Phone numbers stored separately (country_code + local number) to avoid formatting issues
- Passwords stored as SHA256 hashes (never plain text)
- Session tokens are secure random hashes
- All timestamps are GMT+2 (Botswana time)

### Current Test Data
- Household: HSH-2026-TEST01 (Johnson Family)
- Individual: IND-2026-TEST01 (Jane Johnson), IND-2026-TEST02 (John Johnson)
- Status: Approved, Dues Paid
- **Note:** Delete before production go-live

---

## 9. SPREADSHEET STRUCTURE

Each sheet has:
- **Header row** with column names
- **Data validation** on enum fields (dropdowns)
- **Formatting** for dates and currencies
- **Hidden columns** for sensitive data (passwords, tokens)
- **Protected** from accidental deletion (board approval required)

Access control is managed via Google Drive permissions and Apps Script authentication.

---

**End of Schema Document**
