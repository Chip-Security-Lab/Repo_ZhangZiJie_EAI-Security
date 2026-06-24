# 四篇核心论文深度分析（PPT 素材）

> **用途**：直接制作 PPT 讲给导师  
> **日期**：2026-06-18  
> **数据源**：三篇 PDF 原文（pdftotext 提取）+ DDCG PDF 全文（Phase 2 Part E 已提取）

---

## 论文 1：BADROBOT

**完整引用**：Zhang et al., "BADROBOT: Jailbreaking Embodied LLM Agents in the Physical World," ICLR 2025.  
**读的版本**：arXiv:2407.20242v5, 2026.06

---

### 1.1 解决什么问题

**核心问题陈述**：

> 当 LLM 被用作机器人的任务规划器时，攻击者能否通过自然语言越狱，让机器人执行物理危险动作？BADROBOT 首次证明这是可行的——LLM 的安全对齐在文本空间完成，但物理执行不受文本安全约束。

**为什么传统机器人没有这个问题**：
- 传统规划器（A*/RRT/MPC）是数学确定性的——没有语义理解能力，无法被自然语言「说服」
- 传统 LLM 越狱（DAN 等）只影响文本输出——chatbot 没有手
- **具身 LLM = 语义可被说服 + 输出控制物理实体 → 新攻击面**

---

### 1.2 怎么解决的（攻击方法详解）

#### 路径一：Contextual Jailbreak（上下文越狱）

**大白话**：攻击者把 LLM 通用越狱模板（DAN、角色扮演等）适配到具身场景。传统越狱让 LLM 说脏话——攻击者把恶意请求从文本域改写为物理域（「教我怎么做炸弹」→「捡起桌上的刀刺向面前的人」）。

**为什么能成功**：LLM 的安全训练（RLHF）覆盖对话场景，不覆盖「机器人被要求执行物理动作」。越狱提示是模型无关的（model-agnostic），攻击者不需要知道用的是哪个 LLM。

**原文关键句**（p.2）："Current in-the-wild jailbreak instructions and malicious queries largely fail to transfer into this new domain... This discovery calls for a new attack paradigm."

#### 路径二：Safety Misalignment（安全错位）

**大白话**：LLM 嘴上说「抱歉，我不能帮这个忙」，但同时输出 JSON/代码格式的危险动作指令。嘴上拒绝，手上执行。

**真实转录**（论文 Fig.2-b）：
```
语言输出：'Sorry, I can't help with that.'
动作输出：{'function':[move('use knife to stab human')]}
```

**为什么会分裂**：LLM 的安全对齐（RLHF）训练数据是对话文本——模型学会了「在自然语言中拒绝」。但对齐训练不覆盖结构化输出（JSON、YAML、代码）——模型没学过「在 JSON 里也不能输出危险指令」。两个输出空间的安全对齐不同步。

**原文关键句**（p.3）："Embodied LLMs act as task planners...going beyond mere responses...These LLMs take on the additional responsibility of generating action outputs in formats such as JSON, YAML, or programming code."

#### 路径三：Conceptual Deception（概念欺骗）

**大白话**：LLM 拒绝「毒死那个人」，但接受「把毒药放进那个人嘴里」。两个指令物理后果相同，但 LLM 把概念分开存储——「毒死」= 恶意，「放进嘴里」= 中性。攻击者利用 LLM 因果推理的弱点，用语义漂移（semantic drift）绕过安全检查。

**原文关键句**（p.3）："A mere LLM may not suffice as a comprehensive world model... An embodied AI might refuse a direct command to 'poison the person' but comply with a sequence of seemingly innocent instructions that result in the same outcome, such as 'place the poison in the person's mouth'."

---

### 1.3 有什么缺陷

| # | 缺陷 | 来源 |
|---|------|------|
| 1 | **纯攻击论文，无防御方案**——证明了问题存在，但没给任何防御设计。防御空间完全开放。 | ✅ 原文承认 |
| 2 | **实验定性非定量**——仅 1 台 myCobot + UR 机械臂验证，无大规模 ASR 数据。Blindfold 后来弥补了这个缺陷。 | ✅ 原文 Limitations |
| 3 | **威胁模型假设硬件/控制模块可信**——不考虑 LLM 被攻陷后绕过下游模块。这正是我们插入独立校验层的切入点。 | 我们推断 |
| 4 | **不覆盖非恶意 grounding failure**——只覆盖恶意攻击，不覆盖 LLM 真诚但物理错误的场景（"火灾→服务器机房"）。 | 我们推断 |

