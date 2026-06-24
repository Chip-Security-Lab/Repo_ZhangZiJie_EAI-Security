# Phase 2 Prompt：关键论文深度追踪与差异化分析

---

## ⚠️ 执行指令（必读）

1. **产出写入**：完成后将报告写入：
   ```
   E:\BaiduSyncdisk\Enbodied AI\组\prompt\03-产出\02-phase2-深度追踪报告.md
   ```
2. **输入素材**：执行前阅读以下文件了解完整背景：
   - `E:\BaiduSyncdisk\Enbodied AI\组\prompt\03-产出\01-phase1-候选问题池.md`（12 候选池）
   - `E:\BaiduSyncdisk\Enbodied AI\组\prompt\05-外部调研\01-deep-research-产出.md`（独立调研报告）
   - `E:\BaiduSyncdisk\Enbodied AI\组\prompt\04-决策\03-deep-research综合判断.md`（最新方向决策）
3. **原始文献**：`E:\BaiduSyncdisk\Enbodied AI\组\Security Review\` 下有前期综述分析可供参考

---

## 背景：我们当前的问题定义（草稿）

经过两轮调研和讨论，我们将研究问题收敛为：

> **LLM 规划器在具身场景中会产生 grounding failure——它生成计划时隐含地假设了某些物理前提成立（如「地面能走」「这个物体能当工具用」「把任务分解为无害原子动作后物理后果不变」），但这些前提在真实物理环境中可能不成立。LLM 自己是意识不到这一点的——Causal-Plan-Bench 已证明当前 VLM 是「表面 token 预测器」而非物理因果推理器。**
>
> **我们的方向：设计一个独立于 LLM 规划器的物理前提验证层，从传感器数据提取真实物理参数，检测 LLM 计划中隐含的物理前提是否在当前环境中成立，在不成立时拦截执行。**

**这不是「验证指令是否物理可执行」**（那样会帮攻击者确认恶意指令的可行性）。**这是「检测 LLM 的物理理解与传感器观测到的真实物理状态是否一致」**——拦截的标准是前提不成立，而不是动作不可行。

**当前阶段不做方法设计、不提 TEE。只定义问题。**

---

## 你的任务

深度阅读以下 8 篇论文原文，产出每篇的精确分析。核心目标：**坐实「他们做了什么、做到什么程度、哪里没覆盖」——这是问题定义中最关键的部分。**

---

## 论文清单

### 攻击动机（证明问题存在）

| # | 论文 | 出处 | 读它的目的 |
|---|------|------|-----------|
| 1 | **Blindfold: Jailbreaking Embodied LLMs via Action-level Manipulation** (Huang et al., 2026) | SenSys 2026 | 动作级越狱的具体机制——它如何分解恶意意图、为什么现有防御全部失效。这是我们防御的最强动机。 |
| 2 | **Causal-Plan-Bench / Token Predictors Are Not Planners** (Lu et al., 2026) | arXiv:2606.01810, Tsinghua/MSRA | 「VLM 是表面 token 预测器而非物理因果推理器」的证据细节——这个论断直接支撑我们问题的存在性。 |
| 3 | **BADROBOT: Jailbreaking Embodied LLM Agents in the Physical World** (Zhang et al., 2025) | ICLR 2025 | 三种攻击路径（Contextual Jailbreak / Safety Misalignment / Conceptual Deception）的原始出处。确认其威胁模型和我们防御目标的对应关系。 |

### 规划侧防御（他们走的路 vs 我们要走的路）

| # | 论文 | 出处 | 读它的目的 |
|---|------|------|-----------|
| 4 | **ConceptAgent: LLM-Driven Precondition Grounding and Tree Search for Robust Task Planning** (Rivera et al., 2024) | arXiv:2410.06108 | **最接近我们思路的前置工作**——LLM 生成 predicate 前置条件 + 执行前检查。必须精确理解其方法，才能清楚说出我们和它的区别。 |
| 5 | **DDCG: Decoupled Dual-Critic Guidance for Embodied Agents** (Ma et al., 2025) | NeurIPS 2025 | 「可行性判据 C_F + 质量判据 C_Q 分离」框架。它的 C_F 是怎么定义和训练的？为什么它仍依赖 LLM 组件而非独立传感器？ |

### Pre-execution 验证（同一范式，不同手段）

| # | 论文 | 出处 | 读它的目的 |
|---|------|------|-----------|
| 6 | **VerifyLLM: LLM-Based Pre-Execution Task Plan Verification for Robots** (Grigorev et al., 2025) | IROS 2025 | LLM+LTL 形式化验证——pre-execution verification 的代表性工作。它的形式化规范从哪来？覆盖了哪些物理场景、漏了哪些？ |
| 7 | **SAFER: Safety Aware Task Planning via Large Language Models in Robotics** (Khan et al., 2025) | IROS 2025 | 多 LLM（Task LLM + Safety LLM）+ CBF。核心问题：Safety LLM 本身是否也会被越狱？「LLM 审查 LLM」的架构假设是什么？ |
| 8 | **ILION: Deterministic Geometric Verification** (2025) | Zenodo/tech report | 确定性几何验证。它的「几何验证」和「物理验证」的边界在哪里？为什么几何正确 ≠ 物理安全？ |

---

## 对每篇论文的分析要求

每篇论文按以下结构产出（不需要八股文，但信息要完整）：

```
### [论文简称]

