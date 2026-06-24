# 决策记录：Deep Research 产出综合判断

> 日期：2026-06-18
> 来源：[05-外部调研/01-deep-research-产出.md](../../05-外部调研/01-deep-research-产出.md)

---

## 一、三条关键修正

Deep research 独立调研对 Phase 1 的三项评估提出了修正：

### 修正 1：Grounding Failure 的文献基础比我们想的更丰富

| Phase 1 认为 | Deep research 发现 |
|-------------|-------------------|
| 3 篇独立论文 | **≥7 篇**，含 Tsinghua/MSRA 的 Causal-Plan-Bench（2026，1,200 实例，12 任务类别） |
| 防御覆盖率接近 0% | **已有防御但全在规划生成侧**：ConceptAgent（前置条件验证）、DDCG（NeurIPS 2025，可行性/质量判据分离）、ContextMatters（目标松弛）、Causal Reasoner（因果推理训练）。**但没有任何一个做独立于 LLM 的运行时物理校验。** |

**这利好而非削弱我们的方向**：竞争者的存在证明"接地失效"是社区共识问题，而它们全部走「优化 LLM 规划能力」路线，我们走「不信任 LLM、独立验证」路线——差异化更清晰了。

### 修正 2：Pre-execution 验证不是零篇空白

| Phase 1 认为 | Deep research 发现 |
|-------------|-------------------|
| 指令执行前验证零篇 | **至少 5 个已发表系统**：VerifyLLM (IROS 2025)、SAFER (IROS 2025)、Asimov Box (Princeton)、Joint Verification (2024)、ILION (2025) |

**但这同样利好我们**：这些系统全部用形式化方法（LTL/automata）、几何约束或语义安全过滤。**没有任何一个从传感器数据实时提取物理参数（摩擦系数、倾斜角、承重能力）来做安全判据。** 我们的定位从「第一个做 pre-execution 验证的」调整为「第一个做 physics-grounded pre-execution 验证的」——更精确也更站得住。

### 修正 3：纯「TEE + 具身AI」已非全新方向

| Phase 1 认为 | Deep research 发现 |
|-------------|-------------------|
| 技术储备充足但应用完全空白 | PARTEE (ACSAC 2025)：Raspberry Pi TrustZone 保护无人机；DeepTrust^RT (2025)：OP-TEE 运行 DNN 推理；TZ-DATASHIELD (NDSS 2025)：TrustZone 数据流保护；中国具身智能机器人安全技术白皮书 (2026) |

**这直接验证了导师的直觉——如果纯 TEE+具身是我们的核心贡献，确实不够新。** 但关键差异化在于：现有工作保护的是 DNN 推理/数据/可用性，**没有一个保护 LLM 输出的物理安全判据的正确执行**。TEE 的角色从「核心贡献」降级为「工程增强」，与我们之前的定位校准一致。

---

## 二、新发现的重要论文

### Blindfold（SenSys 2026）——动作级越狱，Phase 1 未覆盖

- 将恶意意图分解为表面无害的原子动作序列：如「炸掉手机」→ `find(phone) → pick(phone) → move(oven) → stretch()`
- GPT-4o 上 93.2% ASR，Phi-4-14B 上 98.1%，绕过所有现有防御
- 在真实 6DoF 机械臂上验证
- **这是我们防御的最强动机来源**

### Causal-Plan-Bench（Tsinghua/MSRA, 2026）

- 1,200 实例，12 任务类别，目前最大规模的 grounding failure 诊断基准
- 核心论断：「当前 VLM 是表面 token 预测器，而非物理因果推理器」
- **直接证明了 grounding failure 是系统性缺陷，不是偶发 bug**

---

## 三、方向的精确化调整

基于以上修正，将之前的「候选 10」升级为更精确的定义：

```
调整前：接地失效的物理可行性主动验证（候选 10）
调整后：物理接地的执行前安全验证（Physics-Grounded Pre-Execution Safety Validation）
```

| 维度 | 调整前 | 调整后 |
|------|--------|--------|
| 问题来源 | Grounding Failure（3 篇文献） | Grounding Failure（7+ 篇）+ Pre-execution verification（5 篇系统）+ Blindfold 动作级越狱 |
| 差异化 | "无人做过 pre-execution 验证" | "已有 pre-execution 验证但全是形式化/语义/几何，无人做物理参数提取" |
| 硬件角色 | TEE 作为方案亮点 | TEE 作为工程增强（有先例），**物理判据设计是核心** |
| 实验基线 | SafeLab 等基准 | 可增加 Blindfold 作为攻击基线 |

---

## 四、问题定义框架（"已有工作→做到什么程度→欠缺什么"）

### 4.1 社区已识别的问题

Grounding Failure：LLM planner 在具身场景中产生逻辑自洽但物理不可行的计划。
- 定量证据：Chakraborty et al. 发现幻觉率↑40%；Causal-Plan-Bench（1,200 实例）系统诊断
- 经典案例：Han et al.「火灾→服务器机房」
- 安全后果：Blindfold 证明利用此缺陷可实现 93.2% ASR 的动作级越狱

### 4.2 已有工作做到了什么程度

| 路线 | 代表工作 | 做到了什么 | 做不到什么 |
|------|---------|-----------|-----------|
| **优化 LLM 规划** | ConceptAgent, DDCG (NeurIPS 2025), Causal Reasoner | 提高 LLM 生成计划的质量，减少接地失效 | 无法保证——LLM 仍可被越狱（Blindfold 绕过所有现有防御） |
| **形式化 pre-execution 验证** | VerifyLLM (IROS 2025), SAFER (IROS 2025), ILION | 用 LTL/automata/几何约束做执行前验证 | 不接入传感器数据——不知道当前地面摩擦系数是多少 |
| **TEE 保护机器人** | PARTEE (ACSAC 2025), DeepTrust^RT | 用 TEE 保护 DNN 推理/数据/可用性 | 不保护 LLM 输出的物理安全判据 |

### 4.3 欠缺什么（= 我们的切入空间）

**欠缺一个独立于 LLM 规划器的、从传感器数据提取物理参数的、执行前安全校验层。**

- 独立于 LLM：不依赖 LLM 的自我审查（因为 LLM 不可信）
- 传感器驱动：从 IMU/力传感器/深度相机提取摩擦系数、倾斜角、承重能力
- 执行前：在物理执行前做最后的拦截判断
- 物理判据：用显式物理公式而非 AI 做判据（「不能让 LLM 审查 LLM」）
- 隔离部署（可选）：TEE 防止被已攻陷的 OS 绕过

---

## 五、下一步

1. 将此方向定义写入 Phase 2 prompt，让执行 Agent 做深度文献追踪
2. 核心验证目标：
   - ConceptAgent / DDCG / VerifyLLM / SAFER 的精确局限在哪里（需读原文）
   - Blindfold 的攻击机制是否可以复现作为实验基线
   - 物理参数提取（摩擦系数等）在 Gazebo 中的可行性
3. TEE 不作为 prompt 强调重点——先聚焦问题定义和方法差异化
