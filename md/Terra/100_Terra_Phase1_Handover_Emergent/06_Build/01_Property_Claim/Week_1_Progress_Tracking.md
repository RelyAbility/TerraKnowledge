# Week 1 Progress Tracking – Property Claim Flow

**Week:** 21–28 February 2026  
**Team:** Emergent  
**Feature:** Property Claim Flow (from scratch)  
**Status:** IN PROGRESS  

---

## Daily Progress Log

### 📅 Monday 21 Feb

**Goal:** P0 Claims Persistence (UNPLANNED, PRIORITY SHIFT)
**Status:** ✅ COMPLETED

| Time | Task | Commit | Status |
|------|------|--------|--------|
| EOD | Supabase table created (property_claims) | N/A | ✅ |
| EOD | Backend endpoints migrated (3 endpoints) | N/A | ✅ |
| EOD | Frontend claimsStore.ts created (Zustand) | N/A | ✅ |
| EOD | Persistence verified (50/50 restart tests) | N/A | ✅ |

**Blockers:** None  
**Notes:** 
- Claims now durable (survive backend restart)
- Monitoring state persists across restarts
- Trial activation state persists
- Infrastructure-level reliability achieved
- **Milestone:** This is NOT prototype anymore. Database integrity confirmed.  

---

### 📅 Tuesday 22 Feb

**Goal:** GEE Setup (Brad) + NDVI Schema Foundation (Emergent) - PARALLEL WORKSTREAMS

| Time | Task | Owner | Commit | Status |
|------|------|-------|--------|--------|
| Morning | GEE account creation (8 steps) | Brad | N/A | ✅ In Progress |
| Morning | Supabase NDVI schema planning | Emergent | N/A | ✅ Ready |
| Noon | GEE credentials ready | Brad | N/A | ⏳ |
| Afternoon | NDVI SQL schema deployed | Emergent | N/A | ⏳ |
| Afternoon | Python function skeleton | Emergent | N/A | ⏳ |
| Evening | Credentials integration + local testing | Emergent | N/A | ⏳ |

**Blockers:** Waiting on GEE credentials from Brad  
**Notes:**  
- Claims persistence complete (from Feb 21)
- No blocking - parallel execution started
- Emergent can work on schema while Brad sets up GEE  

---

### 📅 Wednesday 23 Feb

**Goal:** NDVI Python Implementation + Supabase Schema Deployment  
**Status:** ✅ **COMPLETE** — All Feb 23 deliverables done

| Time | Task | Status |
|------|------|--------|
| Morning | GEE credentials ready (env var: GEE_SERVICE_ACCOUNT_JSON) | ✅ Complete |
| Morning | GEE Code Editor setup + connectivity test | ✅ Complete |
| Afternoon | Python ndvi_service.py implementation | ✅ Complete |
| Afternoon | Test with 5 coordinates (Gondwana + edges) | ✅ Complete |
| Evening | NDVI schema deployed to Supabase (10 columns) | ✅ Complete |

**Results (Test Data):**

| Location | Baseline | Current | Delta | Trend |
|----------|----------|---------|-------|-------|
| Gondwana Centre | 0.8400 | 0.8726 | +0.033 | ✅ Improving |
| Gold Creek Corridor | 0.6613 | 0.7326 | +0.071 | ✅ Improving |
| Northern Edge | 0.6116 | 0.7035 | +0.092 | ✅ Improving |
| Southern Edge | 0.3644 | 0.4989 | +0.134 | ✅ **Strongest signal** |
| Western Edge | 0.6871 | 0.6532 | -0.034 | ⚠️ Declining |

**Supabase Schema:** ✅ Deployed  
- 10 NDVI columns added (ndvi_baseline, ndvi_current, ndvi_delta, date windows, status, timestamp, error)
- ALTER TABLE completed successfully
- Schema ready for RQ job integration

**Milestone:** Real Sentinel-2 satellite NDVI data flowing from GEE to local Python ✓  

---

### 📅 Thursday 24 Feb

**Goal:** NDVI RQ Integration + E2E Testing  
**Status:** 🔴 P0 PRIORITY — Test RQ Pipeline

| Phase | Task | Details | Owner | Status |
|-------|------|---------|-------|--------|
| **P0** | RQ Worker | Start worker, create test claim, monitor logs | Emergent | 🔴 NOW |
| **P0** | Status Transitions | Verify `pending → ready` flow in database | Emergent | 🔴 NOW |
| **P0** | NDVI Fields | Confirm all fields persisted (baseline, delta, trend, timestamps) | Emergent | 🔴 NOW |
| **P0** | Error Handling | Simulate error conditions, verify error messages | Emergent | 🔴 NOW |
| **P0** | Idempotency | Verify job can safely retry without corruption | Emergent | 🔴 NOW |
| **P1** | Frontend Design | No mockups needed — follow accessible design spec | Emergent | ⏳ After P0 ✅ |

**P0 Testing Sequence:**
1. `rq worker default` (Terminal 1)
2. Create test claim via `/api/claims` (Terminal 2)
3. Monitor worker logs (expect 5-15s processing)
4. Query Supabase: Verify ndvi_status transitions, fields populated, dates correct
5. Test error case: Invalid coordinates → ndvi_status='error' + error_message
6. Confirm idempotency: Job can re-run safely

