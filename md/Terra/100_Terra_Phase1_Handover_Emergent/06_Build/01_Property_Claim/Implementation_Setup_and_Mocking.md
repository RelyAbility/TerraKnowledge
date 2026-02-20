# Property Claim Flow – Implementation Setup & Mocking Strategy

**Date:** 21 February 2026  
**Status:** Ready for Development  
**Scope:** Phase 1 Claim Flow with Mocked Cadastral Data  

---

## Simple UX (2–3 Screens)

Keeping it simple for 60+ landholders. Three discrete screens:

**Screen 1: Address Search → Boundary Highlight → Confirmation**
- Text input: "Enter your address"
- Auto-suggest from mock DCDB
- On selection: Map shows boundary, highlight parcel
- Overlay: "Is this your property?" [Yes | No]

**Screen 2: Full-Screen Claim Registration** (Map Hidden)
- Collect: Property name, owner name, contact email
- Light theme, 16px+ fonts, minimal fields
- Bottom: [Proceed] button

**Screen 3: Trial Activation + Stripe** (Same Full-Screen)
- Clear text: "Start 30-Day Free Satellite Monitoring?"
- "After 30 days: $20/month. Cancel anytime. No card required now."
- Buttons: [Start Free Trial] | [Register Without Monitoring]
- On [Start Free Trial]: Create claim + activate trial
- Return to map with boundary visible + satellite icon lit

**Flow Duration:** <3 minutes for 60+ users

---

## Cadastral Endpoints (Mocked for Now)

Three lookup methods. All return mock data initially. Real data wires in later when DCDB is ingested into Supabase PostGIS.

### 1. By-Point (PRIMARY)

```
GET /api/cadastral/by-point?lat=-27.4705&lng=152.8449
```

**Mock Response:**
```json
{
  "success": true,
  "matches": [
    {
      "parcel_id": "dcdb_001",
      "lot_number": "1",
      "area_m2": 52500,
      "address": "123 Sample Road, Brookfield QLD 4069",
      "confidence": 0.98,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[-27.4705, 152.8449], ...]]
      }
    }
  ]
}
```

**Why Primary:** Most reliable. User taps/clicks on map → you send pinned lat/lng → instant match.

---

### 2. By-Address (CONVENIENCE)

```
GET /api/cadastral/by-address?address=123%20Sample%20Road,%20Brookfield
```

**Mock Response:**
```json
{
  "success": true,
  "matches": [
    {
      "parcel_id": "dcdb_001",
      "lot_number": "1",
      "area_m2": 52500,
      "address": "123 Sample Road, Brookfield QLD 4069",
      "confidence": 0.85,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[-27.4705, 152.8449], ...]]
      }
    },
    {
      "parcel_id": "dcdb_002",
      "lot_number": "2",
      "area_m2": 75000,
      "address": "123 Sample Road, Brookfield QLD 4069 (Lot 2)",
      "confidence": 0.72,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[-27.471, 152.845], ...]]
      }
    }
  ]
}
```

**Behavior:**
- If confidence > 0.9: Auto-select first match, show boundary
- If confidence 0.7–0.9: Show user 2–3 options, let them choose
- If confidence < 0.7: Suggest RP fallback

---

### 3. By-RP (QLD-ONLY FALLBACK)

```
GET /api/cadastral/by-rp?rp=123456
```

**Mock Response:**
```json
{
  "success": true,
  "rp": "RP123456",
  "lots": [
    {
      "lot_number": "1",
      "area_m2": 50000,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[-27.4705, 152.8449], ...]]
      }
    },
    {
      "lot_number": "2",
      "area_m2": 75000,
      "boundary": {
        "type": "Polygon",
        "coordinates": [[[-27.471, 152.845], ...]]
      }
    }
  ]
}
```

**Behavior:**
- User enters `RP123456`
- If 1 lot: Auto-select, show boundary, confirm
- If >1 lots: Show selection modal (lot + area), user picks, show boundary, confirm

---

## Mocking Strategy

**For Phase 1 Development (Now):**

