# P0 Bug Fix Validation Checklist

**Status:** Testing phase  
**Bugs Fixed:** Navigation loop + full-screen layout  
**Est. Completion:** Today (final validation)

---

## Additional Validation Requirements

### Requirement 1: No "Step Bounce" on Map Zoom / Claim Completion

**What we're testing:** Flow step must NOT revert to AddressSearch due to side effects like map camera animation, re-renders, or modal close/open.

**Acceptance:** After address select, step stays BoundaryConfirm until user explicitly goes back.

**Test Scenarios:**

1. **Map zoom doesn't trigger step revert:**
   - Enter address (e.g., "81 Boscombe Road, Brookfield QLD")
   - Map animates to parcel centroid
   - **Check:** Step is still BoundaryConfirm (not AddressSearch)
   - Tap map multiple times (no interaction should change step)
   - **Check:** Step remains BoundaryConfirm

2. **Modal open/close doesn't trigger step revert:**
   - From BoundaryConfirm, tap any modal control
   - Close any overlays / popovers
   - **Check:** Step remains BoundaryConfirm

3. **Component re-render doesn't trigger step reset:**
   - While viewing BoundaryConfirm, trigger a re-render:
     - Toggle app to background and back (cmd+shift+H on iOS simulator)
     - Rotate device (portrait ↔ landscape)
     - Trigger keyboard appear/dismiss
   - **Check:** Step stays BoundaryConfirm (no bounce back to AddressSearch)

4. **Claim completion resets state correctly (one-way):**
   - Complete the full claim (all steps → Completion)
   - After success, start new claim
   - **Check:** New claim starts fresh at AddressSearch (state was reset)
   - **Check:** Previous claimed parcel is in claimHistory, not current flow

**Expected behavior:** Step transitions are one-directional per user action, never auto-revert due to renders/side effects.

---

### Requirement 2: Persisted State Reset Rules

**What we're testing:** claimFlowStore resets ONLY under specific conditions, not environmental changes.

**Reset SHOULD happen:**
- ✅ Claim completes successfully → new claim starts
- ✅ User taps "Cancel claim" / "Start over" button
- ✅ User logs out / session ends

**Reset should NOT happen:**
- ❌ App resume (backgrounding + reopening)
- ❌ Screen focus change (tab focus in dev tools)
- ❌ Bundle reload (hot reload, Expo reload)
- ❌ Keyboard appear/dismiss
- ❌ Orientation change
- ❌ Map interaction
- ❌ Any NDVI card update

**Test Scenarios:**

1. **App resume doesn't reset state:**
   - Enter address + advance to BoundaryConfirm
   - Background app (cmd+shift+H on simulator, or swipe up on device)
   - Wait 5+ seconds
   - Return to app (restore from background)
   - **Check:** Step is still BoundaryConfirm, address is still selected
   - **Check:** Form field values persist (address, grid selection)

2. **Hot reload doesn't reset state:**
   - In BoundaryConfirm step
   - Trigger hot reload (Expo: Press R)
   - **Check:** Step still BoundaryConfirm (state persisted)
   - **Check:** Zustand store survived reload

3. **User explicitly cancels → state resets:**
   - In BoundaryConfirm step
   - Tap "Cancel claim" or "Start over" button
   - **Check:** State resets to initial (AddressSearch, no address, no parcel)
   - **Check:** Able to start new claim immediately

4. **Claim success → state resets for next claim:**
   - Complete full claim flow (all steps)
   - See Completion screen ("Your claim is being monitored")
   - Tap "Start New Claim" or click away
   - **Check:** New claim starts at AddressSearch (fresh state)
   - **Check:** Previous claim not in current flow (archived in claimsStore.claimed_properties)

---

## Test Device Coverage

**Must test on:**
- [ ] iPhone SE (375pt width, smallest screen)
- [ ] iPhone 14 (standard)
- [ ] iPad (portrait + landscape)

**Should test on:**
- [ ] Android (if available)

---

## Detailed Test Plan

### Phase 1: Step Persistence & No Bounce

**Duration:** ~10 minutes

1. **Step 1: Enter address**
   ```
   Open app
   Tap "This Is My Land"
   Enter "81 Boscombe Road, Brookfield QLD"
   Tap "Find My Property" / Next
   ↓ Map zooms
   ↓ Should land in BoundaryConfirm (not AddressSearch)
   ```

2. **Step 2: Verify no bounce on map interaction**
   ```
   In BoundaryConfirm, tap map multiple times
   Pinch/zoom map
   Pan map around
   ↓ Step should remain BoundaryConfirm
   ✅ NO return to AddressSearch
   ```

3. **Step 3: Verify back button works (one step)**
   ```
   Tap "Back" button
   ↓ Should return to AddressSearch (one step only)
   Verify address field is empty (ready for new input)
   ```

4. **Step 4: Back to BoundaryConfirm again**
   ```
   Re-enter same address
   ↓ Should re-advance to BoundaryConfirm
   (Not stuck on AddressSearch)
   ```

5. **Step 5: Reopen app + verify step persists**
   ```
   While in BoundaryConfirm:
   Close app completely (swipe up / kill process)
   Reopen app
   ↓ Should still be in BoundaryConfirm with same address
   ✅ State survived app close/open
   ```

### Phase 2: State Reset Boundaries

**Duration:** ~10 minutes

