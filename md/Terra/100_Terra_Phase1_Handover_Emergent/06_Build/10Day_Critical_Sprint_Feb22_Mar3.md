# 10-Day Critical Sprint: Feb 22 – Mar 3

**Status:** Infrastructure complete. Now execution.

**The Infrastructure** (✅ DONE):
- Claims persist in Supabase
- Monitoring state durable across restarts
- Trial activation scaffolded
- Backend endpoints wired to database

**What's Missing** (Must complete by Mar 1):
1. Stripe billing pipeline live
2. Real NDVI baseline data flowing
3. Satellite icons visible on map

**Constraint:** 10 days to $0 + real proof

---

## The Three Blockers

### Blocker 1: Stripe Integration
**Status:** Not started  
**Scope:** Wire Day 0-30 trial + subscription billing logic  

What works now:
- Trial state persists
- Day 30 pause scaffolded

What doesn't:
- Actual Stripe charge on Day 31
- Day 25 reminder email
- Payment failure handling
- Auto-pause on Day 30

**Success:** User claims property → Trial activated → Day 25 reminder sent → Day 31 charged → Icon persists

**Effort:** 2-3 days (test keys, webhook handling, error logic)

---

### Blocker 2: Real NDVI Pipeline
**Status:** Not started  
**Scope:** Replace mock satellite data with actual Sentinel-2 NDVI baseline  

What works now:
- Mock NDVI in claimsStore
- Icon state doesn't use real values yet

What doesn't:
- Real Sentinel-2 query against claimed parcels
- Asynchronous NDVI calculation (5+ seconds)
- Baseline capture on claim date
- Update frequency (daily? weekly?)

**Success:** Claim property → NDVI calculated asynchronously → "Refining data" state visible → Real baseline persisted → Icon updates

**Effort:** 2-3 days (Sentinel-2 API + async job + caching)

---

### Blocker 3: Satellite Icon Visibility
**Status:** Partially done (30% — state machine scaffolded, rendering not wired)  
**Scope:** Map displays icon based on monitoring_state + real NDVI  

What works now:
- MonitoringState enum defined
- State transitions logic in claims table
- Icon SVG component created

What doesn't:
- Icon rendered on map for each claimed property
- Icon color/style changes per state (inactive → trial → subscribed → paused)
- Icon shows real NDVI value (not just placeholder)
- Icon survives page refresh

**Success:** Open map → See 5+ satellite icons → Each icon color = monitoring_state → Click icon → See NDVI value

**Effort:** 1-2 days (map layer integration + state binding)

---

## 10-Day Execution Plan

### Days 22-24 (Tue-Thu): Stripe Phase 1
**Goal:** Stripe billing pipeline wired + test

| Day | Task | Deliverable | Success Criteria |
|-----|------|-------------|-----------------|
| Tue 22 | Wire trial_end_date → stripe charge logic | Endpoint test passing | Day 31 charge working |
| Wed 23 | Stripe webhooks (success/failure) | Webhook handler | Real payment simulation works |
| Thu 24 | Day 25 reminder + Day 30 pause automation | Email service + cron job | Timing triggers on schedule |

**Checkpoint (Thu EOD):** 10 test claims → 3 reach Day 25 → Reminders sent → 1 reaches Day 31 → Charge attempted & logged

---

### Days 22-24 (Tue-Thu, Parallel): Real NDVI Start
**Goal:** Sentinel-2 pipeline querying real data

| Day | Task | Deliverable | Success Criteria |
|-----|------|-------------|-----------------|
| Tue 22 | Sentinel-2 API integration (test coords) | Query working | Real NDVI values returned |
| Wed 23 | Async NDVI job queue (PostgreSQL/cron) | Background job | Job completes in <10s |
| Thu 24 | Baseline capture on claim + persistence | NDVI stored in claims table | Real values survive restart |

**Checkpoint (Thu EOD):** 5 test properties → Real NDVI values in database → Values match Sentinel-2

---

### Days 25-26 (Fri-Sat): Icon Integration + Testing
**Goal:** Icons visible on map, all state transitions working

| Day | Task | Deliverable | Success Criteria |
|-----|------|-------------|-----------------|
| Fri 25 | Icon rendering on map layer | Map component updated | 5+ icons visible when claims exist |
| Fri 25 | Icon state binding (inactive → trial → sub → paused) | Icon color logic | Colors change per state |
| Fri 25 | NDVI value display on icon/hover | Tooltip added | Real NDVI visible to user |
| Sat 26 | End-to-end test (claim → trial → charge → icon updates) | Test scenario | Full lifecycle working |

**Checkpoint (Sat EOD):** 
- Claim property → Icon appears within 5 seconds
- Icon shows correct monitoring_state color
- Icon displays real NDVI value
- Icon color updates when state changes (Day 25 → Day 31)
- All claims survive restart

---

### Days 27-28 (Sun-Mon): Buffer + Findings
**Goal:** Fix whatever breaks, prepare for demo

