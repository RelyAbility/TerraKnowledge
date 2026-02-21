# Strategic Decision: Density Before Polish

**Date:** 21 February 2026  
**Decision Point:** End of Week 1  
**Status:** FRAMEWORK FOR POST-STRIPE DECISION  

---

## The Question

After Stripe integration + Claims persistence complete (~22 Feb):

**Should we:**

**A) Soft onboard 5 real landholders (DENSITY)**  
Get real users, real claims, real feedback. Start learning.

**B) Polish UI more (POLISH)**  
Refine 7-screen flow → 3-screen. Animation. Copy audit. Button alignment.

**C) Move to real DCDB data (DATA)**  
Replace mock cadastral with actual QLD government data.

---

## The Answer

**A. Soft onboard 5 real landholders. Immediately.**

Here's why.

---

## Why Density Wins Over Polish

### 1. Real Data Beats Hypothetical

Mock data teaches us **nothing about user behavior**.

Real landholders teaching you everything:
- Does address search work for them? (Our postal system ≠ theirs)
- Does boundary display make sense? (Do they understand parcels?)
- Is $20/month pricing right? (Real objection or not?)
- Does trial mechanics feel natural?

Polish the UI against **real feedback**, not assumptions.

### 2. Trial → Subscription Mechanics Must Work

We can't test Stripe Day 30 logic against ourselves.

Real users show you:
- Do they actually enter payment on Day 25?
- Do they understand the transition from trial → paid?
- What breaks on Day 30? (It always breaks somewhere)

Find the breaks with 5 people. Not with 5,000.

### 3. Satellite Icon State Needs Lives

The icon is meaningless without real monitoring happening.

5 real users = 5 real satellite icons on the map.  
That map changes everything about product feeling.

Mock icon ≠ Real icon with real property history.

### 4. Proof Molecule for Coordinator

On 1 March, showing coordinator:
- "12 properties claimed"
- "Demo properties" vs. **"Real properties from real people"**

Difference between concept and reality.

---

## UI "Fragmentation" Concern

7-screen flow feels fragmented now. May be right for reasons:

1. **Auth interruption** — 1 screen (necessary)
2. **Address search** — 1 screen (focused)
3. **Boundary confirm** — 1 screen (map-driven, intentional)
4. **Registration** — 1 screen (deliberate pause before commitment)
5. **Trial offer** — 1 screen (explicit decision moment)
6. **Completion** — 1 screen (confirmation)
7. **Fallback (RP)** — 1 screen (error recovery)

This might actually **be right for intent clarity**, not fragmentation.

**Don't consolidate until 5 real users tell you it's fragmented.**

Polish moves 3-4 screens onto 1 screen, but loses **explicit decision points**.

For 60+ audience, explicit beats condensed.

---

## Decision Tree Post-Stripe

```
Is Stripe working? (Trial → sub → payment)
  ├─ YES
  │   └─ Soft onboard 5 real landholders
  │       ├─ Track for 5+ days
  │       ├─ Gather feedback
  │       ├─ Fix critical bugs
  │       ├─ Then decide:
  │       │   ├─ More density (add 10 more)?
  │       │   ├─ Polish based on feedback?
  │       │   └─ UI consolidation if they complain?
  │       └─ Proceed to Coordinator 1 March with real proof
  │
  └─ NO
      └─ Fix Stripe in controlled environment first
          └─ Then onboard 5
```

---

## What "Soft Onboard 5" Means

Not:
- ❌ Marketing push
- ❌ "Beta testing blitz"
- ❌ All Founding 50 at once

Is:
- ✅ Hand-select 5 trusted landholders (co-op members you know)
- ✅ Direct contact: "Help us test this. 30-min walkthrough?"
- ✅ Props + encouragement, not formal testing
- ✅ Observe them use it
- ✅ Capture: Pain points, questions, surprises
- ✅ Let them live with it for 3-5 days
- ✅ Gather feedback via conversation (not surveys)

Duration: 5 days  
Effort: 10-15 hours (yours + theirs)  
Outcome: Real intel for Coordinator meeting

---

## Why This Sequence

**Week 1 (21-28 Feb):**
- ✅ Cadastral endpoints (✓ done)
- ✅ Claim UI flow (✓ done)
- ✅ Claims persistence → Supabase (→ doing now)
- ✅ Stripe integration (→ next)

**Week 2 (1-7 Mar):**
- ✅ Soft onboard 5 real landholders (immediately after Stripe)
- ✅ Monitor + gather feedback
- ✅ 1 March: Coordinator meeting with real proof

**Week 3 (8-14 Mar):**
- Iterate based on feedback
- Decide: More density? UI polish? Real data?

---

## Key Insight

**"Density before polish" = Build for behavior, not aesthetics.**

Aesthetics seduce you into thinking a feature works.  
Behavior shows you if it actually works.

5 real people make your product 100x more real than any design sprint.

---

## Decision Implementation

**After Stripe is live:**

1. Brad picks 5 trusted landholders
2. Emergent prepares demo environment (staging, mobile-optimized)
3. Brad does 30-min walkthroughs with each
4. Brad observes, takes notes
5. Landholders use it for 3-5 days
6. Brad collects feedback via calls/messages
7. Emergent + Brad debrief (Day 6)
8. Decide: More density, polish, or data?

---

## Success Metrics (5-Day Soft Onboard)

- [ ] 5 people sign up + claim property ✓
- [ ] All properties show satellite icon ✓
- [ ] Users understand "30-day trial" ✓
- [ ] No critical bugs (app crashes) ✓
- [ ] Positive sentiment on experience ✓
- [ ] Zero user complaints about 7-screen flow (or specific complaint if real) ✓

If 6/6 met → Ready for broader Founding 50 onboarding.  
If not → Take feedback, iterate, try with 5 more.

---

## Message to Emergent

> After Stripe is integrated and claims are persistent:
> 
> Don't ask "Should we polish UI?"
> 
> Ask "Can we onboard 5 real landholders?"
> 
> Real users talking to real software beats beautiful mockups every time.
> 
> Density before polish.

---

## After 1 March Coordinator Meeting

**Coordinator sees:**
- 12+ real property claims
- Real satellite icons
- Real trial subscriptions running
- Real email addresses
- Real landholders (not staff) using it

**Coordinator reaction:**
- "This is real."
- "How many more can we onboard?"
- "What's next?"

That's the moment everything shifts from "interesting concept" to "working product."

Density.
