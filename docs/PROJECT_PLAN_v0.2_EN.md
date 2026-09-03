# Control Layer for Generative Filmmaking — Project Plan v0.2

[Repository home](../README.md) | [中文项目计划](PROJECT_PLAN_v0.2_ZH-CN.md) | [Architecture](ARCHITECTURE_EN.md)

**Document type:** Product + Technical Master Plan  
**Version:** v0.2, architecture revision  
**Source date:** 2026-09-02  
**Publication status:** Research / Architecture Ready — Implementation Deferred  
**Demonstration case:** Cora Lab MVP — Cora discovers a reacting seed in a laboratory

## 0. Status language

This plan uses three evidence labels:

| Label | Meaning | Rule |
|---|---|---|
| **Decided** | A project principle, boundary, or core architectural direction | Do not change in implementation without an explicit decision record |
| **Proposed design** | A default intended to make the architecture implementable | May become Decided after an ADR, review, or experiment |
| **Hypothesis to validate** | A user, technical, cost, or quality claim requiring evidence | Must be paired with a validation method and exit condition |

The project is currently a published research architecture. It contains no implemented product, completed benchmark, verified provider integration, or measured ROI result.

## 1. Executive summary

**Decided.** The system's central value is not a promise that AI will obey perfectly. It is to make **Director Intent**, **Control Delivery**, and output **Conformance** computable and traceable.

Generative video can produce increasingly strong images, but production control still lags. Camera geometry is commonly expressed as prose; staging, gaze, props, and contact drift; multi-shot state is not reliably inherited; and a local error often requires a full reroll. Provider-specific inputs also make the underlying shot design difficult to reuse.

The proposed response is a model-independent layer that separates:

- what the world and approved assets are;
- what happens on a scene-event timeline;
- what a shot samples from that timeline;
- what the director requires for camera, staging, performance, and continuity;
- how those requirements are delivered to a particular provider; and
- how the generated result differs from the intended and delivered control.

```text
Director Intent → Canonical Control → Provider Representation
                                             ↓
                                      Generated Video
                                             ↓
                      Conformance Analysis → Difference
                                             ↓
                               Recompile / Repair / Approve
```

Blender is proposed as the first previs and control runtime. It may author camera and blocking and render clay, depth, mask, motion, or camera references. It is not the only source of truth. Unreal and other DCCs may be added behind the same canonical interfaces later.

The Cora Lab case is a production A/B benchmark, not a visual showcase. It is meant to test reproducibility, shared scene state, continuity, provider compilation, conformance, Time to Approved Shot, reroll cost, and provider replaceability.

## 2. Background and problem definition

### 2.1 Current production failures

- **Camera cannot be reproduced:** focal length, sensor, projection, trajectory, focus, and timing become ambiguous prose.
- **Spatial relations drift:** character positions, orientations, gaze, environment layout, and object relations may change.
- **Asset state is lost:** identity, costume, hair, props, doors, windows, lighting, and damage lack explicit inheritance.
- **Interaction is unstable:** hand-object, foot-floor, eye-line, handoff, and grasp events are difficult to express reliably through prompts alone.
- **Failure cost is too high:** an error in one region or frame range can force a whole-shot reroll.
- **Provider lock-in:** shot intent embedded in one provider's format must be redesigned when the provider changes.
- **Results are difficult to audit:** inputs, hashes, versions, cost, QA evidence, parent/child generations, and approval are fragmented.

### 2.2 Root cause

World facts, directorial requirements, provider representations, and generated outputs are mixed together. Prompts simultaneously carry facts, commands, style, and repair instructions. The information therefore cannot be inherited, converted, validated, or audited reliably.

**Hypothesis to validate.** Separating key controls from the prompt, delivering them explicitly, and measuring conformance will reduce Time to Approved Shot, hard-constraint failures, and full rerolls versus the existing workflow.

## 3. Product position and principles

### 3.1 Position

A provider-agnostic production control layer between Blender/Unreal authoring environments, optional world providers, and generative video render providers. It preserves Film Production State, compiles directorial intent into explicit provider inputs, and evaluates generated output against the shot design.

### 3.2 Eleven principles