1. **Step 1: Background app (doesn't reset)**
   ```
   In BoundaryConfirm, verify selected address visible
   Cmd+Shift+H (simulator) or swipe up (device)
   Wait 5 seconds
   Cmd+Tab back to app (simulator) or swipe left
   ↓ Step should be BoundaryConfirm
   ↓ Address should still be selected
   ✅ State persisted
   ```

2. **Step 2: Hot reload (doesn't reset)**
   ```
   In BoundaryConfirm, note the selected address
   Press R (Expo dev mode) for hot reload
   ↓ Step should be BoundaryConfirm
   ↓ Address should be same
   ✅ Zustand state survived reload
   ```

3. **Step 3: Cancel claim (DOES reset)**
   ```
   In BoundaryConfirm or further step
   Tap "Cancel claim" or "Start over" button
   ↓ Should return to AddressSearch
   ↓ Address field empty
   ✅ State was cleared
   ```

4. **Step 4: Complete claim → reset for next**
   ```
   Go through full flow: AddressSearch → BoundaryConfirm → Registration → Completion
   On Completion screen, tap "Start New Claim" or close
   ↓ Should reset to AddressSearch
   ↓ Previous claim archived (visible in "My Claims" section, not in flow)
   ✅ State reset, ready for next claim
   ```

### Phase 3: Keyboard & CTA Visibility

**Duration:** ~5 minutes

1. **Small screen (iPhone SE - 375pt width):**
   ```
   Open claim flow
   Tap address input
   ↓ Keyboard appears
   ↓ Address field should remain visible
   ↓ "Find My Property" button should remain visible above keyboard
   ✅ No cut-off, no scrolling needed
   ```

2. **Enter long address:**
   ```
   Enter very long address (e.g., "123 Very Long Street Name Brookfield Queensland Australia")
   ↓ Text should wrap or truncate cleanly
   ↓ CTA button still visible
   ✅ No layout breaking
   ```

3. **Keyboard dismiss:**
   ```
   In BoundaryConfirm (no active input)
   Keyboard should be dismissed
   Scroll content, CTA button should follow (always visible)
   ✅ Button never hidden
   ```

---

## Acceptance Criteria (to close P0)

**All of the following must be true:**

- [ ] **Address → BoundaryConfirm is 100% reliable**
  - Step advances after address confirm
  - No bounce-back to AddressSearch on map zoom, re-render, or side effects
  - Back button returns one step only

- [ ] **State persists correctly**
  - App resume doesn't reset state
  - Hot reload doesn't reset state
  - Cancel/Start Over explicitly resets
  - Claim completion resets for next claim

- [ ] **Keyboard/CTA visibility**
  - CTA button always visible (never cut off)
  - Works on small screens (iPhone SE)
  - Address input accessible with keyboard visible

- [ ] **No regressions**
  - Satellite icon still renders
  - Map zoom animation smooth
  - Claimed parcel boundary still renders (visual trust loop)
  - NDVI card functional
  - All previous P0 fixes still work

---

## Report Template

After testing, please report back with:

### Test Summary
- [ ] Device tested: _____ (e.g., iPhone SE, iPhone 14, iPad)
- [ ] Simulator or physical device: _____
- [ ] Expo version: _____ (run `expo --version`)

### Phase 1: Step Persistence & No Bounce
- [ ] Address → BoundaryConfirm advances reliably: ✅ / ❌
- [ ] No step bounce on map zoom: ✅ / ❌
- [ ] Back button one-step: ✅ / ❌
- [ ] App close/reopen preserves step: ✅ / ❌

### Phase 2: State Reset Boundaries
- [ ] App resume preserves state: ✅ / ❌
- [ ] Hot reload preserves state: ✅ / ❌
- [ ] Cancel claim resets state: ✅ / ❌
- [ ] Claim completion resets for next: ✅ / ❌

### Phase 3: Keyboard & CTA
- [ ] CTA visible on small screen keyboard: ✅ / ❌
- [ ] Address input accessible: ✅ / ❌
- [ ] No layout breaking: ✅ / ❌

### Regressions
- [ ] Satellite icon: ✅ / ❌
- [ ] Map zoom animation: ✅ / ❌
- [ ] Claimed boundary rendering: ✅ / ❌
- [ ] NDVI card: ✅ / ❌

### Issues Found (if any)
```
Describe any failures or unexpected behavior:
- ...
- ...
```

### Conclusion
- [ ] **P0 BUG IS FIXED** (all tests pass, ready to move to D1–D3 modal fixes + visual trust loop validation)
- [ ] **ISSUES REMAIN** (list above, needs rework)

---

## Next Steps (After P0 Closes)

Once all acceptance criteria met:

1. ✅ P0 Bug closed (Address step repeats + layout fixed)
2. → Start **D1–D3 Modal Layout Fixes** (3–4 hours)
3. → Re-test **Visual Trust Loop** with mock data
4. → Await DCDB GeoJSON data from Brad
5. → Implement **DCDB Ingestion to Supabase** (7–9 hours)
6. → Swap mock geometry → real cadastral parcels
7. → Then **Stripe Integration** (11–15 hours)

---

## Questions for Emergent

If you're seeing unexpected behavior during testing:

1. **Is claimFlowStore being hydrated from async storage correctly?** (Zustand persistence)
2. **Are there any useEffect dependencies still causing duplicate navigation?** (Check console logs)
3. **Is KeyboardAvoidingView behavior different on real device vs simulator?**
4. **Does overflow: 'hidden' on ScrollView prevent bounce?**

Tag Brad if you hit blockers.
