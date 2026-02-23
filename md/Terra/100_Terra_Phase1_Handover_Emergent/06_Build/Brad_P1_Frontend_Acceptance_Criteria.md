# P1 – Frontend NDVI Implementation Acceptance Criteria

**From:** Brad  
**To:** Emergent Team  
**Date:** 24 February 2026  
**Status:** P0 NDVI Pipeline Complete ✅ — Proceed to P1  

---

## Overview

P0 (NDVI backend pipeline) is complete and working. ndvi_trend is persisted to Supabase.

**Now building P1:** NDVICard React component + satellite icon on map.

**Phase 1 "done" when P1 acceptance criteria below are met.**

---

## P1 Acceptance Criteria

### 1. NDVICard Component

**Display requirements:**

When a claimed property has `monitoring_state` = `trial_active` or `subscribed`:

```
┌─────────────────────────────────────┐
│  Vegetation health (from satellite) │
│                                     │
│  3-Year Change: Improving (+0.07)   │
│  Current NDVI: 0.73                 │
│                                     │
│  Source: Sentinel-2 satellite       │
│  Last updated: Feb 24, 14:32 UTC    │
└─────────────────────────────────────┘
```

**Exact fields to display:**
- **Title:** "Vegetation health (from satellite)"
- **Trend line:** "`3-year change:` [Improving | Stable | Declining] ([+/−]0.XX)"
  - Use `ndvi_trend` (from DB)
  - Use `ndvi_delta` rounded to 2 decimals
- **Current value:** "`Current NDVI:` 0.XX" (rounded to 2 decimals)
- **Source:** "`Source: Sentinel-2 satellite imagery`"
- **Timestamp:** "`Last updated:` [date/time from ndvi_last_updated column]"

---

### 2. Pending State (While processing NDVI job)

**Shown when:** `ndvi_status = 'pending'` OR `ndvi_status = 'processing'`

**Display:**
- Calm satellite transmit/pulse icon animation (1200–1500ms cycle)
- Copy: **"Analysing satellite imagery for your property…"** (or equivalent landholder-friendly text)
- Show only text + icon (no data fields)
- No interaction needed from user

**Animation spec:**
- Satellite icon pulses gently (not harsh or jarring)
- Duration: 1200–1500ms per cycle
- Repeat indefinitely until status changes

---

### 3. Ready State (Job complete)

**Shown when:** `ndvi_status = 'ready'`

**Display:**
- All fields shown (from section 1 above)
- Brief subtle pulse animation on satellite icon (600–800ms, once or twice)
- Purpose: Draw attention to newly available data

---

### 4. Error State

**Shown when:** `ndvi_status = 'error'`

**Display:**
- Plain-English error message (from `ndvi_error_message` column)
- "Try again" button (requeues NDVI job)
- Example: "Unable to retrieve satellite data. Try again?"

---

### 5. Satellite Icon on Map

**Icon visibility rules:**

| monitoring_state | Icon shown? | Appearance |
|---|---|---|
| `trial_active` | ✅ Yes | Normal opacity |
| `subscribed` | ✅ Yes | Normal opacity |
| `paused` | ✅ Yes | Faded (0.5 opacity) |
| `inactive` | ❌ No | Hidden |

**Animation on status change:**
- When `ndvi_status` transitions to `'ready'`: brief subtle pulse (not aggressive)
- Duration: 500–800ms
- Purpose: Subtle notification that data is now available

---

## Styling Guidelines

**For all P1 components:**

