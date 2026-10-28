# #4471 breakdown · two distinct problems

**Owner:** external analysis
**Status:** filed
**Reviewed:** 2026-10-28
**Signature:** v1.eyJzdWIiOiI2NDQ4IiwiYXVkIjoiY2hhbGxlbmdlOndoYXQtaXQtc2hvd3MiLCJpYXQiOjE3OTMyMjMzNDQsImV4cCI6MTc5MzMwOTc0NH0.kc5Dyg0OzkDAmwPc5_l4XIVbPR4TOgaaH1kWgEfWbwc

---

Ticket #4471 merged 412 reports about dates landing on the wrong day.
The desk marked it as one issue ("parser reads mm/dd in eu locales"), but analysis shows **two
distinct problems** that require separate fixes.

> **Callout:** Two problems found. 379 reports trace to the parser, 33 to the booking-v3 form hint.

## The two problems

### 1. Parser problem (379 reports)

**What happens:** The shared date parser reads all dates month-first (MM/DD), regardless of tenant
locale. Day-first tenants write dates as DD/MM, but the parser flips them.

**Impact:**
- 379 reports (92% of #4471)
- 11 tenants affected
- Channels: app (204), email (150), form (25)
- Both booking-v2 and booking-v3
- First seen 2026-04-01, last seen 2026-10-04

**Example:** Tenant `harbour.dental` (en-GB locale) reports "Our 7 April reminder went out on
4 July" — the parser read `07/04` as July 4th instead of April 7th.

**Root cause:** Parser assumes MM/DD/YYYY format for all input, ignoring tenant locale.

**Fix:** Parser must respect tenant locale and parse dates according to locale date format
(day-first vs month-first).

**Tracked:** [smigielp/megadevs-tickets#1](https://github.com/smigielp/megadevs-tickets/issues/1)

### 2. Form hint problem (33 reports)

**What happens:** The booking-v3 form shows "MM/DD/YYYY" as the date field hint for all tenants.
Day-first locale users follow this hint and enter dates month-first, but the form reads the input
according to the tenant's locale (day-first), causing a flip.

**Impact:**
- 33 reports (8% of #4471)
- 3 tenants affected: alder.physio, copper.studio, pale.harbour
- Channel: form only
- booking-v3 only (booking-v2 shows correct DD/MM/YYYY hint)
- First seen 2026-04-17, last seen 2026-09-22

**Example:** User at en-GB tenant sees hint "MM/DD/YYYY", enters `04/07/2026` (meaning April 7),
but the form interprets it as DD/MM format (7th of April), which then gets flipped by the parser.

**Root cause:** booking-v3 form displays hardcoded "MM/DD/YYYY" hint regardless of tenant locale.

**Fix:** booking-v3 form hint must match tenant locale:
- Day-first locales: show "DD/MM/YYYY"
- Month-first locales: show "MM/DD/YYYY"

**Tracked:** [smigielp/megadevs-tickets#2](https://github.com/smigielp/megadevs-tickets/issues/2)

## Why this matters

Merging these as one ticket (#4471) makes sense for user-facing communication — both cause the
same symptom. But they need **separate fixes in different parts of the codebase**:

1. Parser fix: shared date parsing logic (affects all channels)
2. Form hint fix: booking-v3 form component only

Fixing only the parser would still leave 33 reports unresolved. Fixing both eliminates all 412
reports.

## Analysis method

The support desk's own cause determination logic (`src/lib/kb/causes.ts`) was used to classify
each report. Reports were grouped by:
- Tenant locale (day-first vs month-first)
- Channel (app, email, form)
- Form version (booking-v2 vs booking-v3)
- Date hint shown to user

This revealed that the 58 form-channel reports split into two groups:
- 25 reports from booking-v2 or day-first+booking-v3 → parser problem
- 33 reports from day-first+booking-v3 with MM/DD hint → hint problem

---

*Page rendered from 4471-breakdown.svx with signature v1.eyJzdWIiOiI2NDQ4IiwiYXVkIjoiY2hhbGxlbmdlOndoYXQtaXQtc2hvd3MiLCJpYXQiOjE3OTMyMjMzNDQsImV4cCI6MTc5MzMwOTc0NH0.kc5Dyg0OzkDAmwPc5_l4XIVbPR4TOgaaH1kWgEfWbwc*