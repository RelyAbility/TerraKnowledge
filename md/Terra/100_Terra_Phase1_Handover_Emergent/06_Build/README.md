# 06_Build — Feature-Based Build Documentation

**Last Updated:** 23 February 2026  
**Phase:** Week 1 Complete (Claims Persistence + GEE Setup + NDVI Functions)

This folder contains all Phase 1 build specifications, setup instructions, and progress tracking organized by **feature area**.

---

## Directory Structure

```
06_Build/
├── 01_Property_Claim_Flow/
│   └── Week_1_Progress_Tracking.md — Daily progress (Feb 21-25)
│
├── 02_NDVI_Satellite_Monitoring/
│   ├── GEE_Setup_Instructions_Feb23.md — Environment setup guide
│   ├── GEE_Credentials_Ready_Feb23.md — Python NDVI code + tests
│   └── NDVI_Supabase_Schema_Deployment_Feb23.md — Schema & SQL
│
├── 10Day_Critical_Sprint_Feb22_Mar3.md — Master timeline
└── README.md — This navigation guide
```

---

## Quick Links

### For Daily Status
[Week_1_Progress_Tracking.md](01_Property_Claim_Flow/Week_1_Progress_Tracking.md) — Task breakdown by day, blockers, milestones

### For NDVI Implementation (Feb 23-25)
1. **Setup:** [GEE_Setup_Instructions_Feb23.md](02_NDVI_Satellite_Monitoring/GEE_Setup_Instructions_Feb23.md)
2. **Code:** [GEE_Credentials_Ready_Feb23.md](02_NDVI_Satellite_Monitoring/GEE_Credentials_Ready_Feb23.md)
3. **Schema:** [NDVI_Supabase_Schema_Deployment_Feb23.md](02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md)

### For Sprint Overview
[10Day_Critical_Sprint_Feb22_Mar3.md](10Day_Critical_Sprint_Feb22_Mar3.md) — Full timeline, blockers, checkpoints

---

## Status Summary (Feb 23)

| Feature | Status | Notes |
|---------|--------|-------|
| Claims Persistence | ✅ Complete | 50/50 restart tests passed |
| GEE Credentials | ✅ Complete | Deployed to pod environment |
| Python NDVI Functions | ✅ Complete | 5/5 test coordinates validated |
| Supabase Schema | ✅ Deployed | 10 NDVI columns active |
| RQ Integration | 🔄 In Progress | Due Feb 24 |
| FastAPI Endpoint | 🔄 In Progress | Due Feb 24 |
| Frontend NDVICard | ⏳ Pending | Due Feb 25 |
| E2E Testing | ⏳ Pending | Due Feb 26-27 |

**Checkpoint:** Feb 28 Go/No-Go decision (on track)

---

## Test Results (Real NDVI Data)

5 coordinates tested with Sentinel-2 (Feb 23):

| Location | Baseline | Current | Delta | Signal |
|----------|----------|---------|-------|--------|
| Gondwana Centre | 0.8400 | 0.8726 | +0.033 | ✅ Improving |
| Gold Creek | 0.6613 | 0.7326 | +0.071 | ✅ Improving |
| Northern Edge | 0.6116 | 0.7035 | +0.092 | ✅ Improving |
| Southern Edge | 0.3644 | 0.4989 | +0.134 | ✅ **Strongest** |
| Western Edge | 0.6871 | 0.6532 | -0.034 | ⚠️ Declining |

---

## Timeline

| Date | Focus | Status |
|------|-------|--------|
| Feb 21 | Claims persistence | ✅ Done |
| Feb 22 | GEE setup + Schema planning | ✅ Done |
| Feb 23 | Python test + Schema deployed | ✅ Done |
| Feb 24 | RQ integration + FastAPI wiring | 🔄 In Progress |
| Feb 25 | Frontend component | ⏳ Next |
| Feb 26-27 | E2E testing | ⏳ Pending |
| Feb 28 | Go/No-Go decision | ✓ Checkpoint |
| Mar 1-3 | Soft launch (5 real users) | ⏳ Execute |

---

## References

### From md/Terra Root
- [NDVI_Phase1_Implementation_Sentinel2_GEE.md](../../NDVI_Phase1_Implementation_Sentinel2_GEE.md) — Full spec
- [NDVI_Architecture_Decision_GEE_Setup.md](../../NDVI_Architecture_Decision_GEE_Setup.md) — Architecture
- [What_Matters_Now_Priority_Ordering.md](../../What_Matters_Now_Priority_Ordering.md) — Priorities

### From 100_Terra_Phase1_Handover_Emergent Root
- [Phase_1_Execution_Strategy.md](../Phase_1_Execution_Strategy_The_Next_Move.md) — Strategy
- [Claims_Schema_Supabase.md](../Claims_Schema_Supabase.md) — Claims DB design

---

**Built with:** Python + FastAPI + Supabase + Google Earth Engine  
**Owner:** Emergent (implementation) + Brad (GEE setup)  
**Next:** RQ async queue integration (Feb 24)

### 08. Testing & Deployment
**Status:** Coming soon  
**Docs:** (End-to-end test plan, QA checklist, release gates)

---

## 🔗 Related Documents

**Strategy & Vision:**
- Phase 1 Execution Strategy (this folder, top level)
- Terra Constitution (defines guardrails)

**Broader Phase 1 Context:**
- See `100_Terra_Phase1_Handover_Emergent/01_Context/` for foundational specs

---

## 📝 How This Works

**For Emergent:**
1. Read Property Claim docs (all 3) before coding
2. Build features incrementally (auth → UI → endpoints)
3. Each feature gets its own folder as build expands
4. New features: Create folder, add `Specification.md` + `Implementation.md`

**For Brad:**
- Track progress in commit messages
- Move completed features to documentation archive
- Link test results to release gates

---

## ⚡ Quick Links

| Feature | Spec | Implementation | Status |
|---------|------|----------------|--------|
| Property Claim | [✓](./01_Property_Claim/Specification.md) | [✓](./01_Property_Claim/Implementation_Setup_and_Mocking.md) | Ready to Build |
| Satellite icon | TBD | TBD | Blocked on claim |
| Map boundaries | TBD | TBD | Blocked on claim |
| Stripe trial | TBD | TBD | Blocked on claim |
| NDVI baseline | TBD | TBD | Blocked on cadastral |

---

## 🎯 Success Criteria (Phase 1)

- [ ] 10–15 properties claimed in 4.2 km
- [ ] Satellite icons visible on map
- [ ] Stripe Day 30 → Subscription works
- [ ] Coordinator sees proof (25 icons)
- [ ] 1 March meeting with tangible launch

See Phase_1_Execution_Strategy_The_Next_Move.md for complete roadmap.
