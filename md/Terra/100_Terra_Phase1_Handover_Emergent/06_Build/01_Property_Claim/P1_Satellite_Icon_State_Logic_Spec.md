# P1: Satellite Icon State Logic Specification

## Overview
The satellite icon on the map shows whether the user's property is actively being monitored for vegetation health. It serves as a **visual status indicator** tied strictly to the `monitoring_state` field in the claims table.

This is the final piece of the visual trust loop: parcel boundary shows *what* is being monitored, satellite icon shows *whether* monitoring is happening right now.

## Dependency
**Blocked by:** P1_Parcel_Boundary_Rendering_Spec  
**Unblocks:** P2_Stripe_Integration_Spec

---

## Requirements

### 1. Monitoring States and Icon Behavior

The satellite icon has **4 states** tied to the `monitoring_state` field in the `property_claims` table:

#### 1.1 State Machine

```
┌─────────────────────────────────────────────┐
│   User claims property                       │
│   → monitoring_state = "trial_active"        │
│   → trial_begins = today                     │
│   → trial_ends = today + 30 days             │
└──────────────┬──────────────────────────────┘
               │
               ├─ If user pays before day 30
               │  → monitoring_state = "subscribed"
               │
               └─ If day 30 passes without payment
                  → monitoring_state = "paused"
                  → paused_date = day 30
```

#### 1.2 Icon Display Rules

| monitoring_state | Icon Visible? | Appearance | Interaction |
|------------------|---------------|-----------|-------------|
| **inactive** | ❌ Hidden | N/A | N/A |
| **trial_active** | ✅ Visible | Full color (bright green) | Tappable: shows "Trial active until [date]" |
| **subscribed** | ✅ Visible | Full color (bright green) | Tappable: shows "Fully subscribed" |
| **paused** | ✅ Visible | Faded (50% opacity, gray tint) | Tappable: shows "Trial ended. Upgrade to monitor" |

---

### 2. Frontend Implementation

#### 2.1 NativeMapView Enhancement
**File:** `frontend/src/components/NativeMapView.native.tsx`

Add satellite icon marker conditional on monitoring_state:

```tsx
interface NativeMapViewProps {
  grids: any[];
  onGridTap: (gridId: string) => void;
  mapCenter?: MapCenter | null;
  claimedParcel?: ParcelBoundary | null;
  highlightedGridIds?: string[] | null;
  userClaims?: Array<{
    parcel_id: string;
    monitoring_state: 'inactive' | 'trial_active' | 'subscribed' | 'paused';
    trial_ends?: string;  // ISO date string
  }> | null;  // NEW
}

export function NativeMapView({ 
  grids, 
  onGridTap, 
  mapCenter,
  claimedParcel,
  highlightedGridIds,
  userClaims  // NEW
}: NativeMapViewProps) {
  
  // Compute satellite icon visibility and state from user's claims
  const activeSatelliteIcons = useMemo(() => {
    if (!userClaims) return [];
    
    return userClaims
      .filter(claim => 
        claim.monitoring_state === 'trial_active' ||
        claim.monitoring_state === 'subscribed' ||
        claim.monitoring_state === 'paused'
      )
      .map(claim => ({
        parcel_id: claim.parcel_id,
        state: claim.monitoring_state,
        trial_ends: claim.trial_ends,
      }));
  }, [userClaims]);
  
  // Render satellite icon markers
  const satelliteIcons = useMemo(() => {
    return activeSatelliteIcons.map(icon => {
      // Determine icon appearance based on monitoring_state
      let iconColor: string;
      let iconOpacity: number;
      let label: string;
      
      switch (icon.state) {
        case 'trial_active':
          iconColor = '#16A34A'; // Bright green
          iconOpacity = 1.0;
          const trialDate = new Date(icon.trial_ends!);
          const daysLeft = Math.ceil(
            (trialDate.getTime() - Date.now()) / (1000 * 60 * 60 * 24)
          );
          label = `Trial active (${daysLeft}d left)`;
          break;
        
        case 'subscribed':
          iconColor = '#16A34A'; // Bright green
          iconOpacity = 1.0;
          label = 'Monitoring active';
          break;
        
        case 'paused':
          iconColor = '#6B7280'; // Gray
          iconOpacity = 0.5;
          label = 'Trial ended';
          break;
        
        default:
          return null;
      }
      
      // Find claimed parcel to get coordinates
      const claim = userClaims!.find(c => c.parcel_id === icon.parcel_id);
      if (!claim) return null;
      
      const claimedParcelData = claimedParcel?.parcel_id === icon.parcel_id 
        ? claimedParcel 
        : null;
      
      if (!claimedParcelData) return null;
      
      return (
        <Marker
          key={`satellite-icon-${icon.parcel_id}`}
          coordinate={{
            latitude: claimedParcelData.boundary.centroid_lat || centerLat,
            longitude: claimedParcelData.boundary.centroid_lng || centerLng,
          }}
          anchor={{ x: 0.5, y: 1 }}  // Point at bottom of icon
          tracksViewChanges={false}
          onPress={() => handleSatelliteIconPress(icon.parcel_id, icon.state)}
        >
          <View style={[
            styles.satelliteIcon,
            { 
              backgroundColor: iconColor,
              opacity: iconOpacity,
            }
          ]}>
            <Ionicons 
              name="satellite" 
              size={20} 
              color="#FFFFFF" 
            />
          </View>
        </Marker>
      );
    });
  }, [activeSatelliteIcons, claimedParcel, userClaims]);
  
  // Handle satellite icon tap
  const handleSatelliteIconPress = (parcelId: string, state: string) => {
    const claim = userClaims?.find(c => c.parcel_id === parcelId);
    
    if (state === 'trial_active') {
      showAlert(
        'Vegetation Monitoring Active',
        `Your property is being monitored until ${claim?.trial_ends}. Subscribe to continue monitoring after the trial.`
      );
    } else if (state === 'subscribed') {
      showAlert(
        'Monitoring Active',
        'Your property is fully subscribed. Vegetation health updates continue.'
      );
    } else if (state === 'paused') {
      showAlert(
        'Monitoring Paused',
        'Your trial ended. Upgrade to continue monitoring this property.',
        [
          { text: 'Upgrade Now', onPress: () => navigateToSubscription(parcelId) },
          { text: 'Cancel', onPress: () => {} }
        ]
      );
    }
  };
  
  return (
    <View style={styles.container}>
      <MapView {...mapProps}>
        {/* Grid polygons */}
        {mapReady && gridPolygons}
        
        {/* Parcel boundary */}
        {mapReady && claimedParcelPolygon}
        
        {/* Satellite icons - NEW */}
        {mapReady && satelliteIcons}
      </MapView>
    </View>
  );
}

// Add styles
const styles = StyleSheet.create({
  // ... existing styles ...
  satelliteIcon: {
    width: 40,
    height: 40,
    borderRadius: 20,
    justifyContent: 'center',
    alignItems: 'center',
    borderWidth: 2,
    borderColor: '#FFFFFF',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.3,
    shadowRadius: 4,
    elevation: 5,
  },
});
```

