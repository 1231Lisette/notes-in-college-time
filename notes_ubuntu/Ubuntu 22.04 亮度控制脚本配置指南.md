# Ubuntu 22.04 亮度控制脚本配置指南

## 问题背景

在 Ubuntu 22.04 中，开机后系统亮度滑块有时会失效，无法正常调节屏幕亮度。这通常是由于：

- 显卡驱动加载延迟
- 系统亮度控制服务启动较慢
- 权限问题
- 背光设备识别问题

## 原始报错分析

### 错误信息

```
IndexError: list index out of range
```

### 错误原因

- 脚本期望接收一个命令行参数 `argv[1]`（亮度文件路径）
- 但运行时未提供参数，导致访问 `sys.argv[1]` 时索引越界

### 修复方案

1. 添加参数检查逻辑
2. 实现自动检测亮度控制文件功能
3. 完善错误处理机制

## 解决方案

### 1. Python脚本修复

**主要改进：**

- 自动检测亮度控制文件路径
- 添加完善的错误处理
- 支持最大亮度值的动态获取
- 防止黑屏（最低亮度限制）
- 增强的xrandr命令错误处理

**关键函数：**

```python
def find_brightness_file():
    """自动查找系统亮度控制文件"""
    brightness_paths = [
        '/sys/class/backlight/*/brightness',
        '/sys/class/backlight/*/actual_brightness'
    ]
    # 返回第一个找到的有效路径
```

### 2. 权限配置

**添加用户到video组：**

```bash
sudo usermod -a -G video $USER
```

**创建udev规则：**

```bash
sudo tee /etc/udev/rules.d/90-backlight.rules > /dev/null <<EOF
SUBSYSTEM=="backlight", ACTION=="add", RUN+="/bin/chgrp video /sys/class/backlight/%k/brightness"
SUBSYSTEM=="backlight", ACTION=="add", RUN+="/bin/chmod g+w /sys/class/backlight/%k/brightness"
EOF

sudo udevadm control --reload-rules && sudo udevadm trigger
```

我的是/sys/class/backlight/nvidia_wmi_ec_backlight/brightness

### 3. 开机自启动配置

**方法一：启动应用程序（推荐）**

1. **创建启动脚本** `/home/pyy/start-brightness-control.sh`：

```bash
#!/bin/bash
LOG_FILE="/tmp/brightness-control.log"
PYTHON_SCRIPT="/home/pyy/brightness-control.py"

# 等待桌面环境加载
sleep 10

# 检查是否已有实例运行
if pgrep -f "brightness-control.py" > /dev/null; then
    echo "$(date): 脚本已在运行，退出" >> "$LOG_FILE"
    exit 0
fi

# 循环运行，自动重启
while true; do
    echo "$(date): 启动亮度控制进程" >> "$LOG_FILE"
    /usr/bin/python3 "$PYTHON_SCRIPT" >> "$LOG_FILE" 2>&1
    echo "$(date): 脚本退出，5秒后重启" >> "$LOG_FILE"
    sleep 5
done
```

1. **设置执行权限：**

```bash
chmod +x /home/pyy/start-brightness-control.sh
```

1. **添加到启动应用程序：**
   - 打开"启动应用程序"：`gnome-session-properties`
   - 点击"添加"
   - 名称：`亮度控制`
   - 命令：`/home/pyy/start-brightness-control.sh`
   - 注释：`系统亮度自动调节脚本`

## 故障排除

### 常见问题及解决方案

1. **脚本运行一会儿就停止**
   - **原因：** Python脚本遇到异常退出
   - **解决：** 使用带自动重启功能的启动脚本
2. **权限拒绝错误**
   - **检查用户组：** `groups` （应包含video组）
   - **检查文件权限：** `ls -la /sys/class/backlight/*/brightness`
   - **重新应用udev规则**
3. **找不到显示设备**
   - **检查连接的显示器：** `xrandr --query | grep " connected"`
   - **确认显卡驱动正常加载**

### 调试命令

**查看运行日志：**

```bash
# 实时查看
tail -f /tmp/brightness-control.log

# 查看完整日志
cat /tmp/brightness-control.log
```

**检查进程状态：**

```bash
ps aux | grep brightness-control
```

**手动测试亮度调节：**

```bash
# 找到显示设备
DISPLAY_NAME=$(xrandr --query | grep " connected" | head -1 | cut -d' ' -f1)

# 测试调节
xrandr --output $DISPLAY_NAME --brightness 0.8
```

**诊断脚本：**

```bash
#!/bin/bash
echo "=== 亮度控制诊断 ==="
echo "1. 背光设备："
ls -la /sys/class/backlight/*/
echo "2. 显示设备："
xrandr --query | grep " connected"
echo "3. 用户组："
groups
echo "4. 相关进程："
ps aux | grep -i bright | grep -v grep
```

## 使用方法

### 手动运行

```bash
# 直接运行（自动检测亮度文件）
python3 /home/pyy/brightness-control.py

# 指定亮度文件路径
python3 /home/pyy/brightness-control.py /sys/class/backlight/intel_backlight/brightness
```

### 开机自启动

1. 按上述步骤配置权限和启动脚本
2. 重启系统测试
3. 通过日志文件监控运行状态

### 停止运行

```bash
# 停止所有相关进程
pkill -f brightness-control.py
pkill -f start-brightness-control.sh
```

## 文件结构

```
/home/pyy/
├── brightness-control.py          # 主要的Python脚本
├── start-brightness-control.sh    # 启动脚本
└── ~/.config/autostart/
    └── brightness-control.desktop # 自启动配置（可选）

/tmp/
└── brightness-control.log         # 运行日志

/etc/udev/rules.d/
└── 90-backlight.rules            # 权限配置
```

## 总结

这个解决方案通过以下几个关键改进解决了Ubuntu 22.04的亮度控制问题：

1. **自动检测** - 无需手动指定亮度文件路径
2. **错误处理** - 完善的异常捕获和恢复机制
3. **权限管理** - 通过udev规则确保文件访问权限
4. **稳定运行** - 带自动重启功能的启动脚本
5. **易于调试** - 详细的日志记录和诊断工具

配置完成后，系统开机会自动启动亮度控制服务，确保亮度滑块正常工作。