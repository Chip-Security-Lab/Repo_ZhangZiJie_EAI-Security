# 具身AI安全研究问题——独立深度调研报告

> **调研日期**：2026-06-18  
> **调研模式**：Deep Research Full Mode  
> **研究问题**：具身AI中社区已识别但未解决的安全问题，及最适合AI背景团队的首篇论文方向  
> **数据源**：AI顶会（NeurIPS/ICML/ICLR/CVPR/CoRL/RSS）+ 安全四大（S&P/CCS/NDSS/USENIX Security）+ arXiv + ACM Digital Library + IEEE Xplore  
> **时间范围**：2024-2026（奠基性工作不限）  
> **方法论**：系统文献搜索 + 引用链追踪 + 跨会议交叉验证

---

## 1. Executive Summary

本次独立调研对具身AI安全领域进行了系统性文献搜索与验证，**核心发现**：

**（1）Grounding Failure 方向是最优选择，但需要从更精确的角度切入。** 我们发现 Grounding Failure 的文献基础远比 Phase 1 所记录的丰富——至少有 7 篇独立工作从不同角度（前置条件验证、可行性-质量分离、因果推理、3D接地）记录了同一现象。但现有防御工作几乎全部聚焦在**规划生成阶段**（让 LLM 生成更好的计划），而非我们的差异化方向——**在 LLM 输出后、物理执行前的独立安全校验**。

**（2）「指令执行前验证」并非零篇空白——但存在关键的差异化空间。** 我们独立发现了至少 5 个已发表的 pre-execution verification 系统（VerifyLLM/IROS 2025, SAFER/IROS 2025, Asimov Box/Princeton, Joint Verification/arXiv 2024, ILION/2025），这与 Phase 1 "接近 0%"的评估有偏差。**但关键差异化在于**：这些系统全部做的是形式化验证（LTL/automata）、几何约束或语义安全过滤，**没有任何一个从传感器数据实时提取物理参数（摩擦系数、倾斜角、承重能力）来做物理上下文安全校验**。我们的精确差异化定位是：**physics-grounded pre-execution safety validation from sensor data**。

**（3）跨社区空白确实存在，但正在快速收窄。** 我们发现 2025-2026 年间至少有 4 篇论文/系统（PARTEE/ACSAC 2025, DeepTrust^RT/2025, TZ-DATASHIELD/NDSS 2025, 具身智能机器人安全技术白皮书/2026）将 TEE/TrustZone 应用于机器人/嵌入式系统。这证明 TEE 嫁接具身AI 的技术可行性已被验证，但也意味着**纯"TEE+具身"不再是全新方向**。我们的差异化应在**TEE 保护的具体内容**（物理安全判据，而非 DNN 推理或数据保护）和**安全问题的独特性**（语义-物理鸿沟，而非传统传感器攻击）。

**（4）推荐方向**：**Physics-Grounded Pre-Execution Safety Validation（物理接地的执行前安全验证）**——从传感器数据提取物理安全参数，在指令执行前进行独立的安全判据校验，并将校验逻辑部署在 TEE 中作为硬件隔离的最后防线。这个方向是 Grounding Failure 防御 + pre-execution verification + TEE 隔离三者的交叉点，每一侧都有文献支撑但交叉区域完全空白。

---

## 2. Community-Identified Problems（社区已识别的安全问题）

以下列出 **≥7 个**被多篇独立论文从不同角度指出的具身AI安全问题。格式：问题陈述 / 谁提出的 / 证据强度 / 防御状态。

---

### 问题 1：Semantic-Physical Gap（语义-物理鸿沟越狱）

| 维度 | 内容 |
|------|------|
| **问题陈述** | LLM 的安全对齐在文本空间完成，物理执行不受文本安全对齐约束。攻击者通过自然语言越狱可诱导机器人执行物理危险动作，而文本安全过滤器无法检测这些攻击。 |
| **提出者（≥4 篇独立论文）** | **BADROBOT** (Zhang et al., ICLR 2025) — 首次系统识别三种攻击路径（Contextual Jailbreak / Safety Misalignment / Conceptual Deception）；**Blindfold** (Huang et al., SenSys 2026) — 动作级越狱，将恶意意图分解为表面无害的原子动作序列，攻击成功率比 BADROBOT 高 53%；**CHAI** (arXiv 2025) — LVLM 命令劫持绕过安全约束；**BadNAVer** (2025) — 语义越狱触发不安全导航 |
| **证据强度** | **极高**。BADROBOT 在 ICLR 2025 发表，Blindfold 在 SenSys 2026 发表，两者均在真实机器人上验证。Blindfold 在 GPT-4o 上达 93.2% ASR。 |
| **防御状态** | **极低（<10%）**。现有防御（Llama-Guard, SafeDecoding, VeriSafe）全部被 Blindfold 绕过。**关键缺陷**：所有现有防御做的是语义层检查（检查"语言是否危险"），而非物理层检查（检查"动作在当前物理环境下是否危险"）。 |
| **Phase 1 评估准确性** | ✅ 准确。Phase 1 对此问题的评估（文献基础强、防御空白大）与独立调研结果一致。 |

