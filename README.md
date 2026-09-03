# Control Layer for Generative Filmmaking

[English](README.md) | [简体中文](README.zh-CN.md)

> A research architecture for a provider-agnostic production control layer between director intent and generative video systems.

**Status: Research / Architecture Ready — Implementation Deferred**

This repository publishes the v0.2 project plan for a proposed **Control Layer for Generative Filmmaking**. It is not a new video foundation model and it is not a validated product. It describes a production system that would preserve film state, turn directorial intent into explicit controls, compile those controls for different generation providers, and measure how closely generated outputs conform to the intended shot.

No implementation, production benchmark, or claimed performance result is included at this stage. The proposed Cora Lab benchmark and all numerical targets remain future validation work.

## The problem

Generative video quality is improving quickly, but production control remains fragile:

- camera geometry and movement are often compressed into natural-language prompts;
- character position, gaze, props, contact, and environment state can drift;
- adjacent and overlapping coverage shots lack a reliable shared scene state;
- a local failure can force a full reroll;
- shot intent becomes locked to one provider's input format; and
- generation inputs, versions, costs, outputs, QA evidence, and approvals are difficult to audit.

The central proposition is:

> The value of a Control Layer is to establish a computable relationship between director intent and generated results.

## Proposed system

```text
Director Intent
      ↓
Production Canon + Scene Timeline
      ↓
Canonical Shot State
      ↓
Rich Provider-neutral IR
      ↓
Control Compiler + Provider Adapter
      ↓
Control Delivery → Generated Video
      ↓
Conformance Analysis → Difference + Evidence
      ↓
Accept / Review / Recompile / Repair / Regenerate
```

The proposal deliberately separates three things:

1. **Control Intent** — what the director requires.
2. **Control Delivery** — what a provider actually receives, whether native, derived, emulated, or unsupported.
3. **Control Conformance** — how the result differs from the intent and the delivered control.

Generated outputs do not automatically become canon. Human creative approval remains authoritative.

## Core architecture

### Director Intent and constraints

Camera, staging, performance, contacts, and continuity are authored as explicit data. Each controllable property can be classified as:

- **HARD** — must hold or require explicit approval to change;
- **SOFT** — should hold within an accepted tolerance; or
- **FREE** — intentionally left to the model.

### Production Canon

The system keeps versioned facts about the world, characters, costumes, props, and approved assets. A shot references immutable snapshots instead of copying or silently mutating the entire world definition.

### Scene Timeline and Continuity Edge

Edit order is not treated as world time. Each shot samples an interval on a shared scene-event timeline. Explicit edges describe relations such as `continuous`, `overlapping_coverage`, `time_jump`, `flashback`, `montage`, `parallel_action`, and `reset`.

This is especially important for coverage: multiple cameras may observe the same performance interval without inheriting state from whichever shot happens to appear first in the edit.

### Camera Geometry

The canonical camera is rooted in intrinsics and extrinsics rather than a focal-length label alone:

- intrinsics `K`, projection type, image size, and pixel aspect;
- extrinsics `R` and `T` with explicit coordinate metadata;
- lens and timing data such as focal length, sensor, aperture, focus, shutter, frame rate, and animation curves; and
- explicit adapters for Blender, USD, Unreal, and provider boundaries.

The candidate canonical convention is meters, Z-up, right-handed coordinates, quaternion rotation storage, frame origin 0, and rational FPS. This remains a proposed P0 ADR until validated by round-trip fixtures.

### Rich IR and Control Delivery

A provider-neutral **Rich IR** preserves more intent than any one provider may support. The compiler negotiates versioned capabilities and selects the strongest available representation:

- `native` — directly accepted by the provider;
- `derived` — deterministically produced from canonical data;
- `emulated` — approximated through another medium, such as a clay reference; or
- `unsupported` — cannot be delivered reliably.

Unsupported HARD controls must block compilation or produce an explicit, approved downgrade. They must never disappear silently.

### Provider Adapter

Provider-specific logic belongs only in adapters. A stable contract covers capability declaration, package compilation, generation, status, result retrieval, and editing. Changing providers should require recompilation, not redesigning the shot.

### Conformance

Conformance compares the intended control, actual delivery, and generated output. A result should identify category, severity, confidence, time/frame range, spatial evidence, expected and observed values, and a recommended next action.

Technical QA can help assess identity, framing, blocking, props, continuity, and other explicit constraints. It does not determine whether acting, rhythm, composition, emotion, or storytelling are artistically successful.

