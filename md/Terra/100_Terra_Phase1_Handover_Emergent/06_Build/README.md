# Phase 1 Build – Feature-Based Documentation Index

**Last Updated:** 21 February 2026

This folder contains all implementation specifications, technical guides, and development documentation for **Phase 1 build-out**.

Organized by feature for easy navigation as development scales.

---

## 📋 Master Plan

**Start here:**
- [Phase_1_Execution_Strategy_The_Next_Move.md](./Phase_1_Execution_Strategy_The_Next_Move.md) — Strategic roadmap (25 satellite icons = success milestone)

---

## 🏗️ Build Features

### 01. Property Claim
**Status:** Ready for development  
**Audience:** Emergent  
**Docs:**
- [Specification.md](./01_Property_Claim/Specification.md) — Complete UX & feature spec
- [Implementation_Answers.md](./01_Property_Claim/Implementation_Answers.md) — Technical architecture & decisions
- [Implementation_Setup_and_Mocking.md](./01_Property_Claim/Implementation_Setup_and_Mocking.md) — Dev setup, mocking strategy, endpoint specs

**Phase:** Actively building (Week 1-4)

---

### 02. Satellite & NDVI
**Status:** Coming soon  
**Docs:** (Placeholder for future satellite integration spec)

---

### 03. Map Integration
**Status:** Coming soon  
**Docs:** (Placeholder for map display, boundary rendering, icon legend)

---

### 04. Stripe & Billing
**Status:** Coming soon  
**Docs:** (Placeholder for payment flow, subscription logic, Day 30 automation)

---

### 05. Authentication
**Status:** Using Supabase  
**Docs:** (Placeholder for auth integration details as needed)

---

### 06. API Reference
**Status:** Coming soon  
**Docs:** (Complete REST API spec once all endpoints are locked)

---

### 07. Database Schema
**Status:** Coming soon  
**Docs:** (PostGIS/Supabase schema for claims, cadastral data, subscriptions)

---

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
