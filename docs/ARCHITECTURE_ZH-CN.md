# 架构说明｜Control Layer for Generative Filmmaking v0.2

[仓库首页](../README.zh-CN.md) | [English Architecture](ARCHITECTURE_EN.md) | [完整项目计划](PROJECT_PLAN_v0.2_ZH-CN.md)

**状态：** 研究架构提案，实施暂缓。  
**规范边界：** 本文描述目标 Contract 与 Invariant，不代表已经存在可运行软件。

## 1. 架构目标

架构必须始终区分：

- **Production Truth：** Canon、World Facts、Scene Time、Approved Asset Version；
- **Director Intent：** Camera、Staging、Performance、Contact 与 Constraint；
- **Delivery Representation：** 实际用什么媒介把控制交给某个 Provider；
- **Generated Evidence：** Provider Output 与观测到的偏差；
- **Human Judgment：** Technical Override 与 Creative Approval。

同一个 Shot Design 应能够跨 DCC、Provider、Delivery Mechanism 与 Model Version 继续存在。

## 2. 参考流程

```text
┌──────────────────────────────────────────────────────────────┐
│ Authoring                                                    │
│ 第一阶段 Blender；未来其他 DCC                               │
└─────────────────────────────┬────────────────────────────────┘
                              │ 受控读写
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
                      人类技术 / 创意决策
                              │
              accept / recompile / repair / reroll / block
```

Generated Asset 可以影响后续决策，但未经明确的人类审批与 Promote，不能修改 Canon。

## 3. Source of Truth

### 3.1 Spatial World Truth

OpenUSD 是 Scene Hierarchy、Transform、Geometry Reference 与 Spatial Asset Relation 的候选边界。MVP 只使用最小必要范围。USD 不负责 Provider Job、Approval、Cost 或长篇导演语义。

### 3.2 Cinematic Control Metadata

Typed YAML/JSON Schema 负责 Shot Intent、Camera、Constraint、Staging、Performance、Contact、State Sample、Compile Policy 与显式引用。大型媒体和复杂 Geometry 保持为外部资产。

### 3.3 Production / Generation State

Database 负责 Entity Version、Workflow Status、Provider Job、Cost、Manifest、Conformance、Repair、Approval、Dependency、Cache 与 Benchmark Run。建议本地 MVP 使用 SQLite + File Reference。

### 3.4 DCC Runtime Cache

`.blend` 负责本地 Authoring Convenience 与 Preview State，但不是唯一 Canon、Camera Truth 或 Continuity Truth。对象使用与可编辑名称无关的 Stable ID。

## 4. Identity、Version 与 Immutability

每个重要 Entity 都需要 Stable ID 与 Version Reference。候选格式：

```text
<namespace>/<entity>@<semantic-or-content-version>
```

例如：

```text
cora_lab/canon@0.3.0
cora_lab/world@0.3.0
cora_lab/camera_path/LAB_002@1.2
cora_lab/contact_graph/LAB_002@0.4
```

Core Compilation 在不可变 Snapshot 上运行。编辑创建新 Version 或显式 Override。Generated Output 在 Human Approval Promote 之前只是 `GeneratedAsset` 记录。

## 5. Canonical Coordinate / Time Contract

P0 ADR 候选：

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

在 round-trip fixtures 通过前，它仍是建议设计。

所有边界必须声明 Source/Target Convention。任何模块都不能从上下文猜 Axis、Handedness、Euler Order、Scale、Frame Origin 或 FPS。候选 Adapter 包括 Canonical ↔ Blender、USD、Unreal 与 Provider Camera。

Round-trip Test 覆盖 Transform、Animation Curve、Projection、Focus、Shutter 与 Time Sample，并检测 Reflection、×100/÷100 Scale、Quaternion Order、Transform Direction 与 Off-by-one Frame。

## 6. Camera Model

### 6.1 Canonical Root

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

Intrinsics / Extrinsics 是 Canonical Root。Focal Length 与 Sensor 是 UI/Derived Representation，必须保持投影等价。可选字段包括 Distortion、Anamorphic Squeeze、Focus Animation 与 Lens Metadata。

### 6.2 Camera Conformance 边界

Generated Video 通常无法可靠逆推出精确物理焦段，因此 Conformance 应比较可观测结果：Framing、Subject Size/Position、Horizon、Vanishing Behaviour、Background Tracks、Camera-induced Optical Flow 与 Camera/Clay Reference Alignment。

## 7. Scene Timeline 与 State Evaluation

### 7.1 Event Time

