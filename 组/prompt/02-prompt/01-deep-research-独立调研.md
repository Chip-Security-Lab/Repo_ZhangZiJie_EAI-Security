# Deep Research Prompt：具身AI安全研究问题独立调研

---

## ⚠️ 执行指令（必读）

1. **启用 skill**：输入 `/deep-research`，选择 **full research mode**
2. **输入素材**：执行前先阅读 `E:\BaiduSyncdisk\Enbodied AI\组\prompt\03-产出\01-phase1-候选问题池.md` 了解已有调研基础
3. **产出写入**：调研完成后，将完整报告写入：
   ```
   E:\BaiduSyncdisk\Enbodied AI\组\prompt\05-外部调研\01-deep-research-产出.md
   ```
4. **Phase 1 免交互**：如果 skill 的 Phase 1 scoping 阶段需要交互确认，请基于本 prompt 中已明确的「研究问题」「范围边界」「方法论偏好」直接推进，无需等待用户确认。

---

## 背景（给研究 Agent 的完整上下文）

我们是一个具身智能安全方向的研究小组，正在为第一篇论文寻找一个**明确的问题定义**。之前尝试的方向被导师否定，核心问题是：

1. **问题感觉像自己发明的**——没有足够的文献根基证明"这个问题确实存在且被社区认可"
2. **问题定义太模糊**——把多个子问题（物理上下文校验 + 人类指令安全 + 对话监控）揉在一起，范围失控
3. **方法（TEE）与问题脱节**——问题没定义清楚就跳到方法

## 核心约束

**问题必须来自文献（别人提出过），不能是我们自己发明的。** 我们只是问题的发现者和定义者，不是发明者。

## 已有调研基础

完整的 Phase 1 调研产出在 `E:\BaiduSyncdisk\Enbodied AI\组\prompt\03-产出\phase1-候选问题池.md`。核心提要：

- 综述 *Safety in Embodied AI* (2026, 400+ refs) 识别了 8 大具身特色攻击方向 + 6 个开放挑战
- AI 顶会（NeurIPS/ICML/CVPR/CoRL 等）~580 篇中仅 23 篇安全相关（~4%），集中在 VLA 鲁棒性和安全规划，攻击类极少
- 安全四大（S&P/CCS/NDSS/USENIX）2025-2026 ~1,500 篇中具身直接相关不足 15 篇，但 TEE/硬件安全 ~66 篇
- 初步筛选出 12 个候选问题，用户倾向聚焦于"接地失效的物理可行性验证"

## 当前聚焦方向（供 deep-research 验证/挑战）

我们初步倾向于研究 **Grounding Failure（接地失效）**——LLM planner 在具身场景中产生"逻辑自洽但物理不可行"的推理结果。例如：LLM 在火灾中导航机器人去"服务器机房"而非出口，因为它推理出"机房有灭火系统 = 安全"。

已知文献基础：
- Chakraborty et al. [45]：场景-任务不一致时 agent 幻觉率上升 40%
- Han et al. [119]：经典案例"火灾→服务器机房"
- Baraldi et al. [24]：首次定义世界模型预测的病理学标准

但我们需要你**独立验证这个方向的可行性，也开放发现更好的方向。**

---

## 研究问题（供 deep-research Phase 1 直接使用）

### 主问题

What specific, well-documented security problems in embodied AI have been identified by the research community but remain unsolved, and which of them are most suitable as a first-paper topic for a team with AI expertise but limited hardware background?

### 子问题

1. **Grounding Failure 的文献深度验证**：除了 Chakraborty 和 Han，还有哪些论文从不同角度记录了同样的现象？有没有人尝试过防御？防御到什么程度？

2. **语义-物理鸿沟的防御进展**：BADROBOT 之后（2025 至今），有没有新论文提出了针对语义-物理鸿沟的防御方案？如果有，覆盖了 BADROBOT 三种攻击路径中的哪几种？

3. **"指令执行前验证"这个 idea 是否已有先行者**：有没有任何论文明确提出或暗示需要在 LLM planner 输出指令后、物理执行前进行安全性验证？这个 idea 是否已经被别人做过了？