---

### 问题 2：Grounding Failure / Physical Infeasibility（接地失效）

| 维度 | 内容 |
|------|------|
| **问题陈述** | LLM/VLM 在具身场景中产生"逻辑自洽但物理不可行"的推理结果——如推理"火灾→去服务器机房（有灭火系统=安全）"而非出口。LLM 面对不可行指令时不拒绝，而是"将就"危险的替代方案。 |
| **提出者（≥7 篇独立论文）** | 详见 §3 Grounding Failure Deep Dive。核心：Chakraborty et al. (2024) — 幻觉率上升 40% 的定量证据；Han et al. (2024) — "火灾→服务器机房"经典案例；**Token Predictors Are Not Planners** (Lu et al., 2026, Tsinghua/MSRA) — Causal-Plan-Bench 证明当前 VLM 是"表面 token 预测器"而非真正的物理因果推理器；**ConceptAgent** (Rivera et al., 2024) — 前置条件验证防止不可行动作；**DDCG** (Ma et al., NeurIPS 2025) — 信号混杂：现有反馈无法区分物理不可行 vs 策略次优；**ContextMatters** (2025) — LLM+PDDL 目标松弛；**OmniEVA** (2025) — 3D接地+具身感知推理 |
| **证据强度** | **极高**。这是目前具身AI安全领域文献基础最丰富的问题方向。2026 年 Tsinghua/MSRA 的 Causal-Plan-Bench 提供了最大规模的诊断基准（1,200 实例，12 任务类别）。 |
| **防御状态** | **已有初步防御，但集中在规划生成阶段**。ConceptAgent（前置条件验证）、DDCG（可行性判据+质量判据分离）、ContextMatters（目标松弛）、Causal Reasoner（因果推理训练）全部是从"让 LLM 生成更好的计划"角度解决。**没有任何一个从"独立于规划器的物理安全校验"角度做防御**。 |
| **Phase 1 评估准确性** | ⚠️ **部分不准确**。Phase 1 评估防御覆盖率"接近 0%"，但实际已有多个防御系统。**然而 Phase 1 的核心洞察仍然有效**——现有防御全部在规划生成侧，没有一个做运行时独立安全校验。这恰恰强化了我们的差异化。 |

---

### 问题 3：Multi-Modal Fusion Vulnerability（跨模态融合脆弱性）

| 维度 | 内容 |
|------|------|
| **问题陈述** | 跨模态融合（Camera+LiDAR+Radar）的反直觉脆弱性——最弱通道决定整体安全，融合机制本身是攻击面而非保护。 |
| **提出者（≥5 篇独立论文）** | Li et al. (2025) — 200 个 LiDAR 对抗点→99% 攻陷融合；DejaVu (2025) — 传感器同步延迟→mAP 崩溃 88.5%；Li et al. (2025) — 单一物体同时欺骗 3 种传感器；PhantomLiDAR (NDSS 2025)；Physical Adversarial Shadow (USENIX Security 2025) |
| **证据强度** | **极高**。这是 AI 顶会和安全四大**同时关注**的罕见交叉问题（CVPR/ICCV + NDSS/USENIX 均有论文）。 |
| **防御状态** | 低（<20%）。对抗训练为主要手段，仅针对已知攻击模式。对时序错位和全模态欺骗无系统性防御。 |
| **Phase 1 评估准确性** | ✅ 准确。 |

---

### 问题 4：Agent Memory Poisoning（记忆投毒）

| 维度 | 内容 |
|------|------|
| **问题陈述** | Agent 记忆系统（对话历史/RAG 语料/经验日志）独立于模型参数且跨 session 持久存在——投毒一条记忆可导致永久性不安全行为。 |
| **提出者（≥2 篇独立论文）** | AgentPoison (2025) — RAG agent 记忆库投毒，每次检索激活后门；Persistent Backdoor Attacks in Continual Learning (USENIX Security 2025) — 持续学习中后门持久性 |
| **证据强度** | 中等。只有 2 篇直接相关的独立论文，但 AgentPoison 的实验设计严谨。 |
| **防御状态** | 接近 0%。具身场景无任何记忆投毒防御。 |
| **Phase 1 评估准确性** | ✅ 准确。 |

---

### 问题 5：Action-Level Jailbreak via Decomposition（动作级越狱——2026 年新威胁）

