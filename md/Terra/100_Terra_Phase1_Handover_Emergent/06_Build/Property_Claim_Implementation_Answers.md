# Property Claim Flow – Implementation Answers for Emergent

**Date:** 21 February 2026  
**Status:** Technical Clarifications – Ready for Build  
**Scope:** Phase 1 Implementation Details  

---

## Direct Answers to Implementation Questions

### 1. QLD DCDB Data Source

**Answer:** Use QLD DCDB boundaries via internal PostGIS backend.

**Details:**
- DCDB data is ingested into our backend (PostGIS database)
- Do NOT make live external API calls to QLD Government
- Expose simple parcel lookup endpoints from your backend
- Query: `GET /api/cadastral/lookup?address=<address>`
- Response: GeoJSON boundary + parcel metadata (id, lot number, area)

**Your backend should:**
- Pre-compute high-confidence address→DCDB matches
- Return confidence score (0–1.0) with each match
- For high confidence (>0.9): auto-suggest with boundary highlight
- For low confidence (<0.9): trigger "couldn't match" fallback

---

### 2. RP Number Lookup

**Answer:** Query against ingested DCDB in your backend.

**Details:**
- Detect RP pattern: `RP` + 6 digits (e.g., RP123456)
- Query: `GET /api/cadastral/rp/<RP_NUMBER>`
- Response:
  ```json
  {
    "rp": "RP123456",
    "lots": [
      {"lot_number": "1", "area_m2": 50000},
      {"lot_number": "2", "area_m2": 75000}
    ]
  }
  ```

**Single Lot Behavior:**
- If only 1 lot → Auto-select and confirm
- Skip lot selection prompt

**Multiple Lots Behavior:**
- If >1 lot → Show selection overlay
- Display: Lot number + area (m²)
- Include mini preview of boundary with highlighted lot
- User selects lot, confirm, proceed

**Example UI:**
```
Found 2 lots in RP123456

☐ Lot 1 — 50,000 m²
☐ Lot 2 — 75,000 m² (selected)

[Confirm Lot 2]
```

---

### 3. Stripe Integration

**Answer:** Implement now with test keys.

**Details:**

**Trial Logic:**
- 30-day free trial (no card required)
- Satellite icon activates immediately on trial start
- At Day 30: Create Stripe subscription
- If user purchases: Subscription active, monitoring continues
- If user doesn't purchase: Monitoring pauses (graceful degradation)

**Payment Failure:**
- On failed payment: Monitoring pauses (do NOT cancel subscription)
- Show user: "Payment failed. Reactivate to continue monitoring."
- Allow retry or cancel

**Pricing:**
- $20/month (Phase 1 pilot pricing)
- Config-driven (use `process.env.TERRA_MONITORING_PRICE_MONTHLY`)
- Can change without code deploy

**Stripe Setup for Emergent:**
- You have test Stripe API keys in environment
- Create Stripe product: `Terra Monitoring – Phase 1` (reusable monthly plan)
- On trial start: Store trial_end_date in database
- Cron job (nightly): Check for expired trials, create Stripe subscriptions
- Webhook listener: Handle payment.succeeded / payment.failed events

**No charge before Day 30. Period.**

---

### 4. Satellite/NDVI Data

**Answer:** Implement with immediate icon activation + async baseline refinement.

**Details:**

**Icon Activation (Immediate):**
- On trial start: Activate satellite icon instantly (green/lit state)
- Users see proof immediately
- No waiting

**NDVI Baseline Processing:**
- Sentinel-2 is primary source (Phase 1)
- If baseline processing takes >5 seconds: Show "Refining data (up to 24 hours)" state
- User experience:
  ```
  ✓ Satellite tracking is on
  
  We're processing your baseline data.
  Check back in 24 hours for full comparison.
  
  [OK]
  ```
- Return to map. Icon stays lit.
- When baseline ready: User sees baseline + delta visualization

**What "Baseline" Means:**
- Snapshot of vegetation health on Day 1 of claim
- Used to calculate change (delta) over next 3 years
- Compared against historical Sentinel-2 archive where available

**Implementation:**
- On claim confirmation: Queue NDVI baseline job (async)
- Return satellite icon immediately (green)
- Job runs in background (target: complete within 24 hours)
- User doesn't wait

---

### 5. User Authentication

**Answer:** Email-first login/signup, with context preservation during flow.

**Details:**

**Scenario: User Not Logged In**
1. User enters address
2. Before proceeding to boundary confirmation → Show sign-in prompt
3. User signs in (email-based)
4. Return to claim flow without losing address/search context
5. Proceed normally from boundary confirmation

**Scenario: User Already Logged In**
1. Skip auth entirely
2. Proceed directly to address entry

**Sign-In UX:**
```
Create or Log In

[Email input field]
[Password input field]

[Sign In]
[New User? Create Account]
```

**Context Preservation:**
- Store entered address in session/state
- After successful auth: Return user to same screen
- Avoid losing user input

**Integration:**
- Use existing auth system (or build simple JWT-based if not present)
- Check `user` context at start of claim flow
- If null: Pause and prompt sign-in
- If authenticated: Proceed