1. World State is persistent state, not a background prompt reconstructed for each shot.
2. Shot State must be explicit.
3. Camera is convertible geometry and optics, not a lens label or movement adjective.
4. Blocking is spatial data, not descriptive prose.
5. Continuity is evaluated on Scene Timeline and explicit relation edges; edit order is not world time.
6. A provider is a replaceable render provider, not the system core.
7. Generated results do not automatically become a source of truth.
8. Human Creative Approval retains final authority.
9. Hard constraints and creative freedom are separated.
10. Every generation is traceable, and the same versioned inputs produce the same structural Control Package.
11. Provider-agnostic does not mean lowest common denominator; the Rich IR remains more expressive than any one provider.

**Boundary.** Provider-specific logic may exist only inside the relevant adapter.

## 4. Goals and non-goals

### Goals

- Structure world, camera, blocking, performance, continuity, contacts, and constraints.
- Compile one canonical shot for MockProvider and at least one real provider.
- Produce a structured Conformance Result with evidence and next-action guidance.
- Preserve generation, cost, version, QA, repair, and approval history.
- Validate whether the system improves real production outcomes under controlled conditions.

### Non-goals

- Training a new video foundation model.
- Replacing Blender, Unreal, or another DCC.
- Automatically judging whether art, acting, rhythm, emotion, or storytelling is successful.
- Building a complete OpenUSD studio pipeline in the MVP.
- Guaranteeing that a provider obeys every control.
- Making automatic local repair a prerequisite for MVP success.
- Supporting every provider and every control modality at launch.
- Building multi-tenant production infrastructure, a render farm, or a complete editorial system.

## 5. Users and scenarios

| User | Primary task | Value |
|---|---|---|
| Director / previs artist | Author camera, blocking, pacing, and continuity | Turn intent into inspectable shot data |
| AI video artist | Compile packages, submit jobs, compare providers | Reduce repeated prompting and migration cost |
| Technical artist | Maintain DCC runtime and control passes | Standardize exports and spatial control |
| Producer / supervisor | Review status, cost, versions, failures, and approval | Gain predictability and an audit trail |
| Developer / Codex | Implement schemas, compiler, adapters, conformance, and tests | Work from explicit boundaries and acceptance criteria |

Typical scenarios include validating blocking on a low-cost provider before final rendering; changing a 50mm setup to 65mm without rewriting prompts; sampling multiple coverage shots from the same performance interval; and turning a frame-local contact failure into an evidence-backed recompile, repair, or reroll decision.

## 6. End-to-end workflow

1. **Establish Canon / World.** Lock environment, identity, costume, key props, default light, references, and versions.
2. **Build Scene Timeline.** Define events, beats, shot sample ranges, continuity relations, time jumps, resets, and branches.
3. **Design Shot.** Author camera geometry, path, focus, staging, kinematic controls, performance intent, realization, contacts, and constraints.
4. **Validate Canonical Control.** Check schemas, references, coordinate/time conversions, state evaluation, conflicts, dependency edges, and hard-control completeness.
5. **Compile Control Package.** Select native, derived, or emulated representations based on versioned provider capability evidence.
6. **Generate and record.** Create the job and manifest; record hashes, versions, parameters, cost, logs, outputs, and parents.
7. **Analyze Conformance.** Compare intent, delivery, and output; produce differences, evidence, severity, time/space range, and confidence.
8. **Creative Approval.** The director evaluates performance, composition, rhythm, emotion, and narrative success.
9. **Act.** Accept, review, recompile, repair, regenerate, or block. Output never silently overwrites canon.

## 7. System architecture

**Decided.** Authoring, Scene Timeline, Canonical Control, Compiler/Delivery, Provider, Conformance, and Production State must remain separated.

### Core modules

