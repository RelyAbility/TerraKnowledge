# URGENT: P1 Visual Trust Loop - Claimed Property Display & Layout Fixes

## Context
Emergent completed the claim flow, but the visual feedback is missing:
- Claimed property doesn't light up on map
- No parcel boundary render
- No grid highlight for claimed cells
- No monitoring state visual
- Map camera doesn't adjust
- Grid doesn't scale post-selection
- Modal layout issues (not full-height, action button hidden)

**Priority:** Fix visual ownership confirmation BEFORE proceeding to DCDB backend work.

Users need to see: **"I claimed this property. It's highlighted on the map."**

Without this, the trust loop feels broken.

---

## Requirements

### A. Claimed Parcel Boundary Rendering

**What to display:**
- User claims property → PropertyClaimFlow stores claim in database with grid_id + boundary
- Map queries user's claims on mount
- For each claim with monitoring_state = "trial_active" or "subscribed":
  - Render parcel boundary (white stroke, green fill, pulse animation)
  - Show "Your property" label
  - Attach satellite icon at parcel centroid

**Implementation:**

**Step 1: Update Claim Response**
When user confirms claim in PropertyClaimFlow, return boundary geometry:

```json
{
  "status": "success",
  "claim": {
    "claim_id": "claim_123",
    "parcel_id": "1234-567",
    "grid_id": "T_E1_N2",
    "address": "123 Main St, Brookfield",
    "boundary": {
      "type": "Polygon",
      "coordinates": [[[152.915, -27.490], ...]],
      "centroid_lat": -27.4902,
      "centroid_lng": 152.9156
    },
    "monitoring_state": "trial_active",
    "trial_ends": "2026-03-27"
  }
}
```

**Step 2: Update claimsStore**
Add claimed property data to global state:

```tsx
interface Claim {
  claim_id: string;
  parcel_id: string;
  grid_id: string;
  address: string;
  boundary: GeoJSON.Polygon;
  monitoring_state: 'trial_active' | 'subscribed' | 'paused';
}

interface ClaimsStore {
  claimed_properties: Claim[];
  addClaim: (claim: Claim) => void;
  getClaim: (parcel_id: string) => Claim | undefined;
}
```

**Step 3: Update NativeMapView**
Receive claimed properties, render boundaries + icons:

```tsx
interface NativeMapViewProps {
  grids: any[];
  onGridTap: (gridId: string) => void;
  mapCenter?: MapCenter | null;
  claimedProperties?: Claim[] | null;  // NEW
}

export function NativeMapView({ 
  grids, 
  onGridTap, 
  mapCenter,
  claimedProperties  // NEW
}: NativeMapViewProps) {
  
  // Render claimed parcel boundaries
  const claimedBoundaries = useMemo(() => {
    if (!claimedProperties) return [];
    
    return claimedProperties
      .filter(claim => 
        claim.monitoring_state === 'trial_active' || 
        claim.monitoring_state === 'subscribed'
      )
      .map(claim => (
        <Polygon
          key={`claimed-${claim.parcel_id}`}
          coordinates={claim.boundary.coordinates[0].map((coord: number[]) => ({
            latitude: coord[1],
            longitude: coord[0],
          }))}
          fillColor="rgba(34, 197, 94, 0.08)"
          strokeColor="rgba(255, 255, 255, 0.95)"
          strokeWidth={4}
          zIndex={1000}
          tappable={false}
        />
      ));
  }, [claimedProperties]);
  
  // Render satellite icons for claimed properties
  const satelliteIcons = useMemo(() => {
    if (!claimedProperties) return [];
    
    return claimedProperties
      .filter(claim => 
        claim.monitoring_state === 'trial_active' ||
        claim.monitoring_state === 'subscribed' ||
        claim.monitoring_state === 'paused'
      )
      .map(claim => {
        const iconColor = claim.monitoring_state === 'paused' 
          ? '#6B7280' 
          : '#16A34A';
        const opacity = claim.monitoring_state === 'paused' ? 0.5 : 1.0;
        
        return (
          <Marker
            key={`satellite-${claim.parcel_id}`}
            coordinate={{
              latitude: claim.boundary.centroid_lat,
              longitude: claim.boundary.centroid_lng,
            }}
            tracksViewChanges={false}
          >
            <View style={[styles.satelliteIcon, { 
              backgroundColor: iconColor,
              opacity: opacity 
            }]}>
              <Ionicons name="satellite" size={20} color="#FFFFFF" />
            </View>
          </Marker>
        );
      });
  }, [claimedProperties]);
  
  // Render labels
  const claimedLabels = useMemo(() => {
    if (!claimedProperties) return [];
    
    return claimedProperties.map(claim => (
      <Marker
        key={`label-${claim.parcel_id}`}
        coordinate={{
          latitude: claim.boundary.centroid_lat,
          longitude: claim.boundary.centroid_lng,
        }}
        tracksViewChanges={false}
      >
        <View style={styles.propertyLabel}>
          <Text style={styles.propertyLabelText}>Your property</Text>
        </View>
      </Marker>
    ));
  }, [claimedProperties]);
  
  return (
    <MapView {...mapProps}>
      {/* Grid cells */}
      {mapReady && gridPolygons}
      
      {/* Claimed boundaries */}
      {mapReady && claimedBoundaries}
      
      {/* Claimed satellite icons */}
      {mapReady && satelliteIcons}
      
      {/* Labels */}
      {mapReady && claimedLabels}
    </MapView>
  );
}
```