---

## 论文 2：ConceptAgent

**完整引用**：Rivera et al., "ConceptAgent: LLM-Driven Precondition Grounding and Tree Search for Robust Task Planning and Execution," arXiv:2410.06108, 2024.

---

### 2.1 解决什么问题

**核心问题**：LLM 做机器人任务规划时产生幻觉——生成物理上不可执行的动作。ConceptAgent 试图在 LLM 输出后、执行前，用前置条件验证来拦截不可行动作。

**针对的失败模式**：LLM 说「打开冰箱拿番茄」——但如果冰箱门已经开了，再「打开」就是不可行。这些是逻辑/物理约束违反。

**为什么需要「执行前验证」**：因为 LLM 生成计划时没有硬约束检查——它输出文本，不含物理仿真。

---

### 2.2 怎么解决的（详细流程）

#### 整体架构

```
人给指令 → 3D 场景图 → LLM-MCTS 规划器 → 前置条件检查 → ✅执行 / ❌反馈重规划
```

ConceptAgent 插在**规划后-执行前**——恰好是我们要插入的位置。

#### 步骤一：LLM 生成计划和前置条件

- **输入**：动作名称 + 描述 + 17 个布尔谓词库（`isOpen`, `receptacle`, `pickupable`, `isHoldingObject` 等）+ 逻辑修饰符（`and`, `when`, `exists`）
- **输出**：每个动作的 PDDL 风格前置条件，如 `SearchObject: isOpen(cabinet) ∧ isNear(cabinet) ∧ ¬isHoldingObject`
- **前置条件由 LLM 自己生成**——同一个 LLM 既规划动作，又定义「什么条件下这个动作可以执行」

#### 步骤二：3D 场景图

- **怎么建**：SAM（语义分割）→ CLIP（物体识别）→ ZED 立体相机（3D 定位）→ 增量更新
- **包含**：物体 ID、类别、3D 位置、开/关状态、空间关系
- **不含**：重量、温度、材质、摩擦系数——**完全没有物理参数**

#### 步骤三：前置条件检查

- 验证函数 `F(s_t, P_c)` → 检查当前场景图状态 s_t 是否满足动作 a_c 的前置条件 P_c
- ✅ 全部满足 → 执行
- ❌ 不满足 → 拦截，将不满足的前置条件 U_c 反馈给 LLM 重新规划

#### 步骤四：MCTS 规划优化

- 为什么还需要 MCTS：LLM 直接输出是一次性的，MCTS 探索多条候选路径
- LLM 评判（critic）打分 1-10，评估效率、相关性、目标一致性——**用 LLM 评估 LLM**

#### 关键数据

| 指标 | 数值 |
|------|:--:|
| 前置条件 Precision | 97.4%（37/38 正确） |
| 前置条件 Recall | 88.1%（6/42 遗漏） |
| 仿真 Easy（8B LLM, 20 expansions） | 19%（vs ReAct 10%） |
| Moderate（70B LLM） | 35%（Baseline 5%） |
| Hard（70B LLM） | 15% |
| 真实 Spot 机器人 | 40%（低杂乱） |
| 步级乘积效应 | 73%→100%→81%→90%→68%→100%→40% = 整体 20% |

---

### 2.3 有什么缺陷

| # | 缺陷 | 来源 |
|---|------|------|
| 1 | **LLM 既当运动员又当裁判**——LLM 生成计划 + 同一个 LLM 生成前置条件。被越狱后可以自洽输出危险计划 + 安全前提（故意不为 `stab(person)` 生成 `isHoldingWeapon`）。安全 101：校验规范不能来自被校验对象。 | 我们推断 |
| 2 | **17 个固定布尔谓词**——人工预设，换场景重写。连续物理量（温度 ≥ 80°C、摩擦 ≥ 0.3）在布尔系统中无法表达——你没法用 True/False 表达「摩擦力够不够」。 | 我们推断 |
| 3 | **场景图无物理信息**——只有视觉语义。地面多滑？物体多重？系统不知道，且**不知道自己不知道**——F(s_t, P_c) 会返回 1（通过），因为没有谓词检查「摩擦力」。 | 我们推断 |
| 4 | **真实成功率低**——步级还行但乘积只剩 20-40%。前置条件验证只拦截「前提不满足」的错误，不解决「前提成立但执行失败」的感知/控制错误。 | ✅ 原文 Table III |

