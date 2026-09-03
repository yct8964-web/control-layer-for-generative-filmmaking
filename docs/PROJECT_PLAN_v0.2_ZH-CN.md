# Control Layer for Generative Filmmaking｜项目计划 v0.2

[仓库首页](../README.zh-CN.md) | [English Project Plan](PROJECT_PLAN_v0.2_EN.md) | [中文架构说明](ARCHITECTURE_ZH-CN.md)

**文档类型：** Product + Technical Master Plan  
**版本：** v0.2｜架构升级版  
**源文档日期：** 2026-09-02  
**公开状态：** Research / Architecture Ready — Implementation Deferred  
**示范项目：** Cora Lab MVP｜Cora 在实验室发现种子发生反应

## 0. 状态标记

| 标记 | 含义 | 使用规则 |
|---|---|---|
| **已确定** | 已明确的项目原则、边界或核心架构 | 除非有正式 Decision Record，不应在实现中自行改写 |
| **建议设计** | 为形成可实施版本提出的默认方案 | 经过 ADR、评审或实验后可升级为已确定 |
| **待验证假设** | 需要用户、技术、成本或质量证据的判断 | 必须绑定验证方法与退出条件 |

当前项目是公开的研究架构，不包含已完成产品、已跑完的 Benchmark、已验证的 Provider 接入或已测得的 ROI。

## 1. 执行摘要

**已确定。** 系统的核心价值不是承诺 AI 完全服从，而是把 **Director Intent、Control Delivery 与输出 Conformance** 变成可计算、可追溯的生产关系。

生成式视频已经能输出越来越强的画面，但生产控制仍然落后：Camera Geometry 往往退化成自然语言，人物站位、视线、道具和接触关系容易漂移，多镜头状态缺少可靠继承，局部失败常迫使整段重生成，而 Provider 专属输入又让镜头设计难以复用。

本方案把以下内容拆开：

- 世界与批准资产是什么；
- 场景事件时间线上发生了什么；
- Shot 从 Scene Timeline 采样哪一段状态；
- 导演对 Camera、Staging、Performance 和 Continuity 有什么要求；
- 这些要求如何交付给某个 Provider；
- 生成结果相对 Intent 和 Delivery 产生了什么偏差。

```text
Director Intent → Canonical Control → Provider Representation
                                             ↓
                                      Generated Video
                                             ↓
                      Conformance Analysis → Difference
                                             ↓
                               Recompile / Repair / Approve
```

第一阶段建议以 Blender 作为 Previz 与 Control Runtime：它可以设计 Camera / Blocking，并输出 Clay、Depth、Mask、Motion 或 Camera Reference，但不成为唯一 Source of Truth。未来可以在同一 Canonical 接口后接 Unreal 或其他 DCC。

Cora Lab 被定义为生产 A/B Benchmark，而不是单纯展示好看的 Demo。它将验证可复现控制、共享场景状态、Continuity、Provider 编译、Conformance、Time to Approved Shot、重做成本和 Provider 可替换性。

## 2. 背景与问题定义

### 2.1 当前生产失败

- **Camera 不可复现：** focal length、sensor、projection、trajectory、focus 和 timing 被写成模糊描述。
- **空间关系漂移：** 人物位置、朝向、视线、环境结构和物体关系可能变化。
- **资产状态丢失：** 身份、服装、发型、道具、门窗、灯光和损伤缺少显式继承。
- **交互不稳定：** hand-object、foot-floor、eye-line、handoff 和 grasp 很难只靠 Prompt 稳定表达。
- **失败成本高：** 小区域或短帧段错误可能引发整段 reroll。
- **Provider 锁定：** 镜头意图写进一家 Provider 格式，切换时需要重新设计。
- **难以审计：** 输入、哈希、版本、成本、QA、父子生成和审批记录分散。

### 2.2 根本原因

World Facts、导演要求、Provider 表达与 Generated Output 混在一起。Prompt 同时承担事实、命令、风格和补救，信息因此难以继承、转换、验证和追溯。

**待验证假设。** 把关键控制从 Prompt 中分离、显式交付并进行 Conformance 分析，能相对当前流程降低 Time to Approved Shot、Hard Constraint 失败与整段重生成。

## 3. 产品定位与基本原则

### 3.1 定位

