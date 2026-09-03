# Related Work and Evidence Boundary

[Repository home](../README.md) | [中文首页](../README.zh-CN.md)

**Last reviewed:** 2026-09-03  
**Purpose:** distinguish existing evidence from the unvalidated claims of this project.

This file is bilingual by design. English is followed by a concise Chinese interpretation in each section.

## 1. Positioning

The Control Layer proposal does **not** claim to invent Blender blockouts, reference-video control, camera-conditioned generation, motion trajectories, world models, OpenUSD, video-quality benchmarks, or provider adapters.

Its intended contribution is the production-state architecture surrounding those mechanisms:

- Production Canon and immutable versions;
- Scene Timeline independent of edit order;
- Continuity Edge and state evaluation for overlapping coverage;
- canonical camera geometry and explicit coordinate/time conversion;
- a Rich IR that preserves more intent than one provider supports;
- separation of Control Intent, Control Delivery, and Control Conformance;
- dependency-aware invalidation, generation manifests, approval history, and a production A/B benchmark.

**中文说明：** 本项目不声称发明 Blender 白模、Reference Video、Camera-conditioned Generation、Motion Control、World Model、OpenUSD 或 Provider Adapter。它关注的是这些技术外围尚未被统一解决的 Production State 层：Canon、Scene Timeline、Coverage Continuity、Rich IR、Conformance、Dependency、Manifest 与 Approval History。

## 2. Closest open-source workflows

### 2.1 `Yi-111-a/blender-shot-video`

