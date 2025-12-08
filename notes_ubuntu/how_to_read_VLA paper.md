# how to read VLA paper？

#### 1. 数据格式与收集数据的方式 (Data Acquisition & Format)

- **收集方式：**
  - **遥操作 (Teleoperation)：** 使用 VR 手柄、外骨骼手臂（Puppeteer），还是 3D 鼠标（SpaceMouse）？（这决定了数据的质量和精细度）。
  - **人类视频 (Human Video)：** 像 Youtube 视频，还是特定采集的第一人称（Ego-centric）视频？（需要解决 Domain Gap）。
  - **仿真合成 (Simulation)：** 使用 MuJoCo, Isaac Gym, SAPIEN 等生成的数据？
- **数据存储格式：** 这一块通常关注是否使用了统一标准，如 **RLDS (Reinforcement Learning Datasets)** 格式，或者是 HDF5。

#### 2.任务内容与范围 (Scope & Setting)

- **Sim vs. Real:**
  - **Sim-only:** 仅在仿真中验证，关注算法理论上限。
  - **Real-only:** 仅在真机验证，强调鲁棒性。
  - **Sim-to-Real:** 在仿真训练，直接部署到真机（Zero-shot transfer）或微调后部署。
- **任务类型：**
  - **Table-top:** 桌面级操作（拾取、放置、抽屉开关）。
  - **Mobile Manipulation:** 移动操作（导航+操作，如去厨房拿可乐）。
  - **Dexterous Hand:** 灵巧手操作（涉及复杂的手指协同）。

#### 3. 输入与输出的模态（Modality & Representation）

- **输入：**除RGB图像和文本指令外，是否融合了**深度图（Depth）**、**点云（Point Cloud）或趋势**？
- **输出（关键）：**模型是如何“说话”变成“动作”的？
  - **离散化（Discretization）：一个**动作空间（如$x, y, z, roll, pitch, yaw, gripper$）离散化成Token（类似GPT的词表），直接用分类头预测？（如RT-2）。
  - **连续值(Continuous)：**还是输出连续的回归值？
  - **动作空间：**是**关节空间（Joint space）\**还是\**终止笛卡尔空间（End-effector space）**？这决定了泛化性的难易。

#### 4. 骨干网络（Backbone）

- **基座模型：** LLM (Llama, Vicuna) 还是 VLM (CLIP, SigLIP, LLaVA)？
- **模态融合 (The Bridge)：** 视觉特征是如何注入 LLM 的？
  - 简单拼接 (Concatenation)？
  - 通过 Projector (Linear/MLP)？
  - 通过 Q-Former / Perceiver Resampler (如 Flamingo 架构)？
- **微调策略：** 它是全量微调（Full Fine-tuning）还是使用了 LoRA 等参数高效微调？视觉编码器（Visual Encoder）是否被冻结？这关系到训练成本和灾难性遗忘问题。

#### 5.  数据集与数据配比（Data Strategy）

具身智能目前最大的瓶颈是**高质量数据**。

- **数据配比 (Data Mixture)：**
  - Internet Data (VQA, Captioning) vs. Robot Trajectory Data。
  - **Reweighting：** 这是一个关键点。因为机器人数据稀缺，通常需要对机器人数据进行**过采样 (Oversampling)**，否则 LLM 会忘记如何输出动作。
- **数据增强 (Augmentation)：** 是否对图像进行了颜色抖动、裁剪、旋转，以提高鲁棒性？

#### 6. 推理频率与实时性（Inference Speed）

- 机器人的控制通常需要 10Hz - 50Hz。
- 如果模型很大（如 7B 参数），它的推理速度是多少？是否需要量化？如果太慢（如 1-2Hz），它是如何部署到实机上的？（通常需要配合一个高频的小模型作为控制器）。
- **推理速度：** 7B+ 模型通常只有 $1 \sim 3 \text{Hz}$。
- **解决方案 - Action Chunking (关键技术)：**
  - 模型不是每一步输出一个动作 $a_t$，而是**一次输出未来 $k$ 步的动作序列** $\{a_t, a_{t+1}, ..., a_{t+k}\}$。
  - **优势：** 可以在执行这 $k$ 步动作的时间内进行下一次推理，从而掩盖推理延迟；同时动作轨迹更平滑。
  - *阅读时注意：Paper 中是否提到了 Temporal Aggregation 或 Action Chunking？*

#### 7. 泛化能力（Generalization）

- **零样本（Zero-shot）：** 能否识别训练集中没见过的物体？
- **长序列任务：** 能否理解“把桌子上红色的苹果拿起来放到蓝色的盘子里，然后把香蕉给旁边的人”这种多阶段指令？

**泛化层级：**

- **L1: Unseen Instances:** 见过的物体类别（如杯子），但没见过这个具体的颜色或形状。
- **L2: Unseen Categories:** 完全没见过的物体类别。
- **L3: Unseen Backgrounds/Environments:** 换了个桌子，换了个房间。
- **L4: Unseen Instructions:** 换了一种说法（"把苹果给我" vs "我饿了，给点水果"）。

**评估指标 (Metrics)：**

- **Success Rate (SR):** 任务成功率。
- **Goal Conditioned:** 仅仅是到了位置，还是姿态也对了？