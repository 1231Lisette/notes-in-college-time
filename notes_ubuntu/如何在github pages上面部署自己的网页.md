# 如何在github pages上面部署自己的网页

### 1. 准备工作（本地文件整理）

确保你的文件夹结构如下：

Bash

```
resume-site/
├── index.html       # 必须命名为 index.html
├── Resume.pdf       # 你的 PDF 版简历
└── assets/          # 存放图标、照片的资源文件夹
```

> **提示**：如果你的 HTML 还没加上下载 PDF 的链接，请在 `<body>` 标签内合适位置加入：
>
> ```html
> <a href="Resume.pdf" target="_blank">Download PDF</a>
> ```

------

### 2. 命令行部署步骤

请打开终端，进入你的简历文件夹，依次执行以下命令：

#### **Step 1: 初始化本地仓库**

Bash

```
git init
```

#### **Step 2: 将所有文件添加到暂存区**

Bash

```
git add .
```

#### **Step 3: 提交更改**

Bash

```
git commit -m "Deploy my professional resume"
```

#### **Step 4: 关联远程 GitHub 仓库**

在 GitHub 上创建一个名为 `resume` (或任何你喜欢的名字) 的 **Public** 仓库，然后复制仓库地址：

Bash

```
# 请将 <YOUR_GITHUB_URL> 替换为你刚复制的地址
git remote add origin <YOUR_GITHUB_URL>
```

#### **Step 5: 推送到主分支**

Bash

```
git branch -M main
git push -u origin main
```

------

### 3. 在 GitHub 开启访问权限

推送完成后，你需要最后一步操作让网页上线：

1. 打开你的 GitHub 仓库页面，点击顶部的 **Settings**。
2. 在左侧菜单找到 **Pages** 选项。
3. 在 **Build and deployment** > **Branch** 下：
   - 选择 `main`。
   - 文件夹选择 `/ (root)`。
4. 点击 **Save**。

------

### 4. 结果与维护

- **访问链接**：大约 1 分钟后，你会看到类似 `https://<你的用户名>.github.io/resume/` 的链接。
- **更新简历**：以后如果你修改了简历内容，只需执行以下“三连”命令即可自动同步更新：

Bash

```
git add .
git commit -m "Update resume content"
git push
```