---

### B. Claimed Grid Cell Highlighted

**What to highlight:**
- User claims property in grid T_E1_N2
- That grid cell renders with bright green stroke (2.5px)
- Remains highlighted persistently (not just on selection)

**Implementation:**

```tsx
// In NativeMapView, modify grid rendering logic

const gridPolygons = useMemo(() => {
  return grids.map((grid: any) => {
    const isClaimedByUser = claimedProperties?.some(
      claim => claim.grid_id === grid.grid_id
    );
    
    let strokeWidth = 1.5;
    let strokeColor = 'rgba(255, 255, 255, 0.55)';
    let zIndex = 1;
    
    if (isClaimedByUser) {
      // Claimed cell = bright green highlight
      strokeWidth = 2.5;
      strokeColor = 'rgba(34, 197, 94, 0.9)';
      zIndex = 50;
    } else if (grid.grid_id === selectedGridId) {
      // User tapped this cell
      strokeWidth = 3;
      strokeColor = 'rgba(255, 255, 255, 0.95)';
      zIndex = 100;
    }
    
    return (
      <Polygon
        key={grid.grid_id}
        coordinates={coordinates}
        fillColor={colors.fill}
        strokeColor={strokeColor}
        strokeWidth={strokeWidth}
        zIndex={zIndex}
        onPress={() => handleGridSelect(grid.grid_id)}
      />
    );
  });
}, [grids, selectedGridId, claimedProperties]);
```

---

### C. Map State Refresh Post-Claim

**What to fix:**
- After successful claim, map should update immediately
- Show claimed boundary + highlighted grid
- No manual reload needed
- Camera should zoom to claimed property

**Implementation:**

```tsx
// In PropertyClaimScreen or PropertyClaimFlow

const handleClaimSuccess = async (claim: Claim) => {
  // 1. Update global claims store
  useClaimsStore.setState(state => ({
    claimed_properties: [...state.claimed_properties, claim]
  }));
  
  // 2. Refresh map by passing updated claims
  setClaimedProperties(prev => [...(prev || []), claim]);
  
  // 3. Animate map camera to claimed property
  if (mapRef.current && claim.boundary.centroid_lat) {
    mapRef.current.animateToRegion({
      latitude: claim.boundary.centroid_lat,
      longitude: claim.boundary.centroid_lng,
      latitudeDelta: 0.01,
      longitudeDelta: 0.01,
    }, 800);
  }
  
  // 4. Close modal after brief delay (500ms)
  setTimeout(() => {
    closeModal();
  }, 500);
};
```

---

### D. Modal Layout Corrections

**Current issues:**
- Modal not using full screen height
- Green action button sometimes cut off
- Safe area/keyboard handling incorrect

**Fix checklist:**

#### D1. Modal Component (PropertyClaimFlow.tsx)

```tsx
<Modal 
  visible={true} 
  animationType="slide"
  transparent={false}
>
  <SafeAreaView style={{ flex: 1, backgroundColor: '#000' }}>
    
    {/* Map takes 60% of screen */}
    <View style={{ flex: 0.6 }}>
      <NativeMapView 
        grids={grids}
        onGridTap={handleGridTap}
        claimedProperties={claimedProperties}
      />
    </View>
    
    {/* Confirmation card takes 40%, scrollable */}
    <KeyboardAvoidingView 
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
      style={{ flex: 0.4 }}
    >
      <ScrollView 
        contentContainerStyle={styles.confirmationContainer}
        scrollEnabled={true}
      >
        <Text style={styles.title}>Confirm Your Claim</Text>
        <Text style={styles.address}>{selectedGrid?.address}</Text>
        
        {/* Spacer to push button down */}
        <View style={{ flex: 1 }} />
        
        {/* Green action button - ALWAYS VISIBLE */}
        <Button
          title="Confirm Claim"
          onPress={handleConfirmClaim}
          color="#16A34A"
          style={styles.confirmButton}
        />
        
        <Button
          title="Cancel"
          onPress={handleCancel}
          style={styles.cancelButton}
        />
      </ScrollView>
    </KeyboardAvoidingView>
    
  </SafeAreaView>
</Modal>
```

