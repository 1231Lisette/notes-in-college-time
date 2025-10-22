# hugging face token and 使用手册

## 我的token

- name

Lisette

- value

---

用途：上传和下载模型/数据集

## python

1. 上传模型或数据集 (`push_to_hub`)

```python
from transformers import AutoModelForSequenceClassification, AutoTokenizer
from huggingface_hub import HfApi

# 1. 实例化你的模型和分词器
model_name = "my-awesome-robot-model"
# ... 假设你已经加载或训练好了你的模型和分词器对象

# 2. 将它们保存到本地目录
model.save_pretrained("./local_model_dir")
tokenizer.save_pretrained("./local_model_dir")

# 3. 推送到 Hugging Face Hub
# 这里的 "pyy" 是你的用户名
repo_id = f"pyy/{model_name}" 

# 假设你已经用 xxS 登录
# 如果你在命令行登录成功，Python 代码会自动找到令牌
model.push_to_hub(repo_id)
tokenizer.push_to_hub(repo_id)

print(f"Model pushed successfully to https://huggingface.co/{repo_id}")
```



2. 下载模型或数据集 (`from_pretrained`)

```python
from transformers import AutoModel, AutoTokenizer
from datasets import load_dataset

# 下载一个模型（例如 BERT）
model = AutoModel.from_pretrained("bert-base-uncased")
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")

# 下载一个数据集（例如 SQuAD）
dataset = load_dataset("squad")
```

---

## 用huggingfae-cli

#### ① 查看当前登录状态和信息

```bash
# 查看当前登录的用户信息
huggingface-cli whoami
```



#### ② 上传本地文件夹中的文件 (用于 Git LFS)

你可以用它来上传你**没有**通过 Python 库处理的任意文件或大型二进制文件。

1. **创建 Hugging Face 仓库：**

   ```bash
   huggingface-cli repo create my-data-project --type dataset
   ```

2. **克隆到本地：**

   ```bash
   git clone https://huggingface.co/datasets/pyy/my-data-project
   cd my-data-project
   ```

3. **添加文件、提交并推送：** (这里会用到你之前配置的 Git 凭证)

   ```bash
   cp /path/to/my/local/file.bin .
   git add .
   git commit -m "Add my first data file"
   git push
   ```



#### ③ 清理缓存

如果你下载了太多模型和数据集，占用了过多磁盘空间，可以清理缓存。

```bash
# 查看缓存目录信息
huggingface-cli cache info

# 清理所有未被引用的（孤立的）缓存文件
huggingface-cli cache delete
```

---

HF_USER=$(hf auth whoami | head -n 1)

echo $HF_USER