```text
SCENE EVENT TIME ─────────────────────────────────────────────>
   Beat A           Beat B           Beat C           Beat D

   [MASTER 00:00 ─────────────────────────────── 00:30]
         [CORA_CU 00:05 ───────── 00:14]
              [HADES_CU 00:08 ───────── 00:16]
                   [OTS 00:10 ───────────────── 00:22]
```

所有 Shot 独立采样同一 Scene Performance，不依赖 Edit Order。

### 7.2 Relation Semantics

- `continuous`：Target Event-in 紧接 Source Event-out，继承 End State。
- `overlapping_coverage`：从共享 Scene State 采样，绝不按 Edit Order 串联。
- `time_jump`：在目标 Event Time 求值，只继承 Persistent Field。
- `flashback`：使用另一 Timeline Branch 或 Named Snapshot。
- `montage`：每段有显式 Sample 或 Local Rule。
- `parallel_action`：独立 Branch，通过 Sync Anchor 关联。
- `reset`：从 Named Canon/Snapshot 开始并记录原因。

### 7.3 求值顺序

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

所有 Override 记录 Source 与 Reason；在构建 IR 前运行 Conflict Detection。

## 8. Shot、Staging、Performance 与 Contact

Shot 引用 Canon 与 Scene Time Interval，不包含整套 World 的无版本副本。

Staging 记录导演层空间意图：Root Position、Body/Head Direction、Eye Target。Kinematic Control 记录技术实现：Skeleton Pose、IK Target、Trajectory、Contact Constraint。Performance Intent 记录艺术语义；Performance Realization 记录可执行 Timing/Motion。这样 Emotion Label 不会伪装成精确物理事实。

Contact Graph 是版本化资产。Contact Event 声明 Actor Endpoint、Target、Time Range、Tolerance、Visibility Assumption 与 Holding-state Transition。

## 9. Constraint Resolution

所有可控值接受 `HARD`、`SOFT`、`FREE`。Compiler 从 Canon 到最近显式 Override 解析优先级，保存 Decision Trace，并拒绝静默丢失。

```text
if capability is unsupported and constraint == HARD:
    fail compilation
elif representation requires downgrade:
    require an explicit downgrade policy and record evidence
else:
    select the strongest calibrated representation
```

Adapter 不得把 HARD 自行解释为 SOFT。Naturalness 仍是 Benchmark 维度，因为技术合规也可能产生僵硬、不可用的镜头。

## 10. Rich IR

Rich IR 是一个已解析、Provider-neutral、不可变的 Shot Execution 表达，包含：

- Entity / Asset Version；
- Timeline Sample Boundary 上的 Materialized State；
- Canonical Camera 与 Curves；
- Staging、Kinematic、Performance 与 Contact；
- Constraint Level 与 Tolerance；
- Reference 与 Control Render；
- Delivery Preference / Fallback Policy；
- Dependency / Content Hash；
- Provenance / Decision Trace。

Rich IR 不包含 Provider Parameter Name，但可以包含当前没有 Provider 支持的语义。

## 11. Capability Model 与 Compiler

### 11.1 Capability Declaration

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

未知或未测试能力保持 `UNKNOWN`；Provider Marketing Copy 不能把状态升级为 Verified。

### 11.2 Representation Selection

Compiler 在 Constraint、Provider Limit、Available Reference、Cost Policy 与 Known Failure 允许时，按 `native` > `derived` > calibrated `emulated` 选择。`unsupported` 触发 HARD Failure 或显式批准的 Downgrade。

### 11.3 Determinism

相同 Canonical Snapshot、Rich IR Version、Compiler Version、Adapter Version、Capability Snapshot 与 Compile Setting，必须得到相同结构 Package 与 Manifest Hash，声明排除的 Runtime Field 除外。

## 12. Provider Adapter Boundary

```python
class VideoProvider:
    def capabilities(self) -> ProviderCapabilities: ...
    def compile_control_package(self, shot_ir) -> ProviderPackage: ...
    def generate(self, request) -> JobId: ...
    def status(self, job_id) -> JobStatus: ...
    def fetch_result(self, job_id) -> GeneratedAsset: ...
    def edit(self, request) -> JobId: ...
```

Adapter 可以把 Rich IR 转成 Provider Parameter / Media，可以验证 Limit 与 Authentication Readiness；但不能读取 Blender Private State、不能修改 Canon，必须记录 Provider/Adapter Version，并按 Privacy/Rights Policy 保存 Manifest 所需的 Request/Response Evidence。