1. Create `mocks/cadastral.ts` with sample GeoJSON boundaries (3–5 pilot zone properties)
2. All endpoints return mocks
3. Your claim flow works end-to-end with fake data
4. When real DCDB is ingested: Swap mock responses for Supabase queries

**Example Mock Data:**
```typescript
// mocks/cadastral.ts
export const mockDCDB = {
  'by-point': {
    '-27.4705': {
      '152.8449': {
        parcel_id: 'dcdb_001',
        lot_number: '1',
        area_m2: 52500,
        address: '123 Sample Road, Brookfield QLD 4069',
        confidence: 0.98,
        boundary: { /* GeoJSON */ }
      }
    }
  },
  'by-rp': {
    'RP123456': {
      lots: [
        { lot_number: '1', area_m2: 50000, boundary: {} },
        { lot_number: '2', area_m2: 75000, boundary: {} }
      ]
    }
  }
};
```

**Later (When DCDB Ingested):**
```typescript
// Replace mock with real query:
const boundary = await supabase
  .from('dcdb_cadastre')
  .select('*')
  .filter('lot_number', 'eq', data.lot_number)
  .single();
```

No refactoring needed. Same endpoint. Different response source.

---

## Claims Endpoints

### POST /api/claims

**Create a new claim.**

```json
{
  "user_id": "auth_user_123",
  "parcel_id": "dcdb_001",
  "lot_number": "1",
  "property_name": "My Property",
  "boundary": { /* GeoJSON */ },
  "trial_start_date": "2026-02-21T00:00:00Z"
}
```

**Response:**
```json
{
  "claim_id": "claim_abc123",
  "user_id": "auth_user_123",
  "boundary": { /* GeoJSON */ },
  "trial_start_date": "2026-02-21T00:00:00Z",
  "trial_end_date": "2026-03-23T00:00:00Z",
  "subscription_status": "trial",
  "satellite_icon_state": "active"
}
```

---

### POST /api/claims/{claim_id}/start-trial

**Activate satellite monitoring trial.**

```json
{
  "claim_id": "claim_abc123"
}
```

**Response:**
```json
{
  "claim_id": "claim_abc123",
  "trial_active": true,
  "trial_end_date": "2026-03-23T00:00:00Z",
  "satellite_icon_state": "active",
  "ndvi_status": "refining_data"
}
```

**Behavior:**
- Sets `trial_start_date` = now
- Sets `trial_end_date` = now + 30 days
- `subscription_status` = "trial"
- `satellite_icon_state` = "active" (immediate)
- `ndvi_status` = "refining_data" (async job queued)
- Return to map, icon shown as green/lit

---

### GET /api/claims/{claim_id}/status

**Check claim & subscription status.**

**Response:**
```json
{
  "claim_id": "claim_abc123",
  "property_name": "My Property",
  "boundary": { /* GeoJSON */ },
  "trial_active": true,
  "trial_end_date": "2026-03-23T00:00:00Z",
  "days_remaining": 30,
  "subscription_status": "trial",
  "satellite_icon_state": "active",
  "ndvi_status": "ready",
  "ndvi_baseline": { /* Sentinel-2 baseline data */ }
}
```

**Icon State Logic:**
- `subscription_status == "trial"` → Icon green/lit
- `subscription_status == "active"` → Icon green/lit
- `subscription_status == "paused"` or `expired` → Icon grey/faded
- `subscription_status == "none"` → No icon shown

---

### DELETE /api/claims/{claim_id}

**Remove claim (user cancels).**

- Delete from database
- Boundary removed from public view
- Can still query historical data internally
- Return success status

---

## Stripe Integration

**Keep it simple. No auto-subscriptions.**

### Day 0–29: Trial Phase

- User starts trial
- No Stripe interaction
- `subscription_status = "trial"`
- Satellite icon active

### Day 25: Send Reminder

- Send email: "Your free trial ends in 5 days. Add payment to continue."
- Button link to payment page (not auto-charged)

### Day 30: Trial Expires

**No auto-subscription.**

Instead:
- At midnight (Day 30): Set `subscription_status = "paused"`
- Satellite icon turns grey/faded
- App shows: "Trial ended. Reactivate to continue monitoring."
- User taps: [Add Payment] → Stripe checkout → Subscribe explicitly → Monitoring resumes