一层位于 Blender / Unreal 等 Authoring Environment、可选 World Provider 与生成式视频 Render Provider 之间的模型无关生产控制层。它保存 Film Production State，把导演意图编译为显式 Provider 输入，并评估生成结果相对 Shot Design 的符合程度。

### 3.2 十一条原则

1. World State 是持久状态，不是每个 Shot 重新猜测的背景 Prompt。
2. Shot State 必须显式记录。
3. Camera 是可转换的几何与光学数据，不是镜头标签或运镜形容词。
4. Blocking 是空间数据，不是描述性文字。
5. Continuity 在 Scene Timeline 与显式关系边上求值；剪辑顺序不等于世界时间。
6. Provider 是可替换 Render Provider，不是系统核心。
7. 生成结果不会自动成为 Source of Truth。
8. Human Creative Approval 保持最终权威。
9. Hard Constraints 与 Creative Freedom 必须分离。
10. 每次生成都可追溯；相同版本输入应产生相同结构化 Control Package。
11. Provider-agnostic 不等于最低公分母；Rich IR 必须比任何单一 Provider 更丰富。

**边界。** Provider 专属逻辑只能存在于对应 Adapter 内。

## 4. 目标与非目标

### 目标

- 结构化 World、Camera、Blocking、Performance、Continuity、Contact 与 Constraint。
- 将一个 Canonical Shot 编译给 MockProvider 和至少一个真实 Provider。
- 输出带证据与建议动作的结构化 ConformanceResult。
- 保存 Generation、Cost、Version、QA、Repair 与 Approval 历史。
- 在受控实验中验证是否改善真实生产结果。

### 非目标

- 训练新的视频基础模型。
- 替代 Blender、Unreal 或其他 DCC。
- 自动判断艺术、表演、节奏、情绪或故事是否成功。
- 在 MVP 建完整 OpenUSD Studio Pipeline。
- 保证 Provider 100% 服从所有控制。
- 把真实局部 Repair 成功设为 MVP 必须条件。
- 一次支持全部 Provider 与控制方式。
- 建多租户生产平台、渲染农场或完整剪辑系统。

## 5. 用户与场景

| 核心用户 | 主要任务 | 得到的价值 |
|---|---|---|
| 导演 / Previz Artist | 设计 Camera、Blocking、节奏与连续状态 | 把镜头意图变成可检查数据 |
| AI Video Artist | 编译包、提交 Job、比较 Provider | 降低重复提示与迁移成本 |
| Technical Artist | 维护 DCC Runtime 与 Control Pass | 统一导出与空间控制 |
| 制片 / 主管 | 查看状态、成本、版本、失败与审批 | 获得可预测性和审计记录 |
| 开发 / Codex | 实现 Schema、Compiler、Adapter、Conformance 与测试 | 拥有明确边界和验收条件 |

典型场景包括：先用低成本 Provider 验证 Blocking；把 50mm 改成 65mm 而不重写 Prompt；让多个 Coverage Shot 从同一表演区间采样；把第 72–96 帧的接触失败转成有证据的重编译、Repair 或 reroll 决策。

## 6. 端到端工作流

1. **建立 Canon / World：** 锁定环境、身份、服装、关键道具、默认光照、参考和版本。
2. **建立 Scene Timeline：** 定义事件、Beat、Shot 取样区间、Continuity Relation、Time Jump、Reset 与 Branch。
3. **设计 Shot：** 设置 Camera Geometry、Path、Focus、Staging、Kinematic Control、Performance Intent/Realization、Contact 与约束。
4. **验证 Canonical Control：** 检查 Schema、引用、坐标/时间、状态求值、冲突、Dependency 与 HARD 完整性。
5. **编译 Control Package：** 根据版本化 Provider 能力选择 native、derived 或 emulated 表达。
6. **生成并记录：** 创建 Job / Manifest，记录哈希、版本、参数、成本、日志、输出和 parent。
7. **Conformance Analysis：** 比较 Intent、Delivery 和 Output，输出 Difference、Evidence、Severity、时空范围与置信度。
8. **Creative Approval：** 由导演判断表演、构图、节奏、情绪和叙事。
9. **采取动作：** accept、review、recompile、repair、regenerate 或 block；输出不暗中覆盖 Canon。