**Reference:** [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](../Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md)  
**Success Criteria:** All P0 checkpoints ✅ (8 must-haves listed in guide)  
**Next:** P1 frontend accessible design (when P0 complete)  

---

### 📅 Friday 25 Feb

**Goal:** Frontend NDVI Display (Accessible Design)  
**Status:** ⏳ Depends on P0 Complete

| Phase | Task | Details | Owner | Status |
|-------|------|---------|-------|--------|
| **P1** | NDVICard Component | Display vegetation health with large fonts, light theme | Emergent | ⏳ After P0 |
| **P1** | useClaimNDVI Hook | Real-time subscription to ndvi_status changes (Supabase) | Emergent | ⏳ After P0 |
| **P1** | Pending Animation | Satellite pulse (calm, 1200-1500ms cycle), text message | Emergent | ⏳ After P0 |
| **P1** | Ready Animation | Brief pulse on satellite icon when status → ready (600-800ms) | Emergent | ⏳ After P0 |
| **P1** | E2E Frontend Test | Create claim → spinner → data displays with animation | Emergent | ⏳ After P0 |

**Design Guidance (60+ Audience):**
- Light theme, 16px+ fonts, lots of spacing
- Plain language: "Vegetation health (from satellite)" not "NDVI"
- Show: Current, 3-year change (improving/stable/declining), Source, Last updated
- Pending: "Analysing satellite imagery for your site..." (calm pulsing icon)
- Ready: All fields visible + brief pulse animation on icon

**Reference:** [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](../Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md) (design details + component layout)  
**Success Criteria:** P1 works with actual NDVI data from P0 claims  
**Next Checkpoint:** NDVI end-to-end working (claim → RQ job → UI display)  


**Notes:**  
- Status spinner during processing (ndvi_status = 'processing')
- Display NDVI delta + date windows when ready
- Error message display if job fails  

---

### 📅 Saturday 26 Feb

**Goal:** Buffer + Stress Testing (Real NDVI)

| Time | Task | Commit | Status |
|------|------|--------|--------|
| Morning | Real NDVI data verification (10+ test properties) | — | ⏳ |
| Afternoon | Load testing (multiple concurrent claims) | — | ⏳ |
| Evening | Bug fix + final polish | — | ⏳ |

**Blockers:**  
**Notes:**  

---

### 📅 Sunday 27 Feb

**Goal:** Sign-Off + Handoff

| Time | Task | Commit | Status |
|------|------|--------|--------|
| Morning | Final checklist review (10 items complete?) | — | ⏳ |
| Afternoon | Prepare demo environment for Brad + Coordinator | — | ⏳ |
| Evening | Document any known issues / limitations | — | ⏳ |

**Blockers:**  
**Notes:**  

---

### 📅 Monday 28 Feb (Go-No-Go)

**Goal:** Sign-off + Soft Launch Ready

| Time | Task | Commit | Status |
|------|------|--------|--------|
| — | — | — | — |

**Blockers:**  
**Notes:**  

---

## 📊 Weekly Summary

**Commits This Week:** 0  
**Features Complete:** 0/7  
**Tests Passing:** 0/10  
**Blockers Outstanding:** 0  

---

## How to Use This

**For Emergent:**
1. Log commits here as you push
2. Mark status as: `In Progress` | `Complete` | `Blocked`
3. Note blockers immediately (Brad will unblock)
4. Update summary at end of day

**For Brad:**
- Check this daily to see progress
- Identify blockers before Emergent gets stuck
- Fast-track unblocks

---

## Commit Format (for easy tracking)

```
feat: Auth – Supabase sign-in + context preservation
feat: DCDB – Mock /api/cadastral/by-point endpoint
feat: ClaimUI – Address search screen
feat: Stripe – 30-day trial subscription logic
feat: Icons – Satellite state machine (inactive → trial_active → subscribed → paused)
test: Auth – Sign-in during claim flow
test: ClaimFlow – End-to-end (auth → address → confirm → trial → icon)
```

---

## Week 1 Success = All 10 Criteria Met

By Monday 28 Feb:
- [ ] Checklist item 1
- [ ] Checklist item 2
- [ ] Checklist item 3
- [ ] Checklist item 4
- [ ] Checklist item 5
- [ ] Checklist item 6
- [ ] Checklist item 7
- [ ] Checklist item 8
- [ ] Checklist item 9
- [ ] Checklist item 10

See **[Week_1_Development_Checklist.md](./Week_1_Development_Checklist.md)** for full details.

---

## Communication

**Daily standups:** Push commits + update this file  
**Blockers:** Tell Brad immediately (don't wait for EOD)  
**Questions:** Reference spec docs first, then ask Brad  
**PRs:** Link to this tracker in PR description  

---

## Key Dates

- **Today (21 Feb):** Start auth integration
- **25 Feb EOD:** Stripe + icon state working
- **27 Feb EOD:** All testing complete, ready for sign-off
- **28 Feb 09:00:** Go-no-go decision (soft launch ready or not)

**Next milestone:** 1 March — Show coordinator 12 live properties
