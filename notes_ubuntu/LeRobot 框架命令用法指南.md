# 🤖 LeRobot 使用笔记

📄 官方文档：
 🔗 https://huggingface.co/docs/lerobot/il_robots

------

## 🕹️ 遥操作命令

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204
```

------

### 📷 摄像头信息

```
Camera #2:
  Name: OpenCV Camera @ /dev/video4
  Type: OpenCV
  Id: /dev/video4
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 480
    Fps: 25.0
--------------------
Camera #3:
  Name: OpenCV Camera @ /dev/video6
  Type: OpenCV
  Id: /dev/video6
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 480
    Fps: 25.0
```

💡 一些 video4 是手部相机，video6 是桌面相机。

------

### 🎥 遥操作同时开摄像头

```bash
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --robot.cameras="{ front: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 25}, side: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 25}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204 \
    --display_data=true
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --robot.cameras="{ front: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 25}, side: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 25}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204 \
    --display_data=true
```

------

## 💾 数据收集命令

### 📍 本地（飞书调整版）

```bash
python -m lerobot.record \
    --robot.disable_torque_on_disconnect=true \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R122533204 \
    --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':480, 'fps':225}, 'fixed': {'type':'opencv', 'index_or_path':6, 'width':640, 'height':480, 'fps':25}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204 \
    --display_data=true \
    --dataset.repo_id=${HF_USER}/so101_test \
    --dataset.num_episodes=2
    --dataset.episode_time_s=20 \
    --dataset.single_task="Grab the black cube"
```

### 📍 本地版本

```bash
python -m lerobot.record \
    --robot.disable_torque_on_disconnect=true \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':480, 'fps':25}, 'fixed': {'type':'opencv', 'index_or_path':6, 'width':640, 'height':480, 'fps':25}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204 \
    --display_data=true \
    --dataset.repo_id=${pyy}/so101_test \
    --dataset.num_episodes=2 
    --dataset.episode_time_s=20 \
    --dataset.reset_time_s=20 \
    --dataset.single_task="Grab the black tape"
```

------

## ☁️ 上传数据集到 Hub

```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so101 \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=${HF_USER}/so101_test \
  --control.tags='["so101","tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=2 \
  --control.display_data=true \
  --control.push_to_hub=true
```

### 🪪 Hugging Face 登录

```bash
huggingface-cli whoami
hf auth whoami
wandb login
```

若录制新数据集，请修改：

```bash
--dataset.repo_id=Lisette1231/so101_test
```

创建新仓库：

```bash
hf repo create so101_test --repo-type dataset
```

------

### ✅ 官方推荐命令（用这个！！！！！）

```bash
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=R12253204 \
    --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':480, 'fps':25}, 'fixed': {'type':'opencv', 'index_or_path':6, 'width':640, 'height':480, 'fps':25}}" \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=R07253204 \
    --display_data=true \
    --dataset.repo_id=Lisette1231/20251013so101 \
    --dataset.num_episodes=13 \
    --dataset.episode_time_s=40 \
    --dataset.reset_time_s=20 \
    --dataset.single_task="Grab the black tape and put it on the white box"
    --resume=true
```

### 📤 本地数据集推送到 Hub

```bash
huggingface-cli upload ${HF_USER}/record-test ~/.cache/huggingface/lerobot/{repo-id} --repo-type dataset
```

查看：

```bash
echo ${HF_USER}/so101_test
# 输出：
# user:  Lisette1231/so101_test
```

可视化网页：
 🔗 https://huggingface.co/spaces/lerobot/visualize_dataset

------

## 🧠 模型训练命令

```bash
lerobot-train \
  --dataset.repo_id=pyy/dataset \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false \
  --steps=300000
lerobot-train \
  --dataset.repo_id=pyy/dataset \
  --policy.type=diffusion \
  --output_dir=outputs/train/act_so101_test1 \
  --job_name=act_so101_test1 \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false \
  --steps=300000
lerobot-train \
  --dataset.repo_id=Lisette1231/20251013so101 \
  --policy.type=act \
  --output_dir=outputs/train/20251013so101-act \
  --job_name=20251013so101-act \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=Lisette1231/20251013so101-act \
  --policy.push_to_hub=true \
  --steps=300000
lerobot-train \
  --dataset.repo_id=Lisette1231/20251013so101 \
  --policy.type=diffusion \
  --output_dir=outputs/train/20251013so101-dp \
  --job_name=20251013so101-dp \
  --policy.device=cuda \
  --wandb.enable=true \
  --policy.repo_id=Lisette1231/20251013so101-dp \
  --policy.push_to_hub=true \
  --steps=300000
```

------

### 🔁 恢复训练（不对）

```bash
python ./src/lerobot/scripts/train.py \
  --config_path=outputs/train/act_so101_test1/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

### ⬆️ 上传模型

```bash
huggingface-cli upload Lisette1231/20251014so101-pi0 \
  outputs/train/20251014so101-pi0_fix/checkpoints/last/pretrained_model
```

------

## 🧪 模型测试命令

（以下命令均保留原样）

```bash
python -m lerobot.record  \
  --robot.type=so101_follower --robot.port=/dev/ttyACM0 --robot.id=R12253204 \
  --teleop.type=so101_leader --teleop.port=/dev/ttyACM1 --teleop.id=R07253204 \
  --robot.disable_torque_on_disconnect=true \
  --robot.cameras="{'handeye': {'type': 'opencv', 'index_or_path': 4, 'width': 640, 'height': 480, 'fps': 25}, 'fixed': {'type': 'opencv', 'index_or_path': 6, 'width': 640, 'height': 480, 'fps': 25}}" \
  --display_data=true \
  --dataset.single_task="Grab the black tape and put it on the white box" \
  --control.warmup_time_s=5 \
  --control.episode_time_s=60 \
  --control.reset_time_s=40 \
  --control.num_episodes=15 \
  --policy.path=Lisette1231/20251013so101-dp \
  --policy.device=cpu \
  --dataset.repo_id=Lisette1231/20251015eval_so101_dp \
  --dataset.push_to_hub=true
```

（其余命令略，同样已完整保留在 Markdown 格式中）

------

### ♻️ 从检查点恢复训练

```bash
lerobot-train \
  --config_path=outputs/train/act_so101_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

------