## 7. 系统架构

**已确定。** Authoring、Scene Timeline、Canonical Control、Compiler/Delivery、Provider、Conformance 与 Production State 必须分离。

### 核心模块

- **Canonical Data Layer：** 版本化 USD/YAML/JSON 引用和不可变快照。
- **Scene Timeline & Continuity Service：** Event Time、Sample Interval、Relation、State Evaluation、Override 和 Conflict。
- **Coordinate & Camera Geometry：** 单位、轴、手性、Timebase、Camera Intrinsics / Extrinsics 与边界转换。
- **Blender Runtime：** 读取/保存允许字段、设计 Camera/Blocking、渲染控制通道与导出包。
- **Control Compiler：** 解析 Canonical State、构建 Rich IR、协商能力、选择 Delivery Representation、编译 Provider Package。
- **Provider Registry & Adapters：** 声明能力并执行 validate、generate、status、fetch、edit。
- **Generation Orchestrator：** Job、retry、cost、asset 和 Manifest。
- **Conformance Engine：** Intent/Delivery/Output 对比、Evidence 与 Recommended Action。
- **Dependency & Cache Service：** Hash、Dependency Edge、Cache 与精确 Invalidation。
- **Repair Planner：** 把 Difference 变成 recompile、scoped edit、reroll、Provider switch 或人工任务。
- **Approval & Audit：** Technical Pass、Creative Approval、Version Freeze 与 Release Status。

### 事实所有权

| 层 | 负责 | 不负责 |
|---|---|---|
| OpenUSD | 空间层级、Transform、Geometry 引用、Camera/Asset 关系 | Job、成本、审批、长篇导演意图 |
| YAML / JSON | Shot Intent、Constraint、Staging、Performance、State Sample、Compile Policy | 复杂 Geometry 与媒体二进制 |
| Database | Production State、Version、Job、Cost、Manifest、QA、Repair、Approval | DCC 场景与大媒体本体 |
| Blender `.blend` | Authoring Cache、UI State、Local Convenience、Preview | 唯一 Canon、Camera 或 Continuity Truth |

**建议设计。** 本地 MVP 从 Python、typed schemas、SQLite 与文件/对象引用开始；PostgreSQL、Queue、Object Storage 与服务化后置。

## 8. 核心数据模型

### 8.1 领域

- Production：Project、Sequence、Scene、Shot
- World：World、Asset、Reference
- Character：Character、CharacterState
- Prop：Prop、PropState
- Camera：CameraRig、Intrinsics、Extrinsics
- Staging：Staging、KinematicControl、ContactGraph
- Performance：PerformanceIntent、PerformanceRealization
- Continuity：SceneTimeline、ContinuityEdge、StateSample
- Provider：ControlDelivery、GenerationManifest
- Quality：ConformanceResult、RepairTask、Approval
- Build：DependencyEdge、CacheEntry、BenchmarkRun

### 8.2 Shot 最小示例

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

### 8.3 Canonical Coordinate / Time

**P0 ADR 候选默认：** meter、Z-up、right-handed、quaternion `xyzw`、UI degrees、frame origin 0、rational FPS（如 `24000/1001`）。所有边界必须通过显式 Adapter 和 round-trip fixture，避免镜像、×100 尺度、旋转顺序和帧偏移错误。

### 8.4 Camera

- Extrinsics：明确 transform direction、`R`、`T` 与坐标元数据。
- Intrinsics：`K`、image size、pixel aspect 与 projection。
- UI Optics：focal length、sensor、aperture、focus distance。
- Timing：frame rate、shutter、camera/focus curves。
- 可选 Lens Character：distortion 与 anamorphic squeeze。

焦段是 UI 和派生参数，不是 Canonical Camera 的唯一根表达。

### 8.5 Staging、Kinematics、Performance、Contact

- Staging：root position、body/head direction、eye target。
- Kinematic Control：skeleton pose、IK target、trajectory、contact constraint。
- Performance Intent：emotion、intensity、beat、motivation。
- Performance Realization：gaze shift、head turn、pause、gesture、expression 与帧区间。
- Contact Graph：hand/object、foot/floor、character/character、holding、handoff。

每个字段可携带 Constraint Level、Source、Version、Confidence 与 Override Reason。

## 9. 三层约束

