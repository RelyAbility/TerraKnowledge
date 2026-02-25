# Emergent: P0 Implementation Fixes

**Date:** 25 February 2026  
**Priority Order:** Critical → High → Medium  
**Owner:** Emergent  
**Status:** Ready to Implement  

---

## Overview

Based on Emergent's code review, 3 critical fixes needed before E2E testing:

1. ✅ **Full-screen PropertyClaimFlow modal** (highest visibility issue)
2. ✅ **Satellite Icon on Map** (critical P1 feature)
3. ✅ **NDVICard refinements** (exact wording + error handling)

**Note:** Deferring real-time Supabase subscriptions; 5s polling is acceptable for Phase 1.

---

## FIX #1: Full-Screen PropertyClaimFlow Modal

### Issue
PropertyClaimFlow doesn't wrap in Modal, so map may be visible behind claim flow screens. Should be full-screen (white background, covers entire view).

### File to Modify
`frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`

### Implementation

**Find this section (around line 300-350):**
```tsx
export const PropertyClaimFlow: React.FC<PropertyClaimFlowProps> = ({
  onClose,
  onClaimComplete,
  initialAddress,
  initialLocation,
  isAuthenticated: initialAuth,
  userId: initialUserId,
}) => {
  // ... state setup ...
  
  return (
    <SafeAreaView style={{ flex: 1, backgroundColor: '#FFFFFF' }}>
      {/* Current screens render here */}
    </SafeAreaView>
  );
};
```

**Replace the return statement with:**
```tsx
  return (
    <Modal
      visible={true}
      animationType="slide"
      transparent={false}
      hardwareAccelerated={true}
      onRequestClose={onClose}
    >
      <SafeAreaView style={{ flex: 1, backgroundColor: '#FFFFFF' }}>
        {/* All existing screen rendering code stays the same */}
        {showAuthModal && (
          <AuthModal
            onSuccess={handleAuthSuccess}
            onClose={() => setShowAuthModal(false)}
          />
        )}

        {!showAuthModal && (
          <>
            {currentScreen === 'address_search' && (
              <AddressSearchScreen
                onSelect={handleAddressSelect}
                onCancel={onClose}
              />
            )}
            {/* ... rest of screens ... */}
          </>
        )}
      </SafeAreaView>
    </Modal>
  );
```

### Checklist
- [ ] Import `Modal` from `react-native`
- [ ] Wrap entire SafeAreaView in `<Modal>`
- [ ] Set `animationType="slide"` for smooth entry
- [ ] Set `transparent={false}` so background is solid white
- [ ] Keep `onRequestClose={onClose}` to handle back button
- [ ] Test: Start on map → tap grid → tap "This Is My Land" → should see full-screen claim flow
- [ ] Verify: No map visible behind modal
- [ ] Test: Back button closes modal, returns to map

---

## FIX #2: Satellite Icon on Map

### Issue
Satellite icon is not implemented at all. Should appear on map based on claim's `monitoring_state`:
- **visible** (full color) when `trial_active` OR `subscribed`
- **faded** (0.5 opacity, gray color) when `paused`
- **hidden** when `inactive`

### Files to Modify

#### A. Define MonitoringState Type (if not already done)

**File:** `frontend/src/stores/claimsStore.ts`

**Verify this type exists:**
```typescript
export type MonitoringState = 'inactive' | 'trial_active' | 'subscribed' | 'paused';

export interface Claim {
  claim_id: string;
  user_id: string;
  parcel_id: string;
  centroid: { lat: number; lng: number };
  monitoring_state: MonitoringState;
  // ... other fields ...
}
```

If `MonitoringState` type doesn't exist, add it.

#### B. Update NativeMapView Component

**File:** `frontend/src/components/NativeMapView.native.tsx` (or `NativeMapView.tsx` depending on your setup)

**Find the section where map markers are rendered for claims (look for `userClaims` or similar):**

**Search for something like:**
```tsx
{userClaims?.map((claim) => (
  <Marker key={claim.claim_id} ...>
```

