# yolov8 pipline

```php
HT22_ControlBox_Detection/
├── configs/
│   ├── config_m.yaml           # 🔧 模型和训练参数配置（可换数据集）
│   ├── dataset.yaml            # 📂 数据集定义（路径、类别）
│
├── data/
│   ├── data_yolo/              # 训练/验证图片和标签
│   └── data_preprocess/        # 原始图片预处理（可选）
│
├── experiments/
│   ├── runs/                   # 🚀 自动生成的训练实验（best.pt、结果图等）
│   └── results/                # 推理输出图像
│
├── src/
│   ├── data.py                 # 🧩 数据预处理与增强模块
│   ├── utils.py                # 🛠️ 工具函数（日志、绘图、路径管理）
│   ├── infer.py                # 🎯 推理模块（含中文绘制）
│   ├── train.py                # 📈 训练器（封装YOLO训练API）
│   ├── eval.py                 # 📊 评估逻辑
│
├── main.py                     # 🧭 主入口，统一调度train/eval/infer/export
└── README.md                   # 📄 项目说明（可选）

```

数据处理——训练——验证——推理——导出

## 一、`configs/config.yaml`（核心配置文件）

```yaml
# configs/config.yaml

train:
  model: yolov8m.pt           # 模型结构 (可选 yolov8n/s/m/l/x)
  data: configs/dataset.yaml  # 数据集配置文件路径
  epochs: 100
  batch: 16
  imgsz: 640
  device: 0                   # GPU编号
  lr0: 0.001
  optimizer: SGD
  patience: 20
  project: experiments/runs
  name: yolov8m_exp
  save: True
  exist_ok: True

inference:
  conf_threshold: 0.25
  iou_threshold: 0.45
  max_det: 1000

eval:
  save_json: False
```

------

## 二、`configs/dataset.yaml`（数据集定义）

```yaml
path: data/data_yolo
train: images/train
val: images/val
test: images/test  # 可选

nc: 4
names: ['控制适应标识', '无螺丝', '有螺丝', '电路板']
```

##  三、`src/train.py`（训练逻辑封装）

```python
import os
from ultralytics import YOLO
import logging
import yaml

class Trainer:
    def __init__(self, config_path):
        self.logger = logging.getLogger("Trainer")
        self.config = self._load_config(config_path)
        self.model = YOLO(self.config['train']['model'])

    def _load_config(self, config_path):
        with open(config_path, 'r', encoding='utf-8') as f:
            return yaml.safe_load(f)

    def train(self):
        cfg = self.config['train']
        self.logger.info(f"开始训练模型: {cfg['model']}")
        self.model.train(
            data=cfg['data'],
            epochs=cfg['epochs'],
            imgsz=cfg['imgsz'],
            batch=cfg['batch'],
            device=cfg['device'],
            lr0=cfg['lr0'],
            optimizer=cfg['optimizer'],
            patience=cfg['patience'],
            project=cfg['project'],
            name=cfg['name'],
            exist_ok=cfg['exist_ok'],
        )
        self.logger.info("训练完成！模型权重已保存。")
```

------

## 四、`src/infer.py`（推理模块，支持中文绘制）

