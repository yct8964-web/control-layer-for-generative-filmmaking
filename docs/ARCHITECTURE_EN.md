# Architecture — Control Layer for Generative Filmmaking v0.2

[Repository home](../README.md) | [中文架构说明](ARCHITECTURE_ZH-CN.md) | [Full project plan](PROJECT_PLAN_v0.2_EN.md)

**Status:** Proposed research architecture; implementation deferred.  
**Normative boundary:** this document describes intended contracts and invariants. It does not describe running software.

## 1. Architectural objective

The architecture preserves the distinction between:

- **production truth** — canon, world facts, scene time, approved asset versions;
- **directorial intent** — camera, staging, performance, contacts, and constraints;
- **delivery representation** — the actual medium used to communicate control to one provider;
- **generated evidence** — provider output and observed deviations; and
- **human judgment** — technical overrides and creative approval.

The system must allow one shot design to survive changes in DCC, provider, delivery mechanism, and model version.

## 2. Reference flow

```text
┌──────────────────────────────────────────────────────────────┐
│ Authoring                                                    │
│ Blender first; other DCCs later                              │
└─────────────────────────────┬────────────────────────────────┘
                              │ controlled read/write
                              v
┌──────────────────────────────────────────────────────────────┐
│ Canonical Control                                            │
│ Canon + Scene Timeline + Shot State + Constraints + Camera   │
└─────────────────────────────┬────────────────────────────────┘
                              │ resolve + validate
                              v
┌──────────────────────────────────────────────────────────────┐
│ Rich IR + Compiler                                           │
│ capability negotiation + representation selection            │
└─────────────────────────────┬────────────────────────────────┘
                              │ ProviderPackage + Manifest
                              v
┌──────────────────────────────────────────────────────────────┐
│ Provider Adapter                                             │
│ generate / status / fetch / edit                             │
└─────────────────────────────┬────────────────────────────────┘
                              │ GeneratedAsset
                              v
┌──────────────────────────────────────────────────────────────┐
│ Conformance                                                  │
│ Intent + Delivery + Output → Difference + Evidence + Action  │
└─────────────────────────────┬────────────────────────────────┘
                              │
                  human technical / creative decision
                              │
              accept / recompile / repair / reroll / block
```

Generated assets may influence future decisions, but they cannot mutate canon without an explicit human-approved promotion.

## 3. Source-of-truth model

### 3.1 Spatial world truth

OpenUSD is the proposed boundary for scene hierarchy, transforms, geometry references, and spatial asset relations. The MVP should use only the minimum required subset. USD is not treated as the owner of provider jobs, approval, cost, or long-form directorial semantics.

### 3.2 Cinematic control metadata

Typed YAML/JSON schemas own shot intent, camera, constraints, staging, performance, contact, sampled state, compile policy, and explicit references. Large media and complex geometry remain external assets.

### 3.3 Production and generation state

A database owns entity versions, workflow status, provider jobs, costs, manifests, conformance results, repairs, approvals, dependency edges, cache entries, and benchmark runs. SQLite plus file references is the proposed local MVP default.

### 3.4 DCC runtime cache

A `.blend` file owns local authoring convenience and preview state. It does not become the sole canon, camera truth, or continuity truth. Objects carry stable IDs independent of editable DCC names.

## 4. Identity, versions, and immutability

Every material entity requires a stable ID and version reference. Candidate form:

```text
<namespace>/<entity>@<semantic-or-content-version>
```

Examples:

```text
cora_lab/canon@0.3.0
cora_lab/world@0.3.0
cora_lab/camera_path/LAB_002@1.2
cora_lab/contact_graph/LAB_002@0.4
```

Core compilation operates on immutable snapshots. New edits create a new version or explicit override. Generated outputs remain `GeneratedAsset` records until a human approval promotes a reference.

## 5. Canonical coordinate and time contract

The candidate P0 ADR is:

```yaml
canonical_space:
  linear_unit: meter
  up_axis: Z
  handedness: right
  rotation_storage: quaternion_xyzw
  display_rotation: degrees
canonical_time:
  frame_origin: 0
  fps: {numerator: 24000, denominator: 1001}
  timecode_drop_frame: false
```

This is a proposal until round-trip fixtures pass.

All boundaries must declare source and target conventions. No module may infer axis, handedness, Euler order, scale, frame origin, or FPS from context. Candidate adapters include Canonical ↔ Blender, Canonical ↔ USD, Canonical ↔ Unreal, and Canonical ↔ Provider Camera.

Round-trip tests must cover transforms, animated curves, projection, focus, shutter, and time samples. They must detect reflection, scale ×100/÷100, quaternion order, transform-direction, and off-by-one-frame errors.

## 6. Camera model

### 6.1 Canonical roots