**Replace or add this logic:**
```tsx
import { useClaimsStore } from '../stores/claimsStore';

// Inside component
const claimsStore = useClaimsStore();
const userClaims = claimsStore.userClaims || [];

// When rendering markers, add this filter and styling:
{userClaims.map((claim) => {
  // Determine if icon should be visible and what opacity/color
  let isVisible = true;
  let iconOpacity = 1;
  let iconColor = '#16A34A'; // Green for active
  
  if (claim.monitoring_state === 'trial_active' || claim.monitoring_state === 'subscribed') {
    isVisible = true;
    iconOpacity = 1;
    iconColor = '#16A34A'; // Full green
  } else if (claim.monitoring_state === 'paused') {
    isVisible = true;
    iconOpacity = 0.5;
    iconColor = '#9CA3AF'; // Gray, faded
  } else if (claim.monitoring_state === 'inactive') {
    isVisible = false; // Don't render marker at all
  }

  if (!isVisible) return null; // Skip rendering for inactive

  return (
    <Marker
      key={claim.claim_id}
      coordinate={{
        latitude: claim.centroid.lat,
        longitude: claim.centroid.lng,
      }}
      title={claim.property_name || 'Your Land'}
      description={claim.monitoring_state}
      opacity={iconOpacity}
      onPress={() => handleMarkerPress(claim)}
    >
      <View style={{ opacity: iconOpacity }}>
        <Ionicons 
          name="planet" 
          size={32} 
          color={iconColor}
        />
      </View>
    </Marker>
  );
})}
```

