# Week 1 Development Checklist – Property Claim Flow

**Week:** 21–28 February 2026  
**Feature:** New Property Claim Flow (Auth → Mocked Cadastral → Claim UI → Stripe → Icon State)  
**Team:** Emergent  
**Goal:** Complete all 10 items by 28 Feb 09:00  

---

## ✅ Item 1: Supabase Auth Integration (Days 1-2)

**Objective:** Users can sign in during claim flow without losing context.

- [ ] **1.1** Verify Supabase env vars present (`SUPABASE_URL`, `SUPABASE_ANON_KEY`)
  - If missing, tell Brad exact var names needed
- [ ] **1.2** Set up Supabase session detection in claim flow
- [ ] **1.3** Create sign-in modal component
  - Email + password fields
  - Simple, light theme, 16px+ fonts
- [ ] **1.4** Implement context preservation on auth redirect
  - Store entered address in React state before sign-in
  - Resume claim from boundary confirmation after auth
- [ ] **1.5** Test sign-in mid-flow
  - User enters address → gets prompted to sign-in → auth succeeds → resumes with address still there
- [ ] **1.6** Test sign-in for existing users
  - User already logged in → skip auth modal → proceed to address entry
- [ ] **1.7** Commit: `feat: Auth – Supabase sign-in + context preservation`

**Success:** User can start claim, get intercepted for auth if needed, resume seamlessly.

---

## ✅ Item 2: DCDB Mock Endpoints (Day 2)

**Objective:** Backend returns mock cadastral data (3 Gondwana properties).

### 2A: `/api/cadastral/by-point` (PRIMARY)

- [ ] **2A.1** Create endpoint that accepts `?lat=<lat>&lng=<lng>`
- [ ] **2A.2** Return mock GeoJSON boundary for 3 test properties
  - Example: Gondwana property 1 @ -27.4705, 152.8449
  - Must include: `parcel_id`, `lot_number`, `area_m2`, `address`, `confidence`, `boundary` (GeoJSON)
- [ ] **2A.3** Implement confidence scoring (0–1.0)
  - High confidence (>0.9) → Auto-select boundary
  - Medium confidence (0.7–0.9) → Show user options
  - Low confidence (<0.7) → Suggest RP fallback
- [ ] **2A.4** Test with map click (user clicks tile → sends lat/lng → gets boundary back)
- [ ] **2A.5** Commit: `feat: DCDB – Mock /api/cadastral/by-point endpoint`

### 2B: `/api/cadastral/by-address` (CONVENIENCE)

- [ ] **2B.1** Create endpoint that accepts `?address=<address>`
- [ ] **2B.2** Return same mock data (address is wrapper around by-point)
- [ ] **2B.3** Handle partial address matches gracefully
- [ ] **2B.4** Test with user address input (e.g., "123 Sample Road, Brookfield")
- [ ] **2B.5** Commit: `feat: DCDB – Mock /api/cadastral/by-address endpoint`

### 2C: `/api/cadastral/by-rp` (QLD FALLBACK)

- [ ] **2C.1** Create endpoint that accepts `?rp=123456`
- [ ] **2C.2** Return mock RP data with 1–2 lots
  - If 1 lot: Auto-select (simple case)
  - If 2+ lots: Return lot options (lot_number, area_m2)
- [ ] **2C.3** Validate RP pattern (RP + 6 digits)
- [ ] **2C.4** Test single-lot and multi-lot RP scenarios
- [ ] **2C.5** Commit: `feat: DCDB – Mock /api/cadastral/by-rp endpoint`

**Success:** All 3 endpoints return consistent mock data.

---

## ✅ Item 3: RP Flow & Lot Selection (Day 2-3)

**Objective:** User can enter RP number and select lot if multiple.

- [ ] **3.1** Detect RP pattern in address field (RP + 6 digits)
- [ ] **3.2** Add "Enter RP Number" button/option (fallback from failed address match)
- [ ] **3.3** Create RP input screen
  - Input field with placeholder "RP123456"
  - Format validation (RP + 6 digits only)
- [ ] **3.4** Call `/api/cadastral/by-rp` on valid RP entry
- [ ] **3.5** Single-lot auto-select
  - If 1 lot: Auto-confirm, show boundary, move to next screen
- [ ] **3.6** Multi-lot selection UI
  - If 2+ lots: Show modal with lot options (lot number + area)
  - User taps lot → boundary highlights
  - User confirms → proceed