| Day | Task | Deliverable | Success Criteria |
|-----|------|-------------|-----------------|
| Sun 27 | Integration testing (all 3 systems together) | Test results | Zero critical bugs |
| Mon 28 | Known issues doc + demo environment prep | Issues list + staging | Demo ready for Brad |

**Checkpoint (Mon EOD):**
- All 10 checklist items marked complete
- Known issues documented (if any)
- Demo environment stable

---

### Days 1-3 March (Tue-Thu): Coordinator Meeting + Soft Onboard
**Goal:** Show coordinator 12+ real properties on map → Onboard 5 real landholders

| Day | Task | Owner | Deliverable |
|-----|------|-------|------------|
| Mar 1 | Coordinator meeting (12+ properties visible) | Brad | Live demo on map |
| Mar 2-3 | Soft onboard 5 trusted landholders | Brad | Real users, 5 claims active |

**Success:** Coordinator sees real satellite icons + real monitoring states → "This is production, not prototype" → Green light for Founding 50

---

## Definition of "Brutally Focused"

### DO:
✅ Stripe: Day 0-30 logic only (no custom plans, no discount logic, no free tier)  
✅ NDVI: Baseline on claim date + daily updates (no historical backfill, no ML predictions)  
✅ Icon: On map + color state only (no animations, no hover details)  
✅ Testing: 5-10 test claims + real data (not 100 properties)  

### DON'T:
❌ Stripe: Multiple tier pricing, promo codes, billing portal, custom webhooks  
❌ NDVI: Historical data, V-index, trend analysis, heatmaps  
❌ Icon: Clustering, filtering, advanced interactions, custom UI  
❌ Testing: Full QA suite, performance benchmarking, 1000 property load test  

**Philosophy:** Stripe works → Icon works → NDVI flows. Then onboard real users. Everything else waits.

---

## Success Metrics (Mar 1 Morning)

- [ ] 5+ test claims in Supabase
- [ ] Each claim has monitoring_state = trial_active or subscribed
- [ ] Each claim has real Sentinel-2 NDVI baseline
- [ ] Map displays 5+ satellite icons
- [ ] Icon color = monitoring_state (visual verification)
- [ ] Stripe test charge logged (Day 31 simulation)
- [ ] Day 25 reminder sent (email log)
- [ ] Day 30 pause automated (state == paused)
- [ ] All claims survive backend restart
- [ ] Frontend + Backend deployed to demo environment
- [ ] Coordinator demo prepared (12+ properties, all icons visible)

**Go/No-Go Decision:** If 10/10 met → Soft onboard 5. If not → Fix, try again.

---

## Known Risks

1. **Sentinel-2 data latency** — Newest data may be 5+ days old. Plan for that in baseline messaging.
2. **Stripe webhook timing** — Test keys may not fire webhooks reliably. Build in manual override for testing.
3. **NDVI calculation time** — Real Sentinel-2 query can take 5-10 seconds. Show "Refining data" state during processing.
4. **Icon rendering performance** — 100+ icons on map may be slow. Keep test set to 5-10 for now.
5. **Ngrok tunnel instability** — If tunnel breaks, fall back to local testing or Vercel preview.

---

## Daily Standby Format (for Emergent)

Each day, 5pm UTC:

```
## Day X Status

**Done Today:**
- [ ] [Task]
- [ ] [Task]

**Blocking On:**
- None / [Issue]

**Code Pushed:** [Commit SHA or "no commits today"]

**Tomorrow's Plan:**
- [ ] [Task]
```

Brad will check the progress log daily. If blocker appears, unblock immediately.

---

## Timeline Confidence

**22-24 Feb (Stripe + NDVI Start):** 48h ✅ — Both should be wired by Thu EOD  
**25-26 Feb (Icon + Integration):** 24h ✅ — Should be visible + tested by Sat  
**27-28 Feb (Buffer + Demo Prep):** 24h ✅ — Handles unforeseen issues  
**1-3 Mar (Coordinator + Soft Onboard):** 48h ✅ — Time for demo + user onboarding  

**Slack:** 2 days (If Stripe or NDVI slip, still recoverable)

**Crunch Point:** 26 Feb EOD. If icon not rendering by then, Mar 1 is at risk.

---

## Success Looks Like

**Mar 1, 9:00 AM:**
- Open map
- See 12+ blue/green satellite icons
- Each icon labeled with property address
- Click icon → See NDVI value
- Copy of claims table shows monitoring_state for each

**Brad to Coordinator:**
"These are real properties from our test group. Real NDVI data. Real trial subscriptions active. Stripe is live in test mode. Icon state updates automatically every day as NDVI changes. This is the proof you need to soft-launch 50 founders."

**Coordinator Response:**
"When can we onboard the first 5?"

That's the conversation we need to have on Mar 1.

Anything less is not ready.

---

**Owner:** Emergent (code) + Brad (blocking + decisions)  
**Updated:** 21 Feb  
**Next Review:** 22 Feb 18:00 UTC