- **Canonical Data Layer:** versioned USD/YAML/JSON references and immutable snapshots.
- **Scene Timeline & Continuity Service:** event time, sample intervals, relations, state evaluation, overrides, and conflict detection.
- **Coordinate & Camera Geometry:** units, axes, handedness, time base, camera intrinsics/extrinsics, and boundary conversion.
- **Blender Runtime:** load/save permitted control fields, author camera/blocking, render control channels, and export packages.
- **Control Compiler:** resolve canonical state, create Rich IR, negotiate capabilities, select delivery representations, and compile provider packages.
- **Provider Registry & Adapters:** declare capabilities and perform validation, generation, status, retrieval, and editing.
- **Generation Orchestrator:** jobs, retries, costs, assets, and manifests.
- **Conformance Engine:** intent/delivery/output comparisons with evidence and recommended action.
- **Dependency & Cache Service:** hashes, dependency edges, cache entries, and exact invalidation.
- **Repair Planner:** convert differences into recompile, scoped edit, reroll, provider switch, or human tasks.
- **Approval & Audit:** technical pass, creative approval, version freeze, and release status.

### Facts and ownership

| Layer | Owns | Does not own |
|---|---|---|
| OpenUSD | Spatial hierarchy, transforms, geometry references, camera/asset relations | Job state, cost, approval, long-form intent |
| YAML / JSON | Shot intent, constraints, staging, performance, sampled state, compile policy | Heavy geometry or media binaries |
| Database | Production status, versions, jobs, costs, manifests, QA, repair, approval | DCC scenes and large media bodies |
| Blender `.blend` | Authoring cache, UI state, local convenience, previews | The only canon, camera truth, or continuity truth |

**Proposed design.** Begin with Python, typed schemas, SQLite, and file/object references for a local MVP. Service boundaries, PostgreSQL, queues, and object storage are deferred.

## 8. Core data model

### 8.1 Main domains

- Production: Project, Sequence, Scene, Shot
- World: World, Asset, Reference
- Character: Character, CharacterState
- Prop: Prop, PropState
- Camera: CameraRig, Intrinsics, Extrinsics
- Staging: Staging, KinematicControl, ContactGraph
- Performance: PerformanceIntent, PerformanceRealization
- Continuity: SceneTimeline, ContinuityEdge, StateSample
- Provider: ControlDelivery, GenerationManifest
- Quality: ConformanceResult, RepairTask, Approval
- Build: DependencyEdge, CacheEntry, BenchmarkRun

### 8.2 Minimal shot example

```yaml
shot_id: SHOT_012
canon_version: cora_lab_canon@0.3.0
world_ref: usd://cora_lab/world@0.3.0
timeline_sample: {event_in: 12.4s, event_out: 16.9s}
continuity_edge: {type: overlapping_coverage, source: SCENE_01@12.4s}
camera:
  extrinsics: {R: [...], T_m: [...]}
  intrinsics: {K: [[fx, 0, cx], [0, fy, cy], [0, 0, 1]], projection: perspective}
  ui_optics: {focal_length_mm: 50, sensor_width_mm: 36}
staging:
  cora: {position: LabDesk_B, body_direction: SeedTray, eye_target: SeedTray}
performance:
  intent: {emotion: restrained_fear, beat: recognition, intensity: 0.3}
  realization: {gaze_shift_frames: 14, head_turn_deg: 11, pause_s: 0.6}
contacts:
  - {actor: Cora.RightHand, target: SeedTray, start_frame: 72, end_frame: 120}
constraints:
  hard: [identity, costume, camera_path, seed_tray, blocking]
  soft: [lighting, expression, hair_motion]
  free: [dust, reflections, micro_environment_motion]
delivery_policy: {camera_path: HARD, fallback: clay_reference}
state_query: {scene_time: 12.4s}
```

### 8.3 Canonical coordinate and time

**Proposed P0 ADR default:** meters, Z-up, right-handed, quaternion `xyzw`, degrees for UI, frame origin 0, and rational FPS such as `24000/1001`. All boundaries require explicit adapters and round-trip fixtures to prevent mirror errors, ×100 scale errors, rotation-order mistakes, and frame offsets.

### 8.4 Camera

- Extrinsics: explicit transform direction, `R`, `T`, and coordinate metadata.
- Intrinsics: `K`, image size, pixel aspect, and projection.
- UI optics: focal length, sensor dimensions, aperture, and focus distance.
- Timing: frame rate, shutter, camera curves, and focus curves.
- Optional lens character: distortion and anamorphic squeeze.

