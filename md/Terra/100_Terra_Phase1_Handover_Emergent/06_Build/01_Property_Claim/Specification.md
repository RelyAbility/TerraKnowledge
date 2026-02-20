# Terra – Property Claim Flow (Phase 1 Final Specification)

**Status:** Ready for Implementation  
**Date:** 21 February 2026  
**Audience:** Emergent (Development Team)  
**Scope:** Phase 1 Property Claiming Process  

---

## Overview

We are updating the property claim process to simplify UX and improve conversion. This spec defines the complete flow: address entry → cadastral matching → property confirmation → satellite trial → map return.

**Goal:** Make property claiming feel calm, simple, and trustworthy for rural Australian landholders (predominantly 60+ age group).

---

## Core Flow

### Step 1: Address-First Input

1. User enters address
2. Query QLD DCDB for cadastral parcel match
3. **If high confidence match:**
   - Auto-zoom to boundary
   - Highlight parcel
   - Show confirmation overlay: "Is this your property?"
   - Buttons: "Yes, confirm" | "No, try again"

---

### Step 2: Remove Map During Registration

Once property confirmed:
- Switch to **full-screen registration mode**
- Hide map completely during claim process
- Use full vertical space
- Light theme only (no dark mode for Phase 1)

---

### Step 3: Fallback Logic (Address Match Failure)

If address match fails:
- Show message: "We couldn't confidently match that address."
- Provide two options:
  1. **Tap property on map** — Return to map, let user click boundary
  2. **Enter RP number** — QLD-specific RP entry (see below)

---

### Step 4: RP Handling (QLD Only)

**Pattern:** RP + 6 digits (e.g., RP123456)

**Entry Flow:**
1. Detect RP pattern automatically
2. Prefill: `RP______`
3. After entry:
   - **If single lot** → Auto-select and confirm
   - **If multiple lots** → Prompt user to select (show lot number + area in m²)

**Important:** RP logic is QLD-specific for Phase 1. Do not implement multi-state fallbacks.

---

### Step 5: Satellite Trial Offer

After claim confirmation, present full-screen:

```
Start 30-Day Satellite Monitoring

We'll track vegetation health every 10 days using satellite data.
See how your property is doing.

30-day free trial. No card required.
After 30 days: $20/month. Cancel anytime.

[Start Free 30-Day Trial]
[Register Without Monitoring]
```

**Pricing Transparency:**
- Trial: Free
- Ongoing: $20/month (Phase 1 pilot pricing)
- Stripe subscription created at Day 30
- No card required upfront
- On payment failure → Monitoring pauses (no auto-cancel)

---

### Step 6: Post-Completion Behaviour

Return user to map:
1. **Boundary visible** — White outline around claimed property
2. **If trial started** → Satellite icon activated (colour/lit state)
3. **If no trial** → No satellite icon yet
4. **Multiple claims** → All boundaries visible with their respective icon states

---

## Satellite Data Activation

**Icon Activation Timing:**
- Activates **immediately** upon trial start
- NDVI baseline loads immediately using pre-processed data where available
- If deeper processing required: Show "Refining data" state (max 2-3 minutes)

**Map Display:**
- Once processed, satellite baseline appears alongside icon
- Icon colour indicates subscription status:
  - 🟢 Active trial/subscription
  - 🔘 Trial expired (greyed out)
  - None: Property claimed but no monitoring

---

## Claim Management Post-Completion

### Viewing Claims
- Users see all their claimed properties on map
- Boundaries persist indefinitely (or until user removes)
- Icon status shows subscription state

### Removing a Claim
- Users can remove any claimed property
- **Boundary removed from public view**
- **Historical data retained internally** for records/reporting

### Multiple Properties
- Users may claim multiple properties in Phase 1
- Each has independent trial/subscription status
- Each boundary visible simultaneously

---

## UX Requirements

**Typography & Layout:**
- ✅ Increase font sizes across entire claim flow (target 16px+ for body)
- ✅ Remove dark theme (light only for Phase 1)
- ✅ Use 1.5x line-height for clarity
- ✅ Full-width buttons (touch-friendly on mobile)

**Copywriting:**
- ✅ Reduce technical language
- ✅ Use plain English (avoid "cadastral," "parcel," "NDVI baseline")
- ✅ One primary CTA per screen
- ✅ Explain why we need information (e.g., "So we can track your property on satellite")

**Example Phrases:**
- Instead of: "Cadastral parcel match"
  - Use: "Finding your property..."
- Instead of: "NDVI monitoring activated"
  - Use: "Satellite tracking is now on"
- Instead of: "Monitoring paused"
  - Use: "We paused tracking. Reactivate anytime."

---

## Phase 1 Scope Constraints

**✅ Included:**
- Address → QLD DCDB match
- RP fallback (QLD only)
- Property confirmation overlay
- Registration mode (map hidden)
- Satellite icon + baseline reveal
- Stripe $20/month trial
- Claim removal
- Multiple properties per user

**❌ Not Included (Post-Phase 1):**
- Soil data integration
- Acoustic monitoring
- Multi-state RP support
- Dark theme
- Advanced filtering/analytics
- Historical monitoring graphs
- Family/group sharing

---

## Technical Checklist for Emergent

- [ ] QLD DCDB integration tested
- [ ] RP pattern detection and validation
- [ ] Cadastral boundary GeoJSON retrieval
- [ ] Address geocoding (high-confidence threshold defined)
- [ ] Full-screen registration UI (light theme)
- [ ] Satellite icon state management (active/expired/none)
- [ ] Stripe subscription logic at Day 30
- [ ] Payment failure handling (pause, don't cancel)
- [ ] Pre-processed NDVI baseline retrieval
- [ ] "Refining data" loading state
- [ ] Claim removal / boundary de-publication
- [ ] Multi-property support on map
- [ ] Font sizing audit (target 16px+ minimum)
- [ ] Copy review for plain language

---

## Success Criteria (Phase 1)

✅ 10–15 properties claimed within 4.2 km  
✅ Satellite icon visible immediately after trial start  
✅ 30-day trial → Stripe conversion at Day 30  
✅ Zero failed map renders  
✅ Claim process completes in <3 minutes  
✅ Support requests about claim process <5% of total  

---

## Release Gate

Do not release to Founding 50 until:
1. Full claim flow tested end-to-end
2. Stripe trial-to-paid conversion tested
3. Satellite baseline loads within 3 minutes
4. Font sizing audit passed
5. Copy approved by team

---

## Contact

Questions? Reach out directly. Do not interpret or extend this spec.