Repository: [Yi-111-a/blender-shot-video](https://github.com/Yi-111-a/blender-shot-video)

The README describes a practical single-shot pipeline in which Blender creates a plain gray previs, a first frame anchors the visual look, and a director-shot prompt plus reference video drives an AI video provider. The repository includes a shot spec, Blender white-model builder, first-frame adapter, video-generation adapter, and comparison tool. Its camera specification supports start/end/look-at, multiphase keyframes, followed subjects, paths, and easing. Provider calls are treated as pluggable adapters.

What this supports:

- Blender previs can carry camera, motion, framing, and spatial guidance.
- A structured shot spec can drive repeatable control-reference generation.
- The paid generation step can be isolated behind an adapter.
- Human review gates can occur before paid calls.

What it does not establish for this project:

- no evidence of a shared multi-shot Scene Timeline;
- no Production Canon or state evaluation across coverage;
- no general Rich IR / capability negotiation contract;
- no Intent/Delivery/Output Conformance model;
- no dependency graph, production manifest history, or controlled A/B ROI result.

**中文说明：** 这是最接近“Blender 白模锁 Camera / Composition，再让视频模型负责最终视觉”的可运行单镜头工作流。它证明了 Control Delivery 路线的一部分，但不等同于 v0.2 的多镜头 Production State、Continuity 与 Conformance 架构。

### 2.2 `echoxiangzhou/blender-seedance-workflow`

Repository: [echoxiangzhou/blender-seedance-workflow](https://github.com/echoxiangzhou/blender-seedance-workflow)

This repository is a curated collection rather than one product implementation. Its README reports 25 selected public cases covering Blender previs, camera control, multi-character blocking, action choreography, Blender MCP, Codex/Claude-assisted blockouts, FBX/Mixamo, ComfyUI, reference video, production toolchains, and known limitations. It includes cases where Codex builds simple 3D scenes and camera motion for Seedance reference video.

What this supports:

- the Blender → reference video → provider pattern is used in real creator workflows;
- camera, staging, timing, and action previs can be produced with ordinary DCC tools;
- agent-assisted Blender setup can reduce blockout effort;
- real workflows still report limitations and control tradeoffs.

Evidence limitation:

The repository curates creator posts and demos. It is valuable workflow evidence, but it is not a controlled benchmark proving causality, provider-wide reliability, or ROI.

**中文说明：** 该仓库提供了大量创作者实践案例，包括 Codex + Blender MCP，但它是案例资料库，不是统一产品，也不是受控 A/B Benchmark。它适合证明“有人这样做”，不适合证明“本系统一定降低成本”。

## 3. Provider and world-model evidence

### 3.1 Seedance 2.5

Official page: [ByteDance Seed — Seedance 2.5](https://seed.bytedance.com/en/seedance2_5)

The official product page describes reference-video understanding, editing, white-model control, camera movement, and performance blocking. These statements make Seedance a plausible candidate render provider for a control-delivery experiment.

Evidence boundary:

- a public product claim is not proof that every control is available through a stable API;
- it does not establish repeatability, failure distribution, local-edit preservation, or conformance under Cora Lab conditions;
- a provider capability may be marked verified only after an actual contract test and calibration run for a named provider/model/adapter version.

**中文说明：** 官方页面支持“白模、参考、Camera、Blocking 与 Editing 是 Provider 方向”的判断，但不证明 API 已稳定开放，也不证明 Cora Lab 中可重复达到目标。能力矩阵必须在真实 Contract Test 后才能升级为 verified。

### 3.2 World Labs Atlas and World API

Official pages: [Atlas](https://www.worldlabs.ai/blog/atlas) and [World API](https://www.worldlabs.ai/blog/announcing-the-world-api)

World Labs states that Atlas accepts precise camera geometry as a native input, uses spatial context, and can follow manually designed camera paths. Atlas was described as early access at publication. Separately, the public World API exposes generation of navigable worlds from text, images, panoramas, multi-view inputs, and video.

Architectural implication:

- native camera geometry strengthens the case for retaining a canonical K/R/T representation instead of binding the system to clay references;
- world providers may eventually absorb much of today's explicit world-runtime implementation;
- the Control Layer should therefore treat clay, depth, and prompt workarounds as replaceable delivery mechanisms;
- Canon, Scene Timeline, intended state, continuity, approval, and conformance remain separate production concerns.

Evidence boundary:

Atlas marketing and vendor-reported benchmarks are not project-specific conformance data. Early-access availability must not be treated as an MVP dependency without confirmed access and contract tests.

**中文说明：** Atlas 对 Native Camera Geometry 的支持反而说明 Canonical Camera K/R/T 具有长期价值；未来 Delivery 可以从 clay reference 切换到 native API。World Model 可能吸收 World Runtime，但“这一时刻世界应该是什么状态”仍需 Production System 管理。

## 4. Research on explicit camera and motion control

### 4.1 GEN3C

Paper: [GEN3C: 3D-Informed World-Consistent Video Generation with Precise Camera Control (CVPR 2025)](https://openaccess.thecvf.com/content/CVPR2025/html/Ren_GEN3C_3D-Informed_World-Consistent_Video_Generation_with_Precise_Camera_Control_CVPR_2025_paper.html)

GEN3C uses a 3D cache and a user-provided camera trajectory to guide generation. The paper reports improved camera control and temporal 3D consistency over its evaluated baselines.

Relevance:

- explicit camera trajectory and 3D/spatial caches are credible control mechanisms;
- a camera representation richer than cinematic adjectives has research support;
- a future provider may accept native geometry, reducing the need for emulation.

Boundary:

A research result does not imply an equivalent commercial API, the same performance on narrative human-action shots, or validation of this project's production architecture.

### 4.2 MotionPro and motion-prompting research

Paper: [MotionPro: A Precise Motion Controller for Image-to-Video Generation (CVPR 2025)](https://openaccess.thecvf.com/content/CVPR2025/html/Zhang_MotionPro_A_Precise_Motion_Controller_for_Image-to-Video_Generation_CVPR_2025_paper.html)

Motion-control research using trajectories, masks, and space-time guidance supports the decision to keep motion path, region, timing, and contact semantics explicit in the Rich IR.

Boundary:

Motion control alone does not solve identity, prop ownership, physical contact, coverage continuity, or approval history.

**中文说明：** GEN3C 与 MotionPro 等工作支持“Camera Trajectory、3D Cache、Region Trajectory、Motion Mask 是有效显式控制方向”。但研究模型不等于商业 Provider API，也不自动解决 Character Identity、Prop State、Contact 或多镜头 Continuity。

## 5. Evaluation research

### 5.1 VBench 2.0

Paper: [VBench-2.0: Advancing Video Generation Benchmark Suite for Intrinsic Faithfulness](https://arxiv.org/abs/2503.21755)

VBench 2.0 expands evaluation toward Human Fidelity, Controllability, Creativity, Physics, and Commonsense. It reinforces the need to look beyond superficial visual quality.

Use in this project:

- borrow evaluation dimensions and evidence practices where appropriate;
- keep provider/model/evaluator versions explicit;
- do not collapse all quality into one score;
- preserve human creative approval.

### 5.2 T2VPhysBench

Paper: [T2VPhysBench: A First-Principles Benchmark for Physical Consistency in Text-to-Video Generation](https://arxiv.org/abs/2505.00337)

The paper evaluates physical-law compliance and reports substantial limitations in the tested models. It supports the conservative stance that physics and interaction correctness should not be treated as solved or delegated to one automatic score.

Boundary:

Benchmark results apply to the tested model versions and prompts. They do not predict all future provider behavior, and they do not replace shot-specific evidence or human review.

**中文说明：** VBench 2.0 与 T2VPhysBench 支持“不能只看画面好不好看，还要评估 Controllability、Human Fidelity、Physics 与 Commonsense”。但本项目不会用一个自动分数替代 Conformance Evidence 或导演审批。

## 6. OpenUSD

Official documentation: [OpenUSD](https://openusd.org/)

OpenUSD is designed for scalable construction, composition, and interchange of 3D scenes across digital content creation tools. That makes it a suitable candidate boundary for spatial hierarchy, transforms, geometry references, and asset composition.

Boundary:

OpenUSD is not by itself a complete behavior runtime, provider compiler, production database, approval system, or AI control layer. The MVP should use a minimal USD boundary and avoid building a full studio pipeline before the production hypothesis is validated.

**中文说明：** OpenUSD 适合做 Spatial World Truth 与 DCC Interchange，但它不是完整的 Behavior Runtime、Provider Compiler、Production Database 或 Approval System。MVP 应控制 USD 范围。

## 7. Comparison matrix

| Work / area | Camera & spatial control | Single-shot delivery | Multi-shot scene state | Provider abstraction | Conformance & production history | Evidence type |
|---|---:|---:|---:|---:|---:|---|
| `blender-shot-video` | Yes | Yes | Not shown | Pluggable generation adapter | Comparison output, not the proposed conformance model | Open-source implementation/readme |
| `blender-seedance-workflow` | Many cases | Many cases | Not a unified system | Workflow-dependent | Curated cases, not controlled production history | Source-linked collection |
| Seedance 2.5 | Vendor claims white-model/reference/editing controls | Yes, product direction | Not established here | One provider | Not established here | Official product claims |
| World Labs Atlas | Vendor claims native precise camera geometry | Yes | Spatial context, but production semantics differ | One provider/world model | Not established here | Official early-access release |
| GEN3C | Explicit camera trajectory + 3D cache | Research system | Temporal 3D consistency | No | Research metrics | Peer-reviewed paper |
| MotionPro | Explicit motion control | Research system | No production-state layer | No | Research metrics | Peer-reviewed paper |
| VBench 2.0 / T2VPhysBench | Evaluation, not delivery | N/A | N/A | Cross-model benchmark | Evaluation dimensions, not approval history | Research benchmarks |
| **This proposal** | Canonical K/R/T + staging/contacts | Proposed | **Scene Timeline + Continuity Edge** | **Rich IR + capability-aware adapters** | **Intent/Delivery/Output + manifests + approval** | **Unvalidated architecture** |

## 8. What is already de-risked

The following propositions have meaningful external support:

- Blender/previs reference video can influence camera, motion, and spatial layout.
- First-frame plus reference-video workflows can separate look anchoring from spatial motion guidance.
- Explicit camera trajectories and 3D/spatial context are active research and product directions.
- Provider-native control may replace some emulated clay/depth/prompt mechanisms.
- Physics, interaction, and intrinsic faithfulness remain evaluation challenges.
- OpenUSD is a credible spatial interchange/composition boundary.

## 9. What remains unproven

This repository does not yet prove:

- that users will accept the additional previs and data-authoring work;
- that one canonical schema covers real productions without excessive complexity;
- that control delivery materially reduces Time to Approved Shot;
- that HARD constraints can be enforced without making shots rigid;
- that conformance detectors are accurate and useful enough;
- that local repair preserves unaffected content;
- that provider migration reuse reaches the proposed target;
- that the architecture has commercial value or product-market fit.

Only a fixed, same-provider, same-budget production A/B benchmark can begin to answer these questions.

**中文说明：** 外部证据已经降低了“Blender/Previs/Camera Control 是否可行”的风险，但没有证明用户愿意多做一层 Authoring，也没有证明系统会降低 Time to Approved Shot。真正的增量价值仍必须由同 Provider、同预算、固定测试集的 Cora Lab A/B 来验证。

## 10. Maintenance policy

- Prefer primary sources: official repositories, official product/API pages, and original papers.
- Record provider/model/adapter version and verification date.
- Separate `vendor_claim`, `workflow_demo`, `research_result`, `contract_test`, and `production_benchmark` evidence types.
- Never promote a marketing statement to `verified` without a reproducible test.
- Revisit provider capability claims before implementation because this area changes rapidly.
- Add corrections and contrary evidence, not only supporting examples.