| 维度 | 内容 |
|------|------|
| **问题陈述** | **这是 2026 年新出现、Phase 1 未记录的威胁**。攻击者将恶意意图分解为表面无害的原子动作序列——如"炸掉用户手机"→ `find(phone) → pick(phone) → move(oven) → stretch()`——每个单独动作无害，组合后致命。 |
| **提出者** | **Blindfold** (Huang et al., SenSys 2026) — 首个自动化动作级越狱框架。在 GPT-4o 上 93.2% ASR，Phi-4-14B 上 98.1% ASR。绕过 Llama-Guard、SafeDecoding、VeriSafe。 |
| **证据强度** | **极高**。SenSys 2026 已接收，真实 6DoF 机械臂（UFactory xArm 6）验证。 |
| **防御状态** | **零**。Blindfold 作者明确指出："当前防御聚焦语义层语言审查，完全无法理解动作序列的物理后果。" |
| **Phase 1 评估准确性** | ⚠️ Phase 1 未覆盖此威胁（论文 2026 年 3 月才提交，Phase 1 生成时尚未公开）。这是重要的补充发现。 |

---

### 问题 6：Self-Evolution Safety Degradation（自我进化安全退化）

| 维度 | 内容 |
|------|------|
| **问题陈述** | 具身 agent 部署后通过在线学习/记忆积累/工具更新自主进化，安全约束随时间退化（参数漂移、记忆累积不安全经验、工具腐败、工作流退化）。 |
| **提出者（≥3 篇独立论文）** | Shao et al. (2025) — 四条退化路径首次系统识别；Agent-SafetyBench — 无 agent 通过 60% 评估；Moral Anchor (2025) — 自训练降低安全拒绝率；PACT (ICML 2026) — 唯一正面解决退化问题的顶会论文（但仅覆盖扩散策略） |
| **证据强度** | 高。退化路径的分类框架已被多篇独立工作验证。 |
| **防御状态** | 极低（<5%）。PACT 仅覆盖一条退化路径。 |
| **Phase 1 评估准确性** | ✅ 准确。 |

---

### 问题 7：Agent Skill Supply Chain（技能供应链攻击）

| 维度 | 内容 |
|------|------|
| **问题陈述** | 31,000+ agent 技能中 26% 含安全漏洞；预训练编码器后门沿供应链传播（BadEncoder→BadCLIP→BadVision→BadVLA→物理 agent）。 |
| **提出者（≥4 篇独立论文）** | Jiang et al. (2025) — 31K 技能实证扫描；BadEncoder/BadCLIP/BadVision — 后门传播链；SkillJect (2025) — 恶意代码隐藏；BadVLA (NeurIPS 2025) — 供应链终点物理化 |
| **证据强度** | 高。已形成完整的攻击链（编码器→VLM→VLA→物理 agent）。 |
| **防御状态** | 低（<15%）。模型水印（ICLR 2026）仅检测策略来源，无法检测后门。 |
| **Phase 1 评估准确性** | ✅ 准确。 |

---

## 3. Grounding Failure Deep Dive

### 3.1 问题定义演化

Grounding Failure 在 2024-2026 年间经历了显著的问题定义演化：

| 时间 | 论文 | 问题定义 | 核心贡献 |
|------|------|---------|---------|
| 2024 | Chakraborty et al. [45] | 场景-任务不一致时 agent 幻觉率上升 40% | 首次提供 grounding failure 的定量证据 |
| 2024 | Han et al. [119] | "火灾→服务器机房"——逻辑自洽/物理荒谬 | 经典案例确立 |
| 2024.10 | **ConceptAgent** (Rivera et al.) | 前置条件不满足→不可行动作→自动化失败 | **首次提出前置条件验证作为防御**；LLM 生成 predicate 前置条件，执行前检查；97.4% 前置条件生成准确率 |
| 2025 | Baraldi et al. [24] | 世界模型预测的病理学标准（时序/物理/条件一致性） | 首次形式化定义"什么是坏的预测" |
| 2025 | **EMNLP 2025 Findings** | 场景-任务不一致→幻觉率最高 40× 增加 | 首次系统研究，12 模型测试 |
| 2025.09 | **OmniEVA** | "几何适应差距"+"具身约束差距"→理论上有效/实践上不可行 | 3D接地+具身感知推理 |
| 2025.10 | **ContextMatters** | LLM 规划器在目标物理不可达时不调整→失败 | LLM+PDDL 目标松弛；TIAGo 真机验证；+52.45% 成功率 |
| 2025.12 | **DDCG** (NeurIPS 2025) | "信号混杂"——可行性 vs 质量反馈混淆 | 可行性判据 C_F + 质量判据 C_Q 分离；VirtualHome+ScienceWorld 验证 |
| 2026.06 | **Token Predictors Are Not Planners** (Tsinghua/MSRA) | 当前 VLM 是"表面 token 预测器"而非物理因果推理器 | **Causal-Plan-Bench**（1,200 实例）+ **Causal-Plan-1M**（百万级因果推理数据）；因果推理训练后提升 36.3% |

### 3.2 引用链分析

核心引用链揭示了 Grounding Failure 研究的演进路径：