---

### 6. Rebuild ClaimScreen Completely

**Answer:** Build new, clean ClaimScreen rather than refactoring existing.

**Details:**

**Rationale:**
- Existing 5-step flow (Account→Boundary→EPUs→Claim→Trial) doesn't match new spec
- New flow: Address→Match→Confirm→Registration→Trial
- Clean rebuild = fewer bugs, simpler QA, clearer for Founding 50
- Less risk of breaking existing functionality

**Phases:**
1. **Step 1: Address Entry** → Auto-match against DCDB
2. **Step 2: Boundary Confirmation** → "Is this your property?"
3. **Step 3: Registration Mode** → Full-screen, map hidden, collect name/contact
4. **Step 4: Satellite Trial Prompt** → "Start free trial?"
5. **Step 5: Return to Map** → Show boundary + icon state

**Existing ClaimScreen:**
- Keep it if other flows depend on it
- Or remove if this is the only claiming mechanism for Phase 1
- Confirm with your team

---

## Architecture Recommendations

### Backend Endpoints Required

**Cadastral Lookup:**
```
GET /api/cadastral/lookup?address=<address>
  → { confidence, boundary (GeoJSON), parcel_id, lot_number, area }

GET /api/cadastral/rp/<RP_NUMBER>
  → { rp, lots: [{ lot_number, area_m2 }] }
```

**Claim Management:**
```
POST /api/claims
  → { user_id, boundary (GeoJSON), property_name, trial_start_date }

POST /api/claims/{claim_id}/start-trial
  → Creates Stripe subscription at Day 30

GET /api/claims/{claim_id}/status
  → { claim_id, boundary, trial_status, subscription_status }

DELETE /api/claims/{claim_id}
  → Removes claim, de-publishes boundary
```

**Stripe Events:**
```
POST /api/webhooks/stripe
  → Handle payment.succeeded / payment.failed
  → Update claim subscription status
  → Pause/resume monitoring icon
```

### Frontend State Management

Track during claim flow:
- `address` (user input)
- `boundary` (GeoJSON from DCDB match)
- `parcel_id` (DCDB unique identifier)
- `lot_number` (if RP lookup → multiple lots)
- `user_id` (authenticated user)
- `claim_name` (optional property nickname)
- `trial_start_date` (when trial activated)
- `payment_status` (active/paused/expired)

---

## Timeline & Release Gates

**Implementation Order:**
1. Auth integration (enable sign-in mid-flow)
2. DCDB address lookup endpoint + testing
3. RP pattern detection + lot selection
4. ClaimScreen rebuild (address → confirm → registration → trial)
5. Stripe integration (test mode)
6. Satellite icon state management
7. NDVI baseline async job + "Refining data" state
8. Map integration (boundary display + icon legend)
9. End-to-end testing (claim 5 test properties, trigger Day 30 Stripe conversion)

**Do Not Ship Until:**
- [ ] All 6 backend endpoints tested end-to-end
- [ ] Stripe Day 30 → Subscription creation tested in test mode
- [ ] Payment failure → Monitoring pause tested
- [ ] Auth mid-flow tested (user doesn't lose context)
- [ ] Satellite icon activates immediately on trial start
- [ ] NDVI baseline "Refining data" state shows <5 seconds
- [ ] Font audit passed (16px+ minimum)
- [ ] Claim removal tested (boundary removed from public view)
- [ ] No errors in console during full flow

---

## Environment Variables

Provide Emergent with:
```
TERRA_DCDB_BACKEND_URL=<your PostGIS endpoint>
TERRA_STRIPE_PUBLISHABLE_KEY=<test key>
TERRA_STRIPE_SECRET_KEY=<test key>
TERRA_MONITORING_PRICE_MONTHLY=2000  # in cents ($20)
TERRA_SENTINEL2_API_KEY=<if applicable>
TERRA_AUTH_ENDPOINT=<your auth service>
```

---

## Go-No-Go Checklist

Before soft launch to Founding 50:

**Backend:**
- [ ] DCDB lookup returns matches in <1s
- [ ] RP lookup works for single & multiple lots
- [ ] Stripe test subscription created at Day 30
- [ ] Payment failure pauses monitoring (doesn't cancel)
- [ ] Webhooks firing correctly

**Frontend:**
- [ ] Address entry → boundary highlight works
- [ ] RP entry triggers lot selection (if >1 lot)
- [ ] Auth mid-flow preserves address context
- [ ] Registration mode hides map correctly
- [ ] Satellite icon lights up immediately on trial start
- [ ] "Refining data" state appears if needed
- [ ] Return to map shows boundary + icon state
- [ ] Claim removal de-publishes boundary

**UX:**
- [ ] <3 minute claim completion time
- [ ] 16px+ font sizes throughout
- [ ] No dark theme
- [ ] Plain English copywriting (no technical jargon)
- [ ] One primary CTA per screen

---

## Questions?

Do not guess or interpret beyond these answers. If unclear, ask before building.

Commit hash with property claim specification: **13141f1**  
Reference this document simultaneously when implementing.