| 层级 | 定义 | 典型内容 | 失败处理 |
|---|---|---|---|
| HARD | 不可变化，或变化需明确审批 | identity、costume、structure、camera path、blocking、key prop、contact | 阻断；recompile、repair 或 reroll |
| SOFT | 尽量保持，允许容差内偏差 | light、expression、hair、cloth、fog、secondary timing | 报告偏差；阈值与导演决定 |
| FREE | 主动留给模型 | micro-detail、dust、reflection、small environment motion | 不作为失败 |

解析顺序为 Canon → Sequence → Scene → Shot → Repair Override。近层级显式 Override 优先，但必须记录 `override_reason`。Identity、Canon Asset Version 与 Key Prop 不得被 SOFT/FREE 覆盖。Unsupported HARD 必须失败或产生可见的降级决策。

**待验证假设。** HARD 过多可能导致画面僵硬，MVP 需要寻找 Conformance 与 Natural Variation 的有效边界。

## 10. Scene Timeline 与 Continuity

Edit List 中的 Shot N → Shot N+1 不是通用世界时间。状态必须在 Scene Timeline 上求值，再物化为每个 Shot 的 `scene_event_in` / `scene_event_out`。

| Relation | 语义 | 状态规则 |
|---|---|---|
| `continuous` | 目标紧接来源世界时间 | source end → target start |
| `overlapping_coverage` | 多镜头覆盖同一表演区间 | 采样共享 Timeline，禁止按剪辑顺序串联 |
| `time_jump` | 场景内明确跳时 | 在目标时间求值，仅继承声明为 persistent 的状态 |
| `flashback` | 切到另一叙事时间层 | 切换 branch/snapshot 并记录 narrative anchor |
| `montage` | 压缩或非连续事件 | 每段显式 sample 或局部规则 |
| `parallel_action` | 另一地点同步行动 | 独立 branch，以 sync anchor 关联 |
| `reset` | 明确创作/测试重置 | 从命名 Canon/Snapshot 开始并记录原因 |

状态域包括 Character、Prop、Environment、Camera 与 Narrative State。Camera 是 Shot-specific，不参与 World State 串联。

冲突检查包括：无理由的继承覆盖、一个道具被两人同时持有、Blocking/Frustum 矛盾、服装/灯光/门窗无解释跳变、Contact 指向不存在或不可见对象。

## 11. Rich IR、Compiler 与 Provider Adapter

### 11.1 编译顺序

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

### 11.2 Delivery Status

| Status | 定义 | 示例 |
|---|---|---|
| `native` | Provider 直接接收控制 | camera pose、depth、timestamp edit API |
| `derived` | 由 Canonical Data 确定性派生 | 从 K / R/T 渲染 depth/camera reference |
| `emulated` | 用另一媒介近似表达 | clay video 或 prompt 模拟 camera path |
| `unsupported` | 无可靠交付路径 | 阻断 HARD 或要求批准降级 |

Expected Conformance 不应写成主观固定小数。应按 Provider、Version、Capability、Representation、Fixture、样本量、日期和误差分布校准定性等级。

### 11.3 Adapter Contract

```python
class VideoProvider:
    def capabilities(self) -> ProviderCapabilities: ...
    def compile_control_package(self, shot_ir) -> ProviderPackage: ...
    def generate(self, request) -> JobId: ...
    def status(self, job_id) -> JobStatus: ...
    def fetch_result(self, job_id) -> GeneratedAsset: ...
    def edit(self, request) -> JobId: ...
```

能力声明需要包含 status、limits、formats、emulated_by、known_failures、calibration_ref、provider_version、adapter_version 和 `last_verified_at`。Provider 宣传不能直接等同为已验证集成结果。

### 11.4 Control Package

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

验收要求包括确定性结构哈希、Unsupported HARD 的显式处理、Adapter 不读取 Blender 私有状态，以及 Canon/Shot/Reference/Compiler/Adapter/Provider 全链路追溯。

## 12. Blender Previz / Control Runtime

Blender 是第一阶段 Authoring / Control Runtime，不是 Canon Owner。

MVP 操作包括 Shot load/save/navigation；Camera create/load/save/preview；Staging、Character/Prop Marker；粗 rig、IK、Contact、Motion Preview；Clay/Depth/Normal/Segmentation/Camera Preview；Validate、Export 与 Manifest 查看。