```
Chakraborty et al. (2024) ──→ Han et al. (2024) ──→ Baraldi et al. (2025)
        │                            │                        │
        └──→ ConceptAgent (2024.10)  └──→ ContextMatters (2025.10)
             前置条件验证防御               目标松弛防御
                     │
                     └──→ DDCG (NeurIPS 2025)
                          可行性/质量分离
                     │
                     └──→ Causal Reasoner (2026.06)
                          因果推理替代 token 预测
```

**关键观察**：
- ConceptAgent 是最早的防御尝试（2024.10），DDCG 是最精巧的防御设计（2025.12），Causal Reasoner 是最新的方法论突破（2026.06）
- 所有防御沿**同一条路线**：「让 LLM 生成更好的计划」——优化规划生成侧
- **没有任何工作走另一条路线**：「不改变 LLM 规划器，而是在其输出后做独立安全验证」

### 3.3 防御覆盖度重新评估

| 防御方法 | 覆盖 Grounding Failure 的哪些面？ | 不覆盖的盲区 |
|---------|--------------------------------|------------|
| ConceptAgent（前置条件验证） | 可检测"前置条件不满足"的不可行动作 | 仅覆盖规划时的已知前提；无法检测物理环境中隐式约束（如地面摩擦不足） |
| DDCG（可行性判据） | 区分物理不可行 vs 策略次优 | 可行性判据基于训练数据学习（VirtualHome），不接入真实传感器数据 |
| ContextMatters（目标松弛） | 目标不可达时自动降级 | 仅处理"目标不可达"场景，不处理"目标可达但路径危险" |
| Causal Reasoner（因果推理） | 训练 LLM 理解物理因果链 | 模型能力提升≠安全保证；无法证明训练后的模型不会在新的物理场景中犯错 |
| **我们的方向（物理接地执行前校验）** | **覆盖以上所有盲区** | 不试图改进 LLM；在 LLM 输出后、执行前做独立物理校验 |

### 3.4 独立发现的 Phase 1 遗漏

Phase 1 的 Grounding Failure 分析遗漏了以下关键工作（按发现重要性排序）：

1. **Token Predictors Are Not Planners** (Lu et al., 2026.06) — 最新、最大规模的 grounding failure 诊断基准。**必须引用**。
2. **DDCG** (Ma et al., NeurIPS 2025) — 「可行性判据」+「质量判据」分离的框架直接命名了我们的核心问题。**必须引用**。
3. **ConceptAgent** (Rivera et al., 2024.10) — 最接近我们想法的前置工作。**必须引用并差异化**。
4. **ContextMatters** (2025.10) — LLM+PDDL 在真实机器人上做目标松弛，证明了接地问题的工程可行性。
5. **EMNLP 2025 Findings** — 首次系统定量研究 grounding 失败。

---

## 4. Cross-Community Gap Analysis（跨社区空白分析）

### 4.1 AI 顶会 vs 安全四大：空白是否真实存在？

**答案：部分存在，但正在快速收窄。**

#### 确认存在的空白

| 维度 | AI 顶会 | 安全四大 | 空白状态 |
|------|:--:|:--:|------|
| 语义-物理鸿沟防御 | 有攻击（BADROBOT/ICLR, Blindfold/SenSys），**无防御** | **零篇**关注语义层安全 | 🔥🔥🔥 双空白 |
| Grounding Failure 防御 | 有规划侧防御（ConceptAgent, DDCG），**无独立校验** | **零篇**关注 | 🔥🔥🔥 双空白 |
| 动作级越狱（Blindfold） | SenSys 2026 接收，**无防御** | **零篇**关注 | 🔥🔥🔥 双空白 |
| 人机交互安全 | PsySafe 识别威胁，**无防御** | **零篇** | 🔥🔥 双空白 |
| 记忆投毒 | AgentPoison 1 篇，**无防御** | 持久化后门（USENIX），**未针对 agent** | 🔥🔥 方法可迁移但未应用 |

#### 已被填补/正在收窄的空白

| 维度 | 新进展 | 对 Phase 1 评估的修正 |
|------|--------|---------------------|
| **TEE+具身AI** | PARTEE (ACSAC 2025)：Raspberry Pi TrustZone 保护无人机安全关键 enclave；DeepTrust^RT (2025)：OP-TEE 内运行 DNN 推理满足实时约束；TZ-DATASHIELD (NDSS 2025)：ARM TrustZone 数据流保护；中国具身智能机器人安全技术白皮书 (2026)：TEE 异构免疫 | ⚠️ Phase 1 评估"技术储备充足但应用完全空白"**不再准确**。TEE 已开始被应用于机器人/嵌入式系统。但**保护内容不同**——现有工作保护 DNN 推理/数据/可用性，**不保护 LLM 输出的物理安全校验**。 |
| **Pre-execution 验证** | VerifyLLM (IROS 2025)：LTL 形式化验证；SAFER (IROS 2025)：多 LLM + CBF 安全框架；Asimov Box (Princeton)：最后一步拦截；Joint Verification (2024)：automata 形式化验证；ILION (2025)：确定性几何验证 | ⚠️ Phase 1 评估"指令执行前验证零篇"**不准确**。但这个发现**利好而非削弱**我们的方向——它证明 pre-execution verification 是 2025 年的活跃研究前沿，为我们的物理接地变体提供了"时机正确"的论据。 |
| **传感器/物理域攻击** | CVPR/ICCV 有对抗攻击论文 | USENIX/NDSS/CCS 有 10+ 篇传感器攻击 | ✅ Phase 1 评估"高交叉"准确。传统安全问题延伸。 |

