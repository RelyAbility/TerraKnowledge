# P0 UI/UX Refinement Specification

**Date:** 25 February 2026  
**Priority:** P0 (Must complete before P2 Stripe integration)  
**Owner:** Emergent  
**Status:** Ready for Implementation  

---

## Overview

This document specifies 4 critical UI/UX refinements to make the Property Claim flow feel "calm and confident" for 60+ year-old landholders. Changes focus on:

- **Full-screen modal** for claim flow (hide map)
- **Human-first language** in NDVICard (lead with meaning, numbers secondary)
- **Accessibility improvements** (16px+ fonts, light theme, high contrast)
- **TBLS score tooltips** (explain each component, highlight highest driver)
- **Satellite icon state machine** (visibility based on monitoring_state)

---

## Priority 1: Full-Screen Claim Flow Modal

### What to Change
Make the PropertyClaimFlow render as a **full-screen modal that completely hides the map** while the user is in the claim journey. This creates a clear, focused experience.

### Files to Modify
- `frontend/src/flows/PropertyClaim/PropertyClaimFlow.tsx`
- `frontend/src/components/TBLSDetailModal.tsx`

### Implementation Details

#### A. PropertyClaimFlow - Add Full-Screen Modal Wrapper

**Current behavior:** PropertyClaimFlow renders screens but map may be visible.

**Change to:** Wrap entire flow in a `Modal` component that covers full screen.

**Code location:** `PropertyClaimFlow.tsx` - wrapping the entire return JSX

**Acceptance Criteria:**
- [ ] When PropertyClaimFlow starts, entire screen is covered by white/light background
- [ ] Navigation back to map is only via "Cancel" button
- [ ] Modal covers SafeAreaView with no gap
- [ ] Works on both iOS and Android

**Reference Implementation:**
```tsx
// Around line 300 in PropertyClaimFlow.tsx
return (
  <Modal
    visible={true}
    animationType="slide"
    transparent={false}
    onRequestClose={onClose}
  >
    <SafeAreaView style={{ flex: 1, backgroundColor: '#FFFFFF' }}>
      {/* Existing flow screens render here */}
    </SafeAreaView>
  </Modal>
);
```

#### B. TBLSDetailModal - Hide Map When Claim Flow Active

**Current behavior:** Map may be visible behind claim flow.

**Change to:** When `activeScreen === 'claim'`, render the PropertyClaimFlow which now has its own full-screen modal.

**Code location:** `TBLSDetailModal.tsx` - around line 360 where claim flow renders

**Acceptance Criteria:**
- [ ] When "This Is My Land" button pressed, entire screen transitions to claim flow
- [ ] No map visible while in claim flow
- [ ] Modal animation is smooth (slide-up recommended)

---

## Priority 2: NDVICard - Human-First Language & Accessibility

### What to Change
Refactor NDVICard to use plain, clear language that a 60+ year-old would understand. Lead with **meaning**, not numbers.

### Files to Modify
- `frontend/src/components/ndvi/NDVICard.tsx`

### Implementation Details

#### A. Update Title & Subtitle (Friendly)

**Current:**
```tsx
<Text style={styles.readyTitle}>Vegetation Health</Text>
<Text style={styles.readySubtitle}>from satellite imagery</Text>
```

**Change to:**
```tsx
<Text style={styles.readyTitle}>Vegetation Health</Text>
<Text style={styles.readySubtitle}>(from satellite imagery)</Text>
```

**Why:** Parentheses make it clear this is supplementary info.

---

#### B. Update Main Value Label (Lead with Meaning)

**Current:**
```tsx
<Text style={styles.mainValueLabel}>Current Health</Text>
<Text style={styles.mainValue}>{formatNDVI(data.ndvi_current)}</Text>
<Text style={styles.mainValuePeriod}>
  {formatDate(data.date_current_start)} — {formatDate(data.date_current_end)}
</Text>
```

**Change to:**
```tsx
<Text style={styles.mainValueLabel}>Your land's vegetation is:</Text>
<Text style={styles.mainValue}>{formatNDVI(data.ndvi_current)}</Text>
<Text style={styles.mainValuePeriod}>
  Measured {formatDate(data.date_current_end)}
</Text>
```

**Why:** 
- "Your land's vegetation is:" speaks to the user directly
- Shows only the most recent date ("Measured..."), not a date range

---

#### C. Update Trend Indicator (Lead with Meaning)

