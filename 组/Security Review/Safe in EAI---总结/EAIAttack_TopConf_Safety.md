# 具身AI安全 — 顶会论文筛选

> 数据来源：https://github.com/Songwxuan/Embodied-AI-Paper-TopConf
> 合集规模：~580+ 篇（ICML/NeurIPS/ICLR/CVPR/ICCV/CoRL/RSS等）
> 安全相关：**23 篇（约 4%）**

---

## 一、综述收录情况

| 合集 | 综述数量 | CCF-A 收录 |
|------|:------:|:--:|
| [Awesome-Embodied-AI-Safety](https://github.com/Rookie143/Awesome-Embodied-AI-Safety) | 2 篇 | **无**——均为 arXiv 预印本 |

> 具身AI安全方向目前尚无已发表的 CCF-A 综述，领域仍处于形成期。

---

## 二、安全相关论文分类

### 1. 攻击类（2 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| BadVLA: Towards Backdoor Attacks on Vision-Language-Action Models via Objective-Decoupled Optimization | NeurIPS 2025 | VLA 后门攻击——训练时注入触发器，部署时激活恶意行为 |
| Exploring the Adversarial Vulnerabilities of Vision-Language-Action Models in Robotics | ICCV 2025 | VLA 对抗漏洞——探查视觉/语言/跨模态攻击面 |

### 2. 鲁棒性类（4 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| On Robustness of Vision-Language-Action Model against Multi-Modal Perturbations | ICLR 2026 | VLA 对多模态扰动的鲁棒性 |
| Robust Fine-tuning of Vision-Language-Action Robot Policies via Parameter Merging | ICLR 2026 | 参数融合提升 VLA 鲁棒微调 |
| Uncovering Robot Vulnerabilities through Semantic Potential Fields | ICLR 2026 | 通过语义势场发现机器人策略漏洞 |
| Dismantling the Illusion of Vision-Language-Action Models Competence via Explicit Distributional Shifts | ICML 2026 | 通过分布偏移揭示 VLA 虚假能力 |

### 3. 安全规划/控制类（5 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| SafeDec: Constrained Decoding for Safe Autoregressive Generalist Robot Navigation Policies | ICML 2026 | 约束解码保证导航策略安全 |
| SafeFlowMatcher: Safe and Fast Planning using Flow Matching with Control Barrier Functions | ICLR 2026 | 基于 CBF 的安全快速规划 |
| SafeBimanual: Diffusion-based Trajectory Optimization for Safe Bimanual Manipulation | CoRL 2025 | 安全双手操控轨迹优化 |
| Latent Policy Barrier: Learning Robust Visuomotor Policies by Staying In-Distribution | NeurIPS 2025 | 保持分布内学习鲁棒策略 |
| PACT: Self-Evolving Physical Safety Alignment for Diffusion Policies in Embodied Manipulation | ICML 2026 | 扩散策略的物理安全自进化对齐 |

### 4. 故障检测/恢复类（4 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| SAFE: Multitask Failure Detection for Vision-Language-Action Models | NeurIPS 2025 | VLA 多任务故障检测 |
| Failure Prediction at Runtime for Generative Robot Policies | NeurIPS 2025 | 生成式策略运行时故障预测 |
| Can VLMs Diagnose and Recover from VLA Manipulation Faults? | ICML 2026 | VLM 诊断和修复 VLA 操控故障 |
| ReCAPA: Hierarchical Predictive Correction to Mitigate Cascading Failures | ICLR 2026 | 分层预测修正缓解级联故障 |

### 5. 安全基准/评估类（3 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| SafeLab: An Interactive High-Fidelity Benchmark for Embodied Safety in Scientific Robotics | ICML 2026 | 具身安全交互式高保真基准 |
| When would Vision-Proprioception Policies Fail in Robotic Manipulation? | ICLR 2026 | 视觉-本体感觉策略失败条件分析 |
| Ensuring Force Safety in Vision-Guided Robotic Manipulation via Implicit Tactile Calibration | CoRL 2025 | 视觉引导操控中的力安全保证 |

### 6. 其他（5 篇）

| 论文 | 会议 | 研究方向 |
|------|------|---------|
| Emerging Risks from Embodied AI Require Urgent Policy Action | NeurIPS 2025 | 政策/治理——呼吁对具身AI新兴风险采取行动 |
| Remotely Detectable Robot Policy Watermarking | ICLR 2026 | 策略水印——机器人策略知识产权保护 |
| Embodied Interpretability: Linking Causal Understanding to Generalization in Vision-Language-Action Models | ICML 2026 | 可解释性——因果理解与 VLA 泛化 |
| RoboChemist: Long-Horizon and Safety-Compliant Robotic Chemical Experimentation | CoRL 2025 | 安全合规的化学实验机器人 |
| Towards Reliable LLM-based Robots Planning via Combined Uncertainty Estimation | NeurIPS 2025 | LLM 机器人规划的可靠性 |

---

## 三、分布统计

| 维度 | 数量 |
|------|:--:|
| 安全论文总数 | 23 |
| 占总合集比例 | ~4% |
| VLA 相关攻击/鲁棒性 | 8 篇（最热） |
| 安全规划/控制 | 5 篇 |
| 故障检测/恢复 | 4 篇 |
| **后门攻击** | **1 篇**（BadVLA） |
| **越狱攻击** | **0 篇** |
| **人机信任攻击** | **0 篇** |
| **记忆投毒** | **0 篇** |
| **多智能体安全** | **0 篇** |

---

## 四、关键结论

1. **具身AI安全在顶会中占比极低（4%）**，且集中在 VLA 鲁棒性和安全规划
2. **攻击研究极度稀缺**（仅 2 篇），后门 1 篇、越狱 0 篇
3. **综述层面**——尚无任何 CCF-A 综述，领域处于形成期
4. **研究空白**——越狱攻击、人机信任攻击、记忆投毒、多智能体安全攻击在顶会中为零