### 4.2 发现的「反例」——已有人做的交叉工作

以下论文/系统直接跨越了 AI-Security 鸿沟，是 Phase 1 交叉分析的**重要补充**：

| 论文/系统 | 会议/年份 | 交叉方式 | 与我们的差异 |
|----------|----------|---------|------------|
| **PARTEE** | ACSAC 2025 | TEE（TrustZone）用于机器人安全关键 enclave 的可用性保护 | 保护对象是 TEE 内部的**可用性**（防 DoS），不是安全判据的**正确执行** |
| **DeepTrust^RT** | ACM TCPS 2025 | OP-TEE 内运行 DNN 推理 | 保护对象是 **DNN 推理的机密性**，不是物理安全校验逻辑 |
| **SAFER** | IROS 2025 | 安全四大方法论（CBF）用于 AI 顶会场景（LLM 规划器安全） | 安全校验在**软件侧多 LLM 架构**中，可被越狱的 LLM 绕过 |
| **Blindfold** | SenSys 2026 | 安全攻击方法论（对抗性 LLM、意图混淆）用于具身AI | 攻击而非防御；但攻击目标精确描述了我们想防御的威胁 |
| **Semantically Safe Robot Manipulation** | IEEE RAL 2025 | LLM 语义推理 + CBF 物理安全约束 | CBF 约束在控制层而非规划层，且无 TEE 隔离 |

### 4.3 交叉空白的重新定义

基于新发现，我们对交叉空白的定位应修正为：

```
        原定位（Phase 1）                        修正后定位
        ┌──────────────┐                      ┌──────────────────────┐
        │ TEE 嫁接具身  │                      │ 物理接地执行前校验    │
        │ 安全（广）     │        ──→          │ + TEE 隔离部署（精）  │
        └──────────────┘                      └──────────────────────┘
        
问题：纯 TEE+具身 已不是全新方向      优势：校验内容（物理参数）和部署方式
（PARTEE, DeepTrust^RT 等已发表）     （TEE 保护判据执行）的交叉是独有的
```

**我们真正的交叉空白在于**：物理接地（Physics Grounding）+ 执行前验证（Pre-execution Verification）+ 硬件隔离（TEE Isolation）三者的交汇点。每一个单独维度都有工作，但三者的交集**确实为零**。

---

## 5. Candidate Ranking（候选方向排名）

基于独立调研的发现，对候选方向进行四维度排名。排名不仅限于 Phase 1 的 12 个候选，还纳入了新发现的子方向。

### 排名维度定义

| 维度 | 含义 | 评分标准 |
|------|------|---------|
| **(a) 文献根基** | 有多篇独立论文从不同角度指出该问题 | 5=5+篇独立论文含顶会；3=2-3篇；1=仅综述提及 |
| **(b) 具身独有** | 问题是否具身AI独有，传统机器人/纯LLM不覆盖 | 5=严格独有；3=部分重叠；1=可归约为已有问题 |
| **(c) 软硬结合** | 适合软件/AI背景团队，硬件仅作"拿来用" | 5=纯软件可完成核心，硬件可选；3=需要基础硬件；1=硬件为核心贡献 |
| **(d) 实验可行** | 3-4月内可完成仿真实验+写作 | 5=Gazebo/Isaac Sim 即可；3=需要部分真机；1=必须真机 |

### 完整排名

| # | 候选方向 | (a) 文献根基 | (b) 具身独有 | (c) 软硬结合 | (d) 实验可行 | **综合** | 相比 Phase 1 的变化 |
|---|---------|:--:|:--:|:--:|:--:|:--:|------|
| **1** | **物理接地执行前安全验证** | **5** | **5** | **5** | **4** | **⭐⭐⭐** | 🆕 精确化 Grounding Failure + Pre-exec 交叉 |
| 2 | Grounding Failure 因果推理增强 | 5 | 5 | 4 | 4 | ⭐⭐⭐ | 文献更丰富但差异化缩小（Causal Reasoner 已做） |
| 3 | 语义-物理鸿沟物理层防御 | 5 | 5 | 4 | 4 | ⭐⭐⭐ | Blindfold 提供了更强的攻击动机 |
| 4 | 跨模态融合安全架构 | 5 | 4 | 3 | 3 | ⭐⭐ | 需要多传感器平台 |
| 5 | 具身 Agent 记忆完整性保护 | 3 | 4 | 4 | 3 | ⭐⭐ | 文献根基仍偏弱 |
| 6 | 长时域安全不变量监控 | 3 | 4 | 5 | 2 | ⭐⭐ | 实验周期长 |
| 7 | 模型供应链端到端完整性 | 4 | 4 | 4 | 3 | ⭐⭐ | — |
| 8 | 环境条件后门行为检测 | 4 | 4 | 4 | 3 | ⭐⭐ | — |
| 9 | TEE 安全感知硬件架构 | 3 | 3 | 1 | 1 | ⭐ | ⚠️ 已非全新方向（PARTEE 等已发表） |
| 10 | 自我进化安全退化检测 | 3 | 4 | 4 | 2 | ⭐ | 实验周期长 |
| 11 | 多 Agent 语义通信保护 | 4 | 4 | 5 | 2 | ⭐ | 需多 agent 系统 |
| 12 | 人机交互安全监控 | 2 | 3 | 5 | 1 | ⭐ | 需 HRI 实验 |