#### D2. Styles

```tsx
const styles = StyleSheet.create({
  confirmationContainer: {
    flex: 1,
    paddingHorizontal: 16,
    paddingVertical: 12,
    justifyContent: 'space-between',
  },
  title: {
    fontSize: 18,
    fontWeight: '600',
    marginTop: 16,
    marginBottom: 8,
  },
  address: {
    fontSize: 14,
    color: '#9CA3AF',
    marginBottom: 24,
  },
  confirmButton: {
    paddingVertical: 12,
    paddingHorizontal: 16,
    backgroundColor: '#16A34A',
    borderRadius: 8,
    marginBottom: 8,
    minHeight: 44, // Minimum tap target
  },
  cancelButton: {
    paddingVertical: 12,
    backgroundColor: 'transparent',
    borderColor: '#6B7280',
    borderWidth: 1,
    borderRadius: 8,
    minHeight: 44,
  },
});
```

#### D3. Avoid Keyboard Issues

```tsx
// In component initialization
useEffect(() => {
  // Don't let keyboard push content off-screen
  const keyboardShow = Keyboard.addListener(
    Platform.OS === 'ios' ? 'keyboardWillShow' : 'keyboardDidShow',
    () => {
      // Already handled by KeyboardAvoidingView
    }
  );
  
  return () => keyboardShow.remove();
}, []);
```

---

## Testing Checklist

### Visual Confirmation
- [ ] User claims property → boundary appears in white on map
- [ ] Green fill overlay visible (subtle, not dominating)
- [ ] Grid cell for claimed property has green stroke
- [ ] Satellite icon appears at parcel centroid (green if trial/subscribed)
- [ ] "Your property" label shows at centroid
- [ ] Map camera zooms to property automatically

### Layout & UI
- [ ] Modal uses full screen height
- [ ] Confirmation card doesn't overlap action buttons
- [ ] Green "Confirm Claim" button always visible
- [ ] Keyboard doesn't push button off-screen
- [ ] Safe area respected on all devices

### State & Persistence
- [ ] After closing modal, claimed boundary remains visible
- [ ] Claimed grid cell remains highlighted green
- [ ] Multiple claims can be viewed on map
- [ ] Satellite icon updates if monitoring_state changes
- [ ] No manual reload needed

### Edge Cases
- [ ] Multiple parcels claimed → all visible on map
- [ ] Switching between claimed/unclaimed properties works
- [ ] Satellite icon fades when monitoring_state = paused
- [ ] Camera zoom is smooth, not jarring

---

## File Changes Required

| File | Change |
|------|--------|
| `frontend/src/stores/claimsStore.ts` | Add claimed_properties array, expose claimed properties query |
| `frontend/src/components/NativeMapView.native.tsx` | Add claimedProperties prop, render boundaries + icons + highlights |
| `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx` | Update modal layout, add KeyboardAvoidingView, improve spacing |
| `frontend/src/screens/PropertyClaimScreen.tsx` | Pass claimed properties to map, handle map refresh post-claim |
| `backend/server.py` | Ensure claim response includes boundary geometry + centroid |

---

## Timeline

| Task | Hours |
|------|-------|
| Update claimsStore | 0.5 |
| Map rendering (boundaries + icons) | 2-3 |
| Grid cell highlighting | 1 |
| Map camera zoom logic | 1 |
| Modal layout fixes + keyboard | 1.5-2 |
| Testing + refinement | 1-2 |
| **TOTAL** | **7-9.5 hours** |

---

## Success Criteria

- [ ] Claimed property boundary renders immediately on map
- [ ] Parcel boundary is white (4px stroke), green fill (subtle)
- [ ] Claimed grid cell highlights in green (2.5px)
- [ ] Satellite icon appears at parcel centroid
- [ ] Map camera auto-zooms to claimed property
- [ ] No manual reload required
- [ ] Modal uses full screen height
- [ ] Action buttons always visible
- [ ] Keyboard doesn't break layout
- [ ] All 5+ visual tests pass

---

## Why This Matters

**Right now:** User claims property, modal closes, map looks unchanged. **Confusing.**

**After fix:** User claims property, modal zooms map to show their parcel boundary + highlighted grid + satellite icon. **Clear visual ownership.**

This is the emotional anchor of the trust loop. Users need to **see** their land immediately.

---

## Next: DCDB Backend

Once visual trust loop is confirmed working:
1. Integrate DCDB data (Brad provides GeoJSON)
2. Render real parcel boundaries from Supabase PostGIS
3. Replace mock with real cadastral data
4. Then proceed to Stripe integration

Focus: **Visual first. Backend polish second.**