### Checklist
- [ ] Import `useClaimsStore` in NativeMapView
- [ ] Add conditional logic for icon visibility based on `monitoring_state`
- [ ] Full color (#16A34A green) for active claims
- [ ] Gray (#9CA3AF) + 0.5 opacity for paused claims
- [ ] Hidden (not rendered) for inactive claims
- [ ] Test: Create claim in trial_active state → satellite icon visible
- [ ] Test: Change claim to paused state → icon becomes faded gray
- [ ] Test: Change claim to inactive → icon disappears
- [ ] Test with multiple claims in different states

---

## FIX #3: NDVICard Refinements

### Issue A: Exact Title Wording

**File:** `frontend/src/components/ndvi/NDVICard.tsx`

**Find (around line 210):**
```tsx
<Text style={styles.readyTitle}>Vegetation Health</Text>
<Text style={styles.readySubtitle}>from satellite imagery</Text>
```

**Change to:**
```tsx
<Text style={styles.readyTitle}>Vegetation health</Text>
<Text style={styles.readySubtitle}>(from satellite imagery)</Text>
```

**Why:** Brad's spec says exactly "Vegetation health (from satellite)" not "Vegetation Health from..."

### Issue B: Add "Try Again" Button to Error State

**File:** `frontend/src/components/ndvi/NDVICard.tsx`

**Find the ErrorState component (around line 150-160):**
```tsx
const ErrorState: React.FC<{ message: string | null }> = ({ message }) => (
  <View 
    style={styles.errorContainer}
    accessibilityLabel={`Error loading vegetation data: ${message || 'Unknown error'}`}
    accessibilityRole="alert"
  >
    <View style={styles.errorIconContainer}>
      <Ionicons name="alert-circle" size={48} color="#DC2626" />
    </View>
    <Text style={styles.errorTitle}>Unable to Load Data</Text>
    <Text style={styles.errorMessage}>
      {message || 'There was a problem fetching vegetation health data.'}
    </Text>
    <Text style={styles.errorSubtext}>
      We'll retry automatically. If this persists, please contact support.
    </Text>
  </View>
);
```

**Add "Try Again" button:**
```tsx
const ErrorState: React.FC<{ message: string | null; onRetry?: () => void }> = ({ message, onRetry }) => (
  <View 
    style={styles.errorContainer}
    accessibilityLabel={`Error loading vegetation data: ${message || 'Unknown error'}`}
    accessibilityRole="alert"
  >
    <View style={styles.errorIconContainer}>
      <Ionicons name="alert-circle" size={48} color="#DC2626" />
    </View>
    <Text style={styles.errorTitle}>Unable to Load Data</Text>
    <Text style={styles.errorMessage}>
      {message || 'There was a problem fetching vegetation health data.'}
    </Text>
    <Text style={styles.errorSubtext}>
      We'll retry automatically. If this persists, please try again or contact support.
    </Text>
    
    {/* Add Try Again Button */}
    {onRetry && (
      <TouchableOpacity 
        style={styles.tryAgainButton}
        onPress={onRetry}
        accessible={true}
        accessibilityLabel="Try Again"
        accessibilityRole="button"
      >
        <Text style={styles.tryAgainButtonText}>Try Again</Text>
      </TouchableOpacity>
    )}
  </View>
);
```

**Add styles for button:**
```tsx
tryAgainButton: {
  marginTop: 16,
  paddingVertical: 12,
  paddingHorizontal: 24,
  backgroundColor: '#16A34A',
  borderRadius: 8,
  minWidth: 120,
  alignItems: 'center',
},
tryAgainButtonText: {
  fontSize: 16,
  fontWeight: '600',
  color: '#FFFFFF',
},
```

**Update NDVICard to pass onRetry:**

**Find where ErrorState is called (around line 300):**
```tsx
if (data.ndvi_status === 'error') {
  return (
    <View style={styles.card} data-testid="ndvi-card-error">
      <ErrorState message={data.ndvi_error_message || null} onRetry={onRetry} />
    </View>
  );
}
```

**Need to add onRetry function:**
```tsx
// Add prop to NDVICardProps interface
interface NDVICardProps {
  data: NDVIData | null;
  loading?: boolean;
  propertyName?: string;
  onRetry?: () => void; // Add this
}

// In NDVICard component, use it:
const { data, loading = false, propertyName, onRetry } = props;

// Then pass it to ErrorState:
<ErrorState message={data.ndvi_error_message || null} onRetry={onRetry} />
```

### Issue C: Improve Timestamp Format

**File:** `frontend/src/components/ndvi/NDVICard.tsx`

**Find the formatDate function (around line 40):**
```tsx
function formatDate(dateStr: string | null): string {
  if (!dateStr) return '—';
  try {
    const date = new Date(dateStr);
    return date.toLocaleDateString('en-AU', { month: 'short', year: 'numeric' });
  } catch {
    return '—';
  }
}
```

**Update to include time:**
```tsx
function formatDate(dateStr: string | null, includeTime: boolean = false): string {
  if (!dateStr) return '—';
  try {
    const date = new Date(dateStr);
    if (includeTime) {
      return date.toLocaleDateString('en-AU', { 
        month: 'short', 
        day: 'numeric',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        timeZone: 'UTC'
      });
    }
    return date.toLocaleDateString('en-AU', { month: 'short', year: 'numeric' });
  } catch {
    return '—';
  }
}
```

**Update the timestamp display in ReadyState (around line 310):**
```tsx
// Find this line:
<Text style={styles.footerText}>
  Data from Sentinel-2 satellite • Updated {data.ndvi_last_updated ? formatDate(data.ndvi_last_updated) : '—'}
</Text>

// Change to:
<Text style={styles.footerText}>
  Data from Sentinel-2 satellite • Updated {data.ndvi_last_updated ? formatDate(data.ndvi_last_updated, true) : '—'}
</Text>
```

### Checklist
- [ ] Title updated to "Vegetation health" (lowercase)
- [ ] Subtitle wrapped in parentheses: "(from satellite imagery)"
- [ ] "Try Again" button added to error state
- [ ] Button is green (#16A34A), 16px+ font, 44px+ tap target
- [ ] onRetry callback properly wired (should reset ndvi_status to 'pending')
- [ ] Timestamp format shows full date + time (e.g., "Feb 24, 14:32 UTC")
- [ ] Test error state manually: set `ndvi_status = 'error'` and verify button appears
- [ ] Tap "Try Again" → job should requeue (ndvi_status resets to 'pending')

---

## Testing Steps (Complete)

### 1. Full-Screen Modal Test
```
1. Start app on map screen
2. Tap a grid cell
3. Tap "This Is My Land" CTA button
4. Verify: Entire screen is white, no map visible
5. Fill address → boundary → registration
6. Verify: Each screen is full-screen with white background
7. Tap back button → should return to map
```

### 2. Satellite Icon on Map Test
```
1. Create a new claim in trial_active state
2. Refresh map
3. Verify: Satellite icon appears in green (#16A34A) at full opacity
4. Update claim monitoring_state to 'paused' (via backend or manual state change)
5. Refresh map
6. Verify: Icon becomes faded (gray, 0.5 opacity)
7. Update to 'inactive'
8. Verify: Icon disappears entirely
9. Test with multiple claims in different states
```

### 3. NDVICard Refinements Test
```
1. Trigger NDVI processing for a claim
2. While processing: Verify pending state shows "Analysing Satellite Imagery"
3. When ready: Verify title shows "Vegetation health (from satellite)"
4. Verify timestamp shows format like "Feb 24, 14:32 UTC" (with time)
5. Simulate error: Set ndvi_status = 'error' manually
6. Verify: Error message + "Try Again" button visible
7. Tap "Try Again"
8. Verify: Job reprocesses (ndvi_status → 'pending' → processing → ready)
```

---

## Commit & Push

When complete:
```bash
git add frontend/src/
git commit -m "feat: P0 UI polish - full-screen modal, satellite icons, NDVICard refinements"
git push origin main
```

---

## Success Criteria

✅ Full-screen PropertyClaimFlow modal implemented  
✅ Satellite icon visible/faded/hidden based on monitoring_state  
✅ NDVICard title & subtitle match exact spec  
✅ "Try Again" button in error state  
✅ Timestamp includes date + time  
✅ All tests pass  
✅ Ready for E2E test  

---

## Questions?

If any step is unclear or you hit a blocker, ask immediately. Don't guess or wait.

**Timeline:** These fixes should take 2-3 hours total.

**Next:** Once complete and tested, we run full E2E test, then move to P2 Stripe integration.
