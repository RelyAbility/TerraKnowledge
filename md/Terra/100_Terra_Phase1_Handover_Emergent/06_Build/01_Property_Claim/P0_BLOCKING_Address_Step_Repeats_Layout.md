# P0 BLOCKING: Address Step Repeats + Full-Screen Layout

**Status:** CRITICAL BUG  
**Priority:** P0 (blocks all claim flow testing)  
**Est. Time:** 3–5 hours  
**Blocking:** P1 visual trust loop validation, D1–D3 modal layout fixes

---

## Bug Summary

Two critical navigation + layout issues in PropertyClaimFlow:

1. **Address step repeats** — After selecting address and map zoom, flow returns to AddressSearch instead of advancing to BoundaryConfirm
2. **Full-screen layout broken** — Claim screens don't occupy full height; map visible behind; layout squeezed; keyboard hides CTA button

---

## Reproduction Steps (iOS)

1. Open app, tap **"This Is My Land"**
2. Enter address (e.g., `81 Boscombe Road, Brookfield QLD`)
3. App zooms map to correct tile
4. **Bug occurs:** Flow prompts for address again / returns to AddressSearch step
5. **Bug occurs:** Screen doesn't fill available height; map visible behind claim flow; keyboard hides **"Next"** / **"Find My Property"** button

---

## Expected Behavior

**Navigation Flow (no loops):**
```
AddressSearch 
  → (user confirms address)
BoundaryConfirm 
  → (user confirms boundary/grid)
Registration/Trial 
  → (user enters details or starts trial)
Completion 
  → (claim success, map shows boundary)
```

**One-way flow:** No return to AddressSearch unless user explicitly taps **"Edit address"** / **Back** button.

**Layout:**
- Claim screens occupy full available screen height (safe-area aware)
- Remain full-screen when keyboard opens on any input
- Primary CTA button ("Find My Property" / "Next" / "Confirm") always visible, never hidden by keyboard
- Clean visual separation from map behind

---

## Root Cause Analysis

### Navigation Loop Issue

**Likely cause:** Claim flow step state is being reset after map zoom or modal re-render.

Indicators:
- Step state not persisted in a single source of truth (may be local component state instead of store)
- Navigation effects re-firing on every re-render (useEffect without proper guards)
- Map zoom interaction triggering state reset or triggering address search again

**Example of bad pattern:**
```typescript
// ❌ BAD: Step state local, resets on re-render, effect fires repeatedly
function PropertyClaimFlow() {
  const [step, setStep] = useState('address_search');
  
  useEffect(() => {
    // No guard — fires on every render
    setStep('boundary_confirm');
  }, [selectedAddress]); // Dependency fires multiple times
}
```

### Full-Screen Layout Issue

**Likely cause:**
- Claim screens not wrapped in `SafeAreaView` + `KeyboardAvoidingView`
- Top-level container has fixed height or bottom-sheet constraints
- CTA button not pinned to bottom; positioned inside ScrollView
- Modal or screen doesn't flex to fill available space

**Example of bad pattern:**
```typescript
// ❌ BAD: Fixed heights, CTA inside scrollable content, no KeyboardAvoidingView
function PropertyClaimFlow() {
  return (
    <View style={{ height: 500 }}> {/* Fixed height! */}
      <ScrollView>
        {/* Content + CTA button both scroll */}
        <TouchableOpacity>Next</TouchableOpacity>
      </ScrollView>
    </View>
  );
}
```

---

## Fix Requirements

### 1. Persist Flow State in Zustand Store

**File:** `frontend/src/stores/claimFlowStore.ts` (create if missing)

**Required properties:**
```typescript
interface ClaimFlowState {
  // Current step in flow
  step: 'address_search' | 'boundary_confirm' | 'registration' | 'completion';
  
  // Persist across re-renders
  selectedAddress: string | null;
  selectedGridCell: { x: number; y: number } | null;
  resolvedParcel: GeoJSON.Polygon | null;
  claimDraftId: string | null;
  
  // Actions
  setStep: (step: string) => void;
  setSelectedAddress: (address: string) => void;
  setResolvedParcel: (parcel: GeoJSON.Polygon) => void;
  setSelectedGridCell: (cell: { x: number; y: number }) => void;
  reset: () => void; // Reset after claim completes
}

export const useClaimFlowStore = create<ClaimFlowState>((set) => ({
  step: 'address_search',
  selectedAddress: null,
  selectedGridCell: null,
  resolvedParcel: null,
  claimDraftId: null,
  
  setStep: (step) => set({ step }),
  setSelectedAddress: (address) => set({ selectedAddress: address }),
  setResolvedParcel: (parcel) => set({ resolvedParcel: parcel }),
  setSelectedGridCell: (cell) => set({ selectedGridCell: cell }),
  reset: () => set({
    step: 'address_search',
    selectedAddress: null,
    selectedGridCell: null,
    resolvedParcel: null,
    claimDraftId: null,
  }),
}));
```