```yaml
camera:
  projection: perspective
  image_size: [1920, 1080]
  pixel_aspect: 1.0
  intrinsics:
    K: [[fx, 0, cx], [0, fy, cy], [0, 0, 1]]
  extrinsics:
    convention: world_from_camera
    R: [...]
    T_m: [...]
  optics_ui:
    focal_length_mm: 50
    sensor_width_mm: 36
    sensor_height_mm: 20.25
    aperture_f: 2.8
    focus_distance_m: 2.4
  timing:
    fps: {numerator: 24000, denominator: 1001}
    shutter_angle_deg: 180
```

Intrinsics and extrinsics are canonical. Focal length and sensor values are UI/derived representations and must remain projection-equivalent. Optional fields include distortion, anamorphic squeeze, focus animation, and lens metadata.

### 6.2 Camera conformance boundary

Generated video generally does not permit reliable recovery of an exact physical focal length. Conformance therefore compares observable results: framing, subject size and position, horizon, vanishing behavior, background tracks, camera-induced optical flow, and alignment with a camera/clay reference.

## 7. Scene Timeline and state evaluation

### 7.1 Event-time model

```text
SCENE EVENT TIME ─────────────────────────────────────────────>
   Beat A           Beat B           Beat C           Beat D

   [MASTER 00:00 ─────────────────────────────── 00:30]
         [CORA_CU 00:05 ───────── 00:14]
              [HADES_CU 00:08 ───────── 00:16]
                   [OTS 00:10 ───────────────── 00:22]
```

All shots sample the scene performance independently of edit order.

### 7.2 Relation semantics

- `continuous`: target event-in follows source event-out and inherits end state.
- `overlapping_coverage`: target samples shared scene state and never chains from edit order.
- `time_jump`: target evaluates at a later/earlier event time and inherits persistent fields only.
- `flashback`: use another timeline branch or named snapshot.
- `montage`: each segment has explicit samples or local state rules.
- `parallel_action`: independent branch connected by synchronization anchors.
- `reset`: start from a named canon/snapshot with a recorded reason.

### 7.3 Evaluation order

```text
Canon default
  → Sequence override
  → Scene override
  → State at scene event time
  → Continuity relation rule
  → Shot override
  → Repair override
  → materialized Shot Start/End State
```

Every override records source and reason. Conflict detection runs before IR construction.

## 8. Shot, staging, performance, and contact

A Shot references canon and a scene-time interval. It does not contain an unversioned copy of the entire world.

Staging records directorial spatial intent: root position, body/head direction, and eye target. Kinematic Control records technical realization: skeleton pose, IK targets, trajectories, and contact constraints. Performance Intent records artistic semantics; Performance Realization records executable timing and motion. The separation prevents emotion labels from masquerading as exact physical facts.

Contact Graph is a versioned asset. A contact event declares actor endpoint, target, time range, optional tolerance, visibility assumptions, and holding-state transition.

## 9. Constraint resolution

All controllable values accept `HARD`, `SOFT`, or `FREE`. The compiler resolves precedence from Canon to the closest explicit override, preserves a Decision Trace, and refuses silent loss.

Pseudo-rule:

```text
if capability is unsupported and constraint == HARD:
    fail compilation
elif representation requires downgrade:
    require an explicit downgrade policy and record evidence
else:
    select the strongest calibrated representation
```

An adapter may not reinterpret HARD as SOFT. Naturalness remains a benchmark dimension because technical compliance can still produce an unusably rigid shot.

## 10. Rich IR

Rich IR is a resolved, provider-neutral, immutable representation of one shot execution. It includes:

- resolved entity and asset versions;
- materialized state at timeline sample boundaries;
- canonical camera and curves;
- staging, kinematic, performance, and contact data;
- constraint levels and tolerances;
- available references and control renders;
- delivery preferences and fallback policy;
- dependency list and content hashes; and
- provenance and decision trace.

Rich IR must not contain provider parameter names. It may contain semantics no current provider supports.

## 11. Capability model and compiler

### 11.1 Capability declaration

Each provider capability record includes:

```yaml
capability: camera_path
delivery_status: emulated
formats: [video/mp4]
limits: {duration_s_max: null}
emulated_by: clay_reference
known_failures: []
calibration_ref: seedance/example/camera_path@YYYY-MM
last_verified_at: null
provider_version: null
adapter_version: 0.0.0
```

Unknown or untested capabilities remain `UNKNOWN`; public marketing copy does not promote them to verified.

### 11.2 Representation selection

The compiler ranks `native` > `derived` > calibrated `emulated`, subject to the requested constraint, provider limits, available references, cost policy, and known failures. `unsupported` triggers a hard failure or explicit approved downgrade.

### 11.3 Determinism

Given the same canonical snapshot, Rich IR version, compiler version, adapter version, provider capability snapshot, and declared compile settings, the structural package and manifest hashes must be identical except for explicitly excluded runtime fields.

## 12. Provider adapter boundary

```python
class VideoProvider:
    def capabilities(self) -> ProviderCapabilities: ...
    def compile_control_package(self, shot_ir) -> ProviderPackage: ...
    def generate(self, request) -> JobId: ...
    def status(self, job_id) -> JobStatus: ...
    def fetch_result(self, job_id) -> GeneratedAsset: ...
    def edit(self, request) -> JobId: ...
```