在真实接入前必须有 MockProvider，用来测试 Schema、Deterministic Compilation、Manifest、Job Transition、Failure 与 Contract，而不产生付费调用。

## 13. Generation Manifest

Manifest 记录：

- Project/Scene/Shot/Run ID；
- Canon、Timeline、Shot、IR、Compiler、Adapter、Provider Version；
- Capability Snapshot 与 Decision Trace；
- Reference、Control Render、Prompt、Parameter 与 Hash；
- Job Timestamp / State；
- Cost / Billing Unit；
- Output、Parent Generation 与 Edit Scope；
- Conformance、Technical Pass、Creative Approval 与 Human Override；
- Reference/Output 的 Rights/Source Metadata。

历史事件 Append-only；纠错通过新增事件完成，不能无痕改写旧记录。

## 14. Conformance Model

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

Conformance Engine 区分 Detection 与 Decision。Confidence 与 Evidence 支持 Human Review。Creative Approval 是独立记录，QA Model 不能自动产生。

## 15. Repair Planning

Repair Planner 读取 Difference、Provider Edit Capability、Preserve Control 与 Cost Policy，可建议：

- 使用更强或不同 Representation 重新编译；
- Local Spatial Edit；
- 带 Boundary Anchor 的 Temporal-range Edit；
- 重生成一段或整个 Shot；
- 切换 Provider；
- 人工处理。

每个 RepairTask 保留 `parent_generation_id`、Time/Space Scope、Preserve List、Intent/Delivery Ref 与成功/失败 Evidence。真实 Repair 质量不作为验证核心架构的前置条件。

## 16. Dependency Graph 与 Cache Invalidation

```text
CharacterCanon@0.4 ─┬─> ShotIR/LAB_002 ─> ProviderPackage ─> Generation
World@0.3 ──────────┤
CameraPath@1.2 ─────┘

change CameraPath@1.2 → 1.3
  invalidates: ShotIR/LAB_002, downstream packages, conformance baseline
  preserves: unrelated shots, approved canon, raw provider outputs
```

每个派生产物保存 Content Hash、Producing Tool/Version、Dependency Hash 与 Cache Key。Invalidation 遍历 Direct / Transitive Edge。Raw Provider Output 与 Historical Manifest 是不可变 Evidence，不能因上游变更而删除。

## 17. State Machine

候选 Generation State：

```text
DRAFT → VALIDATED → COMPILED → SUBMITTED → RUNNING
  → SUCCEEDED | FAILED | CANCELLED
  → CONFORMANCE_REVIEWED
  → TECHNICAL_PASS | TECHNICAL_FAIL
  → CREATIVE_APPROVED | CREATIVE_REJECTED
```

Technical Status 与 Creative Status 必要时正交：Shot 可以 Technical Pass 但 Creative Reject。

## 18. Security、Privacy 与 Rights

API Key 不得进入 Shot Schema、Manifest 或提交到仓库的 Provider Package。Secret 留在 Local/Runtime Secret Store。Reference Source、License、Allowed Use、Access Class、Retention 与 Deletion Status 属于 Asset Metadata。公开 Benchmark Fixture 必须使用可公开素材。

## 19. Test Strategy

### Contract / Structural

- Schema Round-trip 与 Unknown-field Policy；
- Stable ID / Immutable Version Reference；
- Deterministic Compilation / Package Hash；
- MockProvider Contract / Job Transition；
- Adapter 与 DCC Private State 隔离；
- Manifest Completeness；
- Dependency Invalidation Precision。

### Geometry / Time

- Canonical ↔ Blender / USD / Provider Transform；
- K 与 Focal/Sensor Projection Equivalence；
- Animation Curve、Time Sample、Rational FPS、Frame Origin；
- Coverage、Time Jump、Reset、Conflict、Override Fixture。

### Production Benchmark

使用固定 Cora Lab Set，在同 Provider、同预算、同输出要求与同 Approval Procedure 下采集 Time to Approved Shot、Active Human Time、Full Reroll、Constraint Failure、Cost 与 Qualitative Naturalness。不能把挑选出的最佳输出当作 Benchmark。

## 20. Deferred Decisions

实现前必须通过 Coordinate/Time、Versioning、Minimal USD Scope、Camera Transform Direction、State Evaluation Semantics、Content Hashing、Provider Capability Evidence、HARD Tolerance 与 Benchmark Procedure 的 ADR。

在此之前，本架构刻意保持“可讨论、可证伪”的研究提案状态，而不是 Production-ready Software。