Focal length is a UI and derived parameter, not the only canonical camera root.

### 8.5 Staging, kinematics, performance, and contacts

- Staging: root position, body direction, head direction, eye target.
- Kinematic control: skeleton pose, IK target, trajectory, contact constraint.
- Performance intent: emotion, intensity, beat, motivation.
- Performance realization: gaze shift, head turn, pause, gesture, expression, and frame intervals.
- Contact Graph: hand/object, foot/floor, character/character, holding and handoff relations.

Each field may carry a constraint level, source, version, confidence, and override reason.

## 9. Constraint system

| Level | Meaning | Typical content | Failure behavior |
|---|---|---|---|
| HARD | Cannot change without explicit approval | identity, costume, structure, camera path, blocking, key prop, contact | block pass; recompile, repair, or reroll |
| SOFT | Preserve when possible within tolerance | light, expression, hair, cloth, fog, secondary timing | report deviation; threshold and director decide |
| FREE | Deliberately available to the model | micro-detail, dust, reflection, small environment motion | not a failure |

Resolution order is Canon → Sequence → Scene → Shot → Repair Override, with closer explicit overrides taking precedence and recording `override_reason`. Identity, canon asset version, and key props cannot be overwritten by SOFT or FREE values. Unsupported HARD controls must fail or generate a visible downgrade decision.

**Hypothesis to validate.** Too many HARD controls may make outputs rigid. The MVP must explore the useful boundary between conformance and natural variation.

## 10. Scene Timeline and continuity

Shot N → Shot N+1 in an edit list is not a universal world-time relationship. State is evaluated on Scene Timeline, then materialized at each shot's `scene_event_in` and `scene_event_out`.

| Relation | Meaning | State rule |
|---|---|---|
| `continuous` | target follows source in world time | source end → target start |
| `overlapping_coverage` | shots cover the same performance interval | sample shared timeline; never chain by edit order |
| `time_jump` | explicit jump within a scene | evaluate at target time; inherit declared persistent state only |
| `flashback` | another narrative time layer | switch branch/snapshot with narrative anchor |
| `montage` | compressed or non-continuous events | explicit samples or local rules per segment |
| `parallel_action` | simultaneous action elsewhere | independent branch linked by sync anchors |
| `reset` | explicit creative/test reset | restart from a named canon or snapshot and record reason |

State domains include Character, Prop, Environment, Camera, and Narrative State. Camera state is shot-specific and does not participate in world-state chaining.

Conflict checks include unexplained inherited overrides, double-held props, blocking/frustum contradictions, unexplained costume/light/door jumps, and contact events targeting absent or invisible objects.

## 11. Rich IR, compiler, and provider adapter

### 11.1 Compiler sequence

```text
Canonical Shot
  → resolve versions and inheritance
  → validate hard constraints
  → build provider-neutral Rich IR
  → query versioned provider capabilities
  → select native / derived / emulated representation
  → compile package + delivery report + decision trace
  → write manifest + hashes
```

### 11.2 Delivery status

| Status | Definition | Example |
|---|---|---|
| `native` | provider directly accepts the control | camera pose, depth, or timestamp edit API |
| `derived` | deterministically produced from canonical data | render a depth/camera reference from K and R/T |
| `emulated` | another medium approximates the control | clay video or prompt used for camera path |
| `unsupported` | no reliable delivery path | block HARD or require approved downgrade |

Expected conformance should not be a subjective hard-coded decimal. Calibrate qualitative levels by provider, version, capability, representation, fixture, sample size, date, and error distribution.

### 11.3 Adapter contract

```python
class VideoProvider:
    def capabilities(self) -> ProviderCapabilities: ...
    def compile_control_package(self, shot_ir) -> ProviderPackage: ...
    def generate(self, request) -> JobId: ...
    def status(self, job_id) -> JobStatus: ...
    def fetch_result(self, job_id) -> GeneratedAsset: ...
    def edit(self, request) -> JobId: ...
```

Capabilities include status, limits, formats, emulation route, known failures, calibration reference, provider version, adapter version, and `last_verified_at`. Marketing claims are not treated as verified integration results.

### 11.4 Package structure