UI 可以显示 focal length、sensor、aperture、focus，但持久化使用 Canonical K、R/T、projection、image size、timebase、shutter 与 curves。所有对象必须有 Stable ID；保存只回写允许字段；所有 Control Render 都进入 GenerationManifest。

## 13. Conformance、Technical QA 与 Repair

```text
Control Intent ───────────────┐
Control Delivery ─────────────┼─> Conformance Analysis
Generated Video ──────────────┘          ↓
                              Difference + Evidence
                                         ↓
                  Accept / Review / Recompile / Repair /
                              Regenerate / Block
```

Technical QA 检查明确、可复核的技术约束；Creative Approval 始终由人完成。

候选自动证据包括 Manifest/State consistency、Prop presence、Identity similarity、Screen-space blocking、Pose/Trajectory、Background feature tracks、Horizon、Optical flow 与 Clay alignment。系统不承诺准确逆推出 focal length，也不承诺自动判断所有 Physics 或 Emotion。

Camera Conformance 应比较 Expected Clay / Camera Reference 与 Screen-space Motion、Subject Bounds、Framing、Horizon、Vanishing Behaviour、Camera-induced Flow，而不是武断地声称生成视频“就是精确 50mm”。

ConformanceResult 至少包含 Category、Constraint Level、Severity、Confidence、Frame/Time Range、Spatial Region/Mask、Expected/Observed、Evidence、Intent/Delivery/Output Ref、Metric/Tolerance/Deviation、Recommended Action、Model/Rule Version 与 Human Override。

Repair 可以限定为小区域、短时间、单控制通道或全局 HARD 失败。局部 Repair 是探索性收益，不是 MVP Gate；可回退到更强 Recompile、扩大编辑范围、切换 Provider、整段 reroll 或返回 Shot Design。

## 14. Cora Lab MVP 与 A/B Benchmark

### Shot Design

| Shot | 叙事动作 | 关键控制 | 结果状态 |
|---|---|---|---|
| `LAB_001` | Cora 整理 Seed Tray，出现微弱反应 | Lab Canon、Costume、50mm、Desk Blocking、Tray | Cora 注意到光；Tray 在桌上 |
| `LAB_002` | Cora 靠近并伸手 | Gaze、Push-in、RightHand Contact | 手接触 Tray；门关闭；午后光 |
| `LAB_003` | 反应增强，Cora 拿起 Tray | Holder Change、Start State 继承、Soft Expression | Tray in hand；Cora at LabDesk_B |
| `LAB_002B` | 同一表演时刻的侧面 Coverage | Overlapping Coverage、共享 Event Time、独立 K/R/T | 不产生新 World End State |

4 个设计可扩展为 10 个固定 Benchmark Shot。

### MVP Scope

- 一个 World/Scene Canon：Cora、Seed Tray、Desk、Door、Light；
- Canonical Coordinate/Time、Camera K/R/T、Staging、Kinematics 的 load/edit/save/playback；
- Scene Timeline、Relation、State Materialization、Override 与 Conflict；
- HARD/SOFT/FREE；
- MockProvider + 1 个真实 Adapter；
- Package、Manifest、Job、Cost；
- 至少 Identity、Camera/Framing、Blocking、Prop、Continuity 五类 Conformance；
- Dependency Graph 与精确 Cache Invalidation；
- Baseline A / System B 数据采集；
- Repair Planner 与可复现 Fixture，真实局部 Repair 为 bonus。

### 验收

- 相同版本输入重建相同结构文件与哈希，排除声明的时间字段；
- Continuous 与 Coverage 状态求值正确；
- Blender round-trip 保持 Camera、Staging、Rig/IK/Contact、Stable ID 与 Transform 在容差内；
- Provider 差异只存在于 Adapter Output；
- Conformance 输出结构化 Difference、Evidence 与 Action；
- Dependency Change 只使正确下游失效；
- 每次 Generation 可追溯到输入版本、Compiler、Adapter、Provider、Cost、Output 与 Approval；
- A/B 报告包含 Time to Approved Shot、reroll、active human time、cost 和 failure category，而不是只展示最好看的一次。

## 15. 候选 Roadmap 与资源