#### 2.2 Claims Store Enhancement
**File:** `frontend/src/stores/claimsStore.ts`

Add user claims to the store with monitoring_state:

```tsx
interface Claim {
  parcel_id: string;
  user_id: string;
  monitoring_state: 'inactive' | 'trial_active' | 'subscribed' | 'paused';
  trial_begins: string;
  trial_ends: string;
  created_at: string;
  // ... other fields
}

interface ClaimsStore {
  claims: Claim[];
  setClaims: (claims: Claim[]) => void;
  getClaim: (parcelId: string) => Claim | undefined;
  getMonitoringState: (parcelId: string) => string;
}

export const useClaimsStore = create<ClaimsStore>((set, get) => ({
  claims: [],
  
  setClaims: (claims) => set({ claims }),
  
  getClaim: (parcelId) => {
    return get().claims.find(c => c.parcel_id === parcelId);
  },
  
  getMonitoringState: (parcelId) => {
    const claim = get().getClaim(parcelId);
    return claim?.monitoring_state || 'inactive';
  },
}));
```

#### 2.3 PropertyClaimScreen Integration
**File:** `frontend/src/screens/PropertyClaimScreen.tsx`

Pass user claims to NativeMapView:

```tsx
export function PropertyClaimScreen() {
  const { claims } = useClaimsStore();
  const [userClaims, setUserClaims] = useState(claims);
  
  // Fetch user's claims on mount
  useEffect(() => {
    const fetchClaims = async () => {
      const response = await fetch(`/api/user/claims`);
      const data = await response.json();
      setUserClaims(data.claims);
    };
    
    fetchClaims();
  }, []);
  
  return (
    <NativeMapView
      grids={grids}
      onGridTap={handleGridTap}
      claimedParcel={selectedParcel}
      userClaims={userClaims}
    />
  );
}
```

---

### 3. Backend Implementation

#### 3.1 Database Schema
**monitoring_state** column already exists in `property_claims` table:

```sql
-- Expected schema
ALTER TABLE property_claims ADD COLUMN monitoring_state VARCHAR(20) DEFAULT 'trial_active';
ALTER TABLE property_claims ADD COLUMN trial_begins TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
ALTER TABLE property_claims ADD COLUMN trial_ends TIMESTAMP;
ALTER TABLE property_claims ADD COLUMN paused_date TIMESTAMP;
```

#### 3.2 API Endpoint: Get User's Claims
**Endpoint:** `GET /api/user/claims`