**Current:**
```tsx
<Text style={[styles.trendTitle, { color: trendColor }]}>
  {data.ndvi_trend ? data.ndvi_trend.charAt(0).toUpperCase() + data.ndvi_trend.slice(1) : 'Unknown'}
</Text>
<Text style={styles.trendDelta}>
  {formatDelta(data.ndvi_delta)} over 3 years
</Text>
```

**Change to:**
```tsx
<Text style={[styles.trendTitle, { color: trendColor }]}>
  3-Year Change: {data.ndvi_trend ? data.ndvi_trend.charAt(0).toUpperCase() + data.ndvi_trend.slice(1) : 'Stable'}
</Text>
<Text style={styles.trendDelta}>
  {formatDelta(data.ndvi_delta)} since {formatDate(data.date_baseline_start)}
</Text>
```

**Why:**
- Lead with "3-Year Change:" to make it clear this is a long-term indicator
- Show the baseline date so user knows the comparison period
- Default to "Stable" instead of "Unknown"

---

#### D. Replace Comparison Row with Simple Info Layout

**Current:**
```tsx
{/* Comparison Row */}
<View style={styles.comparisonRow}>
  <View style={styles.comparisonItem}>
    <Text style={styles.comparisonLabel}>3-Year Baseline</Text>
    <Text style={styles.comparisonValue}>{formatNDVI(data.ndvi_baseline)}</Text>
    <Text style={styles.comparisonPeriod}>
      {formatDate(data.date_baseline_start)} — {formatDate(data.date_baseline_end)}
    </Text>
  </View>
  <View style={styles.comparisonDivider} />
  <View style={styles.comparisonItem}>
    <Text style={styles.comparisonLabel}>Change</Text>
    <Text style={[
      styles.comparisonValue,
      { color: trendColor }
    ]}>
      {formatDelta(data.ndvi_delta)}
    </Text>
    <Text style={styles.comparisonPeriod}>since baseline</Text>
  </View>
</View>
```

**Change to:**
```tsx
{/* Detailed Information */}
<View style={styles.detailedInfoContainer}>
  <View style={styles.infoRow}>
    <Text style={styles.infoLabel}>Baseline (3 years ago):</Text>
    <Text style={styles.infoValue}>{formatNDVI(data.ndvi_baseline)}</Text>
  </View>
  <View style={styles.infoRow}>
    <Text style={styles.infoLabel}>Current:</Text>
    <Text style={styles.infoValue}>{formatNDVI(data.ndvi_current)}</Text>
  </View>
  <View style={styles.infoRow}>
    <Text style={styles.infoLabel}>Data source:</Text>
    <Text style={styles.infoValue}>Sentinel-2 satellite</Text>
  </View>
  <View style={styles.infoRow}>
    <Text style={styles.infoLabel}>Last updated:</Text>
    <Text style={styles.infoValue}>{formatDate(data.ndvi_last_updated)}</Text>
  </View>
</View>
```

**Why:**
- Simple label → value pairs are easier to scan
- Plain English labels ("Data source", "Last updated")
- All text is readable (no abbreviations like "WATER" or "CONSERVE")
- Each row clearly separated

---

#### E. Update Footer Text (Plain Language)

**Current:**
```tsx
<Text style={styles.footerText}>
  Data from Sentinel-2 satellite • Updated {data.ndvi_last_updated ? formatDate(data.ndvi_last_updated) : '—'}
</Text>
```

**Change to:**
```tsx
<Text style={styles.footerText}>
  This shows how green your vegetation is. Higher means healthier. Updates monthly.
</Text>
```

**Why:**
- "How green your vegetation is" is plain English that doesn't require knowing what NDVI means
- "Higher means healthier" clearly explains what the numbers mean
- "Updates monthly" sets expectations

---

#### F. Add/Update Styles for Accessibility

**Add these new styles to `StyleSheet.create()`:**

```tsx
// Detailed Information Container
detailedInfoContainer: {
  backgroundColor: '#F9FAFB',
  borderRadius: 12,
  padding: 16,
  marginBottom: 16,
  gap: 12,
},
infoRow: {
  flexDirection: 'row',
  justifyContent: 'space-between',
  alignItems: 'center',
  paddingBottom: 12,
  borderBottomWidth: 1,
  borderBottomColor: '#E5E7EB',
},
infoLabel: {
  fontSize: 16,  // 16px minimum for accessibility
  fontWeight: '500',
  color: '#6B7280',
},
infoValue: {
  fontSize: 16,  // 16px minimum for accessibility
  fontWeight: '600',
  color: '#111827',
},
```

**Update existing styles for accessibility:**

