# §2 Perception 感知层 — 攻击手段总结

> 分析自：*Safety in Embodied AI: A Survey of Risks, Attacks, and Defenses* (arXiv:2605.02900v1, 2026.03)
> 标注：🔥 = 具身AI特色鲜明，适合报告展开；其余为传统攻击，简略带过。

---

## 感知层在具身AI Pipeline 中的位置

感知层是具身AI的最内层——AI 通过视觉、听觉、空间、运动、跨模态五个子领域感知物理世界。感知错误会向认知→规划→执行层层级联放大。

---

## §2.1 Visual Perception（视觉感知）

**攻击面：** 2D 相机图像 → 分类/检测/跟踪/分割 → 驱动下游任务

### 攻击类型

| 攻击类型 | 子类 | 做什么 | 代表方法 | 具身特色？ |
|---------|------|--------|---------|:--:|
| Adversarial（白盒） | 数字域 | 像素级扰动欺骗分类器/检测器/跟踪器/分割器 | PGD变体、Adversarial Overlay、HitM、RTAA、TrackPGD、CAA | 传统CV |
| | 物理域 | 打印贴纸、投影图案、路面伪装、衣服印花欺骗真实摄像头 | RP2、SLAP、DRP、AdvT、ShapeShifter、MFDA | 传统，物理域是具身前提 |
| Adversarial（黑盒） | 数字域 | 替代模型/查询优化/迁移攻击 | AdvTraj、Fang et al. (粒子群优化)、PB-UAP | 传统CV |
| | 物理域 | 激光投影、光学幻影、道路纹理篡改 | ELA、CAMOU、MobilBye、SSPA、NS Attack、ControlLoc | 传统 |
| Backdoor | 训练操纵 | 位翻转/提示投毒植入后门 | TrojViT、SWARM | 传统 |
| |  编码器供应链投毒 | **一个CLIP编码器被投毒 → 所有下游VLM+agent全带后门** | BadEncoder→BadCLIP→BadVision | 🔥 |

### 🔥 展开——编码器供应链后门

传统后门攻击针对单个任务模型（一个分类器、一个检测器），影响范围有限。现代具身系统普遍使用 CLIP/SigLIP 这类预训练视觉编码器作为感知 backbone——**一个编码器被投毒，所有依赖它的下游 VLM、导航、操控模块全部继承后门行为。**

具体攻击链：
- **BadEncoder [155]**：首次证明预训练编码器的后门随编码器传播到所有下游任务
- **BadCLIP [211]**：用贝叶斯推理优化触发器，后门更隐蔽
- **BadVision [228]**：自监督编码器后门诱导大视觉语言模型（LVLM）产生**视觉幻觉**——不是检测错误，而是"看到不存在的场景"

---

## §2.2 Auditory Perception（听觉感知）

**攻击面：** 语音识别 + 说话人验证 → 语音控制入口

### 攻击类型

| 攻击类型 | 子类 | 做什么 | 代表方法 | 具身特色？ |
|---------|------|--------|---------|:--:|
| Adversarial（白盒） | 物理域 | 空中传输对抗音频，人听不出异常但机器识别为恶意指令 | Carlini et al.、CommanderSong、Metamorph、SpecPatch、AdvPulse | 传统 |
| Adversarial（黑盒） | 数字+物理 | 替代模型/速度调节/穿墙攻击 | TSMAE、Occam、BarrierBypass、Devil's Whisper | 传统 |
| Backdoor | 训练操纵 | 声学模型后门植入 | TrojanModel | 传统 |
| **防御** | — | **后门防御完全空白（零研究）** | — | 研究缺口 |

**简评：** 整体与传统语音对抗攻击重叠度高，方法论无本质区别。具身场景的增量在于"空中传输+物理环境噪声下"的攻击鲁棒性。后门防御是明显空白。

---

## §2.3 Spatial Perception（空间/3D感知）

**攻击面：** LiDAR点云 / 深度估计 / SLAM / NeRF / 3DGS → 3D场景理解 → 导航与避障

### 攻击类型

| 攻击类型 | 子类 | 做什么 | 代表方法 | 具身特色？ |
|---------|------|--------|---------|:--:|
| Adversarial（白盒） | 数字域-3D检测/跟踪 | 点云扰动欺骗3D检测器/跟踪器 | FLAT、SlowLiDAR、Zheng et al.、Wang et al.、Cheng et al.、TAN | 传统 |
| | 数字域-SLAM | 注入扰动破坏LiDAR扫描匹配 | Yoshida et al. | 传统 |
| |  数字域-NeRF/3DGS | **扰动NeRF输入 → 渲染出虚假场景；投毒3DGS → 场景重建崩溃(DoS)** | Horváth & Józsa、Poison-Splat | 🔥 |
| | 物理域 | 优化对抗3D mesh/Morphing/阴影操控欺骗LiDAR | LiDAR-Adv、Tu et al.、AE-Morpher、Adv3D、ShadowHack | 传统，物理域是前提 |
| Adversarial（黑盒） | 数字+物理 | 全局搜索+局部细化、非反射对抗点、轨迹预测欺骗 | Hau et al.、TAPG、SpotAttack、LiDAttack、Lou et al. | 传统 |
| Backdoor | 数据投毒 | BEV触发/点扰动/伪造模式植入3D检测分割后门 | Zhang et al.、BadLiDet、BadLiSeg | 传统 |
| **防御** | — | **空间后门防御完全空白** | — | 研究缺口 |