**Response:**
```json
{
  "user_id": "user_123",
  "claims": [
    {
      "parcel_id": "1234-567",
      "address": "123 Main Street, Brookfield",
      "monitoring_state": "trial_active",
      "trial_begins": "2026-02-25T00:00:00Z",
      "trial_ends": "2026-03-27T00:00:00Z",
      "ndvi": {
        "current": 0.52,
        "baseline": 0.48,
        "delta": 0.04,
        "trend": "improving"
      }
    },
    {
      "parcel_id": "5678-901",
      "address": "456 Oak Street, Brookfield",
      "monitoring_state": "paused",
      "trial_begins": "2026-01-15T00:00:00Z",
      "trial_ends": "2026-02-14T00:00:00Z",
      "paused_date": "2026-02-15T00:00:00Z",
      "ndvi": null
    }
  ]
}
```

---

### 4. Visual Hierarchy on Map

**Satellite Icon Appearance by State:**

```
TRIAL_ACTIVE               SUBSCRIBED               PAUSED
┌──────────────┐          ┌──────────────┐         ┌──────────────┐
│   🛰️         │          │   🛰️         │         │   🛰️         │
│ Bright Green │          │ Bright Green │         │ Gray (50%)   │
│ Full opacity │          │ Full opacity │         │ Faded        │
│ White ring   │          │ White ring   │         │ White ring   │
└──────────────┘          └──────────────┘         └──────────────┘

"Trial active          "Monitoring active"      "Trial ended
(3d left)"                                       Click to upgrade"
```

---

### 5. Testing Checklist

#### Unit Tests
- [ ] `test_satellite_icon_visible_for_trial_active()` - Icon renders when monitoring_state = trial_active
- [ ] `test_satellite_icon_visible_for_subscribed()` - Icon renders when monitoring_state = subscribed
- [ ] `test_satellite_icon_visible_for_paused()` - Icon renders (faded) when monitoring_state = paused
- [ ] `test_satellite_icon_hidden_for_inactive()` - Icon does not render when monitoring_state = inactive
- [ ] `test_icon_color_trial_active()` - Green color applied for trial_active
- [ ] `test_icon_color_paused()` - Gray color applied for paused
- [ ] `test_icon_opacity_paused()` - 50% opacity applied for paused

#### Integration Tests
- [ ] GET /api/user/claims returns user's claims with monitoring_state
- [ ] NativeMapView receives user claims via prop
- [ ] Satellite icon renders at correct parcel coordinate
- [ ] Icon state matches monitoring_state from backend
- [ ] Icon tap shows correct alert message for each state

#### E2E Tests
- [ ] User claims property → icon appears (green, trial_active)
- [ ] Icon shows "Trial active (30d left)"
- [ ] User taps icon → shows trial message + upgrade option
- [ ] Day 30 passes → monitoring_state = paused, icon fades (gray, 50% opacity)
- [ ] User upgrades → monitoring_state = subscribed, icon remains bright green
- [ ] User with paused icon taps it → shows upgrade CTA
- [ ] User with no claims → no satellite icons visible

#### Visual QA
- [ ] Green satellite icon is clearly visible on map
- [ ] Gray icon is noticeably faded on paused state
- [ ] Icon doesn't obscure parcel boundary or grid cells
- [ ] Icon tap area is easy to hit (40x40px)
- [ ] Alert messages are clear and actionable

---

### 6. Success Criteria

- [ ] Satellite icon appears only for active/paused monitoring states
- [ ] Icon is bright green for trial_active and subscribed states
- [ ] Icon is gray with 50% opacity for paused state
- [ ] Icon is hidden for inactive state (no claim or no monitoring)
- [ ] Icon tap displays context-appropriate message
- [ ] Days remaining shown correctly for trial_active state
- [ ] All E2E test cases pass
- [ ] Icon coordinates align with parcel boundary

---

### 7. Files to Modify

| File | Changes |
|------|---------|
| `frontend/src/components/NativeMapView.native.tsx` | Add `userClaims` prop, render satellite icon markers with state-based styling |
| `frontend/src/stores/claimsStore.ts` | Add `monitoring_state` field, add `getMonitoringState()` method |
| `frontend/src/screens/PropertyClaimScreen.tsx` | Fetch user claims, pass to NativeMapView |
| `backend/server.py` | Add `GET /api/user/claims` endpoint |
| Database schema | Ensure `monitoring_state`, `trial_begins`, `trial_ends`, `paused_date` columns exist |

---

### 8. Timeline Estimate
- Frontend icon rendering: **2 hours**
- Backend API endpoint: **1 hour**
- Testing + QA: **1-2 hours**
- **Total: 4-5 hours**

---

## Why This Matters

The satellite icon is the **status indicator** of the trust loop:
- **Green icon = I'm being monitored. My land is being watched from space.**
- **Gray icon = I was monitored, but the trial ended. I can reactivate with a subscription.**

It's a simple, visual way to communicate monitoring state without words. One glance at the map tells the user: "Is my regeneration being measured right now?"

---

## Next Steps (After Satellite Icon Complete)

Once this spec is implemented:
1. Proceed to **P2_Stripe_Integration_Spec** - Full payment lifecycle tied to monitoring states
2. Payment → monitoring_state updates (trial_active at claim creation, paused at day 30 if unpaid)
