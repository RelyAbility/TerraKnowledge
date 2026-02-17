# Terra -- Conceptual Data Model (Schema) v1.0

This is a conceptual schema diagram showing the key entities and how
they relate across phases. It is intended to guide database and API
design (not prescribe implementation details).

![](media/image1.png){width="6.8in" height="4.00126312335958in"}

## Key Notes

• Plot → Zone → Portfolio hierarchy is mandatory for future market
activation.

• Tier/Confidence/Elite flags exist from Phase 1, even if dormant in UI.

• Derived metrics are versioned to keep results reproducible across
method upgrades.

• Workflows (briefs, missions, alerts) are driven by derived metrics +
score objects.