### 🔥 展开——NeRF/3DGS 场景篡改

传统机器人用显式几何地图（点云、occupancy grid），攻击传感器即可改变地图。现代具身AI用神经网络隐式表征场景（NeRF/3DGS）——**攻击的不是传感器，而是AI"学出来的世界认知"**。

具体：
- **Horváth & Józsa [131]**：给 NeRF 训练输入加扰动 → NeRF 渲染出逼真但完全虚假的场景。AI "看到"的 obstacle 不存在，"看不到"的墙其实存在。传感器读数正常，但 AI 的"空间记忆"被篡改
- **Poison-Splat [243]**：投毒 3DGS 的自适应密度控制 → 不是让场景变假，而是让场景重建耗尽内存和计算资源，直接崩溃

> 与"世界模型攻击"的区别：NeRF攻击篡改的是**几何/外观表征**（AI看到的世界长什么样），世界模型攻击篡改的是**物理/因果规律**（AI理解的世界怎么运转）。

---

## §2.4 Motion Perception（运动/传感器感知）

**攻击面：** IMU / GNSS / 超声波 / mmWave 雷达 → 自身状态估计（位置、速度、姿态）

### 攻击类型

| 攻击类型 | 子类 | 做什么 | 代表方法 |
|---------|------|--------|---------|
| Spoofing | GNSS重放 | 捕获+延迟重传真实GPS信号 | Lenhart et al.、Wang et al. (OSNMA漏洞) |
| | GNSS生成 | SDR合成伪造GPS信号 | Horton et al.、FusionRipper、Dasgupta et al. |
| | 声学注入IMU | 超声波激发MEMS陀螺仪机械谐振→伪造角速度 | Son et al.、WALNUT、KITE |
| | 声学注入超声波 | 伪造回声信号欺骗测距 | Yan et al.、Xu et al.、Gluck et al. |
| | RF注入雷达 | 电磁信号伪造mmWave雷达点云 | Sun et al.、Komissarov et al.、mmSpoof |
| | 物理操纵 | 超材料表面操控雷达反射 | MetaWave、TileMask、mmHide |
| Jamming | 声学/电磁 | 高功率信号阻塞传感器 | Lim et al.、Jang et al. |

**简评：** 这节本质上是传统机器人/控制系统的传感器安全，攻击手段在具身AI出现前已存在十余年。报告中一句话带过即可。

---

## §2.5 Cross-Modal Perception（跨模态融合感知）🔥

**攻击面：** Camera + LiDAR + Radar 融合机制 → 联合感知

### 攻击类型

| 攻击类型 | 子类 | 做什么 | 代表方法 | 具身特色？ |
|---------|------|--------|---------|:--:|
| Adversarial |  数字域-单通道攻陷融合 | **仅攻击LiDAR→200个点→99%攻陷整个camera+LiDAR融合系统** | Li et al. [189] |  |
| |  数字域-时序错位 | **单帧LiDAR延迟→3D检测mAP崩溃88.5%，攻击传感器同步机制** | DejaVu [133] | 🔥 |
| |  物理域-三模态同时欺骗 | **一个物理对抗物体同时欺骗camera+LiDAR+radar** | Li et al. [207] | 🔥 |
| | 物理域-跨模态对抗物体 | 3D打印物体同时欺骗camera+LiDAR；黑盒LiDAR欺骗攻陷8种融合算法 | Cao et al.、Hallyburton et al. |  |

### 🔥 展开——融合机制本身就是攻击面

直觉告诉我们"多加传感器更安全"。但这节的研究证明了**相反的结论**：

1. **最弱通道决定整体安全**：融合没有纠正错误，反而把单通道的污染传播到全局。Li et al. [189] 仅攻击 LiDAR 就把 camera+LiDAR 融合检测器拿下（99%成功率，仅200个对抗点）
2. **时序是隐藏攻击面**：DejaVu [133] 发现传感器同步延迟（真实物理环境中极易发生）对融合系统是灾难性的——不是攻击任何模型，而是攻击"系统架构的假设前提"（假设所有传感器始终同步）
3. **全模态系统性欺骗**：Li et al. [207] 证明单一物体可以同时欺骗三种异构传感器——攻击者不必选择最弱环节，可以系统性欺骗所有感知通道

---

## 感知层总结

| 子领域 | 传统成分 | 具身AI特色 | 防御空白 |
|--------|---------|-------------|:--:|
| 2.1 Visual | 数字/物理对抗攻击、数据投毒后门 | 编码器供应链后门（BadEncoder/BadCLIP/BadVision） | — |
| 2.2 Auditory | 语音对抗、空中传输 | （无本质特色） | 后门防御空白 |
| 2.3 Spatial | 点云对抗、3D检测攻击 | NeRF/3DGS场景篡改 | 空间后门防御空白 |
| 2.4 Motion | GNSS/IMU/雷达 spoofing & jamming | （全传统） | — |
| 2.5 Cross-Modal | 融合对抗训练 | 单通道攻陷融合、时序错位、全模态欺骗 | — |
