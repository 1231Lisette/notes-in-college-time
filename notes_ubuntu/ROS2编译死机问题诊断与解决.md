# 📝 ROS 2 编译死机问题诊断与解决

## 1. 问题现象

执行 `colcon build` 时，系统 CPU 和内存瞬间满载，导致鼠标键盘无法响应（死机），只能强制重启。

## 2. 核心原因

- **内存耗尽 (OOM)**：ROS 2 编译（尤其是 C++ 源码）非常吃内存。
- **缺乏缓冲**：系统未配置 **Swap（交换空间）**，内存满载后没有任何退路，直接导致内核保护性锁死。
- **并行过激**：`colcon` 默认会开启与 CPU 核心数等同的线程数，导致内存占用呈指数级上升。

------

## 3. 诊断命令

在操作前，先确认系统现状：

```
# 查看内存与交换空间情况
free -h

# 查看硬盘剩余空间（确保根目录 / 剩余 > 10GB）
df -h

# 查看逻辑核心数
nproc
```

------

## 4. 解决方案：建立“救命稻草” (Swap)

执行以下命令，手动创建一个 8GB 的交换文件：

```
# 1. 创建 8G 的空文件 (在根目录下)
sudo fallocate -l 8G /swapfile

# 2. 修改权限，仅允许 root 读写
sudo chmod 600 /swapfile

# 3. 设置为交换格式
sudo mkswap /swapfile

# 4. 立即启用
sudo swapon /swapfile

# 5. 永久生效：将其写入系统挂载表
# 执行下面这一行命令即可
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab

# 6. (可选) 优化积极性：将 swappiness 设为 10
echo 'vm.swappiness=10' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

------

## 5. 安全编译指令需要我帮你把这段内容保存成一个具体的 .md 文件，还是现在就带你把那个 Swap 空间跑起来？ Would you like me to guide you through the Swap creation step-by-step?

有了 Swap 后，虽然不会死机，但为了保护硬盘寿命和系统流畅度，建议使用 **“限流”** 编译：

### 推荐方案（单包、双核）：

Bash

```
colcon build --packages-select <你的包名> --parallel-workers 1 --make-args -j2
```

- `--parallel-workers 1`：一次只编一个包（防止内存并发爆炸）。
- `--make-args -j2`：每个包内部最多用 2 个核（防止 CPU 占满）。

------

## 6. 监控工具

在编译时，建议开一个侧边窗口盯着资源：

- **实时刷新模式**：`watch -n 1 free -h`
- **交互图形模式**：`htop`（如果没有，用 `sudo apt install htop` 安装）

------

> **总结：**
>
> 16GB 内存 + 8GB Swap 是 ROS 2 开发的“标准配置”。以后遇到编译大型 C++ 库（如 OpenCV、Nav2）时，记得带上 `--make-args -j` 参数！

