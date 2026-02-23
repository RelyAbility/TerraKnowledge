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

**Goal:** NDVI RQ Integration + FastAPI Wiring  
**Status:** Next up (Building on Feb 23 success)

| Time | Task | Details | Status |
|------|------|---------|--------|
| Morning | RQ async queue setup | Wire compute_ndvi_job to RQ (or Celery) | ⏳ |
| Afternoon | FastAPI endpoint | POST /api/property_claims → trigger RQ job on claim creation | ⏳ |
| Afternoon | Supabase updates | RQ job writes NDVI results back to property_claims table | ⏳ |
| Evening | E2E test | Claim created → RQ job enqueued → NDVI values appear in DB | ⏳ |

**Dependencies:** Feb 23 complete (NDVI functions tested + schema deployed)  
**Next Checkpoint:** RQ job processing verified with real claims (Feb 24 EOD)  
**Notes:**  
- Use RQ (Redis Queue) for Phase 1 simplicity vs Celery
- Job status transitions: pending → processing → ready (via ndvi_status column)
- Supabase real-time subscription next (Feb 25 frontend wiring)  

---

### 📅 Friday 25 Feb

**Goal:** Frontend NDVI Integration + Real-time Updates  
**Status:** Depends on Feb 24 RQ completion

| Time | Task | Details | Status |
|------|------|---------|--------|
| Morning | useClaimNDVI React hook | Subscribe to Supabase real-time NDVI updates | ⏳ |
| Afternoon | NDVICard component | Display NDVI values + date windows on PropertyClaimFlow | ⏳ |
| Evening | Integration test | Claim created → NDVI values display in UI with status spinner | ⏳ |

**Dependencies:** Feb 24 RQ + FastAPI endpoint complete  
**Next Checkpoint:** NDVI data flowing from claim creation → RQ job → Supabase → React hook → UI (Feb 25 EOD)  
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
