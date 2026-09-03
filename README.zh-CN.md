# 生成式影视控制层

[English](README.md) | [简体中文](README.zh-CN.md)

> 一套位于导演意图与生成式视频系统之间、模型无关的影视生产控制层研究架构。

**当前状态：Research / Architecture Ready — Implementation Deferred（研究与架构已就绪，实施暂缓）**

本仓库公开《Control Layer for Generative Filmmaking》v0.2 项目方案。它不是一个新的视频基础模型，也不是已经验证的产品。它描述的是一套可能的生产系统：保存影视生产状态，把导演意图转换为显式控制，为不同生成 Provider 编译控制包，并衡量生成结果与镜头设计之间的符合程度。

当前阶段没有宣称已完成实现、真实生产 Benchmark 或性能结果。Cora Lab A/B Benchmark 与所有数值目标均属于后续待验证工作。

## 要解决的问题

生成式视频画质正在快速提升，但生产控制仍然脆弱：

- 相机几何和运动经常被压缩成自然语言；
- 人物站位、视线、道具、接触关系和环境状态容易漂移；
- 相邻镜头和重叠 Coverage 缺少可靠的共享场景状态；
- 局部错误可能迫使整段重新生成；
- 镜头意图容易被锁进某个 Provider 的输入格式；
- 生成输入、版本、成本、输出、QA 证据和审批难以统一追溯。

核心命题是：

> Control Layer 的价值，是建立导演意图与生成结果之间的可计算关系。

## 系统提案

```text
导演意图
   ↓
Production Canon + Scene Timeline
   ↓
Canonical Shot State
   ↓
模型无关 Rich IR
   ↓
Control Compiler + Provider Adapter
   ↓
Control Delivery → Generated Video
   ↓
Conformance Analysis → Difference + Evidence
   ↓
接受 / 复核 / 重编译 / 修复 / 重新生成
```

方案刻意区分三件事：

1. **Control Intent**：导演要求什么。
2. **Control Delivery**：Provider 实际收到了什么，属于 native、derived、emulated 还是 unsupported。
3. **Control Conformance**：结果相对意图和实际交付产生了什么偏差。

生成结果不会自动成为 Canon；Human Creative Approval 始终拥有最终权威。

## 核心架构

### Director Intent 与约束

Camera、Staging、Performance、Contact 和 Continuity 都以显式数据表达。每个可控属性可以标记为：

- **HARD**：不可变化，或变化必须明确审批；
- **SOFT**：应尽量保持，允许在容差内偏差；
- **FREE**：主动留给模型发挥。

### Production Canon

系统保存世界、角色、服装、道具和批准资产的版本化事实。Shot 引用不可变快照，而不是复制或暗中修改整套世界定义。

### Scene Timeline 与 Continuity Edge

剪辑顺序不等于世界时间。每个 Shot 从共享的 Scene Event Timeline 采样一个区间，并通过显式关系边表达 `continuous`、`overlapping_coverage`、`time_jump`、`flashback`、`montage`、`parallel_action` 和 `reset`。

这对 Coverage 尤其重要：多个机位可以观察同一段表演，而不应该继承剪辑列表中“上一镜”的结束状态。

### Camera Geometry

Canonical Camera 以 Intrinsics / Extrinsics 为根，而不是只保存“50mm”标签：

- Intrinsics `K`、投影类型、图像尺寸和像素宽高比；
- Extrinsics `R`、`T` 与明确的坐标元数据；
- focal length、sensor、aperture、focus、shutter、frame rate 和动画曲线；
- Blender、USD、Unreal 与 Provider 边界的显式 Adapter。

候选 Canonical 约定为 meter、Z-up、右手系、quaternion rotation、frame origin 0 和 rational FPS。它仍是待通过 round-trip fixtures 验证的 P0 ADR，而非既定实现事实。

### Rich IR 与 Control Delivery

模型无关 **Rich IR** 保留比任一 Provider 更丰富的意图。Compiler 根据版本化能力选择最强表达：

- `native`：Provider 原生接收；
- `derived`：由 Canonical 数据确定性派生；
- `emulated`：通过 Clay Reference 等替代媒介近似表达；
- `unsupported`：无法可靠交付。

如果某个 HARD Control 不受支持，系统必须阻断编译，或产生有证据、经过批准的降级记录，绝不能静默丢弃。

### Provider Adapter

Provider 专属逻辑只能存在于 Adapter。稳定接口负责能力声明、包编译、生成、状态查询、结果抓取和编辑。更换 Provider 应只需要重新编译，而不是重新设计镜头。

### Conformance

Conformance 同时比较 Intent、Delivery 和 Generated Output。结果应包含类别、严重度、置信度、时间/帧范围、空间证据、预期值、观测值和建议动作。

