# Property Claim Flow – Development Handoff Confirmation

**Date:** 21 February 2026  
**Status:** Emergent Ready to Code  
**Scope:** Week 1 P0 Implementation  

---

## ✅ Go-Ahead Confirmed

You are **cleared to start immediately** on Week 1 P0 priorities. Everything below removes blockers.

---

## 🔐 Authentication (Supabase)

**What to use:**
- Existing Supabase setup (credentials env-configured in pod)
- Do NOT use any "Emergent LLM key"

**If you can't see env variables:**
- Required variables: `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
- Tell Brad exactly which are missing → he'll add them

**Implementation:**
- User can start claim flow without auth
- At address entry (before cadastral lookup): Check for user session
- If not logged in: Show sign-in modal
- On successful auth: Preserve entered address in state, resume from boundary confirmation
- Don't lose user input during auth redirect

---

## 💳 Stripe Integration

**What to use:**
- Stripe test key available in pod environment
- Do NOT hardcode keys in repo

**If test key is missing:**
- Tell Brad which env vars you need: `STRIPE_PUBLISHABLE_KEY`, `STRIPE_SECRET_KEY`
- He'll supply them

**Implementation:**
- Use env var placeholders if needed: `process.env.REACT_APP_STRIPE_PUBLISHABLE_KEY`
- 30-day free trial (Day 0-29): No card required, icon active
- Day 25: Send automated reminder email (mock for now, can wire real email later)
- Day 30: Trial expires, monitoring pauses (icon fades)
- User must explicitly add payment to reactivate
- On payment failure: Pause monitoring (don't cancel)

---

## 📖 Specification Documents

**Read all 3 before coding:**

1. [Specification.md](./01_Property_Claim/Specification.md)
   - Complete UX, flow, feature gates

2. [Implementation_Answers.md](./01_Property_Claim/Implementation_Answers.md)
   - Backend architecture, endpoint specs, Stripe logic

3. [Implementation_Setup_and_Mocking.md](./01_Property_Claim/Implementation_Setup_and_Mocking.md)
   - Mock endpoints, state management, DB schema

**Why:** These lock exact copy, timing, and error states. Don't guess.

**But:** Don't wait to start scaffolding. Read docs + build skeleton simultaneously.

---

## 🎯 Week 1 Scope (Strictly P0)

**INCLUDE:**
- ✅ 3-screen claim flow (Address → Confirm → Trial)
- ✅ Supabase auth + context preservation
- ✅ Mock cadastral endpoints (by-point, by-address, by-rp)
- ✅ Stripe trial subscription setup
- ✅ Satellite icon state machine
- ✅ Basic end-to-end testing

**DEFER (Not Week 1):**
- ❌ Real Gondwana government data (placeholder: wait for DCDB ingest)
- ❌ Animated loading screen
- ❌ Vector tiles
- ❌ Biodiversity score
- ❌ Mission engine
- ❌ BPA/WildNet data

---

## 🎨 UX Constraints (Non-Negotiable)

**Typography:**
- Light theme only (no dark mode)
- 16px+ fonts minimum
- Generous line-height (1.5x)
- Full-width buttons (touch-optimized)

**Language:**
- Plain English (no technical jargon)
- No ecology terminology
- Explain why we need info (e.g., "So we can track your property")
- One primary CTA per screen

**Map:**
- Map VISIBLE only for: Address search + Boundary confirmation
- Map HIDDEN during: Full-screen registration + Trial activation
- Return to map after trial activation with boundary visible + icon state

---

## 🔄 State Machines (Explicit)

**Critical:** Implement these states exactly. Prevents confusion later.

### Parcel States

```
unselected
  ↓
highlighted (user hovered/searched, boundary shown on map)
  ↓
confirmed (user tapped "Is this your property?" YES)
  ↓
claimed (registration complete, boundary persisted)
```

**Implementation:**
```typescript
type ParcelState = 'unselected' | 'highlighted' | 'confirmed' | 'claimed';

// Example flow:
// User enters address → cadastral matches → ParcelState = 'highlighted'
// User confirms boundary → ParcelState = 'confirmed'
// User completes registration → ParcelState = 'claimed'
```

### Monitoring States

```
inactive
  ↓ (user taps "Start Free Trial")
trial_active (Day 0-30)
  ↓ (on Day 30)
paused (waiting for user to add payment)
  ↓ (user adds payment)
subscribed (ongoing $20/month)
  ↓ (on payment failure)
paused (again)
```

**Implementation:**
```typescript
type MonitoringState = 'inactive' | 'trial_active' | 'subscribed' | 'paused';

