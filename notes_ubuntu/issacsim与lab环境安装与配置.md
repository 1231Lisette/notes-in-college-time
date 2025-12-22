# issacsim与lab环境安装与配置

https://blog.csdn.net/m0_65805744/article/details/150344985

### 环境准备

1. ubuntu22.04
2. 内存16GB，显存8GB（吃力）
3. ros（humble）
4. miniconda3

---

### isaacsim

在https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/download.html

下载3个资产和环境

---

在个人目录下创建isaacsim文件夹和isaacsim_assets文件夹，然后把linux那个zip放到isaacsim目录下面，3个assets的zip放在isaacsim_assets目录下面

1. isaacsim下面

```bash
unzip isaac-sim-standalone-5.0.0-linux-x86_64.zip #解压文件
./post_install.sh #整理文件正确的位置，完成软链接、权限和环境配置。
./isaac-sim.selector.sh #进行验证 
```

2. isaacsim_assets下面

```bash
# 我这里无法用unzip，所以下载了一个7z
# 安装 7z（如果还没有）
sudo apt install p7zip-full

# 解压多卷文件
7z x isaac-sim-assets-complete-5.0.0.zip.001 # 然后它就会自动解压3个assets
```

#### 导入资产

https://docs.isaacsim.omniverse.nvidia.com/5.0.0/installation/install_faq.html

在这个页面的Assets折叠页展开下（真想吐槽一下这个内容真的很难找。。。）

需要编辑你的` /home/<username>/isaacsim/apps/isaacsim.exp.base.kit`文件

（马的网页上的复制键也是一托是）

注意自己改自己的用户名还有对应创建的文件夹是不是一样的，vsc打开比较好，因为可以全部替换

```bash
[settings]
persistent.isaac.asset_root.default = "/home/<username>/isaacsim_assets/Assets/Isaac/5.0"

exts."isaacsim.asset.browser".folders = [
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Robots",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/People",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/IsaacLab",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Props",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Environments",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Materials",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Samples",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Sensors",
]

exts."isaacsim.gui.content_browser".folders = [
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Robots",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/People",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/IsaacLab",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Props",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Environments",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Materials",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Samples",
    "/home/<username>/isaacsim_assets/Assets/Isaac/5.0/Isaac/Sensors",
]
```

然后就可以run了

```bash
./isaac-sim.sh --/persistent/isaac/asset_root/default="/home/<username>/isaacsim_assets/Assets/Isaac/5.0"
```

可以在isaacsim页面的左下角点击一个🔓的文件夹，看看它的路径是不是本地

---

### isaaclab

先在bashrc加上

记得source一下

```bash
# Isaac Sim root directory
export ISAACSIM_PATH="${HOME}/isaacsim"
# Isaac Sim python executable
export ISAACSIM_PYTHON_EXE="${ISAACSIM_PATH}/python.sh"
```

然后测试一下能否打开页面

```bash
${ISAACSIM_PATH}/isaac-sim.sh
```

可以打开后，可以cd到你的issaclab文件夹下或者直接home目录

```bash
git clone https://github.com/isaac-sim/IsaacLab.git
```

然后链接isaacsim和isaaclab，是在isaaclab的文件夹里创建一个 直接链接到isaacsim文件夹的链接，桥梁。

```bash
ln -s ${HOME}/isaacsim _isaac_sim
```

这时候iu会多一个文件是_isaac_sim

然后可以用conda创建一个虚拟环境 ，激活，

```bash
./isaaclab.sh --conda 
```

```bash
conda activate env_isaaclab 
```

```bash
sudo apt install cmake build-essential
```

```bash
./isaaclab.sh --install
```

逐次输入以下代码进行验证：

1. 直接测试是否能创建新的环境

```bash
./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py
```

2. 测试虚拟环境内的python能否创建新环境

```bash
python scripts/tutorials/00_sim/create_empty.py
```

3. 测试训练一个机器人

```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py --task=Isaac-Ant-v0 --headless
```

若均运行无误，则完成Isaaclab的安装

---

### isaaclab的vsc配置

（其实这个我还不知道要怎么用呢哈哈哈）

首先用vsc打开isaacsim文件

然后ctrl+shift+p打开命令行

选择run task

选择setup_python_env

回车，选择默认的continue without scanning the task out

--

同样用vsc打开isaaclab

然后ctrl+shift+p打开命令行

选择编译器，即让isaaclab在我们之前创建的环境下运行

选择刚才创建的虚拟环境，env_isaacab

点击左边栏的run 和debug

随后配置launch.json文件

---

1. 利用leisaac的文档完成仿真与代码的入门
2. 用xlerobot完成仿真学习
3. 任务制定与训练
4. 调试真机
5. sim2real任务