下列内容是规划框架，不是已承诺排期。

| Milestone | 候选周 | 主要交付 | 退出条件 |
|---|---:|---|---|
| M0 基础 | 1–2 | ID/Version、Coordinate/Time ADR、Timeline、Schema、SQLite、Fixture | State/Camera round-trip 通过 |
| M1 Blender | 3–5 | K/R/T、Staging、粗 rig、IK/Contact、Control Render | Cora Lab continuous + coverage 往返 |
| M2 Compiler | 6–7 | Rich IR、Delivery、MockProvider、Graph、Manifest | 确定性编译与失效 |
| M3 Provider | 8–9 | 1 个真实 Adapter、Job、Result、Cost | 完成一条真实链路 |
| M4 Conformance | 10–11 | 5 类 Evidence、Action Planning | Intent → Difference → Action |
| M5 Benchmark | 12 | Baseline A/System B 与 Go/No-Go | 依据生产指标决策 |

候选投入包括 Product/Director Lead、Pipeline Tech Lead、Blender Technical Artist、Backend/Tooling Engineer、兼职 CV/QA 与兼职 Production QA/UI。这是源文档估算，不代表当前已有团队或承诺。

## 16. 商业价值与潜在护城河

潜在价值包括降低返工、提高镜头可预测性、降低 Provider 锁定、积累可复用 Production State、连接创作与工程，以及给制片提供状态/成本/失败/审批可见性。

可能持续积累的资产包括统一 Shot/State Schema、Capability-aware Compiler、Scene Timeline/Continuity Graph、Conformance/Repair Evidence、DCC Integration、Cross-provider Benchmark、Generation History 与 Approval History。

护城河不能是单一 Camera workaround。随着 Provider 原生吸收 Camera、Motion、Identity 与 Editing，Delivery Mechanism 应该变简单，而 Canon、Timeline、State、Conformance、Versioning 与 Audit 继续保留价值。

## 17. 风险与应对

| 风险 | 影响 | 应对 / 决策门槛 |
|---|---|---|
| Provider 吸收原生控制 | 单项控制商品化 | 聚焦 Canon、Timeline、Compiler、Conformance、History、DCC Workflow |
| API 快速变化 | Adapter/Delivery 过期 | Versioned Capability、Contract Test、定期校准 |
| 过度约束 | 合规但僵硬 | 限制 HARD、保留 FREE、人工 Naturalness 评分 |
| Schema 过早膨胀 | 开发慢、字段不用 | 只从 Cora Lab 证据新增字段 |
| USD 范围过大 | 基础设施拖累 MVP | 保持最小 Spatial Boundary |
| QA 误报/漏报 | 阻塞创作或放过失败 | Evidence、Confidence、Threshold、Human Override |
| Repair 不可靠 | 修复失败或破坏其他内容 | 不把 Repair 设为 Gate |
| 坐标/时间错误 | 镜像、尺度、旋转、帧偏移 | P0 ADR、Adapter、Golden Round-trip Test |
| DCC/Canon 双写 | State Drift | Stable ID、Controlled Write、Conflict Block |
| Demo 好看但不可重复 | 误判成熟度 | Fixed Fixtures、Repeated Runs、Hashes、Metric Report |
| 版权与数据治理 | Reference/Output 无法安全复用 | License/Source Metadata、Access、Retention、Deletion |

## 18. 成功指标与 Go/No-Go

所有阈值在 Baseline 校准前都只是候选。

| 指标 | 定义 | 候选目标 |
|---|---|---|
| 确定性编译 | 相同输入/版本得到相同结构哈希 | 100% |
| 可追溯 | Generation 链接 Canon、Shot、Compiler、Adapter、Cost、Approval | 100% |
| 状态继承覆盖 | 要求继承字段被解析并检查 | 100% |
| HARD 通过率 | Fixed Set 中关键约束通过比例 | ≥90%，待校准 |
| Camera/Blocking 往返 | 关键数据重载在容差内 | 100% 结构一致 |
| Time to Approved Shot | 从开始到 Technical Pass + Director Approval 的墙钟时间 | 核心 KPI；相对 Baseline 显著下降 |
| Active Human Time | 创意/技术主动操作分钟 | Previz 开销小于 reroll/repair 节省 |
| 整段 reroll | Approved 前完整重生成次数 | 下降 ≥30%，候选 |
| Repair Success | 修复通过且不引入新 HARD Failure | ≥50%，探索性 |
| Provider Migration Reuse | 切 Provider 后复用 Shot Design 字段比例 | ≥80%，候选 |
| New Adapter Cycle | 稳定 API 后接入工程时间 | ≤2 工程周，候选 |