- [ ] **3.7** Test single & multi-lot scenarios
- [ ] **3.8** Commit: `feat: DCDB – RP pattern detection + lot selection`

**Success:** User can fall back to RP, see proper lot selection UI.

---

## ✅ Item 4: Claim UI Skeleton – 3-Screen Flow (Days 3-4)

**Objective:** Build the 3-screen UI (Address → Confirm → Trial) without integration yet.

### 4A: Screen 1 – Address Search

- [ ] **4A.1** Create AddressSearch screen component
- [ ] **4A.2** Text input: "Enter your address"
  - Placeholder: "e.g., 123 Sample Road, Brookfield QLD"
  - Light theme, 16px+ font, generous padding
- [ ] **4A.3** Auto-suggest dropdown (mock suggestions)
- [ ] **4A.4** "Can't find your property?" → Option to use RP fallback
- [ ] **4A.5** On address select: Show loading state "Finding your property..."
- [ ] **4A.6** Commit: `feat: ClaimUI – Screen 1 (Address Search)`

### 4B: Screen 2 – Boundary Confirmation (Map Visible)

- [ ] **4B.1** Create BoundaryConfirm screen component
- [ ] **4B.2** Map visible with highlighted parcel boundary
- [ ] **4B.3** Overlay: "Is this your property?"
  - Show property address
  - Show lot number + area
  - Light theme, centered overlay
- [ ] **4B.4** Buttons: [Yes, Confirm] | [No, Try Again]
- [ ] **4B.5** On "Yes": Proceed to registration (full-screen, map hidden)
- [ ] **4B.6** On "No": Return to address search
- [ ] **4B.7** Commit: `feat: ClaimUI – Screen 2 (Boundary Confirmation)`

### 4C: Screen 3 – Full-Screen Registration (Map Hidden)

- [ ] **4C.1** Create Registration screen component
- [ ] **4C.2** Full-screen form, light theme, no map
- [ ] **4C.3** Fields:
  - Property Name (optional)
  - Owner Name (required)
  - Email (required)
  - All 16px+ fonts, simple language
- [ ] **4C.4** Bottom button: [Proceed to Trial Offer]
- [ ] **4C.5** Simple form validation (email format, name required)
- [ ] **4C.6** Commit: `feat: ClaimUI – Screen 3 (Full-Screen Registration)`

### 4D: Screen 4 – Trial Activation (Map Hidden)

- [ ] **4D.1** Create TrialOffer screen component
- [ ] **4D.2** Full-screen, light theme
- [ ] **4D.3** Clear text:
  ```
  Start 30-Day Free Satellite Monitoring?
  
  We'll track vegetation health using satellite data.
  After 30 days: $20/month. Cancel anytime.
  No card required now.
  ```
- [ ] **4D.4** Buttons: [Start Free Trial] | [Skip for Now]
- [ ] **4D.5** On [Start]: Go to mock Stripe subscription, return to map
- [ ] **4D.6** On [Skip]: Mark monitoring as inactive, return to map
- [ ] **4D.7** Commit: `feat: ClaimUI – Screen 4 (Trial Activation)`

**Success:** All 4 screens built, light theme, 16px+, map hidden where needed.

---

## ✅ Item 5: Stripe Integration (Days 3-4)

**Objective:** Trial subscription logic works (no real charges).

- [ ] **5.1** Verify Stripe test key in env (`STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`)
  - If missing, tell Brad exact var names
- [ ] **5.2** Initialize Stripe library (frontend + backend)
- [ ] **5.3** On [Start Free Trial] click:
  - Create claim record in database
  - Set `trial_start_date` = now
  - Set `trial_end_date` = now + 30 days
  - Set `monitoring_state` = "trial_active"
- [ ] **5.4** Implement Day 30 check (mock cron or manual trigger for testing)
  - At Day 30: Set `monitoring_state` = "paused"
  - Test by manually advancing date in development
- [ ] **5.5** Implement Day 25 reminder (hardcoded email for now, can wire SendGrid later)
  - Mock email send (just log to console)
- [ ] **5.6** Implement payment failure handling
  - On Stripe webhook `payment.failed`: Set `monitoring_state` = "paused"
  - Do NOT auto-cancel subscription
- [ ] **5.7** Test full trial flow (Day 0, Day 25, Day 30)
- [ ] **5.8** Commit: `feat: Stripe – 30-day trial subscription logic`

**Success:** Trial runs 30 days, pauses on Day 30, payment failure pauses monitoring.

---

## ✅ Item 6: Satellite Icon State Machine (Day 4)

