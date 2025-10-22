#  Git 上传更新流程

------

## 🚀 一次性初始化上传（第一次上传项目）

假设你的项目路径是：

```
~/HT22_ControlBox_Detection
```

执行以下命令：

```bash
# 1️⃣ 进入项目目录
cd ~/HT22_ControlBox_Detection

# 2️⃣ 初始化本地仓库（仅第一次）
git init

# 3️⃣ 关联远程仓库（你已经创建好的 GitHub 仓库）
git remote add origin git@github.com:1231Lisette/htt2_classificvation.git

# 4️⃣ 添加所有文件到暂存区
git add .

# 5️⃣ 提交到本地仓库
git commit -m "Initial commit"

# 6️⃣ 推送到远程仓库（main 分支）
git branch -M main
git push -u origin main --force
```

> ✅ 执行完这些，你的本地项目就会被完整上传到 GitHub 仓库。

------

## 🔁 后续更新上传（以后每次修改后）

以后你只要更新了代码、配置文件或文档，只需要执行 3 步：

```bash
# 1️⃣ 添加修改
git add .

# 2️⃣ 提交修改
git commit -m "更新了YOLOv8配置文件"   # 写上你的修改说明

# 3️⃣ 推送到远程仓库
git push
```

> 💡 这三条命令就是你要的“pipeline”，相当于：
>  “保存修改 → 记录修改 → 上传修改”

------

## 🧭 如果你在另一台电脑要更新项目

当你在另一台电脑上继续开发时：

```bash
# 1️⃣ 克隆远程仓库到本地
git clone git@github.com:1231Lisette/htt2_classificvation.git

# 2️⃣ 进入项目目录
cd htt2_classificvation

# 3️⃣ 编辑代码后，重复这三步：
git add .
git commit -m "更新说明"
git push
```

------

## ⚡ 如果你远程更新过，另一台本地要同步

当你在 GitHub 上直接改了内容，或别人推送了修改，
 在你本地更新代码前先执行：

```bash
git pull
```

即可把远程修改拉到本地。

------

## 🧩 一键命令版（适合常用）

如果你想更方便，可以在项目目录下创建一个脚本，例如：

```bash
nano git_pipeline.sh
```

内容写：

```bash
#!/bin/bash
git add .
git commit -m "${1:-update}"
git push
```

保存后给执行权限：

```bash
chmod +x git_pipeline.sh
```

以后你只需要一条命令就能上传：

```bash
./git_pipeline.sh "更新说明"
```

------

