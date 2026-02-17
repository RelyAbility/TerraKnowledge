# Gondwana / Terra -- Functional Specification (Phase 1)

## 1. User Inputs

• Plot boundary (polygon draw or upload)

• Location selection within 10 km zone

## 2. System Processing

• Retrieve regional ecosystem mapping intersecting plot

• Generate monthly NDVI composite for plot

• Calculate structural heterogeneity index

• Overlay disturbance and connectivity layers

• Match ecosystem to native vegetation preference library

## 3. System Outputs

• Vegetation health indicator (satellite-derived)

• Structural complexity indicator

• Disturbance alerts

• Recommended native vegetation structure (canopy/mid-storey/ground
layer)

• Suggested missions (e.g., mid-storey infill, edge buffering)

## 4. Data Sources

• Sentinel-2 (NDVI/EVI)

• Sentinel-1 (structural proxy where available)

• Landsat archive (historical change)

• VIIRS (fire detection)

• Queensland Regional Ecosystem GIS layers

## 5. Edge Cases

• Multiple RE types within single plot

• Cloud-heavy months

• Small plot below pixel resolution threshold

• Manual ecosystem override by user
