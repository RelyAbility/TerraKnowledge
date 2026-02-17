# Terra -- Security, Roles, and Governance Specification v1.0

Purpose: Define security controls, roles, audit trails, data governance,
and operational policies for Terra Phase 1, with forward compatibility
for external validation and market activation.

## 1. Roles & Permissions

• user: create/edit own plots (within org), view briefs, complete
missions.

• org_admin: manage users, portfolios, zones, triggers for reprocessing
(limited).

• reviewer: read-only access to evidence packages; submit external
review decisions (Phase 3).

• system: backend workers for processing and notifications.

## 2. Authentication & Session

• JWT access tokens with short TTL; refresh tokens stored securely.

• Optional MFA for org_admin and reviewer roles.

• Device registration for offline sync (device_id).

## 3. Data Access Controls

• Row-level security by org_id and portfolio membership.

• Geometry access restricted to authorised users to reduce leakage risk.

• Export controls: bulk export only allowed for org_admin (and logged).

## 4. Audit Trails

All mutations logged with: actor_id, org_id, resource_type, resource_id,
action, before/after diff, timestamp, request_id.

Method governance: any method_version changes require an immutable
record in method_versions with parameters JSON.

## 5. Data Retention & Privacy

• User-generated content (notes/photos) retained per org policy (default
7 years).

• Derived metrics retained indefinitely for longitudinal evidence, with
method versioning.

• PII minimisation: store only necessary identity attributes.

• Location sensitivity: provide precision controls for public sharing
(e.g., jitter / aggregation).

## 6. Integrity Controls

• No manual edits to derived metrics without creating a new
method_version or correction record.

• Brief publishing gated by ValidPixelFraction and QA checks.

• Tier labels are enforced server-side (not client-controlled).

• Elite flags remain dormant in Phase 1; any evaluation requires logged
rule set.

## 7. Operational Security

• Secrets management for API keys (satellite providers, push services).

• Least privilege IAM for processing workers.

• Regular vulnerability scanning and dependency checks.

• Backups: daily DB snapshots; monthly long-term archive.
