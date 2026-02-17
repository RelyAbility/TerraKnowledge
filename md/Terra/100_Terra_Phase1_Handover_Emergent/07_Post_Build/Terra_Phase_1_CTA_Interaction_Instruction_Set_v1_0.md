# Terra Phase 1 -- CTA Interaction Instruction Set

Version 1.0

## Purpose

Define the functional behaviour, navigation logic, and user experience
for the two primary CTAs within the EPU bottom sheet:\
1. \'See What You Can Do\'\
2. \'This Is My Land\'

## Context

Phase 1 Terra is grid-first (EPU-based), not property-first. All
interactions must reinforce landscape logic while remaining
property-friendly.

## 1. SEE WHAT YOU CAN DO -- Functional Specification

### Objective

Convert ecological insight into practical, location-specific action.

### Trigger Behaviour

On button press:

- • Navigate to Action Recommendations Screen

- • Pass EPU_ID and sub-score drivers

### Action Recommendation Engine (Phase 1 Logic)

Generate top 3 recommended actions based on dominant score drivers.

- Example drivers → example actions:

- Low remnant % → Native regeneration planting guidance

- Riverine proximity → Riparian buffer restoration

- High conservation value → Protect & monitor habitat

### Action Screen Must Display

• EPU_ID

• Dominant ecosystem

• 3 Recommended actions

• Why this matters (1 sentence explanation per action)

• Optional: \'Start 14-Day Monitoring Trial\' CTA

### Non-Scope (Phase 1)

• No AI action planning

• No detailed restoration budget modelling

• No compliance reporting

## 2. THIS IS MY LAND -- Functional Specification

### Objective

Initiate property claim, account creation, and monitoring eligibility.

### Trigger Behaviour

On button press:

- • If not logged in → Navigate to Account Creation Screen

- • If logged in → Attach parcel to user account

### Claim Flow Steps

1\. Confirm property boundary (if available)

2\. Show EPUs intersecting property

3\. Display weighted Nature Opportunity Rating

4\. Confirm claim

5\. Offer Monitoring Trial

### Post-Claim State

• Property tagged to user profile

• Monitoring eligibility enabled

• Quarterly updates scheduled

### Non-Scope (Phase 1)

• No legal verification

• No title validation integration

• No compliance scoring

## 3. UX Requirements

• Buttons must have visible press feedback

• Increase colour contrast for accessibility (WCAG AA)

• Navigation transitions must complete in \<200ms

• Errors must show friendly user-facing messages

## Success Criteria

• User can move from EPU view → Action screen in one tap

• User can claim land in under 60 seconds

• User clearly understands next step
