在 ROS 2 中，**workspace（ws）就是你的开发工作区**，所有功能包（package）都会放在里面。理解它的结构后，你就知道怎么往里面添加项目包了。

------

# 一、ROS2 Workspace 的标准结构

一个典型的 ROS2 工作区通常是这样的：

```
ros2_ws/
├── src/
│   ├── pkg1/
│   ├── pkg2/
│   └── pkg3/
├── build/
├── install/
└── log/
```

各目录作用：

| 目录        | 作用                                     |
| ----------- | ---------------------------------------- |
| **src**     | 存放所有ROS2功能包（你写的代码就在这里） |
| **build**   | 编译时产生的中间文件                     |
| **install** | 编译完成后安装的可执行文件和库           |
| **log**     | 编译日志                                 |

最核心的其实只有一个：

**src/**

因为：

> ROS2只会扫描 `src` 里面的 package。

------

# 二、一个 package 的结构

例如：

```
ros2_ws/
└── src/
    └── my_robot_pkg/
        ├── package.xml
        ├── setup.py / CMakeLists.txt
        ├── resource/
        ├── my_robot_pkg/
        │   └── node.py
        └── launch/
```

关键文件：

| 文件           | 作用       |
| -------------- | ---------- |
| package.xml    | 包的描述   |
| CMakeLists.txt | C++构建    |
| setup.py       | Python构建 |
| launch         | 启动文件   |
| src/代码       | node程序   |

------

# 三、如何在 workspace 里增加一个包

假设你的工作区是

```
~/ros2_ws
```

进入 src：

```
cd ~/ros2_ws/src
```

然后创建 package：

### Python包

```
ros2 pkg create my_pkg --build-type ament_python
```

生成：

```
src/
└── my_pkg/
    ├── package.xml
    ├── setup.py
    └── my_pkg/
```

------

### C++包

```
ros2 pkg create my_cpp_pkg --build-type ament_cmake
```

生成：

```
src/
└── my_cpp_pkg/
    ├── package.xml
    ├── CMakeLists.txt
    └── src/
```

------

# 四、添加完包之后要做什么

回到 workspace 根目录：

```
cd ~/ros2_ws
```

编译：

```
colcon build
```

然后加载环境：

```
source install/setup.bash
```

这样 ROS2 才能找到新包。

------

# 五、工作区结构示例（真实项目）

例如一个机器人项目：

```
ros2_ws/
├── src/
│   ├── robot_description/
│   ├── robot_control/
│   ├── robot_navigation/
│   └── robot_bringup/
├── build/
├── install/
└── log/
```

不同包负责不同功能：

| 包                | 作用        |
| ----------------- | ----------- |
| robot_description | URDF模型    |
| robot_control     | 控制算法    |
| robot_navigation  | SLAM / 导航 |
| robot_bringup     | 启动系统    |

------

# 六、很多新手容易犯的错误

### ❌ 把包放错地方

错误：

```
ros2_ws/
    my_pkg/
```

正确：

```
ros2_ws/
└── src/
    └── my_pkg/
```

------

### ❌ 新包没编译

创建后必须：

```
colcon build
```

------

# 七、如果你做机器人项目（非常推荐这种结构）

比如你现在的小车 + STM32 + ROS：

```
ros2_ws/
└── src/
    ├── robot_interfaces
    ├── robot_driver
    ├── robot_control
    ├── robot_navigation
    └── robot_bringup
```

这是**工业级结构**。

------

✅ **一句话总结**

ROS2 工作区核心逻辑：

```
workspace
   ↓
 src
   ↓
 packages
```

只要：

```
新包放到 src
→ colcon build
→ source install/setup.bash
```

ROS2 就能识别。