### 关键变化说明

1. **候选 1（物理接地执行前安全验证）升至第一**：这是 Grounding Failure 防御 + Pre-execution Verification + TEE 隔离的精确交叉点。每个维度都有文献支撑，但交叉区域空白。实验可行性高（Gazebo 仿真即可）。
2. **候选 9（TEE 安全感知硬件架构）降至第九**：纯 TEE+具身 已非全新方向。PARTEE、DeepTrust^RT、TZ-DATASHIELD 等已在 2025 年发表。只有与特定安全问题（如物理安全判据）结合才有差异化。
3. **新增 Blindfold 驱动的子方向**：Blindfold (SenSys 2026) 的动作级越狱为候选 1 和 3 提供了更强的攻击动机和评估基线。

---

## 6. Recommendation（推荐方向）

### 6.1 首要推荐：Physics-Grounded Pre-Execution Safety Validation（物理接地的执行前安全验证）

**一句话定义**：
> 在 LLM 规划器输出动作指令后、机器人物理执行前，从传感器数据实时提取物理安全参数（摩擦系数、倾斜角、承重能力），进行独立于 LLM 的物理安全判据校验，并将校验逻辑部署在 TEE 中作为硬件隔离的最后防线。

**为什么这是最优方向**：

| 论证维度 | 内容 |
|---------|------|
| **文献根基** | (a) Grounding Failure：≥7 篇独立论文记录同一现象（Chakraborty→Han→ConceptAgent→DDCG→Causal Reasoner→ContextMatters→OmniEVA）；(b) Pre-execution verification：≥5 篇独立系统（VerifyLLM, SAFER, Asimov Box, Joint Verification, ILION）证明了这个范式在 2025 年已成为活跃前沿；(c) TEE+机器人：PARTEE, DeepTrust^RT 证明技术可行性 |
| **差异化** | 现有 Grounding Failure 防御全部在「规划生成侧」（让 LLM 更好地规划）。现有 Pre-execution verification 全部用形式化方法或语义过滤。**没有任何一个用传感器数据做物理参数提取+安全判据**。我们的定位是「规划生成侧的对立面」——独立于 LLM 的物理校验。 |
| **具身独有** | 物理上下文感知的安全判据（同一句话在平地安全、在斜坡危险）——传统机器人不需要（规划器可信），纯 LLM 不需要（无物理后果），纯安全系统不需要（不做物理参数提取） |
| **软硬结合** | TEE 是工程增强而非核心贡献，符合团队能力。安全判据的设计（从传感器数据→物理参数→安全判据→拦截决策）是纯软件/AI 问题。 |
| **实验可行** | Gazebo 仿真可提供地面摩擦系数、倾斜角等地形参数；可设计攻击场景（被越狱的 LLM 输出危险搬运指令）→验证安全校验拦截率；一个仿真环境 + 若干 LLM planner（GPT-4o/Gemini/开源）+ 规则化物理判据 |
| **论文规模匹配** | 可拆分为：短文的物理校验方法（核心贡献）+ TEE 部署（工程验证） |

**建议的问题陈述（Problem Statement）**：
> Current approaches to grounding failure in embodied AI focus on improving LLM planners' physical reasoning capabilities. However, given the demonstrated vulnerability of LLM planners to semantic jailbreaks (BADROBOT, Blindfold) and the fundamental difficulty of guaranteeing correct physical reasoning in LLMs (Causal-Plan-Bench), we argue that a complementary defense is needed: an independent, physics-grounded safety validation layer that sits between the LLM planner and physical execution, verifies the physical feasibility of planned actions using real-time sensor data, and is deployed in a hardware-isolated environment (TEE) to prevent bypass by a compromised planner.

**与已有工作的精确差异化**：