### Dependency graph

Derived artifacts are content-addressed and linked to upstream versions. A change should invalidate only dependent Shot IR, provider packages, and QA baselines, while preserving unrelated shots, approved canon, and raw provider outputs.

## Cora Lab validation proposal

The proposed MVP is a production A/B benchmark, not a beauty demo. Its smallest narrative follows Cora discovering a reacting seed in a lab:

- three continuous shots;
- one overlapping coverage shot sampling the same performance interval;
- a minimal world and character canon;
- camera, staging, gaze, prop, and hand-to-tray contact controls;
- MockProvider plus one real provider adapter;
- at least five conformance categories; and
- Baseline A versus System B under the same provider and budget.

The primary future metric is **Time to Approved Shot**. Other proposed measures include deterministic compilation, traceability, continuity coverage, hard-constraint pass rate, active human time, full rerolls, provider migration reuse, and adapter integration time.

These are experimental criteria, not reported results.

## Roadmap

Implementation is intentionally deferred while provider capabilities continue to change rapidly. If the project resumes, the candidate sequence is:

1. **M0 — Foundations:** IDs, versioning, coordinate/time ADR, typed schemas, Scene Timeline, SQLite, and golden fixtures.
2. **M1 — Blender runtime:** canonical camera round trips, staging, coarse rigs, IK/contact, and control renders.
3. **M2 — Compiler:** Rich IR, delivery/fidelity records, MockProvider, dependency graph, deterministic manifests.
4. **M3 — Real provider:** one adapter, job lifecycle, result retrieval, and cost records.
5. **M4 — Conformance:** structured evidence for five categories and Difference-to-Action planning.
6. **M5 — Benchmark:** fixed Baseline A/System B test set and an evidence-based Go/No-Go decision.

The 12-week sequence in the v0.2 plan is a candidate planning frame, not a committed schedule.

## Why this is public

I am sharing this work openly because I do not see these ideas merely as something to possess, but as something I have received and am willing to share, test, refine, and develop with others exploring the future of generative filmmaking.

Publishing the architecture now also makes its status honest: the research and system framing are ready to discuss, while implementation and production evidence remain deferred.

## Related work

Existing work already demonstrates important parts of the control-delivery path, especially Blender blockouts, camera previs, reference video, motion guidance, and provider adapters. This proposal does not claim to invent those techniques. Its intended contribution is the production-state layer around them: canon, shared scene time, continuity relations, Rich IR, conformance, dependency tracking, manifests, and approval history.

See [research/RELATED_WORK.md](research/RELATED_WORK.md) for evidence, boundaries, and links.

## Repository map

```text
README.md
README.zh-CN.md
docs/
  PROJECT_PLAN_v0.2_EN.md
  PROJECT_PLAN_v0.2_ZH-CN.md
  ARCHITECTURE_EN.md
  ARCHITECTURE_ZH-CN.md
research/
  RELATED_WORK.md
LICENSE_RECOMMENDATION.md
```

## Read the full documents

- [Project Plan v0.2 — English](docs/PROJECT_PLAN_v0.2_EN.md)
- [项目计划 v0.2 — 中文](docs/PROJECT_PLAN_v0.2_ZH-CN.md)
- [Architecture — English](docs/ARCHITECTURE_EN.md)
- [架构说明 — 中文](docs/ARCHITECTURE_ZH-CN.md)

## License recommendation

No license has been asserted in this draft repository package. The recommended approach is:

- **CC BY 4.0** for documentation and diagrams; and
- **Apache License 2.0** for future source code, if implementation begins.

This dual-license structure keeps public research attributable while giving future code a conventional permissive license with an explicit patent grant. See [LICENSE_RECOMMENDATION.md](LICENSE_RECOMMENDATION.md). A real `LICENSE` file should be added only after the repository owner confirms the choice.

## Suggested GitHub metadata

- **Repository name:** `control-layer-for-generative-filmmaking`
- **Description:** `Research architecture for a provider-agnostic control, continuity, and conformance layer for generative filmmaking.`
- **Visibility:** Public
- **Suggested topics:** `generative-video`, `filmmaking`, `blender`, `previs`, `camera-control`, `continuity`, `production-pipeline`, `ai-video`

Contributions are welcome as discussion, evidence, critique, experiments, and corrections. Please distinguish verified behavior from proposed design and avoid presenting provider marketing claims as measured conformance.