4. **硬件辅助具身安全的最新进展**：是否有论文将 TEE/TrustZone/安全 enclave 等硬件机制应用于具身AI/机器人安全？如果有，具体怎么用的？和我们的初步设想有什么区别？

5. **跨社区空白验证**：AI 顶会和安全四大之间，是否确实存在我们观察到的"AI 有问题定义无安全机制，安全有机制无具身视角"的鸿沟？有没有反例（即有人已经做了交叉工作）？

---

## 范围边界（供 deep-research Phase 1 的 scoping）

| 维度 | In Scope | Out of Scope |
|------|---------|-------------|
| 时间 | 2024-2026 发表 | 2023 及更早（除非是奠基性工作） |
| 领域 | 具身AI安全、机器人安全、LLM agent 物理安全 | 纯 LLM 文本安全、传统工业机器人安全（无 AI 组件） |
| 方法 | 安全校验、验证、监控、隔离 | 纯对抗训练、纯数据增强、传统避障 |
| 硬件 | 支持性角色（TEE 隔离、可信执行） | 芯片设计、新型传感器硬件 |
| 攻击类型 | 利用 LLM 架构特征的攻击 | 纯传感器 spoofing/jamming（传统安全四大会已覆盖） |

---

## 方法论偏好（供 deep-research Phase 1 的 methodology 设计）

- 研究范式：文献驱动的 gap analysis（非实证研究、非系统综述）
- 主要方法：系统文献搜索 + 引用链追踪 + 跨会议交叉验证
- 数据源：AI 顶会（NeurIPS/ICML/ICLR/CVPR/CoRL/RSS）+ 安全四大（S&P/CCS/NDSS/USENIX Security）+ arXiv 预印本
- 质量评估：优先顶会已接收论文，arXiv 预印本作为补充信号（不作为主要依据）

---

## 团队约束（供 deep-research 评估候选方向可行性时参考）

1. **硬件能力有限**：团队以软件/AI 背景为主，不了解也不喜欢深度硬件工作。硬件只能作为"拿来用"的工程支撑（如用 TEE 做隔离部署），不能作为核心科学贡献。
2. **仿真可验证**：实验环境首选 Gazebo/Isaac Sim 等仿真平台，暂不具备真机实验条件。
3. **论文规模**：第一篇论文，目标顶会短文（short paper）或 workshop paper，不需要完整的系统实现。
4. **时间周期**：3-4 个月内完成实验 + 写作。

---

## 输出要求

请将完整调研报告写入：

```
E:\BaiduSyncdisk\Enbodied AI\组\prompt\05-外部调研\deep-research-20260618.md
```

报告结构：

1. **Executive Summary**（300 字以内）
2. **Community-Identified Problems**：列出至少 5 个被多篇独立论文指出的具身AI安全问题。每个包含：谁提出的、什么证据、有没有防御。
3. **Grounding Failure Deep Dive**：对 grounding failure 的文献深度分析——所有相关论文、引用链、防御状态。
4. **Cross-Community Gap Analysis**：验证我们观察到的「AI 顶会 vs 安全四大」空白是否真实存在。
5. **Candidate Ranking**：对你发现的所有候选问题进行排名（不仅限于我们的 12 个），排名维度：
   - (a) 文献根基有多扎实
   - (b) 是否具身独有
   - (c) 软硬件结合的可行性（不要求深度硬件）
   - (d) 实验可行性
6. **Recommendation**：推荐 1-2 个方向，明确说明理由，特别是对 grounding failure 方向的验证或反驳。
7. **Key Papers to Read**：5-10 篇最该读的论文（带完整引用和推荐理由）。
8. **Limitations**：本次调研的方法论局限和不确定性。

---

## 特别说明

1. **不要只复述我们已有的调研**——请做独立搜索和验证。如果发现我们对某个问题的判断有误，直接指出。
2. **如果 grounding failure 方向确实是最优的**，请给出更强的文献支撑和更精确的问题界定。
3. **如果发现了更好的方向**，请详细论证，不要被我们的初步偏好约束。
4. **所有关键声明必须有引用**，无法确认的标注"待验证"。