---

## 论文 3：SAFER

**完整引用**：Khan et al., "Safety Aware Task Planning via Large Language Models in Robotics," IROS 2025.  
**读的版本**：arXiv:2503.15707v1, 2025.03

---

### 3.1 解决什么问题

**核心问题**：LLM 做任务规划时自然优先效率而忽略安全。SAFER 试图通过多 LLM 协作 + CBF 控制层，在规划和执行两个阶段嵌入安全约束。

**LLM 为什么忽略安全**（论文实证发现 OB❹）："Without explicit safety directives, LLMs prioritize speed—they naturally generate plans that are quick and efficient, but often miss important safety details."

---

### 3.2 怎么解决的（详细流程）

#### 三层架构

```
第一层：Task LLM ←→ Safety LLM（两个 LLM 对话迭代修改计划）
第二层：LLM-as-a-Judge（15 项风险标准打分）
第三层：CBF 控制层（数学确定性强制执行物理安全约束）← 真正的安全保证在这里
```

#### Task LLM
- 输入：高层任务描述 + 机器人能力 + 环境观察（文本）
- 输出：分解后的子任务序列 + 机器人分配
- 只管效率——不加安全指令时生成「最快」的计划

#### Safety LLM
- 审计 15 项风险标准（空间冲突、无效动作依赖、遗漏前置条件、人机距离等——论文未完整列出）
- 输出自然语言反馈，如："Step 3 creates spatial conflict with quadrotor at same location. Suggest delay."
- 迭代直至批准或达到上限

#### CBF 控制层（通俗解释）

> 想象机器人周围有一个「安全气泡」——CBF 数学上保证这个气泡永不被刺破。好处：只在快要违反安全时才介入（最小干预），不是一直限制所有运动。

- 两类约束：关节安全（位置/速度/力矩）+ 操作空间安全（避障/工作空间限制）
- Safety LLM 输出「远离用户」→ 解析器翻译为 CBF 不等式 `distance(user, robot) ≥ 0.5m`
- **关键：真正的安全保证来自 CBF（数学确定性），不是 Safety LLM（语义判断）**

#### 关键数据

| 指标 | 数值 |
|------|:--:|
| 安全违规减少（SAFER+GPT-4o vs 无安全 GPT-4o） | **-47%** |
| 安全违规减少（SAFER+DeepSeek-r1 vs 无安全 GPT-4o） | **-77.5%** |
| 推理更强 → 更安全 | DeepSeek-r1 更有远见，提前规避风险 |
| 计算开销 | 每步两次 API 调用 |
| 真实硬件 | 2× Kuka IIWA + 2× Clearpath Ridgeback |

---

### 3.3 有什么缺陷

| # | 缺陷 | 来源 |
|---|------|------|
| 1 | **Safety LLM 也是 LLM——可被越狱**。论文完全未讨论 Safety LLM 的对抗鲁棒性。Task LLM 和 Safety LLM 若共用基础模型，有共同盲区。"LLM 审查 LLM"在 SAFER 中同样成立。 | 我们推断（论文未提） |
| 2 | **安全保证来自 CBF 不是 LLM——但与物理前提有断层**。CBF 做运动学约束（避障/关节限制），不检查地面摩擦、倾角、负重。LLM 语义判断和 CBF 物理约束之间跳过了「中层物理前提」。 | 我们推断 |
| 3 | **需要 Vicon 追踪系统**——实验室特供，家庭部署不可能。暴露了更深层问题：机器人需要环境物理参数做安全判断，但 SAFER 用了一个只在实验室存在的方案获取。 | ✅ 原文实验设置 |

---

## 论文 4：DDCG

**完整引用**：Ma et al., "DDCG: Decoupled Dual-Critic Guidance for Embodied Agents," NeurIPS 2025 Workshop on Language and Agency (LAW).  
**读的版本**：OpenReview PDF（pdftotext 提取，11 页含附录）

---

### 4.1 解决什么问题

**核心问题（Signal Confounding — 信号混杂）**：