**Objective:** Icon appears on map, reflects monitoring state only.

### 6A: Monitoring State Machine

- [ ] **6A.1** Define MonitoringState enum:
  ```typescript
  type MonitoringState = 'inactive' | 'trial_active' | 'subscribed' | 'paused'
  ```
- [ ] **6A.2** Store MonitoringState in claim record
- [ ] **6A.3** Transitions:
  - Claim created → `inactive` (user hasn't started trial)
  - User starts trial → `trial_active`
  - Day 30, no payment → `paused`
  - User subscribes → `subscribed`
  - Payment fails → `paused`

### 6B: Icon Rendering

- [ ] **6B.1** Create SatelliteIcon component
- [ ] **6B.2** Icon appearance based on MonitoringState:
  - `inactive` → No icon shown (or greyed out)
  - `trial_active` → 🟢 Green/lit (active)
  - `subscribed` → 🟢 Green/lit (active)
  - `paused` → ⚫ Grey/faded (inactive)
- [ ] **6B.3** Icon appears ONLY when claim exists
- [ ] **6B.4** Icon persists on map after claim complete
- [ ] **6B.5** Icon updates in real-time on state changes
- [ ] **6B.6** Test icon transitions (inactive → trial_active → paused)
- [ ] **6B.7** Commit: `feat: Icons – Satellite state machine (inactive → trial_active → subscribed → paused)`

**Success:** Icon reflects monitoring state accurately, updates in real-time.

---

## ✅ Item 7: State Machine Explicit Implementation (Days 3-4)

**Objective:** Parcel state + Monitoring state clearly defined.

### 7A: Parcel State Machine

- [ ] **7A.1** Define ParcelState enum:
  ```typescript
  type ParcelState = 'unselected' | 'highlighted' | 'confirmed' | 'claimed'
  ```
- [ ] **7A.2** Transitions:
  - User enters address → `highlighted` (boundary shown on map)
  - User confirms "Is this your property?" → `confirmed`
  - User completes registration → `claimed`
- [ ] **7A.3** Store ParcelState in component state (or database if persisting)
- [ ] **7A.4** Verify state transitions at each screen
- [ ] **7A.5** Commit: `feat: State – Parcel state machine (unselected → highlighted → confirmed → claimed)`

### 7B: Documentation

- [ ] **7B.1** Add state machine diagrams to code comments
- [ ] **7B.2** Document state transitions in README
- [ ] **7B.3** Ensure no conflation of Parcel state + Monitoring state
- [ ] **7B.4** Commit: `feat: Docs – State machine documentation`

**Success:** State machines explicit, transitions clear, no confusion.

---

## ✅ Item 8: Integration & End-to-End Flow (Day 4-5)

**Objective:** All 7 items wired together, full flow works.

- [ ] **8.1** Wire Auth → Address Search
  - User not logged in? Show sign-in → resume address entry
- [ ] **8.2** Wire Address Search → DCDB endpoints
  - User enters address → calls `/api/cadastral/by-address` → shows boundary
  - Or fallback to `/api/cadastral/by-rp`
- [ ] **8.3** Wire Boundary Confirm → Registration
  - User confirms property → move to registration screen
- [ ] **8.4** Wire Registration → Trial Offer
  - User completes form → show trial offer
- [ ] **8.5** Wire Trial Offer → Stripe → Icon State
  - User starts trial → Stripe call → Claim created → Icon appears (green)
- [ ] **8.6** Wire return to Map
  - After trial activated: Return to map, show boundary + icon
  - Icon should be lit/green and bound to claim
- [ ] **8.7** Test full E2E flow (5 times minimum)
  - Auth (if not logged in)
  - Address search (mock endpoint)
  - Boundary confirm
  - Registration
  - Trial activation
  - Return to map with icon
- [ ] **8.8** Commit: `feat: Integration – End-to-end claim flow wired`

**Success:** Full flow works without manual data entry or hacks.

---

## ✅ Item 9: Comprehensive Testing (Day 5)

**Objective:** All 10 test cases pass.

### 9A: Auth & Context Tests

- [ ] **9A.1** User not logged in → gets sign-in prompt mid-flow ✓
- [ ] **9A.2** User already logged in → skips auth ✓
- [ ] **9A.3** Address context preserved through auth redirect ✓

### 9B: Cadastral & RP Tests

- [ ] **9B.1** Address search returns mock boundary (by-point) ✓
- [ ] **9B.2** Address search returns options on medium confidence ✓
- [ ] **9B.3** RP single-lot: Auto-selects, shows boundary ✓
- [ ] **9B.4** RP multi-lot: Shows selection modal, user selects lot ✓

### 9C: Claim & Stripe Tests

- [ ] **9C.1** Registration form validation (email required, name required) ✓
- [ ] **9C.2** Trial activation creates claim + sets trial dates ✓
- [ ] **9C.3** Day 30 simulator: Trial expires, monitoring pauses ✓
- [ ] **9C.4** Payment failure: Monitoring pauses (not cancelled) ✓

### 9D: Icon State Tests

- [ ] **9D.1** Icon appears only when monitoring is active (trial_active or subscribed) ✓
- [ ] **9D.2** Icon is green when trial_active ✓
- [ ] **9D.3** Icon is grey when paused ✓
- [ ] **9D.4** Icon reflects real-time state changes ✓

### 9E: UX Tests

- [ ] **9E.1** All screens: Light theme only (no dark) ✓
- [ ] **9E.2** All screens: 16px+ fonts ✓
- [ ] **9E.3** All screens: Plain English, no jargon ✓
- [ ] **9E.4** Map hidden during registration/trial steps ✓
- [ ] **9E.5** Map visible during address/boundary confirm ✓

- [ ] **9E.6** Full claim flow completes in <3 minutes ✓
- [ ] **9E.7** No console errors during full flow ✓
- [ ] **9E.8** Responsive on mobile (test on device or simulator) ✓

- [ ] **9.9** Document test results in pull request
- [ ] **9.10** Commit: `test: Comprehensive – End-to-end test suite (10/10 passing)`

**Success:** All 10 test cases green.

---

## ✅ Item 10: Sign-Off & Release Readiness (Day 5-6)

**Objective:** Feature ready for soft launch to Founding 50.

- [ ] **10.1** Code review checklist:
  - No console errors
  - No hardcoded keys
  - State machines explicit
  - Comments on tricky bits
- [ ] **10.2** PR + commit message clear
  - Link to Week 1 tracking doc
  - Reference spec docs used
- [ ] **10.3** Brad reviews + approves
- [ ] **10.4** Merge to main
- [ ] **10.5** Deploy to staging environment
- [ ] **10.6** Test staging deployment (full flow, real environment)
- [ ] **10.7** Prepare release notes:
  - What works (3-screen flow, mock data, Stripe trial, icon state)
  - What's coming (real DCDB data, NDVI baseline, map integrations)
  - Known limitations (mock data only, Day 30 manual testing)
- [ ] **10.8** Commit: `release: Property Claim Flow – Week 1 complete, ready for Founding 50 soft launch`
- [ ] **10.9** Tag commit: `v1.0.0-claim-flow`
- [ ] **10.10** Update progress tracker: Mark all items complete

**Success:** Feature merged, staging tested, release notes ready, Founding 50 onboarding can begin.

---

## 📊 Checkpoint Dates

| Checkpoint | Date | Owner | Status |
|-----------|------|-------|--------|
| Auth integrated | 22 Feb | Emergent | — |
| DCDB + RP endpoints live | 23 Feb | Emergent | — |
| Claim UI skeleton complete | 24 Feb | Emergent | — |
| Stripe + icon state working | 25 Feb | Emergent | — |
| E2E testing complete | 26 Feb | Emergent | — |
| Release ready for Brad sign-off | 27 Feb | Emergent | — |
| Go / No-Go decision | 28 Feb 09:00 | Brad | — |
| Soft launch to Founding 50 | ~1 Mar | Brad | — |

---

## 🎯 Success = All 10 Items Complete & Tested

By **Monday 28 Feb 09:00**:
- [ ] Item 1: Auth ✓
- [ ] Item 2: DCDB Endpoints ✓
- [ ] Item 3: RP Flow ✓
- [ ] Item 4: Claim UI ✓
- [ ] Item 5: Stripe ✓
- [ ] Item 6: Icon State ✓
- [ ] Item 7: State Machines ✓
- [ ] Item 8: Integration ✓
- [ ] Item 9: Testing ✓
- [ ] Item 10: Release Ready ✓

**All 10 = Go for Founding 50 soft launch. 🚀**

---

## Communication

**Questions:** Reference spec docs first  
**Blockers:** Tell Brad immediately  
**Daily:** Push commits, update progress tracker  
**PR descriptions:** Link to this checklist, mark items as complete  

Let's ship Phase 1. 🚀