```text
control_package/
  shot.json
  references/{character,environment,costume,prop}/
  render/{clay.mp4,depth.mp4,normal.mp4,mask.mp4,camera_preview.mp4}
  audio/{dialogue.wav,reference.wav}
  prompt/provider_prompt.txt
  delivery/control_delivery.json
  manifest.json
```

Acceptance requires deterministic structural hashes, explicit treatment of unsupported HARD controls, no adapter access to private Blender state, and traceability to canon, shot, reference, compiler, adapter, and provider versions.

## 12. Blender previs / control runtime

Blender is the first authoring and control runtime, not the canon owner.

MVP operations cover Shot load/save/navigation; Camera create/load/save/preview; Staging and Character/Prop markers; coarse rig, IK, contacts, and motion preview; clay/depth/normal/segmentation/camera preview renders; validation; package export; and manifest inspection.

The UI may expose focal length, sensor, aperture, and focus, while persistence uses canonical K, R/T, projection, image size, timebase, shutter, and curves. Stable IDs are mandatory. Save operations write only permitted fields, and all control renders enter the Generation Manifest.

## 13. Conformance, Technical QA, and repair

```text
Control Intent ───────────────┐
Control Delivery ─────────────┼─> Conformance Analysis
Generated Video ──────────────┘          ↓
                              Difference + Evidence
                                         ↓
                  Accept / Review / Recompile / Repair /
                              Regenerate / Block
```

### QA boundary

Technical QA evaluates explicit, reviewable constraints. Creative Approval remains a human decision.

Candidate automated evidence includes manifest/state consistency, prop presence, identity similarity, screen-space blocking, pose/trajectory, background feature tracks, horizon, optical flow, and clay alignment. Exact focal-length recovery, universal physics judgment, and emotion judgment are not promised.

Camera conformance should compare expected clay/camera references with screen-space motion, subject bounds, framing, horizon, vanishing behavior, and camera-induced flow rather than asserting that a generated clip is exactly 50mm.

### ConformanceResult

Required fields include category, constraint level, severity, confidence, frame/time range, spatial region or mask, expected and observed values, evidence references, intent/delivery/output references, metric/tolerance/deviation, recommended action, model/rule versions, and any human override.

### Repair

Repair scope may be a small region, short time range, one control channel, or a full hard-constraint failure. Local repair is an exploratory benefit, not an MVP gate. The fallback may be stronger recompilation, larger-range editing, a provider switch, full reroll, or a return to shot design.

## 14. Cora Lab MVP and A/B benchmark

### Shot design

| Shot | Narrative action | Key controls | Resulting state |
|---|---|---|---|
| `LAB_001` | Cora sorts a Seed Tray; a faint reaction appears | lab canon, costume, 50mm, desk blocking, tray | Cora notices the light; tray on desk |
| `LAB_002` | Cora leans closer and reaches toward the tray | gaze, push-in, right-hand contact | right hand contacts tray; door closed; afternoon light |
| `LAB_003` | reaction strengthens; Cora lifts tray | holder change, inherited start state, soft expression | tray in hand; Cora at LabDesk_B |
| `LAB_002B` | side coverage of the same performance moment | overlapping coverage, shared event time, independent K/R/T | creates no new world end state |

The four designs may expand into ten fixed benchmark shots.

### MVP scope

- one World/Scene Canon with Cora, Seed Tray, desk, door, and light;
- canonical coordinates/time, camera K/R/T, staging, kinematics, save/load/playback;
- Scene Timeline, relations, state materialization, overrides, and conflicts;
- HARD/SOFT/FREE constraint processing;
- MockProvider and one real adapter;
- package, manifest, job state, cost records;
- at least identity, camera/framing, blocking, prop, and continuity conformance;
- dependency graph and exact cache invalidation;
- Baseline A/System B data collection; and
- Repair Planner plus reproducible fixtures, with real local repair as a bonus.

### Acceptance

