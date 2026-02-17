*Design Directives & Implementation Guidance*

Supplementary guidance provided during development, to be read alongside
the core PRD.

# 1. Visual Design: Earth-Tone Palette

## Rationale

Grid overlays should feel like they \"emerged from the land\" --- not
like a heatmap or traffic light system. The palette must remain readable
on satellite imagery without overwhelming the landscape itself.

## Palette Specification (6-step gradient)

  --------------------------------------------------------------------------
  **Score        **Color Name** **Hex Code**   **RGBA Fill**  **Use Case**
  Range**                                                     
  -------------- -------------- -------------- -------------- --------------
  80--100        Rich forest    #2E5A27        rgba(46, 90,   Remnant
                                               39, 0.45)      vegetation
                                                              cores

  65--79         Deep olive     #556B2F        rgba(85, 107,  Strategic
                                               47, 0.42)      ecological
                                                              value

  50--64         Sage green     #6B8E23        rgba(107, 142, High uplift
                                               35, 0.38)      potential

  35--49         Warm amber     #B8860B        rgba(184, 134, Corridor
                                               11, 0.35)      support areas

  20--34         Sandy brown    #C2A478        rgba(194, 164, Lower priority
                                               120, 0.32)     

  0--19          Light tan      #D2B48C        rgba(210, 180, Baseline
                                               140, 0.28)     landscape
  --------------------------------------------------------------------------

## Overlay Rules

• Fill opacity: 28--45% (semi-transparent so satellite layer stays
visible)

• Stroke: Subtle border (1.5px) at \~60-80% opacity for grid edge
legibility

• Stroke color: Same hue as fill, but higher opacity

# 2. Map Implementation: Phased Approach

## Phase 1: Stabilise (Current)

• Render grid polygons as static overlays

• No tap interactions yet

• Focus on: correct centering, polygon render performance, visual
legibility

## Phase 2: Interaction (Next)

• Add onPress handlers to polygons

• Open simplified \"trust-first\" grid detail modal

• Animate transitions smoothly

## Phase 3: Property Claiming

• Connect taps to property claiming flow

• Multi-select for landholders with multiple grids

# 3. Performance Optimisation

## Polygon Rendering

• Memoize polygons: Use React.memo and useMemo to precompute polygon
arrays

• Avoid unnecessary re-renders: Polygon coordinates are static

• Conditional rendering: Only render polygons after onMapReady fires

• Batch state updates: Don\'t trigger re-renders on every map gesture

## Data Loading

• Load grid data once on mount

• Cache boundary/grid data in Zustand store

• Consider pagination for future scale (56 grids now, potentially
hundreds later)

# 4. Platform Strategy

## Native (iOS/Android via Expo Go)

• Primary experience: Full satellite map with interactive polygons

• Use react-native-maps with mapType=\"hybrid\"

• Disable unnecessary map features: compass, scale, rotation, pitch

## Web

• Fallback experience: Not a broken state, but a deliberate redirect

• Show Terra branding, stats summary, sample parcels

• Clear messaging: \"The full map experience is on mobile\"

• Maintain trust-first aesthetic

## Technical Pattern

• Single file with conditional rendering (Platform.OS === \'web\')

• Native-only code isolated in components that web never calls

• Avoid platform-specific file extensions for route files (Metro
bundling issues)

# 5. Iteration Philosophy

## \"Stabilise, then Enhance\"

1\. Get the basic version working and stable

2\. Test on actual device (Expo Go)

3\. Add interactions/enhancements in subsequent iteration

4\. Don\'t bundle complexity into first pass

## User Testing Checkpoints

Before adding complexity, verify:

☐ Does it load correctly?

☐ Is performance acceptable?

☐ Does the visual design match the trust-first aesthetic?

☐ Is it usable by a 60+ year-old landholder?

# 6. UI/UX Principles

## Emotional Hierarchy

**Ownership → Pride → Calm → Curiosity**

## Trust-First Interface Rules

1\. The landscape is the hero --- map fills the screen, UI overlays are
minimal

2\. Progressive disclosure --- start simple, reveal complexity on demand

3\. Familiar patterns --- feel like Google Maps, not a complex dashboard

4\. Calming aesthetic --- earth tones, generous whitespace, no alarm
colors

5\. Friendly language --- \"What you can do\" not \"Available missions\"

## Navigation

• 4 tabs: Explore, Actions, Community, You

• Explore tab: No header (full-screen map)

• Other tabs: Simple, friendly headers

# 7. Backend Data Notes

## Mock Data (MVP)

All ecological scores are currently mocked (deterministic seed for
consistency):

• 56 grid cells generated

• Scores based on distance from center + simulated corridor

• 2 clusters detected automatically

## Future Integration Points

• Replace mock scores with real BPPF model output

• Connect to satellite NDVI data

• Integrate property boundary verification (RP numbers)

# 8. Key Decisions Log

  -----------------------------------------------------------------------
  **Decision**                        **Rationale**
  ----------------------------------- -----------------------------------
  Use react-native-maps over MapLibre Expo Go compatibility; true native
                                      feel without custom dev build

  Earth tones over traffic-light      Feels organic; readable on
  colors                              satellite; avoids alarm aesthetic

  Conditional rendering over platform Metro bundler issues with .web.tsx
  files                               in route files

  Phase 1 = no tap interactions       Stabilise map rendering first; add
                                      interactivity in next iteration

  Web fallback shows stats, not error Every platform should feel
                                      intentional, not broken
  -----------------------------------------------------------------------

Document created: February 2026

*To be updated as new directives emerge during development.*