| 已有工作 | 他们做了什么 | 我们做什么（差异化） |
|---------|------------|-------------------|
| ConceptAgent (2024) | LLM 生成前置条件，执行前验证 | **我们不依赖 LLM 生成前置条件**——直接从传感器数据提取物理参数，用显式物理公式（摩擦方程）做判据 |
| DDCG (NeurIPS 2025) | 可行性判据 C_F + 质量判据 C_Q | **我们的判据不在 LLM 组件内**——独立于 LLM 部署在隔离环境中 |
| VerifyLLM (IROS 2025) | LLM 做 LTL 形式化验证 | **我们不做形式化验证**——做物理参数提取+物理公式判据 |
| SAFER (IROS 2025) | 多 LLM（Task LLM + Safety LLM） | **我们不让 LLM 审查 LLM**——安全判据是规则化/物理公式的 |
| ILION (2025) | 确定性几何验证（嵌入空间） | 几何验证≠物理验证——我们的判据基于物理参数（摩擦、倾角） |
| PARTEE (ACSAC 2025) | TEE 保护机器人安全关键 enclave 可用性 | 保护内容不同：我们保护物理安全判据的正确执行 |

### 6.2 次选推荐：Semantic-Physical Jailbreak Defense with Blindfold Baseline

如果首要方向的技术路线受阻，可以做更纯粹的「语义-物理鸿沟防御」方向：

- **动机**：Blindfold (SenSys 2026) 提供了更强的攻击基线（93.2% ASR on GPT-4o），但防御完全空白
- **方向**：设计针对动作级越狱（将恶意意图分解为无害原子动作）的检测机制
- **优势**：问题更新（2026.03）、基线明确（Blindfold）、防御空白最大
- **劣势**：可能更偏向纯语义/ML 检测而非物理校验→具身独特性减弱

---

## 7. Key Papers to Read（推荐精读的 10 篇论文）

按推荐阅读优先级排列。标注「🆕」为本次独立调研新发现、Phase 1 未覆盖的论文。

| # | 论文 | 出处 | 推荐理由 | Phase 1 覆盖？ |
|:--:|------|------|---------|:--:|
| 1 | **Token Predictors Are Not Planners: Building Physically Grounded Causal Reasoners** (Lu et al., 2026) | arXiv:2606.01810, Tsinghua/MSRA | 🆕 最新最大的 grounding failure 诊断基准（Causal-Plan-Bench, 1,200 实例）。核心论断"当前 VLM 是表面 token 预测器而非物理因果推理器"直接支撑我们问题的存在性。 | ❌ 未覆盖 |
| 2 | **Blindfold: Jailbreaking Embodied LLMs via Action-level Manipulation** (Huang et al., 2026) | SenSys 2026 | 🆕 2026 年最新攻击，动作级越狱，93.2% ASR on GPT-4o。为我们的防御提供了最强的攻击动机和评估基线。 | ❌ 未覆盖 |
| 3 | **ConceptAgent: LLM-Driven Precondition Grounding and Tree Search for Robust Task Planning** (Rivera et al., 2024) | arXiv:2410.06108 | 🆕 最接近我们防御思路的前置工作——前置条件验证+执行前检查。必须引用并差异化。97.4% 前置条件准确率。 | ❌ 未覆盖 |
| 4 | **DDCG: Decoupled Dual-Critic Guidance for Embodied Agents** (Ma et al., 2025) | NeurIPS 2025 | 🆕 「可行性判据」+「质量判据」分离框架。直接命名"信号混杂"问题——物理不可行 vs 策略次优无法区分。 | ❌ 未覆盖 |
| 5 | **Trust in LLM-controlled Robotics: A Survey** (Huang et al., 2025) | arXiv:2601.02377 | 🆕 最新的 LLM 控制机器人安全综述。引入"embodiment gap"概念。系统化了防御分类（形式化规范/运行时执行/多 LLM 监督/提示硬化/环境感知对齐）。 | ❌ 未覆盖 |
| 6 | **BADROBOT: Jailbreaking Embodied LLM Agents in the Physical World** (Zhang et al., 2025) | ICLR 2025 | Phase 1 已记录。问题定义的核心文献——三种攻击路径的原始出处。我们的防御目标。 | ✅ 已覆盖 |
| 7 | **SAFER: Safety Aware Task Planning via Large Language Models in Robotics** (Khan et al., 2025) | IROS 2025 | 🆕 2025 年 pre-execution verification 的代表性工作。多 LLM + CBF + LLM-as-a-Judge。我们差异化的对照物。 | ❌ 未覆盖 |
| 8 | **PARTEE: Practical Multi-Enclave Availability Through Partitioning and Asynchrony** (Habeeb et al., 2025) | ACSAC 2025 | 🆕 TEE（ARM TrustZone）用于机器人（Raspberry Pi 无人机）的首次实用设计。证明 TEE+机器人的工程可行性。 | ❌ 未覆盖 |
| 9 | **VerifyLLM: LLM-Based Pre-Execution Task Plan Verification for Robots** (Grigorev et al., 2025) | IROS 2025 | 🆕 Pre-execution verification 的代表性工作——LLM+LTL 形式化验证。我们差异化的对照物。 | ❌ 未覆盖 |
| 10 | **Towards Robust and Secure Embodied AI: A Survey on Vulnerabilities and Attacks** (Xing et al., 2025) | ACM Computing Surveys | 🆕 2025 年 ACM Computing Surveys 接收的具身AI安全综述。外源/内源漏洞分类框架。可以作为我们文献综述的参考结构。 | ❌ 未覆盖 |