```python
import cv2
import os
import numpy as np
from PIL import ImageFont, ImageDraw, Image
from ultralytics import YOLO

class Inference:
    def __init__(self, model_path, config):
        self.model = YOLO(model_path)
        self.config = config
        self.class_names = ['控制适应标识', '无螺丝', '有螺丝', '电路板']
        self.font = ImageFont.truetype("/usr/share/fonts/truetype/wqy/wqy-microhei.ttc", 20)

    def predict(self, source_dir, output_dir):
        os.makedirs(output_dir, exist_ok=True)
        imgs = [f for f in os.listdir(source_dir) if f.endswith(('.jpg', '.png'))]

        for img_name in imgs:
            img_path = os.path.join(source_dir, img_name)
            results = self.model.predict(img_path,
                                         conf=self.config['inference']['conf_threshold'],
                                         iou=self.config['inference']['iou_threshold'])
            res = results[0]
            img = self._visualize(img_path, res)
            cv2.imwrite(os.path.join(output_dir, f"pred_{img_name}"), img)

    def _visualize(self, img_path, result):
        img = cv2.imread(img_path)
        img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        img_pil = Image.fromarray(img_rgb)
        draw = ImageDraw.Draw(img_pil)

        for box in result.boxes:
            cls_id = int(box.cls[0])
            conf = float(box.conf[0])
            x1, y1, x2, y2 = map(int, box.xyxy[0])
            color = (255, 0, 0)
            draw.rectangle([x1, y1, x2, y2], outline=color, width=3)
            draw.text((x1, y1 - 25), f"{self.class_names[cls_id]} {conf:.2f}", fill=(255, 255, 255), font=self.font)

        return cv2.cvtColor(np.array(img_pil), cv2.COLOR_RGB2BGR)
```

------

##  五、`src/eval.py`（验证逻辑）

```python
from ultralytics import YOLO
import logging

class Evaluator:
    def __init__(self, model_path, data_yaml):
        self.logger = logging.getLogger("Evaluator")
        self.model = YOLO(model_path)
        self.data_yaml = data_yaml

    def evaluate(self):
        self.logger.info("开始验证...")
        results = self.model.val(data=self.data_yaml)
        self.logger.info(f"验证完成 - mAP50: {results.box.map50:.4f}, mAP50-95: {results.box.map:.4f}")
        return results
```

------

## 六、`src/utils.py`（日志与工具函数）

```python
import logging
import os

def setup_logger(name="HT22_Detector"):
    logger = logging.getLogger(name)
    logger.setLevel(logging.INFO)
    ch = logging.StreamHandler()
    ch.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(levelname)s - %(message)s"))
    logger.addHandler(ch)
    return logger

def ensure_dir(path):
    os.makedirs(path, exist_ok=True)
```

------

## 七、`main.py`（统一入口）

```python
import argparse
import logging
from src.train import Trainer
from src.infer import Inference
from src.eval import Evaluator
from src.utils import setup_logger
import yaml

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--mode", type=str, required=True, choices=["train", "eval", "infer"])
    parser.add_argument("--config", type=str, default="configs/config.yaml")
    parser.add_argument("--weights", type=str, default=None)
    parser.add_argument("--source", type=str, default=None)
    parser.add_argument("--output", type=str, default="experiments/results")
    args = parser.parse_args()

    logger = setup_logger()

    with open(args.config, 'r', encoding='utf-8') as f:
        config = yaml.safe_load(f)

    if args.mode == "train":
        trainer = Trainer(args.config)
        trainer.train()

    elif args.mode == "eval":
        evaluator = Evaluator(args.weights, config['train']['data'])
        evaluator.evaluate()

    elif args.mode == "infer":
        infer = Inference(args.weights, config)
        infer.predict(args.source, args.output)

if __name__ == "__main__":
    main()
```

------

## 八、运行命令示例

```bash
# 训练
python main.py --mode train --config configs/config.yaml

# 验证
python main.py --mode eval --config configs/config.yaml --weights experiments/runs/yolov8m_exp/weights/best.pt

# 推理
python main.py --mode infer --config configs/config.yaml \
  --weights experiments/runs/yolov8m_exp/weights/best.pt \
  --source data/data_yolo/val/images \
  --output experiments/results
```

------

# 核心类：`YOLO`

这是最常用的 API 类。

```
from ultralytics import YOLO

# 1️⃣ 加载预训练模型
model = YOLO('yolov8m.pt')

# 2️⃣ 训练
model.train(data='data.yaml', epochs=100, imgsz=640)

# 3️⃣ 验证
model.val()

# 4️⃣ 推理
results = model.predict(source='data/images')

# 5️⃣ 导出（例如 ONNX）
model.export(format='onnx')
```

💡 可以看到，它的使用逻辑和 PyTorch Lightning 类似：
 **模型是对象，任务是方法**。