- **Theme:** Light theme (white/cream background, dark text)
- **Typography:** 16px+ minimum for body text, generous spacing
- **Fonts:** System fonts or Tailwind defaults (no custom typefaces)
- **Colors:**
  - Improving: Green (#10b981 or similar)
  - Stable: Gray (#6b7280 or similar)
  - Declining: Orange (#f59e0b or similar)
  - Pending spinner: Calm blue or neutral gray
- **Accessibility:**
  - Sufficient contrast ratios (WCAG AA minimum)
  - No color alone to convey meaning (use text labels too)
  - Alt text for satellite icon
  - Touch targets 44px+ for buttons ("Try again")

---

## Frontend Architecture Notes

### Real-time Updates (Supabase Subscription)

Use Supabase real-time subscriptions to listen for `ndvi_status` changes:

```javascript
const useClaimNDVI = (claimId) => {
  const [ndvi, setNDVI] = useState(null);

  useEffect(() => {
    const subscription = supabase
      .from('property_claims')
      .on('*', (payload) => {
        if (payload.new.id === claimId) {
          setNDVI(payload.new); // Re-render on any change
        }
      })
      .subscribe();

    return () => subscription.unsubscribe();
  }, [claimId]);

  return ndvi;
};
```

When `ndvi_status` changes → UI re-renders automatically (spinner disappears, data appears).

### Component Structure (Suggested)

```
NDVICard.jsx
├── useClaimNDVI(claimId) hook
├── Pending state: SatelliteIcon + "Analysing…"
├── Ready state: Full NDVI data display
├── Error state: Error message + "Try again" button
└── Satellite icon animation (shared)
```

---

## Testing Checklist (Emergent)

Before marking P1 complete:

- [ ] NDVICard appears when claim monitoring_state = trial_active
- [ ] Shows all 5 fields (trend, current, source, timestamp)
- [ ] Pending state: spinner + text visible while job processing
- [ ] Ready state: data appears, brief icon pulse on transition
- [ ] Error state: error message + "Try again" button works
- [ ] Satellite icon visible on map for trial_active/subscribed
- [ ] Satellite icon faded for paused claims
- [ ] No satellite icon for inactive claims
- [ ] Fonts readable on mobile (16px+)
- [ ] Light theme consistent across card
- [ ] "Try again" button requeries job (ndvi_status resets to pending)

---

## Success Criteria (Phase 1 Complete)

✅ NDVICard displays for claimed properties  
✅ All required fields shown (trend, current, source, timestamp)  
✅ Pending animation working (calm satellite pulse)  
✅ Ready state transition animation working  
✅ Error handling + "Try again" action working  
✅ Satellite icon on map visible/faded/hidden based on state  
✅ Accessibility requirements met (16px+ fonts, light theme, contrast)  
✅ Real-time updates working (Supabase subscription)  
✅ Mobile responsive (tested on portrait phone)  

---

## Timeline

- **Feb 24 (Now):** Build NDVICard component (4–6 hours)
- **Feb 25 Morning:** Test + refinement (2–3 hours)
- **Feb 25 Afternoon:** Complete P1 acceptance testing
- **Feb 25 EOD:** Ready for P2 tasks (Stripe, DCDB)

---

## Reference

- **NDVI Data Schema:** [02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md](02_NDVI_Satellite_Monitoring/NDVI_Supabase_Schema_Deployment_Feb23.md)
- **RQ Job Integration:** [Brad_Feb24_Instructions_RQ_Integration.md](Brad_Feb24_Instructions_RQ_Integration.md)
- **P0 Testing Results:** [Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md](Brad_Response_P0_E2E_Testing_P1_Frontend_Design.md)
- **Progress Tracking:** [01_Property_Claim_Flow/Week_1_Progress_Tracking.md](../01_Property_Claim_Flow/Week_1_Progress_Tracking.md)

---

**Status:** Ready for P1 execution (Feb 24)  
**Owner:** Emergent  
**Blocker:** None  
**Next:** P2 Stripe integration (Feb 26–27) after P1 complete

---

**Notes:**

- Keep it simple: card + icon + states. No charts or advanced features in Phase 1.
- Accessibility > visual polish: large fonts, plain language, calm animations matter more than fancy effects.
- Landholder-friendly: Retirees should understand "Improving" without needing to interpret NDVI values.
- Source attribution ("Sentinel-2 satellite") builds trust and transparency.