- identical versioned inputs rebuild the same structural files and hashes, excluding declared time fields;
- continuous and coverage state evaluation is correct;
- Blender round trips preserve camera, staging, rig/IK/contact, stable IDs, and transforms within tolerance;
- provider differences are isolated to adapter output;
- conformance produces structured differences, evidence, and action suggestions;
- dependency changes invalidate only the correct downstream artifacts;
- every generation traces to input versions, compiler, adapter, provider, cost, outputs, and approval; and
- the A/B report includes Time to Approved Shot, rerolls, active human time, cost, and failure categories, not only a selected best-looking output.

## 15. Candidate roadmap and resources

This is a planning frame, not a committed delivery schedule.

| Milestone | Candidate weeks | Main output | Exit condition |
|---|---:|---|---|
| M0 Foundations | 1–2 | IDs/versioning, coordinate/time ADR, timeline, schemas, SQLite, fixtures | validated state/camera round trips |
| M1 Blender | 3–5 | K/R/T, staging, coarse rig, IK/contact, control renders | continuous + coverage Cora Lab round trip |
| M2 Compiler | 6–7 | Rich IR, delivery, MockProvider, graph, manifest | deterministic compilation and invalidation |
| M3 Provider | 8–9 | one real adapter, jobs, results, cost | one real end-to-end generation path |
| M4 Conformance | 10–11 | evidence for five categories, action planning | Intent → Difference → Action loop |
| M5 Benchmark | 12 | Baseline A/System B and Go/No-Go | decision based on production metrics |

Candidate staffing: product/director lead, pipeline tech lead, Blender technical artist, backend/tooling engineer, part-time CV/QA engineer, and part-time production QA/UI support. This is an estimate from the planning document, not a current team or commitment.

## 16. Business value and potential moat

Potential value lies in reduced rework, more predictable shots, lower provider lock-in, reusable production state, better creative/engineering translation, and management visibility.

Potential compounding assets are the unified Shot/State schema, capability-aware compiler, Scene Timeline/Continuity Graph, conformance and repair evidence, DCC integration, cross-provider benchmark data, generation history, and approval history.

The moat cannot be a single camera workaround. As providers absorb native camera, motion, identity, and editing controls, delivery mechanisms should simplify while Canon, Timeline, State, Conformance, Versioning, and Audit remain valuable.

## 17. Risks and responses

| Risk | Impact | Response / decision gate |
|---|---|---|
| Providers absorb native controls | individual control features become commoditized | focus on canon, timeline, compiler, conformance, history, DCC workflow |
| Provider APIs change rapidly | adapters and delivery strategies age | version capabilities, contract-test adapters, recalibrate regularly |
| Excess constraint causes rigidity | technically compliant but lifeless output | limit HARD, preserve FREE, score naturalness with humans |
| Schema expands too early | slow implementation and unused fields | add fields only from Cora Lab evidence |
| USD scope grows | infrastructure consumes MVP | keep a minimal spatial boundary |
| QA false positives/negatives | blocks creativity or misses failures | evidence, confidence, thresholds, human override |
| Repair is unreliable | local fix fails or damages preserved content | do not make repair an MVP gate |
| Coordinate/time conversion errors | mirror, scale, rotation, frame errors | P0 ADR, adapters, golden round-trip tests |
| DCC/canon double writes | state drift | stable IDs, controlled writes, conflict blocking |
| Attractive demo is not repeatable | false maturity signal | fixed fixtures, repeated runs, hashes, metric reports |
| Rights and data governance | references or outputs cannot be reused safely | license/source metadata, access, retention, deletion policy |

## 18. Success metrics and Go/No-Go

All thresholds are candidates until baseline calibration.

| Metric | Definition | Candidate objective |
|---|---|---|
| Deterministic compilation | same inputs/versions yield same structural hash | 100% |
| Traceability | generation links to canon, shot, compiler, adapter, cost, approval | 100% |
| State inheritance coverage | required inherited fields are resolved and checked | 100% |
| Hard-constraint pass rate | fixed-set shots passing key constraints | ≥90%, after calibration |
| Camera/blocking round trip | key data reloaded within tolerance | 100% structural consistency |
| Time to Approved Shot | wall-clock from start to technical pass and director approval | core KPI; significant reduction from baseline |
| Active human time | hands-on creative/technical minutes | previs overhead lower than reroll/repair savings |
| Full rerolls | full generations before approval | ≥30% reduction, candidate |
| Repair success | scoped repair passes without new hard failure | ≥50%, exploratory |
| Provider migration reuse | shot-design fields reused across provider change | ≥80%, candidate |
| New adapter cycle | engineering time after stable API availability | ≤2 engineering weeks, candidate |

