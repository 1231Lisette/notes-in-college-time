VLM：Accept text and images as input and predicts the next text tokens and optionally images

VLA: Accept text, images, and state as input and predicts robot actions

| Robotics policy usage pattern | NLP analogy                                | Current state                                        |
| ----------------------------- | ------------------------------------------ | ---------------------------------------------------- |
| 1. No end-to-end policies     | Classic NLP (pre-NN)                       | Many resources are available (ROS2)                  |
| 2. Train from scratch         | Problem-specific models (RNNs)             | Can be done at home (ACT, Diffusion, VQ-BET, DOT...) |
| 3. Fine-tune on the task(s)   | Big pretrained models (ELMo, ULMFiT, BERT) | Can be done at home (Pi0, Gr00t, SmolVLA...)         |
| 4. Fine-tune for the robot    | Text-to-text (T5, GPT-2/3)                 | Frontier labs focus (Pi0.5, Gemini Robotics...)      |

# Vision-Language-Action Models for Robotics: A Review Towards Real-World Applications

## 定义 

VLA是一种以视觉观察和自然语言指令作为必要输入的系统，并可能融入其他感知模态。它通过直接生成控制命令来生成机器人动作。因此，高级策略(例如，VLM backbone)仅从一组预训练技能或控制原语中进行选择的模型不算VLA。

## 三大挑战

### 1. 数据

VLA模型的训练需要大规模、多样化且标注精确的数据，这些数据需将视觉观察、自然语言指令和相应动作紧密结合。然而，现有数据集存在明显短板:像COCO-Captions这类视觉语言数据集虽语言信息丰富，却缺乏机器人所需的动作信息;而机器人演示数据集则往往语言多样性不足或任务范围过于狭窄。

### 2. 实体迁移

机器人形态各异，从只有手臂的机械臂到带轮子或腿的移动机器人，它们的关节、结构、传感器和外观差异巨大。尽管VLA模型越来越多地使用来自不同形态机器人的数据进行训练，但让模型策略在不同机器人之间迁移仍然是一个巨大挑战。每个机器人的动作空间和本体感知空间(反映自由度、传感器和运动结构的差异)都是独特的。

### 3. 计算成本

由于输入数据包含视觉、语言和动作等高维多模态信息，训练VLA模型需要巨大的计算资源。在推理阶段，将这些模型部署到资源有限的机器人平台上，也面临着延迟和内存占用的挑战。

## VLA模型发展阶段总结

### 1. 早期基于CNN的端到端架构CLIPort

### 2. 基于Transformer的序列结构

### 3. 结合预训练VLM的统一真实世界策略

RT-1是首个统一了大量真实世界机器人任务的模型它通过早期融合视觉和语言特征，并结合大规模真实数据训练实现了通用控制。其后续模型RT-2则开创性地将一个在互联网数据上预训练好的大型视觉语言模型(VLM)作为主干，再用机器人数据进行微调，极大地提升了泛化能力，这-“VLM作为主干”的设计成为了后续VLA模型的标准范式。而RT-X则证明了使用来自多机器人的数据集进行训练，能开发出更通用的VLA模型。

### 4. 扩散策略（Diffusion Policy）

Octo作为继RT系列之后的一个重要开源模型，首次将**扩散策略(Diffusion Policy)**引入VLA，用于生成**连续的机器人动作**，并**支持包括语言和目标图像在内的灵活指令**

### 5. 扩散Transformer架构（Diffusion Transformer）

RDT-1B将扩散过程直接整合到Transformer的解码器中，而不仅是作为输出动作的模块。这种架构利用**DiT (Diffusion Transformer)**作为其主干，通过交替注入图像和文本条件来训练模型，从而生成动作。

### 6. 流匹配策略架构（Flow Matching Policy）

π0在PaliGemma模型基础上，引入了名为“动作专家”的定制化动作输出模块，该模块利用**流匹配(Flow Matching)**技术以高达50Hz的频率生成平滑且实时的连续动作，尤其适合需要快速响应的真实世界控制。
从视频中学习潜在动作LAPA提出了一种创新方法，通过**在无标签的视频数据上进行预训练来学习“潜在动作"表征**。这种方法使得模型能有效利用人类演示视频，增强了对不同机器人形态的适应性，非常适合真实世界的部署。

