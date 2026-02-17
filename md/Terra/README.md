# Terra Documentation (Source of Truth)

This directory is the **authoritative, version-controlled knowledge base** for the Terra program.

It exists to make Terra’s intent, decisions, specifications, and delivery plan easy to:
- review (clean diffs, clear history)
- collaborate on (standard Markdown workflows)
- operationalise (turn narrative into requirements, architecture, and implementation)

## Purpose

Terra documentation is written to align stakeholders and teams on:
- why Terra exists and what success looks like
- how the program is governed and secured
- what the Terra app must do (and why)
- how data is modelled, processed, and integrated
- how the system is engineered and evolved over time

## What you’ll find here

Rather than being a single document, this knowledge base captures Terra from multiple angles:

### Strategy and decision context
Executive summaries, deployment framing, cost/performance envelopes, and structured risk thinking that inform downstream product and technical choices.

Key documents:
- [Executive summary](00_Strategy_&_Vision/HHI_Executive_Summary_v1_0.md)
- [Corridor-first deployment (Founding 50)](00_Strategy_&_Vision/Phase_1_Corridor_First_Deployment_Founding_50_centered_on_Gondwana.md)
- [Founder/board strategic summary](00_Strategy_&_Vision/Terra_Founder_Board_Strategic_Summary_v1_0.md)
- [Performance envelope and cost model](00_Strategy_&_Vision/Terra_Performance_Envelope_and_Cost_Model_v1_0.md)
- [Structured risk review](00_Strategy_&_Vision/Terra_Structured_Risk_Review_v1_0.md)

### Governance and standards
The participation model, constitutional/gov packs, pillar definitions, alignment addenda, and the security/governance spec that constrain how the platform is built and operated.

Key documents:
- [Governance & participation framework (Founding 50)](01_Governance_&_Standards/Founding%2050%20Ecological%20Observatory%20%20Governance%20%26%20Participation%20Framework%20%28v1.0%29.md)
- [HHI Constitution](01_Governance_&_Standards/HHI_Constitution_v1_0.md)
- [HHI Governance Pack](01_Governance_&_Standards/HHI_Governance_Pack_v1_0.md)
- [NRM alignment addendum](01_Governance_&_Standards/HHI_NRM_Alignment_Addendum_v1_0.md)
- [Pillar definitions](01_Governance_&_Standards/HHI_Pillar_Definitions_v1_0.md)
- [Structural addendum](01_Governance_&_Standards/HHI_Structural_Addendum_v1_0.md)
- [Security and governance spec](01_Governance_&_Standards/Terra_Security_and_Governance_Spec_v1_0.md)

### Product intent (Terra app)
Functional requirements, UX concepts, and roadmap artefacts that translate strategy into user-facing outcomes.

Key documents:
- [Phase 1 objective](02_Product_Terra_App/02.1%20Product%20Requirements/Phase1_Objective_v1_0.md)
- [Functional spec (Phase 1, active)](02_Product_Terra_App/02.1%20Product%20Requirements/Terra_Functional_Spec_Phase1_v1_1_ACTIVE.md)
- [Product requirements (Phase 1, active)](02_Product_Terra_App/02.1%20Product%20Requirements/Terra_Product_Requirements_Phase1_v1_2_ACTIVE.md)
- [App concept (Phase 1)](02_Product_Terra_App/02.2%20UX%20%26%20Concepts/Gondwana_Terra_Phase_1_App_Concept_For_Review.md)
- [Older versions (requirements/specs)](02_Product_Terra_App/02.1%20Product%20Requirements/Older_versions/)

### Data and algorithms
Algorithm and processing specifications, data model work, and external integration notes (including geospatial) that define how Terra converts inputs into usable outputs.

Key documents:
- [Algorithm and data processing spec (v1.0)](03_Data_&_Algorithms/Terra_Algorithm_and_Data_Processing_Spec_v1_0.md)
- [Algorithm and data processing addendum (v1.0)](03_Data_&_Algorithms/Terra_Algorithm_and_Data_Processing_Spec_v1_0_Addendum.md)
- [Algorithm and data processing spec (v1.1 consolidated)](03_Data_&_Algorithms/Terra_Algorithm_and_Data_Processing_Spec_v1_1_Consolidated.md)
- [API contract spec](03_Data_&_Algorithms/03.2%20Data%20Model/Terra_API_Contract_Spec_v1_0.md)
- [Conceptual data model](03_Data_&_Algorithms/03.2%20Data%20Model/Terra_Conceptual_Data_Model_v1_0.md)
- [Database schema draft](03_Data_&_Algorithms/03.2%20Data%20Model/Terra_Database_Schema_Draft_v1_0.md)
- [Cadastral RP integration addendum](03_Data_&_Algorithms/03.3%20External%20Data%20Integrations/Geospatial%20Integration/Terra_Cadastral_RP_Integration_Addendum_v1_0.md)

### Technical architecture and engineering guidance
Reference architecture, engineering packs, and an evolution framework to keep implementation coherent as capabilities expand.

Key documents:
- [HHI Technical Architecture](04_Technical_Architecture/HHI_Technical_Architecture_v1_0.md)
- [Terra Engineering Pack](04_Technical_Architecture/Terra_Engineering_Pack_v1_0.md)
- [Strategic architecture evolution framework](04_Technical_Architecture/Terra_Strategic_Architecture_Evolution_Framework_v1_0.md)
- [NDVI method (v1.0)](04_Technical_Architecture/04.01_Methodology/NDVI_Method_v1_0.md)

### Delivery, handover, and operations
Phase 1 handover materials intended to support build completion, transition, and post-build operational clarity.

Key documents:
- [Phase 1 handover dossier](100_Terra_Phase1_Handover_Emergent/06_Build/Terra_Phase_1_Handover_Dossier.md)
- [Technical implementation brief](100_Terra_Phase1_Handover_Emergent/06_Build/Terra_Technical_Implementation_Brief.md)
- [Delivery milestone plan](100_Terra_Phase1_Handover_Emergent/06_Build/Terra_Delivery_Milestone_Plan.md)
- [Emergent pack structure](100_Terra_Phase1_Handover_Emergent/06_Build/EMERGENT%20PACK%20STRUCTURE.md)
- [Phase 1 EPU instruction set](100_Terra_Phase1_Handover_Emergent/07_Post_Build/Implemented/Terra_Phase_1_EPU_Instruction_Set.md)

## Curation note

Some migrated documents may be staged under “Original Files” while they are being curated into their long-term home. The intention is to progressively consolidate content into the main domain areas as the knowledge base matures.

## Editing principles

- Treat Markdown here as the source of truth; keep edits incremental and reviewable.
- Prefer clarity over verbosity: capture decisions, constraints, and interfaces explicitly.
- Keep filenames stable when practical to preserve history and minimise link churn.
