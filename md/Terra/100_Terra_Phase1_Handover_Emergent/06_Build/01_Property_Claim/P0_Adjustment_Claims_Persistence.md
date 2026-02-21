# P0 Priority Adjustment: Claims Persistence → Supabase

**Date:** 21 February 2026  
**Status:** URGENT – Do Before Stripe Integration  
**Team:** Emergent  

---

## ⚠️ The Issue

Claims are currently stored **in-memory only**. This blocks everything downstream:

- [ ] Claims disappear on app restart
- [ ] Trial state breaks
- [ ] Stripe integration has nowhere reliable to track subscription
- [ ] Testing becomes non-deterministic
- [ ] No audit trail

**Stripe integration on in-memory state = Building on sand.**

---

## 🎯 New P0 (Before Stripe)

**Move claims persistence from in-memory → Supabase.**

Once complete, proceed with:
1. Stripe integration
2. Satellite icon state
3. Frontend testing
4. Soft onboarding

---

## 📋 Schema Required

**Table: `claims`**

```sql
CREATE TABLE claims (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id),
  
  -- Property Data
  parcel_id TEXT NOT NULL,
  lot_number TEXT,
  property_name TEXT,
  owner_name TEXT NOT NULL,
  email TEXT NOT NULL,
  boundary JSONB NOT NULL, -- GeoJSON
  area_m2 INT,
  address TEXT,
  
  -- Monitoring State
  monitoring_state TEXT NOT NULL DEFAULT 'inactive', -- inactive | trial_active | subscribed | paused
  
  -- Trial Dates
  trial_start_date TIMESTAMP,
  trial_end_date TIMESTAMP,
  
  -- Stripe Integration (nullable initially)
  stripe_customer_id TEXT,
  stripe_subscription_id TEXT,
  
  -- Audit
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  
  -- Constraints
  UNIQUE(user_id, parcel_id, lot_number) -- One claim per property/lot per user
);
```

**Why this schema:**
- `user_id` links to authenticated user (Supabase auth)
- `monitoring_state` drives icon visibility + behavior
- `trial_start_date` + `trial_end_date` for Day 30 logic
- `stripe_*` fields ready for Stripe hookup (nullable for now)
- `boundary` + `address` for map display
- `UNIQUE` constraint prevents duplicate claims

---

## 🔄 Migration Steps

### 1. Create Supabase Table (10 min)

In Supabase dashboard or via SQL:
```sql
-- Run the schema above in Supabase SQL editor
-- Ensure RLS is enabled: SELECT user_id
-- (Users can only see their own claims)
```

### 2. Update Backend Endpoints (30 min)

**POST /api/claims** (Create)
```json
{
  "parcel_id": "dcdb_001",
  "lot_number": "1",
  "property_name": "My Property",
  "owner_name": "John Doe",
  "email": "john@example.com",
  "boundary": { /* GeoJSON */ },
  "area_m2": 52500,
  "address": "123 Sample Road, Brookfield"
}
```
→ Insert into `claims` table with `user_id` from auth token  
→ Set `monitoring_state = 'inactive'`  
→ Return `claim_id`

**POST /api/claims/{claim_id}/start-trial** (Start Trial)
```json
{
  "claim_id": "uuid"
}
```
→ Update claim:
- Set `trial_start_date = NOW()`
- Set `trial_end_date = NOW() + INTERVAL '30 days'`
- Set `monitoring_state = 'trial_active'`
- Return updated claim

**GET /api/claims/{claim_id}** (Get Claim)
→ Fetch from `claims` table  
→ Return current `monitoring_state`

**GET /api/claims** (List User's Claims)
→ Fetch all claims for authenticated `user_id`  
→ Return array with current states

**DELETE /api/claims/{claim_id}** (Delete)
→ Remove claim from `claims` table

### 3. Update Frontend (30 min)

**PropertyClaimFlow:**
- On trial activation: POST `/api/claims/start-trial`
- Receive persistent `claim_id`
- Store in React state + localStorage (backup)
- On page reload: Fetch persisted state from `/api/claims/{claim_id}`

**SatelliteIcon:**
- Query `/api/claims/{claim_id}` for `monitoring_state`
- Render icon based on state (trial_active | subscribed | paused | inactive)
- Poll every 30s or on visibility change

### 4. Testing (1 hour)

- [ ] Create claim → verify in Supabase dashboard
- [ ] Claim persists after app restart
- [ ] Trial activation updates `monitoring_state`
- [ ] Icon reflects correct state
- [ ] Multiple claims per user works
- [ ] User can't see other user's claims (RLS test)

---

## ⏱️ Timeline

**Today (21 Feb) — EOD:**
- Create Supabase table
- Update 3 endpoints (create, start-trial, get)

**Tomorrow (22 Feb) — Morning:**
- Frontend integration (trial activation → Supabase storage)
- Verify persistence across app restart

**Tomorrow (22 Feb) — Afternoon:**
- Testing + bug fixes
- Ready for Stripe integration

---

## ✅ Success Criteria

- [ ] Claims table created in Supabase
- [ ] All endpoints persist to Supabase (not in-memory)
- [ ] User authentication enforced (can only see own claims)
- [ ] Trial activation creates persistent claim record
- [ ] Icon state driven by `monitoring_state` field
- [ ] Claims survive app restart
- [ ] 24/24 API tests still pass (with Supabase instead of in-memory)
- [ ] Commit: `refactor: Claims – In-memory → Supabase persistence`

---

## 🚨 Important

**Do NOT proceed to Stripe integration until this is complete.**

Stripe webhooks need reliable state to track subscriptions. In-memory breaks that completely.

---

## After Claims Persistence ✅

Once Supabase persistence is live:

### Next: Stripe Integration (2 days)
- Wire `/api/claims/{claim_id}/start-trial` to Stripe
- Day 30 check → update `monitoring_state = 'paused'`
- Payment webhook → update `monitoring_state` + `stripe_subscription_id`

### Then: Satellite Icon Visual (1 day)
- Icon state = `monitoring_state` only
- trial_active | subscribed → 🟢 Green/lit
- paused | inactive → ⚫ Grey/faded

### Then: Frontend Testing (1 day)
- Full E2E with Supabase persistence
- Stripe Day 30 simulation
- Icon state transitions

### Then: Soft Onboarding (Decision Point)
- **Ask:** Should we onboard 5 real landholders now?
- **Answer:** Yes. Density before polish.
- Real users → real feedback → real data
- Then iterate.

---

## Questions?

Don't guess. Ask Brad if:
- Supabase connection details needed
- Schema changes required
- Endpoint behavior unclear

Move fast. Ship Supabase persistence today. Then we move to Stripe with confidence.