**Go:** deterministic compilation, timeline/continuity, invalidation, and traceability reach 100%, and Cora Lab on one real provider reduces Time to Approved Shot or materially reduces critical failure/rework.  
**Adjust:** if previs overhead exceeds reroll savings, narrow the interaction model or supported shot types.  
**No-Go:** should be based on failed production value and core-control evidence, not local-repair failure alone.

## 19. Implementation order if resumed

1. Approve coordinate/time ADR, IDs, versions, typed schemas, and golden fixtures.
2. Implement Scene Timeline, Continuity Edge, state evaluation, override, coverage, and conflict tests.
3. Implement Blender load/save, canonical camera, staging, coarse rig, IK/contact, and control renders.
4. Implement Rich IR, delivery records, MockProvider, dependency graph, deterministic compiler, and manifest.
5. Add one real provider and job lifecycle.
6. Add five conformance categories and Difference-to-Action planning.
7. Run the fixed Baseline A/System B benchmark and publish the evidence-backed decision.

Suggested future code boundaries:

```text
schemas/        canonical typed models
core/           timeline, state, continuity, constraints, coordinates
compiler/       IR, selection, packaging
adapters/       provider-specific code only
blender_addon/  authoring/runtime integration
orchestration/  jobs, manifests, cost, assets
conformance/    intent/delivery/output evidence
dependency/     graph, hashes, invalidation
repair/         repair planning and requests
fixtures/cora_lab/
tests/
docs/           ADRs, specs, capability evidence
```

## 20. Open decisions

P0 decisions require owners, deadlines, evidence, and decision records:

- Canonical coordinate/time convention and round-trip tolerance.
- Scene Timeline semantics and coverage fixtures.
- Canonical camera representation and projection equivalence.
- First real provider, selected by actual API availability and testability.
- Fixed MVP benchmark set and baseline procedure.
- Minimal USD scope.
- Category-specific hard-constraint tolerances.

P1 decisions include SQLite/file storage boundaries, conformance techniques, fidelity calibration, minimum Repair Planner behavior, and initial internal users. Unreal timing and multi-tenant services remain P2.

## 21. Maintenance rules

- Keep this master plan focused on position, boundaries, decisions, and milestone status; move implementation detail into TECH_SPEC, ADRs, and backlog items.
- Record the reason, impact, alternatives, and effective version for changes to Decided design.
- Promote Proposed Design only after review or experiment.
- Tie every Hypothesis to a metric, fixture, owner, and decision date.
- Record provider capability status, evidence, calibration set, verification date, provider version, and adapter version.
- Update risks, metrics, resources, and Go/No-Go conclusions after each milestone.

## 22. Terminology

- **Canon:** the currently approved versions of project world, character, asset, and factual definitions.
- **World State:** explicit state of a scene and its objects at a time.
- **Shot State:** camera, blocking, performance, constraints, and sampled start/end state required by a shot.
- **Scene Timeline:** event and beat timeline independent of edit order.
- **Continuity Edge:** explicit temporal/state relationship between shots or events.
- **Control Intent:** what the director requires.
- **Control Delivery:** how the compiler actually communicates a control to a provider.
- **Control Conformance:** evidence-backed difference between output, intent, and delivery.
- **Rich IR:** provider-neutral intermediate representation retaining full intent.
- **Generation Manifest:** input, version, parameter, cost, output, hash, and parent record for one generation.
- **Technical QA:** detection of explicit technical constraints, not artistic judgment.
- **Creative Approval:** final human judgment of the shot's artistic success.

## 23. Evidence boundary

External provider pages, open-source workflows, and research papers support the feasibility of individual control mechanisms. They do not prove that this proposed system has a usable API integration, stable conformance, better ROI, or product-market fit. See [Related Work](../research/RELATED_WORK.md) for the maintained evidence map.