```tsx
// Increase satellite icon size
readySatelliteIcon: {
  width: 44,
  height: 44,
  borderRadius: 22,
  backgroundColor: '#F0FDF4',
  justifyContent: 'center',
  alignItems: 'center',
},

// Icon inside header
// Change from size={24} to size={32} in the JSX

// Update footer for readability
footerText: {
  fontSize: 16,  // Was 12, increase to 16px
  color: '#6B7280',
  lineHeight: 22,
  flex: 1,
},
```

---

#### G. Update Pending State Message

**Current:**
```tsx
<Text style={styles.pendingTitle}>Analysing Satellite Imagery</Text>
<Text style={styles.pendingMessage}>
  Downloading your site's vegetation health from satellite...
</Text>
<Text style={styles.pendingSubtext}>
  This usually takes less than 30 seconds
</Text>
```

**Keep as-is** — this is already good plain language.

---

### Acceptance Criteria for NDVICard

- [ ] All text is 16px or larger
- [ ] No abbreviations (no "NDVI", use "vegetation health")
- [ ] Lead with meaning: "Your land's vegetation is X" instead of "Current Health: X"
- [ ] 3-Year Change section clearly shows baseline date and change amount
- [ ] Footer explains in plain English what the card shows
- [ ] Info rows are readable with good contrast
- [ ] Light theme throughout (white/cream background, dark text)
- [ ] Tested on simulator with both pending and ready states

---

## Priority 3: TBLS Radar Chart & Tooltip System

### What to Change
Add **info buttons** to each score component so users can understand what "Conservation", "Waterways", etc. mean. Fix the "What's Driving Your Score" section to show the **highest-scoring component**, not an arbitrary one.

### Files to Modify
- `frontend/src/components/TBLSRadarChart.tsx`
- `frontend/src/components/TBLSDetailModal.tsx`

### Implementation Details

#### A. Add Tooltip Descriptions (New File)

**Create file:** `frontend/src/components/tbls/TBLSComponentsHelp.ts`

```typescript
/**
 * Landholder-friendly descriptions for TBLS score components
 */

export const TBLS_COMPONENT_HELP = {
  conservation: {
    label: 'Conservation',
    description: 'Native vegetation that remains on your land. Acts as habitat and helps prevent erosion.',
    examples: 'Remnant forest, grassland, or shrubland',
  },
  waterways: {
    label: 'Waterways',
    description: 'Creeks, streams, and the vegetation along them. Healthy waterways support fish and wildlife.',
    examples: 'Riparian zones, creek-side trees',
  },
  wildlife: {
    label: 'Wildlife',
    description: 'Native animals that can live on your land. Includes birds, mammals, reptiles, and insects.',
    examples: 'Habitat corridors for native species',
  },
  connectivity: {
    label: 'Habitat Links',
    description: 'How well your land connects to neighbouring habitats. Animals need to move between patches.',
    examples: 'Corridors linking to nearby nature reserves',
  },
  restoration: {
    label: 'Potential',
    description: 'Opportunity to restore or improve degraded areas. Your land may have unused restoration potential.',
    examples: 'Areas suitable for tree planting or weed removal',
  },
};

export const getComponentHelp = (key: string) => TBLS_COMPONENT_HELP[key as keyof typeof TBLS_COMPONENT_HELP];
```

#### B. Update TBLSRadarChart to Show Info Buttons

**File:** `frontend/src/components/TBLSRadarChart.tsx`

Add info button next to each dimension label:

```tsx
// Around the dimension labels section, add:
import { TouchableOpacity, Modal, Text, View } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { TBLS_COMPONENT_HELP } from './TBLSComponentsHelp';

// Add state management at component level
const [selectedComponent, setSelectedComponent] = useState<string | null>(null);

// Inside DIMENSIONS rendering, wrap label with info button:
{DIMENSIONS.map((dim) => (
  <View key={dim.key} style={{ flexDirection: 'row', alignItems: 'center', gap: 4 }}>
    <Text>{dim.label}</Text>
    <TouchableOpacity 
      onPress={() => setSelectedComponent(dim.key)}
      accessible={true}
      accessibilityLabel={`More information about ${dim.label}`}
      accessibilityRole="button"
    >
      <Ionicons name="information-circle-outline" size={16} color="#9CA3AF" />
    </TouchableOpacity>
  </View>
))}

// Add modal to show help text
{selectedComponent && (
  <Modal visible={true} transparent={true} animationType="fade">
    <View style={{ flex: 1, backgroundColor: 'rgba(0,0,0,0.5)', justifyContent: 'center', padding: 16 }}>
      <View style={{ backgroundColor: '#FFFFFF', borderRadius: 12, padding: 20 }}>
        <Text style={{ fontSize: 18, fontWeight: '600', marginBottom: 12 }}>
          {TBLS_COMPONENT_HELP[selectedComponent as keyof typeof TBLS_COMPONENT_HELP]?.label}
        </Text>
        <Text style={{ fontSize: 16, lineHeight: 24, marginBottom: 12, color: '#6B7280' }}>
          {TBLS_COMPONENT_HELP[selectedComponent as keyof typeof TBLS_COMPONENT_HELP]?.description}
        </Text>
        <Text style={{ fontSize: 14, color: '#9CA3AF', marginBottom: 16 }}>
          {TBLS_COMPONENT_HELP[selectedComponent as keyof typeof TBLS_COMPONENT_HELP]?.examples}
        </Text>
        <TouchableOpacity 
          onPress={() => setSelectedComponent(null)}
          style={{ paddingVertical: 12, paddingHorizontal: 16, backgroundColor: '#10B981', borderRadius: 8 }}
        >
          <Text style={{ color: '#FFFFFF', fontWeight: '600', textAlign: 'center' }}>Got it</Text>
        </TouchableOpacity>
      </View>
    </View>
  </Modal>
)}
```

