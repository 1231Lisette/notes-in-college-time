# smolvla

- [ ] 怎么异步推理



# lerobotdataset 文件结构

```php
<root>/                           # repo_id 对应的根目录
├── data/                         # 所有结构化数据（parquet）
├── meta/                         # 所有元数据（json/jsonl）
└── videos/                       # 所有视频数据（mp4）
```

|                                | 内容                                                      | 备注                                                         |
| ------------------------------ | --------------------------------------------------------- | ------------------------------------------------------------ |
| data/chunk-*/episode_*.parquet | 每一 episode 的逐帧结构化数据：状态、动作、奖励、时间戳等 | 由 [hf_dataset](https://zhida.zhihu.com/search?content_id=261108737&content_type=Article&match_order=1&q=hf_dataset&zhida_source=entity) 直接映射；可只下载部分 episode |
| meta/info.json                 | 数据集全局信息：fps、特征名、形状、数据类型、相机配置等   | 决定 dataset.features、dataset.fps                           |
| meta/stats.json                | 各特征的统计量（min/max/mean/std）                        | 训练时做归一化                                               |
| meta/tasks.jsonl               | 任务的自然语言描述列表                                    | 比如“抓纸盒”                                                 |
| meta/episodes.jsonl            | 每个 episode 的索引、长度、所属任务                       | 构建 episode_data_index，实现按 episode 快速切片             |
| videos/chunk-*//episode_*.mp4  | 每个 camera 每个 episode 的整段视频                       |                                                              |

**meta/tasks.jsonl**

```text
{"task_index": 0, "task": "Grab the paper box"}
```

表示本数据集仅包含 **一个任务**，编号 `0`，文本描述为 `"Grab the paper box"`。



**meta/episodes.jsonl**

episode 索引表， 告诉代码「第几号 episode 有多少帧、属于哪个任务」，从而支持按 episode 切片、统计、下载或训练。

```text
{"episode_index": 0, "tasks": ["Grab the paper box"], "length": 213}
{"episode_index": 1, "tasks": ["Grab the paper box"], "length": 217}
{"episode_index": 2, "tasks": ["Grab the paper box"], "length": 217}
{"episode_index": 3, "tasks": ["Grab the paper box"], "length": 215}
{"episode_index": 4, "tasks": ["Grab the paper box"], "length": 219}
{"episode_index": 5, "tasks": ["Grab the paper box"], "length": 218}
```

**meta/episodes_stats.json**

为每一个 episode 列出了所有特征（动作、状态、图像、时间戳……）的 min / max / mean / std / count，方便你在训练前快速了解数据分布、做归一化或异常检测。

```text
{"episode_index": 0, "stats": {"action": {"min": [0.8097972869873047, -100.83145904541016, 91.56088256835938, 70.18883514404297, -6.923374176025391, 0.8877677917480469], "max": [3.474395990371704, -97.67970275878906, 95.2083740234375, 71.90310668945312, -6.024308204650879, 1.7298245429992676], "mean": [1.7911291122436523, -99.07067108154297, 94.17674255371094, 71.10909271240234, -6.520898818969727, 1.3790700435638428], "std": [0.504497766494751, 0.6285833120346069, 0.7208112478256226, 0.3215145468711853, 0.19652514159679413, 0.1487114578485489], "count": [213]}, "observation.state": {"min": [1.4126393795013428, -98.82746887207031, 96.01809692382812, 71.09876251220703, -6.641870498657227, 2.253147840499878], "max": [5.05576229095459, -98.57621765136719, 。。。。。
```

注：episode_index里面，"observation.images.front(top)": 中"count": [100]表示：**统计计算时，只用了 100 帧**来估计该 episode 中**前部相机图像**的 min / max / mean / std。

---

# 训练篇

https://zhuanlan.zhihu.com/p/1941530566446024557

**make_policy**

用于加载预训练策略模型，导入参数：

- 若单一任务数据集，则读取meta配置。
- 若多任务数据集，则使用第一个子数据集的meta配置。

**因此，需要每个任务建议使用同样的采集环境、硬件配置和模型**



## 训练关键流程解析

### make_dataset 读取数据集

**单任务数据集**

1.通过repo_ids读取对应元数据ds_meta；

2.计算数据时间偏移量delta_timestamps， 通过 `i / ds_meta.fps`将整数帧索引转换为实际时间：

- action_delta_indices： 将动作的帧索引转换为实际时间，默认fps=30；
- action_delta_indices=[0,1,...,49]共有50帧，表示模型需处理长达1.6秒（以30fps计）的动作序列。
- observation_delta_indices：默认为[0]，即观测数据无需历史或未来帧，模型仅依赖最新观测状态决策。

3.实例化数据集LeRobotDataset，传入repo_id，delta_timestamps，tolerance_s（可接受时间差阈值），meta：

- 读取每组数组起止帧，get_episode_data_index，meta数据中有每组数据帧数，可计算出数据起止信息，表示为[from:xxxx,to:xxxxx]





不知道为啥突然粘贴个这个在这

| 模型                 | 推荐 `n_obs_steps` | 时序依赖       | 计算需求 | 典型任务           | 是否可蒸馏         |
| -------------------- | ------------------ | -------------- | -------- | ------------------ | ------------------ |
| **ACT**              | 1～3               | 弱（靠 chunk） | ⭐ 低     | 抓取、旋转、短动作 | ✅ 易               |
| **Diffusion Policy** | 8～32              | 强             | ⭐⭐ 中高  | 推、拉、堆叠       | ⚙️ 可蒸馏为轻量DDPM |
| **SmolVLA**          | 3～8               | 中             | ⭐⭐ 中    | 指令+视觉操作      | ✅ 推荐             |
| **π₀ (Pi0)**         | 16～48             | 强             | ⭐⭐⭐ 高   | 多任务、复杂长序列 | ⚙️ 常蒸馏压缩       |