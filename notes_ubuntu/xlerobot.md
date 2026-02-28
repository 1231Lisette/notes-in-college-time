#  xlerobot_import_issue

------

~~~md
# LeRobot 中 xlerobot 无法导入的问题分析与解决

## 问题现象

在运行示例时出现如下错误：

```text
ModuleNotFoundError: No module named 'lerobot.robots.xlerobot'
~~~

但在本地源码目录中，确认存在：

```text
lerobot/src/lerobot/robots/xlerobot/
```

说明该模块在源码中真实存在。

------

## 问题根本原因

**Python 实际加载的 lerobot 包，并不是本地源码，而是 conda / pip 环境中的旧版本。**

具体表现为：

```bash
python - << 'EOF'
import lerobot
print(lerobot.__file__)
EOF
```

输出类似：

```text
/home/pyy/miniconda3/envs/lerobot/lib/python3.10/site-packages/lerobot/__init__.py
```

这说明：

- Python 使用的是 `site-packages` 中的 lerobot
- 而不是 `~/lerobot/src/lerobot`
- 该旧版本中不包含 `xlerobot` 模块

------

## 为什么会发生这个问题？

LeRobot 的示例代码（`examples/`）默认假设：

- 用户使用的是 **源码开发模式（editable install）**
- 即 Python 直接从 `src/lerobot` 加载包

如果只是 `pip install lerobot`，就会出现：

- 示例代码版本 > 本地安装版本
- API / 模块不一致
- 导致 import 失败

------

## 正确的解决方法（标准做法）

### 1. 进入 lerobot 项目根目录

```bash
cd ~/lerobot
```

### 2. 卸载当前环境中的旧 lerobot

```bash
pip uninstall lerobot -y
```

### 3. 以开发模式安装本地源码

```bash
pip install -e .
```

其中 `-e` 表示 **editable install**，含义是：

- Python 直接使用 `src/lerobot`
- 源码修改立即生效
- 与 examples 目录保持一致

------

## 安装后的验证方式（必须）

```bash
python - << 'EOF'
import lerobot
print("lerobot loaded from:", lerobot.__file__)
from lerobot.robots.xlerobot import XLerobot
print("XLerobot import OK")
EOF
```

### 正确输出应类似：

```text
lerobot loaded from: /home/pyy/lerobot/src/lerobot/__init__.py
XLerobot import OK
```

至此，`xlerobot` 无法导入的问题彻底解决。

---

用了robocrew之后，设置udev原则
/dev/arm_left
/dev/arm_right
/dev/camera_left
/dev/camera_center
/dev/camera_right