#### C. Fix "What's Driving Your Score" Logic (TBLSDetailModal)

**File:** `frontend/src/components/TBLSDetailModal.tsx` — around line 250-280

**Current issue:** Shows arbitrary drivers, not the highest-scoring ones.

**Change:** Calculate which components have the highest scores and show those.

```tsx
// Find highest scoring components
const getTopDrivers = (subScores: TBLSSubScores) => {
  const components = [
    { key: 'protection', label: 'Conservation Value', score: subScores.protection },
    { key: 'hydrology', label: 'Waterways', score: subScores.hydrology },
    { key: 'species_gravity', label: 'Wildlife Presence', score: subScores.species_gravity },
    { key: 'connectivity', label: 'Habitat Links', score: subScores.connectivity },
    { key: 'restoration', label: 'Improvement Potential', score: subScores.restoration },
  ];
  
  // Sort by score, return top 3
  return components.sort((a, b) => b.score - a.score).slice(0, 3);
};

// In renderOverviewTab, replace topDrivers section:
{tblsData?.sub_scores && (
  <View style={styles.driversSection}>
    <Text style={styles.sectionTitle}>What's Driving Your Score</Text>
    {getTopDrivers(tblsData.sub_scores).map((driver, idx) => (
      <View key={idx} style={styles.driverRow}>
        <View style={styles.driverHighlight}>
          <Text style={styles.driverScore}>{driver.score.toFixed(0)}</Text>
        </View>
        <View style={styles.driverInfo}>
          <Text style={styles.driverName}>{driver.label}</Text>
          <Text style={styles.driverDescription}>
            {TBLS_COMPONENT_HELP[driver.key]?.description}
          </Text>
        </View>
      </View>
    ))}
  </View>
)}

// Update styles to show scores more prominently
driverHighlight: {
  width: 50,
  height: 50,
  borderRadius: 25,
  backgroundColor: '#10B981',
  justifyContent: 'center',
  alignItems: 'center',
  marginRight: 12,
},
driverScore: {
  fontSize: 18,
  fontWeight: '700',
  color: '#FFFFFF',
},
driverName: {
  fontSize: 16,
  fontWeight: '600',
  color: '#111827',
},
driverDescription: {
  fontSize: 14,
  color: '#6B7280',
  marginTop: 4,
  lineHeight: 20,
},
```

### Acceptance Criteria for TBLS Tooltips

- [ ] Each score component (Conservation, Waterways, Wildlife, Habitat Links, Potential) has an info icon
- [ ] Tapping info icon shows a plain-language explanation
- [ ] Explanation includes what it means and examples
- [ ] "What's Driving Your Score" shows the 3 highest-scoring components, not arbitrary ones
- [ ] Top drivers are clearly ranked by score
- [ ] All text is 16px or larger
- [ ] Light theme, high contrast

---

## Priority 4: Satellite Icon State Machine

### What to Change
The satellite icon on the map should change appearance based on the claim's `monitoring_state`:
- **visible** (full color) when `trial_active` or `subscribed`
- **faded** (0.5 opacity) when `paused`
- **hidden** when `inactive`

### Files to Modify
- `frontend/src/components/NativeMapView.native.tsx` (or `.tsx` depending on platform)
- `frontend/src/stores/claimsStore.ts`

### Implementation Details

#### A. Get Claim's Monitoring State in Map Component

**File:** `frontend/src/components/NativeMapView.native.tsx`