**保留 Phase 1 推荐的论文**（已覆盖，不再重复排名但建议保持关注）：
- TAT (USENIX Security 2026) — 安全四大中唯一直接做机械臂轨迹证明的论文
- TensorShield (CCS 2025) — 软硬结合保护 DNN 推理的范例
- EXIA (NDSS 2026) — TEE 验证外部输入真实性的方法论

---

## 8. Limitations（本次调研的局限性）

1. **搜索覆盖度**：本次调研主要依赖公开搜索（Google Scholar, arXiv, ACM DL, IEEE Xplore），未访问付费数据库（如 Scopus, Web of Science）。可能遗漏部分仅在这些数据库中索引的论文。

2. **中文文献覆盖不足**：中国在具身AI安全领域有活跃研究（如具身智能机器人安全技术白皮书），但中文学术数据库（知网/万方）未被系统搜索。部分中国团队的工作可能未被充分捕获。

3. **预印本质量不均**：引用的 arXiv 预印本未全部经过同行评审。已标注会议/期刊接收状态的为可确信引用，纯 arXiv 预印本（特别是 2026 年提交的）需在正式引用前验证其最终发表状态。

4. **时间窗口限制**：调研聚焦 2024-2026。2023 及更早的奠基性工作可能被遗漏——特别是传统机器人安全和控制理论中的相关工作（如运行时验证 runtime verification、安全关键系统中的监控器设计），这些可能为我们的方案提供方法论基础。

5. **安全四大的具身AI覆盖可能被低估**：安全四大（特别是 USENIX Security 和 CCS）的论文标题常不包含"embodied"或"robot"关键词，而是用更通用的"CPS"、"cyber-physical"、"embedded system"等。部分将 CPS 安全技术应用于具身AI 场景的工作可能被漏检。

6. **ILION 框架的可信度需要验证**：ILION 的文档主要在 Zenodo 上，且提及"patent pending"。其声称的 100% 决策确定性需要经过独立验证。在正式引用前，建议确认其是否经过同行评审。

7. **Grounding Failure 概念边界模糊**：在 2024-2026 年间，"grounding failure"、"physical infeasibility"、"embodiment gap"、"semantic-physical gap"等概念在使用中存在交叉和混用。我们的文献分类可能将某些工作归入"grounding failure"而原作者使用的是不同理论框架。

8. **本调研不构成系统性综述**：本次调研是 gap analysis，不是 PRISMA-compliant 的系统性综述。如果后续需要发表文献综述部分，需要重新以系统性综述标准执行。

---

## 附录：本次调研的方法论说明

### 搜索策略

| 轮次 | 搜索重点 | 数据源 | 关键词 |
|:--:|---------|--------|--------|
| 1 | Grounding Failure 文献 | Google Scholar, arXiv | "grounding failure" + "embodied AI" + "LLM planner" + "physical infeasibility" |
| 2 | BADROBOT 后续防御 | Google Scholar, arXiv, ACM DL | "BADROBOT" + "semantic-physical gap" + "defense" + "LLM jailbreak robot" |
| 3 | Pre-execution 验证 | Google Scholar, IEEE Xplore | "pre-execution" + "safety verification" + "LLM planner" + "robot" + "instruction validation" |
| 4 | TEE/TrustZone + 具身 | Google Scholar, ACM DL, IEEE Xplore | "TEE" + "TrustZone" + "secure enclave" + "embodied AI" + "robot safety" |
| 5 | 跨社区空白 | Google Scholar, 各会议官网 | "embodied AI safety" + "S&P" + "CCS" + "NDSS" + "USENIX" |
| 6 | 深度验证（关键论文） | arXiv, ACM DL, IEEE Xplore | 目标论文标题精确搜索（ConceptAgent, DDCG, SAFER, Blindfold, Causal-Plan-Bench 等） |

### 引用筛选标准

- **纳入**：2024-2026 发表的同行评审论文/顶会接收论文；高质量 arXiv 预印本（有引用基础、有代码/实验验证）
- **排除**：纯 LLM 文本安全（无具身组件）；传统工业机器人安全（无 AI 组件）；芯片设计论文；仅标题匹配但内容不相关的论文
- **灰色地带处理**：arXiv 预印本标注"待验证"；Zenodo/tech report 标注来源类型

---

> **报告完成时间**：2026-06-18  
> **独立调研执行**：Deep Research Agent Team (Full Mode)  
> **AI 辅助声明**：本调研使用 AI 辅助文献搜索与分析。所有关键发现均基于可追溯的文献来源。标注"待验证"的声明需要进一步人工确认。