### 7. 分层策略架构(Hierachical Policy)

最新一代的VLA模型普遍采用分层策略，以连接高层的语言理解与底层的电机控制。RT-H是这一设计的典型代表，它包含一个高层策略用于生成抽象的“语言动作”计划，再由一个低层策略将其细化为具体动作。GROOT N1等模型则集成了潜在动作学习、扩散生成和流匹配控制器等多种技术，形成了一个统一的多阶段策略，代表了当前可扩展和自适应VLA模型的最前沿方法。

---

有个重要的timeline

1. RT系列引入VLM大大提升了泛化能力
2. π0使用flow matching的action expert将控制频率提到50Hz能够产生平滑的动作
3. 分层策略机构，今年算是标配了。来源是

## 模型架构分类与数据模态
### 1. Transformr+离散动作令牌（Discrete action token）
这类架构将图像和语言等输入统一转换为令牌(tokens)，再送入一个Transformer模型来预测**离散化的动作。**

代表性工作包括采用自回归方式输出动作的Gato和VIMA，以及采用非自回归方式并成为后续通用设计的RT-1。

### 2. Transformr+扩散动作头（Diffussion action head）
该架构在Transformer之后加入一个扩散策略(DP)作为动作输出头，以生成平滑且连续的机器人动作，弥补了离散动作实时性不足的缺点。

Octo和NoMAD是典型代表

### 3. 扩散Transformer（Diffusion Transformer）
这种架构将扩散过程直接整合到Transformer模型内部，使其能够直接在图像和语言令牌的条件下执行扩散来生成动作，而不是将扩散模型作为独立输出模块。

代表模型包括RDT-1B和Large Behavior Models(LBMs).

### 4. VLM+离散动作令牌（Discrete action token）
该架构用一个在海量互联网数据上预训练好的大型视觉语言模型(VLM)替代了普通的Transformer，利用VLM的常识知识和上下文学习能力，极大地提升了模型的泛化性。

开创性的RT-2以及主流开源模型OpenVLA均采用此设计

### 5. VLM+扩散动作头（Diffusion action head）
此架构结合了VLM的强大泛化能力与扩散模型生成平滑连续动作的优点，即将(B)中的Transformer替换为VLM。

Diffusion-VLA和DexVLA等模型是这一方向的代表。

### 6. VLM+流匹配动作头（Flow matching action head）
该架构将(E)中的扩散模型替换为响应速度更快的流匹配(Flow Matching)动作头，以在保持平滑控制的同时实现更高的实时性。

基于PaliGemma的π0是其代表，可实现高达50Hz的控制频率。

### 7. VLM+扩散Transformer（Diffusion Transformer）
这种架构将VLM与扩散Transformer相结合，通常形成一个分层策略:VLM作为高级策略(系统2)进行规划，扩散Transformer作为低级策略(系统1)执行具体动作。

代表模型GROOT N1就应用了此设计。
---

各种模型架构的介绍顺序与模型架构演变顺序基本一致，趋势是backbone由transformer到VLM，输出部分由离散动作token到扩散模型输出连续动作。

由于VLM的频率比较低，分层架构出现也是十分自然的。

## 训练方法
### supervised learning
与LLM和VLM训练类似，VLA的supervisedlearning分为pre-training 和 post-training，参照图1 Pi-0的训练范式。

pre-training 使用数据集非常diversity，例如人类演示、异构机器人演示或与机器人规划相关的 VQA 数据集。

与 LLM 类似，数据规模在 VLA 预训练中起着至关重要的作用，在这种相似性下，大多数VLM工作会直接使用pre-trianed VLM。post-training 使用机器人特定domain的数据集，这个阶段的训练中，数据的质量要比规模更重要。

与LLM类似，VLA也存在一定的in-context learningVLA in-context learning 使它在推理时根据少量人类遥控轨迹推断适当的动作。这是挺有意思的，感兴趣的可以看一下这篇论文(In-Context lmitation Learningvia Next-Token Prediction)

### self-supervised learning

self-supervised learning 主要在研究三个问题Modality alignment , Visual representation 和 Latentaction representation learning。最后一个是指通过从初始图像和目标图像中提取潜在动作，并使用初始图
### reinforcement leaning