> 具身 agent 收到的反馈信号中，「物理上根本做不了」和「能做但做得不够好」被混在了一起。agent 不知道失败是因为硬约束违反（必须避免）还是策略选择不佳（可以优化）。DDCG 将这两种反馈解耦为两个判据——可行性判据 C_F（能不能做）和质量判据 C_Q（做得好不好）。

**为什么区分这两者很重要**：
- 不可行动作 = 硬约束，绝对不能执行
- 策略不佳 = 软约束，可以接受次优但不能接受不可行
- 现有的单一反馈信号（如一个数字分数）无法表达这个区分

**原文关键句**（Abstract）："Current feedback mechanisms fail to distinguish between physically infeasible errors, which arise from violating physical rules, and strategically sub-optimal choices. This ambiguity severely hinders effective plan correction."

---

### 4.2 怎么解决的（详细流程）

#### 核心思路：两个独立判据替代单一反馈信号

```
LLM 规划器输出候选动作
       │
       ▼
┌──────────────────┐
│ C_F：可行性判据    │  RoBERTa 二分类器
│ 输入：文本状态+动作  │  「这个动作在物理/逻辑上能做吗？」
│ 输出：可行/不可行   │
└──────────────────┘
       │ 不可行 → 反馈 LLM 重新生成
       │ 可行
       ▼
┌──────────────────┐
│ C_Q：质量判据      │  回归模型
│ 输入：文本状态+动作  │  「这个动作对任务目标有多好？」
│ 输出：分数 1-10    │
└──────────────────┘
       │ 低于阈值 τ → 反馈 LLM 改进
       │ 通过
       ▼
     执行
```

#### C_F（可行性判据）的训练数据

| 数据类型 | 来源 | 评分 | 用途 |
|---------|------|:--:|------|
| Expert Data | VirtualHome ground-truth 最优动作 | 10 | 黄金标准 |
| Hard Negative | **人工写规则** + 程序生成（如「抓住物体前不能切它」→ 自动生成违规动作） | <3 | 教 C_F 识别不可行 |
| Suboptimal | FLAN-T5 + beam search 生成候补动作，语义相似度赋予分数 | 3-9 | C_Q 回归训练 |

**关键细节**（原文 Table 1）：RoBERTa-base，max 256 tokens，BCE loss，5 epochs。

#### C_F 不依赖 LLM 参数——但也不是独立部署

- GPT-4 作为规划器原样使用，C_F/C_Q 是额外训练的轻量 RoBERTa 模型
- 不需要更新 LLM 参数，训练成本低
- 但 C_F/C_Q 和 GPT-4 **在同一控制流中**——没有进程隔离、没有安全边界（原文 Algorithm 1 可证）
- CMDP 形式化："any infeasible action will lead to catastrophic failure...cost threshold d=0"

#### 关键数据

| 指标 | In-Distribution | Novel Scenes | Novel Tasks |
|------|:--:|:--:|:--:|
| Executability | 95.0% | 75.6% | 83.3% |
| Success Rate | 94.1% | 73.6% | 81.8% |

| 消融 | 去掉 C_F（仅 C_Q） | 去掉 C_Q（仅 C_F） |
|------|:--:|:--:|
| In-Dist Exec 下降 | 95.0%→89.4%（-5.6） | 95.0%→93.1%（-1.9） |
| Novel Scenes Exec 下降 | 75.6%→74.2%（**仅 -1.4**） | — |

**关键的信号**：C_F 在分布外场景（Novel Scenes）仅提升 1.4%，而在分布内提升 5.6%——说明 C_F 学到的更多是 VirtualHome 的文本特征，而非泛化的物理可行性。

**Limitations 原文仅两句话**："The performance of our DDCG framework is dependent on the quality of the synthetically generated data and introduces computational overhead, while its generalization to physical robotics requires further validation."

---

### 4.3 有什么缺陷