An adapter:

- may translate Rich IR into provider parameters and media;
- may validate provider limits and authentication readiness;
- may not query private Blender scene state;
- may not mutate canonical state;
- must record provider and adapter versions; and
- must preserve request/response evidence required by the manifest, subject to privacy and rights policy.

MockProvider is required before any real integration. It tests schema, deterministic compilation, manifests, job transitions, failures, and contract behavior without paid generation.

## 13. Generation Manifest

The manifest records:

- project/scene/shot/run IDs;
- canon, timeline, shot, IR, compiler, adapter, and provider versions;
- capability snapshot and Decision Trace;
- references, control renders, prompt, parameters, and hashes;
- job timestamps and states;
- cost and billing unit where available;
- output assets, parent generation, and edit scope;
- conformance, technical pass, creative approval, and human override references; and
- rights/source metadata required for retention and reuse.

The manifest is append-only for historical events. Corrections create new events rather than rewriting the record invisibly.

## 14. Conformance model

```yaml
conformance_result:
  category: contact
  constraint_level: HARD
  severity: major
  confidence: 0.82
  frame_range: [72, 96]
  spatial_region: right_hand
  expected_value: contact(Cora.RightHand, SeedTray)
  observed_value: separation_detected
  evidence_refs: [track://..., mask://...]
  intent_ref: shot://LAB_002@...
  delivery_ref: delivery://...
  generated_asset_ref: generation://...
  metric: pixel_or_world_proxy_distance
  tolerance: ...
  deviation: ...
  recommended_action: review
  qa_model_version: ...
  rule_version: ...
```

The engine distinguishes detection from decision. Confidence and evidence allow human review. Creative Approval is a separate record and cannot be emitted by the QA model.

## 15. Repair planning

The Repair Planner consumes differences, provider edit capabilities, preserved controls, and cost policy. It can propose:

- recompile with a stronger or different representation;
- localized spatial edit;
- temporal-range edit with boundary anchors;
- regeneration of one segment or the full shot;
- provider switch; or
- human intervention.

Every RepairTask retains `parent_generation_id`, time/space scope, preserve list, intent/delivery references, and success/failure evidence. Real repair quality is explicitly not a prerequisite for validating the core architecture.

## 16. Dependency graph and cache invalidation

```text
CharacterCanon@0.4 ─┬─> ShotIR/LAB_002 ─> ProviderPackage ─> Generation
World@0.3 ──────────┤
CameraPath@1.2 ─────┘

change CameraPath@1.2 → 1.3
  invalidates: ShotIR/LAB_002, downstream packages, conformance baseline
  preserves: unrelated shots, approved canon, raw provider outputs
```

Every derived artifact stores content hash, producing tool/version, dependency hashes, and cache key. Invalidation walks direct and transitive dependency edges. Raw provider outputs and historical manifests are immutable evidence and are never deleted merely because an upstream version changes.

## 17. State machines

Candidate generation states:

```text
DRAFT → VALIDATED → COMPILED → SUBMITTED → RUNNING
  → SUCCEEDED | FAILED | CANCELLED
  → CONFORMANCE_REVIEWED
  → TECHNICAL_PASS | TECHNICAL_FAIL
  → CREATIVE_APPROVED | CREATIVE_REJECTED
```

Technical and creative status remain orthogonal where necessary: a shot may technically pass and still be creatively rejected.

## 18. Security, privacy, and rights

The architecture must not place API keys in shot schemas, manifests, or committed provider packages. Secrets remain in local/runtime secret stores. Reference sources, licenses, allowed uses, access class, retention policy, and deletion status belong in asset metadata. Public benchmark fixtures must use publishable material.

## 19. Test strategy

### Contract and structural tests

- schema round trip with unknown-field policy;
- stable IDs and immutable version references;
- deterministic compilation and package hashes;
- MockProvider contract and job transitions;
- adapter isolation from DCC private state;
- manifest completeness;
- dependency invalidation precision.

### Geometry and time tests

- Canonical ↔ Blender / USD / provider transforms;
- K and focal/sensor projection equivalence;
- animation curves, time samples, rational FPS, and frame origin;
- coverage, time jump, reset, conflict, and override fixtures.

### Production benchmark

Use the fixed Cora Lab set with the same provider, budget, output requirements, and approval procedure. Collect Time to Approved Shot, active human time, full rerolls, constraint failures, cost, and qualitative naturalness. Do not treat a selected best output as a benchmark.

## 20. Deferred decisions

Before implementation, approve ADRs for coordinates/time, versioning, minimum USD scope, camera transform direction, state-evaluation semantics, content hashing, provider capability evidence, hard-constraint tolerances, and benchmark procedure.

Until then, the architecture is intentionally descriptive and falsifiable rather than presented as production-ready software.