**核心方法**（3-5 句话：输入是什么、做什么处理、输出是什么）

**实验设置**（什么环境、什么机器人/仿真、什么任务）

**关键数据**（成功率和我们关心的指标——特别是：在什么条件下失败？失败的 case 长什么样？）

**安全假设**（这篇论文的安全模型假设了什么？哪些组件是可信的、哪些是不可信的？）

**对我们的意义**（1-2 句话：这篇论文支撑了我们问题定义的哪个部分？是攻击动机、差异化对照、还是方法借鉴？）

**局限/盲区**（从我们的视角看，这篇论文没有覆盖什么？为什么它的方法不能直接解决我们定义的问题？）
```

---

## 最后产出：差异化矩阵

在 8 篇论文分析完成后，产出一张汇总表：

| 论文 | 做了什么 | 物理前提验证？ | 独立于 LLM？ | 接入传感器？ | 能拦 Blindfold？ |
|------|---------|:--:|:--:|:--:|:--:|
| ConceptAgent | LLM 生成前置条件 + 检查 | 部分（仅已知前提） | ❌ | ❌ | ❌ |
| DDCG | 可行性/质量判据分离 | 部分（训练数据驱动） | ❌ | ❌ | ❌ |
| VerifyLLM | LTL 形式化验证 | ❌（形式化规范） | ✅ | ❌ | ❌ |
| SAFER | 多 LLM + CBF | ❌（语义+控制） | ❌（Safety LLM 仍是 LLM） | ❌ | ❌ |
| ILION | 确定性几何验证 | ❌（几何≠物理） | ✅ | ❌ | ❌ |
| **我们的方向** | **传感器驱动物理前提验证** | **✅** | **✅** | **✅** | **✅** |

---

## 特别说明

1. **搜索论文**：如果部分论文（特别是 IROS、SenSys、Zenodo 上的）在公开渠道无法直接获取全文，请标注并尝试通过 arXiv 版本、作者主页、Google Scholar 获取。确实无法获取的，基于摘要和引用信息给出分析，标注「基于摘要分析，待获取全文确认」。

2. **区分事实和判断**：对论文方法的描述是事实（需要准确），对盲区的分析是你的判断（需要标注推理依据）。不要混在一起。

3. **不确定就标注**：如果某个论文的某个细节你无法从原文中找到，标注「原文此点未明确」，不要脑补。

4. **关注失败案例**：每篇论文的实验部分，特别关注它们报告的失败模式——这些失败案例往往直接落入我们想覆盖的盲区。
