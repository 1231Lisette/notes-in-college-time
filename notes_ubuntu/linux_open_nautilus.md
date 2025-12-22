# Linux 终端一键打开图形化文件管理器（Nautilus）

## 🎯 目的

在终端中 `cd` 到某个目录后，希望：

- 快速查看文件结构
- 拖拽 / 双击打开文件
- 不想手动点文件管理器

实现：**终端 → 图形界面 一键切换**

---

## 🖥️ 系统环境

- OS：Ubuntu 22.04
- 桌面环境：GNOME
- 文件管理器：Nautilus
- Shell：bash

---

## ✅ 基础命令

### 打开当前目录
```bash
nautilus .
打开指定目录
bash
复制代码
nautilus ~/下载
🚀 懒人方案（最终采用）
使用 open 作为快捷命令。

🔧 配置方法
1️⃣ 写入 alias
bash
复制代码
echo "alias open='nautilus . >/dev/null 2>&1 &'" >> ~/.bashrc
2️⃣ 立刻生效
bash
复制代码
source ~/.bashrc
🧪 使用方式
bash
复制代码
cd 任意目录
open
效果：

📂 打开当前目录的 Nautilus

🔇 无终端输出

🧘 终端不被占用

🧠 命令说明
bash
复制代码
nautilus .          # 打开当前目录
>/dev/null          # 丢弃标准输出
2>&1                # 丢弃错误输出
&                   # 后台运行
这是 Linux 中启动 GUI 程序的标准写法。
```

