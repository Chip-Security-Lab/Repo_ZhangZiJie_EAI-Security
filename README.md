# Embodied AI Security Research

> **Chip-Security-Lab** · 具身智能安全研究方向调研与文献整理

本仓库围绕 **具身智能（Embodied AI）安全** 这一新兴交叉方向，系统性整理了相关文献、基准、技术方案与安全分析，涵盖从多智能体协同、VLA 模型到物理攻击面的完整研究链路。

---

## 📂 仓库结构

| 目录 | 内容 |
|------|------|
| **`综述/`** | 多智能体协同决策综述、具身 AI 综述（含 400+ 引文 Safety Survey） |
| **`Tech/`** | 技术方案：多智能体 LLM 协作框架、Dual-System VLA、RoboOS、REMAC 等 |
| **`bench/`** | 具身 AI 评测基准：HABITAT 3.0、LOTA-BENCH、EMBODIEDBENCH、PARTNR、CoELA 等 |
| **`muitl-agent-SOTA/`** | 多智能体 SOTA 方法：RoCo、MindAgent、COMBO |
| **`组/Security Review/`** | 🔥 **安全四大顶会调研** + 具身安全攻击面分析 + 研究方向探讨 |
| **`组/`** | 组内其他材料：空地协同平台、ROS、周中汇报 |
| **`邓老师/`** | 文献汇报材料：VLA 模型、Dual-System、IoT-LLM、Genie、π₀ 等 |

### 🔐 安全方向核心内容 (`组/Security Review/`)

```
Security Review/
├── 安全四大调研.md              ← 安全四大 (S&P/CCS/NDSS/USENIX) 2025-2026 论文筛选
├── 研究讨论过程与汇报章程.md      ← 研究演进脉络 & PPT 汇报章程
├── Safe in EAI---总结/
│   ├── EAIAttack_Summary.md      ← 五层攻击面总览
│   ├── EAIAttack_Perception.md   ← 感知层攻击分析
│   ├── EAIAttack_Cognition.md    ← 认知层攻击分析
│   ├── EAIAttack_Planning.md     ← 规划层攻击分析
│   ├── EAIAttack_Action.md       ← 执行/交互层攻击分析
│   ├── EAIAttack_Agentic.md      ← Agentic 层攻击分析
│   ├── EAIAttack_TopConf_Safety.md ← AI 顶会安全论文筛选
│   └── 具身安全方向.md           ← 研究方向收窄 & 可行性分析
```

**核心发现：** 安全四大（~1,500 篇）几乎不做具身 AI 安全，但有大量可嫁接的硬件安全、TEE、CPS 研究。AI 顶会（~580 篇具身论文）仅 ~4% 与安全相关，越狱/人机信任/记忆投毒方向几乎空白。

---

## 🎯 研究方向

围绕 **"语义-物理鸿沟"** 这一具身 AI 独有安全特征，主要方向包括：

- **物理上下文感知的安全性校验**：环境变化导致安全属性翻转
- **跨模态融合攻击面**：融合机制本身构成攻击入口
- **具身越狱（BADROBOT）**：Contextual Jailbreak / Safety Misalignment / Conceptual Deception
- **Grounding Failure**：推理自洽但物理不可行的安全问题
- **VLA 模型鲁棒性**：视觉-语言-动作模型的对抗攻击与防御

---

## 🔗 快速链接

- **安全四大调研笔记（CDN 加速）**：[jsDelivr CDN](https://cdn.jsdelivr.net/gh/Chip-Security-Lab/Repo_ZhangZiJie_EAI-Security@master/%E7%BB%84/Security%20Review/%E5%AE%89%E5%85%A8%E5%9B%9B%E5%A4%A7%E8%B0%83%E7%A0%94.md)
- **研究讨论过程与汇报章程**：[GitHub 直达](组/Security%20Review/研究讨论过程与汇报章程.md)
- **具身安全方向探讨**：[GitHub 直达](组/Security%20Review/Safe%20in%20EAI---总结/具身安全方向.md)

---

## 👤 Maintainer

- **张子杰 (Zijie Zhang)** · [@111crab](https://github.com/111crab)
- **Lab:** [Chip-Security-Lab](https://github.com/Chip-Security-Lab)

---

> 📝 本仓库持续更新中。如有问题或合作意向，欢迎提 Issue。