### 2. Idempotent Step Transitions

**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

**Pattern 1: Guard against duplicate navigation**
```typescript
function PropertyClaimFlow() {
  const { step, setStep, selectedAddress, resolvedParcel } = useClaimFlowStore();

  // Only auto-advance if data exists AND step hasn't already transitioned
  useEffect(() => {
    if (!selectedAddress) return; // No address selected
    if (step !== 'address_search') return; // Already advanced
    if (!resolvedParcel) return; // No parcel resolved yet
    
    // Safe to advance
    setStep('boundary_confirm');
  }, [selectedAddress, resolvedParcel, step, setStep]);

  return (
    <>
      {step === 'address_search' && <AddressSearchStep />}
      {step === 'boundary_confirm' && <BoundaryConfirmStep />}
      {step === 'registration' && <RegistrationStep />}
      {step === 'completion' && <CompletionStep />}
    </>
  );
}
```

**Pattern 2: Explicit user actions, no auto-advance**
```typescript
function AddressSearchStep() {
  const { setStep, setSelectedAddress, setResolvedParcel } = useClaimFlowStore();
  
  const handleAddressConfirm = async (address: string) => {
    // Resolve address to parcel
    const parcel = await resolveAddressToParcel(address);
    
    // Update store
    setSelectedAddress(address);
    setResolvedParcel(parcel);
    
    // Zoom map (doesn't change step)
    await animateMapToParcel(parcel);
    
    // Manually advance step (not auto-triggered)
    setStep('boundary_confirm');
  };
  
  return <AddressInput onConfirm={handleAddressConfirm} />;
}
```

**Pattern 3: Back button only goes one step**
```typescript
function handleBackButton() {
  const { step, setStep } = useClaimFlowStore();
  
  switch (step) {
    case 'boundary_confirm':
      setStep('address_search');
      break;
    case 'registration':
      setStep('boundary_confirm');
      break;
    case 'completion':
      // Don't go back from completion (or reset flow)
      break;
  }
}
```

### 3. Stop Duplicate Effect Triggers

**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

**Guard all useEffect navigation:**
```typescript
// ✅ GOOD: Well-guarded effect, only fires when needed
useEffect(() => {
  if (!resolvedParcel) return; // No parcel yet
  if (step !== 'address_search') return; // Already advanced
  
  // Only runs once: when parcel resolved AND step is still address_search
  setStep('boundary_confirm');
}, [resolvedParcel, step, setStep]);

// ✅ GOOD: Map zoom is isolated, doesn't trigger navigation
async function animateMapToParcel(parcel: GeoJSON.Polygon) {
  const centroid = calculateCentroid(parcel);
  await mapRef.current?.animateToRegion({
    latitude: centroid[1],
    longitude: centroid[0],
    latitudeDelta: 0.01,
    longitudeDelta: 0.01,
  });
  // Step state unchanged
}
```

### 4. Full-Screen Layout Fix

**File:** `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

**Wrap entire flow in SafeAreaView + KeyboardAvoidingView:**
```typescript
import { SafeAreaView, KeyboardAvoidingView, Platform } from 'react-native';
import { useSafeAreaInsets } from 'react-native-safe-area-context';