// Satellite icon reflects ONLY monitoring state:
// trial_active || subscribed → 🟢 Icon lit/green
// inactive || paused → ⚫ Icon faded/grey
```

**Critical Rule:**
- Icon state = MonitoringState only
- DO NOT conflate icon with claim status
- Icon should be off until user explicitly starts trial

---

## 🛠️ Implementation Order (Locked)

**1. Auth Integration (Day 1-2)**
- Supabase session detection
- Sign-in modal during claim (preserve address context)
- Post-auth redirect back to claim flow

**2. DCDB Mock Endpoints (Day 2)**
- `/api/cadastral/by-point?lat=&lng=` → Mock 3-5 Gondwana properties
- `/api/cadastral/by-address?address=` → Same mock data
- `/api/cadastral/by-rp?rp=` → Mock RP lookup

See [Implementation_Setup_and_Mocking.md](./01_Property_Claim/Implementation_Setup_and_Mocking.md) for exact response shapes.

**3. Claim UI Skeleton (Day 2-3)**
- Screen 1: Address search input → Auto-suggest → Cadastral lookup → Boundary highlight on map
- Screen 2: "Is this your property?" overlay → [Yes | No]
- Screen 3: Full-screen registration form (PropertyName, OwnerName, Email)
- Screen 4: Full-screen trial offer → [Start Trial | Skip]

**4. Auth + Skeleton Integration (Day 3)**
- Detect if user logged in at claim start
- If not: Show sign-in before cadastral lookup
- Preserve address input through auth
- Resume flow seamlessly

**5. Stripe Integration (Day 3-4)**
- Create Stripe subscription logic
- Day 30 → Set monitoring to "paused"
- Day 25 → Send reminder (hardcoded email for now, can wire SendGrid later)
- Payment failure → Pause (don't cancel)

**6. Satellite Icon State (Day 4)**
- Implement icon as function of MonitoringState
- Icon visible on map only after trial starts
- Icon color tied to subscription status

**7. Testing (Day 4-5)**
- Full flow: Auth → Address search → Boundary confirm → Registration → Trial → Icon appears
- Test Day 30 expiry (monitoring → paused)
- Test payment failure → Monitoring pause
- Verify context preservation through auth redirect

---

## 📝 State Diagram (Reference)

```
┌─────────────────────────────────────────────────────┐
│ Claim Flow: Parcel → Monitoring                      │
└─────────────────────────────────────────────────────┘

Screen 1: Address Search
  [Input] → cadastral lookup → 
    ParcelState = 'highlighted'
    Map shows boundary

Screen 2: Confirm Boundary
  "Is this your property?" → [Yes]
    ParcelState = 'confirmed'
    Move to registration

Screen 3: Full-Screen Registration
  [Collect property name, owner, email]
    ParcelState = 'claimed'
    Boundary stored

Screen 4: Full-Screen Trial Offer
  "Start 30-day free trial?" → [Start]
    MonitoringState = 'trial_active'
    Stripe trial period begins (Day 0-30)

Return to Map
  ✓ Boundary visible (white outline)
  ✓ Satellite icon active (🟢 green)
  ✓ User can claim more properties or explore
```

---

## 🚫 Common Pitfalls (Avoid)

1. **Don't hardcode api keys** → Use env vars
2. **Don't conflate icon state with claim state** → Icon reflects MonitoringState only
3. **Don't auto-subscribe at Day 30** → Pause, require explicit user action
4. **Don't lose form context during auth** → Preserve address input
5. **Don't show dark theme** → Light only
6. **Don't use ecology jargon** → Plain English
7. **Don't require card upfront** → 30-day free first

---

## ✅ Checklist Before Coding

- [ ] Read all 3 spec documents
- [ ] Confirm env vars present (SUPABASE_*, STRIPE_*)
- [ ] If missing, tell Brad exactly which ones
- [ ] Understand ParcelState machine
- [ ] Understand MonitoringState machine
- [ ] Understand icon = MonitoringState only
- [ ] Confirm 3-screen + auth flow with Brad
- [ ] Clarify any spec ambiguities NOW

---

## 🎯 Success Criteria (End of Week 1)

- [ ] User can sign in (auth works, context preserved)
- [ ] User can enter address → see boundary → confirm property
- [ ] User can complete registration → claim property
- [ ] User can start 30-day trial → satellite icon appears (green)
- [ ] Mock cadastral endpoints return consistent data
- [ ] Stripe test subscription created at Day 30 (in test mode)
- [ ] MonitoringState transitions working (inactive → trial_active → paused)
- [ ] Icon reflects monitoring state (lit/faded)
- [ ] All text plain English, 16px+, light theme
- [ ] Map hidden during registration/trial screens
- [ ] End-to-end test: Claim 1 property → Icon activates → Success

---

## 🚀 Ready?

**You have:**
- ✅ Spec documents (3 files)
- ✅ State machines (Parcel + Monitoring)
- ✅ Implementation order (7 steps)
- ✅ Auth unblocked (Supabase)
- ✅ Stripe unblocked (test key in env)
- ✅ UX constraints (light, 16px+, plain language)

**Go build. Report blockers. Commit early, push often.**

Commit message format:
```
feat: Property Claim – Auth context preservation
feat: Property Claim – DCDB mock endpoints
feat: Property Claim – 3-screen UI skeleton
etc.
```

**Brad will review + merge as you push.**

Let's ship Phase 1. 🚀