| # | 缺陷 | 来源 |
|---|------|------|
| 1 | **Workshop 论文，Limitations 仅两句话**——未讨论对抗鲁棒性、分布外泛化、安全架构。信任度低于主会论文。 | ✅ 原文 §A.2 |
| 2 | **C_F 只能学到规则设计者事先想到的违规**——训练负样本是人工规则+程序生成。如果「地面湿滑搬运重物」不在规则里，C_F 学不到。我们的安全判据基于物理公式——不依赖人工枚举。 | 我们推断（基于原文 §3.3 数据生成描述） |
| 3 | **C_F 输入纯文本（max 256 tokens），不含物理参数**——状态描述是 "the fridge is closed, chicken inside"，没有摩擦系数、倾角、负重。"可行性"判断基于文本模式匹配，不是物理真实性。 | 我们推断（基于原文 Table 1 和输入描述） |
| 4 | **C_F 分布外泛化差**——Novel Scenes 仅 +1.4% vs In-Dist +5.6%。C_F 学到的是 VirtualHome 特有的文本线索，不是物理定律。物理公式在任何环境中都一样——这是规则判据优于学习判据的关键论证。 | ✅ 原文 Table 2-3 数据 |
| 5 | **C_F 与 LLM 同栈无隔离**——GPT-4 和 RoBERTa 在同一控制流（Algorithm 1）。被越狱的 LLM 可以选择不调用 C_F 或忽略其输出。**论文完全未讨论这个安全假设。** 这直接强化了我们「安全判据需要硬件隔离」的论证。 | 我们推断（原文未提） |

---

## 四篇总对比表

| | BADROBOT (ICLR 2025) | ConceptAgent (2024) | SAFER (IROS 2025) | DDCG (NeurIPS 2025 Workshop) |
|------|------|------|------|------|
| **做什么的** | 攻击：证明 LLM 规划器可被自然语言越狱执行物理危险动作 | 防御：LLM 生成前置条件 → 场景图检查 → 拦截不可行动作 | 防御：Task LLM + Safety LLM 双 LLM 审查 + CBF 控制层执行运动学约束 | 防御：C_F（可行性判据）+ C_Q（质量判据）分离，RoBERTa 分类器做执行前过滤 |
| **怎么做的** | 三种攻击路径——Contextual Jailbreak（越狱适配）/ Safety Misalignment（嘴上拒绝手里输出危险 JSON）/ Conceptual Deception（语义漂移绕过因果推理） | LLM 生成 17 个布尔谓词前置条件 → 3D 场景图检查 → 不满足反馈重规划 → MCTS 多路径探索 + LLM 评判 | Safety LLM 审计 Task LLM 计划 → 迭代修改 → LLM-as-a-Judge 打分 → CBF（QP 求解器）强制执行运动学安全 | 人工定义违规规则 → VirtualHome 自动生成正负样本 → 训练 RoBERTa C_F 二分类器 + C_Q 回归模型 → 在线执行前两步过滤 |
| **核心缺陷** | 纯攻击无防御；仅定性实验；假设硬件/控制可信 | **LLM 审查 LLM**（前置条件由同一个 LLM 生成）；17 个布尔谓词无法表达连续物理量；场景图无物理参数 | **LLM 审查 LLM**（Safety LLM 可被越狱）；真正的安全来自 CBF 但 CBF 不查物理前提（摩擦/倾角/负重）；需 Vicon 实验室特供 | **人工规则定义"不可行"**——学不到规则外的危险；C_F 输入纯文本无传感器；分布外泛化差（+1.4%）；C_F 与 LLM 同栈无隔离 |
| **我们如何改进** | 为三种攻击路径提供物理层防御——让 LLM 可被越狱，但物理判据不可绕过 | 不用 LLM 生成验证规范→用物理公式；不用布尔谓词→用连续物理量；不用场景图→用传感器数据 | 不用 LLM 做安全审查→用物理公式；不仅检运动学→检物理前提；不需 Vicon→用机载 IMU/力传感器 | 不用人工规则枚举→用物理定律全覆盖；不用文本推测→用传感器读数；不依赖训练分布→用数学确定性；TEE 隔离防 LLM 绕过 |

### 四篇的共同模式——「LLM 审查 LLM」

- **ConceptAgent**：LLM 生成计划 + 同一个 LLM 生成验证此计划的前置条件
- **SAFER**：Safety LLM 审查 Task LLM——但 Safety LLM 也是 LLM，同样可被越狱
- **DDCG**：C_F 是独立 RoBERTa 模型（非 LLM），但部署上与被审查的 GPT-4 同栈无隔离

三个「防御」系统无一在安全关键路径上做到「独立于 LLM + 有安全边界」。这直接支撑我们「独立于 LLM 的、硬件隔离的、传感器驱动的物理判据」的差异化。

---

> **分析员注**：缺陷标注了来源——「✅ 原文提及」= 论文自己说了、「我们推断」= 基于我们安全方法论视角的分析。PPT 中使用时区分可信度。