export function PropertyClaimFlow() {
  const insets = useSafeAreaInsets();

  return (
    <SafeAreaView style={{ flex: 1 }}>
      <KeyboardAvoidingView 
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
      >
        <View style={styles.container}>
          {/* Scrollable content area */}
          <ScrollView 
            style={styles.content}
            scrollEnabled={true}
            keyboardDismissMode="on-drag"
          >
            {/* Step rendering */}
            {step === 'address_search' && <AddressSearchStep />}
            {step === 'boundary_confirm' && <BoundaryConfirmStep />}
            {/* etc */}
          </ScrollView>

          {/* CTA button - ALWAYS VISIBLE, bottom-pinned */}
          <View style={[styles.ctaContainer, { paddingBottom: insets.bottom }]}>
            <TouchableOpacity style={styles.ctaButton}>
              <Text style={styles.ctaText}>Find My Property</Text>
            </TouchableOpacity>
          </View>
        </View>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'space-between', // Pushes content + CTA apart
  },
  content: {
    flex: 1, // Fills available space
    paddingHorizontal: 16,
    paddingVertical: 12,
  },
  ctaContainer: {
    paddingHorizontal: 16,
    paddingVertical: 12,
    backgroundColor: '#fff',
    borderTopWidth: 1,
    borderTopColor: '#e0e0e0',
  },
  ctaButton: {
    backgroundColor: '#2ecc71',
    paddingVertical: 14,
    borderRadius: 8,
    alignItems: 'center',
  },
  ctaText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '600',
  },
});
```

**Key points:**
- `flex: 1` on root container fills all available height
- `SafeAreaView` handles notch/home indicator
- `KeyboardAvoidingView` lifts content above keyboard
- CTA button outside `ScrollView` (always visible)
- `paddingBottom: insets.bottom` accounts for home indicator

---

## Implementation Checklist

### Navigation State
- [ ] Create `claimFlowStore.ts` with step + address + parcel + gridcell state
- [ ] Update PropertyClaimFlow to use store instead of local state
- [ ] Add guards to useEffect (check step before advancing)
- [ ] Test: Enter address → zoom map → land in BoundaryConfirm (not AddressSearch)
- [ ] Test: Back button returns one step only
- [ ] Test: No duplicate address prompts

### Layout Fixes
- [ ] Wrap PropertyClaimFlow in SafeAreaView + KeyboardAvoidingView
- [ ] Set flex: 1 on root container + ScrollView
- [ ] Pin CTA button to bottom (outside ScrollView)
- [ ] Add insets.bottom padding for home indicator
- [ ] Remove any fixed heights / bottom-sheet constraints
- [ ] Test on small screen (iPhone SE)
- [ ] Test: Keyboard opens → CTA button remains visible
- [ ] Test: Screens fill entire height (no gaps)

### Acceptance Tests
1. **Navigation flow:**
   - [ ] Tap "This Is My Land"
   - [ ] Enter address
   - [ ] Map zooms to tile
   - [ ] **Land in BoundaryConfirm** (not AddressSearch)
   - [ ] Tap "Confirm Boundary"
   - [ ] **Land in Registration**
   - [ ] Back button returns to BoundaryConfirm (one step)

2. **Layout:**
   - [ ] All screens fill full height (no gaps, no map visible behind)
   - [ ] CTA button always visible
   - [ ] Keyboard appears → CTA lifts above keyboard
   - [ ] Works on iPhone SE (small screen)
   - [ ] Works on iPhone 14 Pro Max (large screen)
   - [ ] Works on iPad (portrait/landscape)

3. **No regressions:**
   - [ ] Satellite icon still works (green active, gray paused)
   - [ ] Map zoom animation smooth
   - [ ] NDVI card still visible / functional
   - [ ] Visual trust loop still renders claimed boundaries

---

## Next Steps After Fix

Once this P0 bug is fixed:
1. **D1–D3 Modal Layout Fixes** can be validated properly (already have spec)
2. **Visual Trust Loop** can be re-tested with mock data
3. When Brad provides DCDB data, swap mock → real geometry

---

## Files to Update

1. `frontend/src/stores/claimFlowStore.ts` — Create store
2. `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx` — Navigation + layout fixes
3. `frontend/src/flows/PropertyClaim/AddressSearchStep.tsx` — Use store
4. `frontend/src/flows/PropertyClaim/BoundaryConfirmStep.tsx` — Use store
5. `frontend/src/flows/PropertyClaim/RegistrationStep.tsx` — Use store

---

## Resources

- [Zustand store setup](https://github.com/pmndrs/zustand)
- [KeyboardAvoidingView docs](https://reactnative.dev/docs/keyboardavoidingview)
- [SafeAreaView + react-native-safe-area-context](https://github.com/th3rdEye/react-native-safe-area-context)

---

## Questions?

Tag Brad in Slack if you hit:
- Issues with Zustand store hydration
- Keyboard behavior different on Android
- Map zoom animation conflicts with navigation