**Go：** 确定性编译、Timeline/Continuity、Invalidation 与 Traceability 达到 100%，且 Cora Lab 在至少一个真实 Provider 上缩短 Time to Approved Shot 或显著减少关键失败/重做。  
**Adjust：** 如果 Previz 成本大于 reroll 节省，缩小交互或目标 Shot 类型。  
**No-Go：** 应由 Production Value 与 Core Control Evidence 失败决定，而不是仅由局部 Repair 失败决定。

## 19. 若恢复实施的顺序

1. 通过 Coordinate/Time ADR、ID、Version、Typed Schema 与 Golden Fixtures。
2. 实现 Scene Timeline、Continuity Edge、State Evaluation、Override、Coverage 与 Conflict Test。
3. 实现 Blender Load/Save、Canonical Camera、Staging、粗 rig、IK/Contact 与 Control Render。
4. 实现 Rich IR、Delivery Record、MockProvider、Dependency Graph、Deterministic Compiler 与 Manifest。
5. 接 1 个真实 Provider 与 Job Lifecycle。
6. 实现 5 类 Conformance 与 Difference-to-Action。
7. 运行固定 Baseline A/System B，并公开基于证据的决策。

建议未来代码边界：

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

## 20. 待决策问题

P0 需要 Owner、Deadline、Evidence 与 Decision Record：

- Canonical Coordinate/Time 与 round-trip tolerance；
- Scene Timeline 语义与 Coverage Fixture；
- Canonical Camera 与 projection equivalence；
- 依据实际 API 可用性和可测试性选择第一个真实 Provider；
- 固定 MVP Benchmark 与 Baseline Procedure；
- 最小 USD Scope；
- 各类别 HARD Constraint Tolerance。

P1 包括 SQLite/File Storage 边界、Conformance 技术路线、Fidelity Calibration、Repair Planner 最小行为与首批内部用户。Unreal 接入与 Multi-tenant Service 为 P2。

## 21. 持续维护规则

- 主文档保留定位、边界、决策与 Milestone Status；实现细节拆入 TECH_SPEC、ADR 与 Backlog。
- 已确定设计的变更记录原因、影响、替代方案与生效版本。
- 建议设计只有在评审或实验后升级。
- 待验证假设绑定 Metric、Fixture、Owner 与 Decision Date。
- Provider Capability 记录 Status、Evidence、Calibration Set、Verification Date、Provider Version 与 Adapter Version。
- 每个 Milestone 后更新 Risk、Metric、Resource 与 Go/No-Go。

## 22. 术语

- **Canon：** 项目当前正式批准的 World、Character、Asset 与 Fact 版本。
- **World State：** 场景与对象在某一时刻的显式状态。
- **Shot State：** Shot 所需 Camera、Blocking、Performance、Constraint 与采样 Start/End State。
- **Scene Timeline：** 独立于剪辑顺序的 Event/Beat Timeline。
- **Continuity Edge：** Shot/Event 之间的显式时间与状态关系。
- **Control Intent：** 导演要求什么。
- **Control Delivery：** Compiler 如何把控制交给 Provider。
- **Control Conformance：** Output 相对 Intent/Delivery 的证据化偏差。
- **Rich IR：** 保留完整意图的 Provider-neutral Intermediate Representation。
- **Generation Manifest：** 一次生成的 Input、Version、Parameter、Cost、Output、Hash 与 Parent 记录。
- **Technical QA：** 对显式技术约束的检测，不是艺术判断。
- **Creative Approval：** 人对镜头艺术成功与否的最终判断。

## 23. 证据边界

Provider 官方页面、开源工作流和研究论文可以证明单项 Control Mechanism 的可行性，但不能证明本系统已经拥有可用 API Integration、稳定 Conformance、更好 ROI 或 Product-Market Fit。持续维护的证据图谱见 [Related Work](../research/RELATED_WORK.md)。