### Payment Failure

- Stripe webhook receives `payment.failed` event
- Update `subscription_status = "paused"`
- Show user: "Payment failed. [Retry] or [Cancel]"
- Do NOT auto-cancel subscription

### Stripe Config

```
STRIPE_PUBLISHABLE_KEY=<test_pk_live_xxx>
STRIPE_SECRET_KEY=<test_sk_live_xxx>
TERRA_MONITORING_PRICE_MONTHLY=2000  # $20 in cents
TERRA_TRIAL_DAYS=30
```

---

## Auth Integration (Supabase)

**Assume user auth already exists.** Do not build custom JWT.

### Sign-In Mid-Flow

**Scenario:** User starts claim → address search → realizes they're not signed in

**Flow:**
1. Detect `!user` at claim start
2. Show sign-in modal (email + password)
3. On successful auth: Close modal, **preserve entered address in state**
4. Resume claim flow from boundary confirmation
5. User doesn't re-enter address

**Implementation:**
```typescript
// In PropertyClaimFlow.tsx
const [user] = useAuth(); // Supabase session
const [searchedAddress, setSearchedAddress] = useState('');

useEffect(() => {
  if (!user && searchedAddress) {
    // Show sign-in modal, preserve searchedAddress in state
    showSignInModal();
  }
}, [user, searchedAddress]);

const handleAddressSearch = (address) => {
  if (!user) {
    setSearchedAddress(address); // Store it
    showSignInModal(); // Block claim
    return;
  }
  // Continue with cadastral lookup
  lookupCadastral(address);
};
```

---

## Satellite Icon State Machine

**Simple state management:**

```typescript
type SubscriptionStatus = 
  | "trial"        // Free 30-day trial active
  | "active"       // User subscribed, payment active
  | "paused"       // Trial expired or payment failed
  | "none";        // User declined trial at start

type IconState = 
  | "lit"          // Green, active, monitoring on
  | "faded";       // Grey, paused, monitoring off

// Logic
const iconState = (status: SubscriptionStatus) => {
  if (status === "trial" || status === "active") return "lit";
  return "faded";
};
```

---

## Implementation Start Checklist

**Week 1: Foundation**
- [ ] Auth integration (Supabase sign-in preserve context)
- [ ] Mock cadastral endpoints (by-point, by-address, by-rp)
- [ ] Property claim skeleton UI (3 screens)
- [ ] Mock claim API endpoints (POST, GET, DELETE)

**Week 2: Claim Flow**
- [ ] Address search → boundary highlight (with mocks)
- [ ] Full-screen registration mode (light theme, 16px+)
- [ ] Trial activation button → mock API call
- [ ] Return to map with boundary + icon

**Week 3: Stripe & Icons**
- [ ] Stripe test integration (Day 30 logic, not auto-subscription)
- [ ] Satellite icon state machine (lit/faded)
- [ ] Day 25 reminder email logic
- [ ] Payment failure handling

**Week 4: Testing & Polish**
- [ ] End-to-end test: Claim → Trial → Icon activation
- [ ] Test Day 30 trial expiry (should pause, not auto-subscribe)
- [ ] NDVI "Refining data" state
- [ ] Font audit (16px+ minimum)
- [ ] Copy audit (plain English)

---

## Ready to Start?

**You have:**
1. ✅ 3-screen UX (simple, 60+ friendly)
2. ✅ Mock cadastral endpoints
3. ✅ Mocking strategy (swap later with real DCDB)
4. ✅ Stripe logic (Day 25 reminder, Day 30 pause, explicit user action)
5. ✅ Auth integration (Supabase, context preservation)
6. ✅ Icon state machine

**Next steps:**
1. Create `mocks/cadastral.ts` with 3–5 sample properties from Gondwana
2. Build claim UI skeleton (3 screens)
3. Wire mocked endpoints
4. Test end-to-end

**When real DCDB is ready:** Replace mocks with Supabase queries. Zero UI changes.

Proceed. Build. Report blockers.