Technical QA 可以辅助检查身份、构图、Blocking、道具和 Continuity 等明确约束，但不会自动判断表演、节奏、构图、情绪或叙事是否成功。

### Dependency Graph

派生产物按内容寻址，并记录上游依赖。某项变化应只使相关 Shot IR、Provider Package 和 QA Baseline 失效，同时保留无关镜头、Approved Canon 与原始 Provider 输出。

## Cora Lab 验证提案

MVP 被定义为生产 A/B Benchmark，而不是“做出一个好看的 Demo”。最小叙事是 Cora 在实验室发现种子发生反应：

- 3 个连续镜头；
- 1 个采样同一表演区间的重叠 Coverage Shot；
- 最小 World / Character Canon；
- Camera、Staging、Gaze、Prop 与 Hand-to-Tray Contact 控制；
- MockProvider + 1 个真实 Provider Adapter；
- 至少 5 类 Conformance；
- 同 Provider、同预算的 Baseline A / System B 对照。

首要未来指标是 **Time to Approved Shot**。其他候选指标包括确定性编译、可追溯覆盖、连续性覆盖、Hard Constraint 通过率、人工操作时间、整段重生成次数、Provider 迁移复用率和 Adapter 接入周期。

这些都是实验标准，不是已经取得的结果。

## Roadmap

由于 Provider 能力仍在快速变化，项目目前主动暂缓实施。未来若恢复，候选顺序为：

1. **M0 — 基础：** ID、版本、坐标/时间 ADR、typed schemas、Scene Timeline、SQLite 与 golden fixtures。
2. **M1 — Blender Runtime：** Camera 往返、Staging、粗 rig、IK/Contact 与控制通道渲染。
3. **M2 — Compiler：** Rich IR、Delivery/Fidelity、MockProvider、Dependency Graph 与确定性 Manifest。
4. **M3 — 真实 Provider：** 1 个 Adapter、Job 生命周期、结果与成本记录。
5. **M4 — Conformance：** 5 类结构化证据与 Difference-to-Action 规划。
6. **M5 — Benchmark：** 固定 Baseline A / System B 测试集与基于证据的 Go/No-Go。

v0.2 中的 12 周只是候选规划框架，不是已经承诺的排期。

## 为什么公开

我选择公开这份方案，因为对我而言，这些思路并不只是一个需要占有的商业点子，而是我所领受、也愿意分享出来继续被验证、修正和发展的东西。希望它能对正在探索生成式影视未来工作流的人有所帮助。

现在公开，也能诚实地表达项目状态：研究与系统架构已可讨论，但实现与真实生产证据仍然暂缓。况且我有什么不是领受的呢？白白地得来，也要白白地舍去。。。

## Related Work

现有开源工作已经证明 Control Delivery 路线中的重要部分，例如 Blender 白模、Camera Previs、Reference Video、Motion Guidance 与 Provider Adapter。本项目不声称发明这些技术。它试图补充的是外围的 Production State 层：Canon、共享 Scene Time、Continuity Relation、Rich IR、Conformance、Dependency、Manifest 与 Approval History。

一手证据、适用边界与链接见 [research/RELATED_WORK.md](research/RELATED_WORK.md)。

## 仓库结构

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

## 完整文档

- [Project Plan v0.2 — English](docs/PROJECT_PLAN_v0.2_EN.md)
- [项目计划 v0.2 — 中文](docs/PROJECT_PLAN_v0.2_ZH-CN.md)
- [Architecture — English](docs/ARCHITECTURE_EN.md)
- [架构说明 — 中文](docs/ARCHITECTURE_ZH-CN.md)

## License 建议

当前仓库包没有擅自写入正式 License。建议未来采用：

- 文档与图示：**CC BY 4.0**；
- 如果开始实现代码：**Apache License 2.0**。

这种双许可证结构既保留公开研究的署名要求，也让未来代码拥有带明确专利授权的常见宽松许可证。详情见 [LICENSE_RECOMMENDATION.md](LICENSE_RECOMMENDATION.md)。只有在仓库所有者确认后，才应加入正式 `LICENSE` 文件。

## 建议的 GitHub 信息

- **仓库名：** `control-layer-for-generative-filmmaking`
- **Description：** `Research architecture for a provider-agnostic control, continuity, and conformance layer for generative filmmaking.`
- **Visibility：** Public
- **Topics：** `generative-video`、`filmmaking`、`blender`、`previs`、`camera-control`、`continuity`、`production-pipeline`、`ai-video`

欢迎通过讨论、证据、实验、批评和纠错参与。请明确区分 verified behavior 与 proposed design，不要把 Provider 的宣传表述包装成本项目已经测得的 Conformance 结果。