```tsx
// At component level, add:
const claimsStore = useClaimsStore();
const userClaims = claimsStore.userClaims || [];

// When rendering markers for claims, check monitoring_state:
{userClaims.map((claim) => {
  // Determine icon visibility and opacity
  let iconOpacity = 0;
  let isVisible = false;
  
  if (claim.monitoring_state === 'trial_active' || claim.monitoring_state === 'subscribed') {
    iconOpacity = 1;
    isVisible = true;
  } else if (claim.monitoring_state === 'paused') {
    iconOpacity = 0.5;
    isVisible = true;
  }
  // if 'inactive', isVisible stays false
  
  return isVisible ? (
    <Marker
      key={claim.claim_id}
      coordinate={{
        latitude: claim.centroid.lat,
        longitude: claim.centroid.lng,
      }}
      opacity={iconOpacity}
    >
      <View style={{ opacity: iconOpacity }}>
        <Ionicons 
          name="planet" 
          size={32} 
          color={claim.monitoring_state === 'paused' ? '#D1D5DB' : '#16A34A'} 
        />
      </View>
    </Marker>
  ) : null;
})}
```

#### B. Update ClaimsStore Type Definitions

**File:** `frontend/src/stores/claimsStore.ts`

Ensure `monitoring_state` is part of the Claim type:

```typescript
export type MonitoringState = 'inactive' | 'trial_active' | 'subscribed' | 'paused';

export interface Claim {
  claim_id: string;
  user_id: string;
  parcel_id: string;
  centroid: { lat: number; lng: number };
  monitoring_state: MonitoringState;
  created_at: string;
  // ... other fields
}
```

### Acceptance Criteria for Satellite Icon State Machine

- [ ] Icon is visible (full color) for `trial_active` claims
- [ ] Icon is visible (full color) for `subscribed` claims
- [ ] Icon is faded (gray, 0.5 opacity) for `paused` claims
- [ ] Icon is hidden for `inactive` claims
- [ ] State updates when claim monitoring_state changes
- [ ] Works on both iOS and Android
- [ ] No performance issues with multiple claims on map

---

## Testing Checklist (All 4 Priorities)

### P1: Full-Screen Modal
- [ ] Start on map, tap grid, tap "This Is My Land"
- [ ] Claim flow appears as full-screen modal
- [ ] Map is completely hidden
- [ ] No white space around modal (covers SafeAreaView)
- [ ] Can close via back button or cancel
- [ ] Transition is smooth (slide-up animation)
- [ ] Tested on iOS and Android

### P2: NDVICard Language
- [ ] Create a claim and trigger NDVI processing
- [ ] When ready, view NDVICard
- [ ] All text is readable (no small fonts)
- [ ] Leading text makes sense ("Your land's vegetation is:")
- [ ] Numbers are secondary to the meaning
- [ ] 60+ year-old can understand without technical knowledge
- [ ] Light theme is consistent (no dark backgrounds)

### P3: TBLS Tooltips
- [ ] View TBLS grid detail modal
- [ ] Tap info icon on one of the score components
- [ ] Modal appears with explanation
- [ ] Text is clear and uses plain language
- [ ] "What's Driving Your Score" shows highest-scoring components
- [ ] Components are ranked by score number (e.g., Conservation 88 > Waterways 75)

### P4: Satellite Icon State
- [ ] Create claim in trial_active state
- [ ] Satellite icon appears on map (full color and size)
- [ ] Move claim to paused state
- [ ] Icon becomes faded (gray, lower opacity)
- [ ] Move claim to inactive state
- [ ] Icon disappears from map
- [ ] Test with multiple claims in different states

---

## Definition of Done (P0 Complete)

✅ All 4 priorities implemented  
✅ All acceptance criteria met  
✅ Testing checklist passed  
✅ Code review completed  
✅ No console warnings or errors  
✅ Tested on iOS and Android (simulator)  
✅ Ready for P1 E2E testing (NDVI flow with UI)  

---

## Next Steps

Once P0 is complete:
1. **E2E test:** Create claim → trigger NDVI → view polished UI
2. **Screen record:** Record full flow for Land for Wildlife coordinator
3. **Move to P2:** Stripe integration ($20/month, 30-day trial, no card upfront)

---

## Questions for Clarification?

If anything is unclear, ask before starting implementation. This spec is meant to be followed step-by-step.

**Estimated effort:** 4-6 hours for all P0 changes  
**Risk level:** Low (mostly UI/text updates, no backend changes)  
**Testing required:** Medium (requires simulator/emulator or device)
