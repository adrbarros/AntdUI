[首页](Home.md)・[更新日志](UpdateLog.md)・[配置](Config.md)・[主题](Theme.md)

## 在 [Gitee](https://gitee.com) 向 [AntdUI](https://gitee.com/AntdUI/AntdUI) 贡献

为演示如何使用 [**Pull Request**](https://gitee.com/AntdUI/AntdUI/compare/main...main)（以下统称“PR”）贡献代码，将使用 **MOSS** 空间地址（每个人的空间地址不同）来演示

***

### 1. 首先在 [AntdUI](https://gitee.com/AntdUI/AntdUI) 官方仓库，Fork 到自己账号

> 点击右上角 **Fork** 按钮

![1](Img/PR_1.png)

### 2. 选择 Fork 目标空间

> 默认个人，也就是自己，点击 **确认** 继续

![2](Img/PR_2.png)

> 等待 Fork 完成，完成后会自动跳转
> ![3](Img/PR_3.png)

### 3. 在自己空间地址克隆项目

> 点击 **克隆/下载**，⚠ 注意是自己的**空间地址**

![4](Img/PR_4.png)

点击第一个 `.git` 地址复制

![5](Img/PR_5.png)

### 4. 提交贡献代码到自己仓库

> 打开 [Visual Studio](https://visualstudio.microsoft.com) 克隆刚刚复制的地址，选择目录，点击 **克隆** 继续

![6](Img/PR_6.png)

克隆成功后，提交需要贡献的代码

### 5. 发起 [PR](https://gitee.com/AntdUI/AntdUI/compare/main...main)

> 点击 **Pull requests** 选项卡，或者[点击**Pull Request**跳转提交PR页面](https://gitee.com/AntdUI/AntdUI/compare/main...main)

![7](Img/PR_7.png)

点击 **新建 Pull Request**

![8](Img/PR_8.png)

### 大功告成

提交内容调整核实后，点击 **创建 Pull Request**

![9](Img/PR_9.png)

> 收到后，我会尽快审核，🧙 生活愉快

---

### 6. PR 合并后同步自己的 Fork 仓库

> 维护者合并你的 PR 后，官方仓库已向前推进，但**你的 Fork 不会自动更新**。下次贡献前，请务必将自己的 Fork 与上游 `main` 分支保持一致。

打开你 Fork 后的 Gitee 仓库，在仓库主页点击 **同步**，将上游最新提交合并到自己的 Fork；若提示无更新则说明已同步。

随后在本地拉取最新代码，使工作区与上游一致：

```bash
# 确保处于 main 分支
git checkout main

# 从你的 Fork（origin）拉取最新代码
git pull origin main
```

> 小技巧：如果配置了 `upstream` 远程指向官方仓库，可直接从上游同步：
> ```bash
> git remote add upstream https://gitee.com/AntdUI/AntdUI.git
> git fetch upstream
> git merge upstream/main
> git push origin main
> ```

### 7. 异常情况处理

本节介绍贡献流程中常见的异常场景与处理办法。

#### 7.1 Fork 与官方仓库不一致

**症状**：提交新 PR 时 Gitee 提示“此分支已过时”/“存在冲突，必须解决”。

**处理**：先按 [第 6 步：同步 Fork 仓库](#6-pr-合并后同步自己的-fork-仓库) 同步。如果分支上已经存在冲突：

```bash
git fetch upstream
git merge upstream/main
# 在编辑器中解决冲突后：
git add .
git commit
git push origin your-branch
```

#### 7.2 提交错乱——删除 Fork 重新克隆

如果本地提交历史严重错乱、rebase 失败或冲突无法解决，最干净的做法是**删除 Fork 重新开始**：

1. **手动备份本地修改**。将你仍需保留的修改文件复制到仓库外的目录（例如 `D:\AntdUI_backup\`）。重新 Fork 会丢失所有提交和分支。
2. **删除 Fork**：Gitee 进入你的 Fork 仓库 → **管理** → 滚动到底部 → **删除仓库**，按提示输入完整仓库名确认。
3. **重新 Fork 官方仓库**：按 [第 1 步](#1-首先在-antdui官方仓库fork-到自己账号) 重新操作。
4. **重新克隆**：按 [第 3 步](#3-在自己空间地址克隆项目) 重新克隆新 Fork 到本地。
5. **手动恢复你保留的改动**：从备份目录将代码覆盖回新仓库，提交并发起新的 PR。

> ⚠ 警告：此操作具有破坏性。**删除 Fork 前**务必确认备份完整。任何未提交到远程且未备份的本地改动将永久丢失。

#### 7.3 提交错分支了

如果不小心直接提交到 `main` 而非功能分支，可将提交迁移到新分支：

```bash
# 创建并切换到新分支，保留刚才的提交
git checkout -b my-feature-branch
# 将 main 重置回上游
git checkout main
git reset --hard upstream/main
git checkout my-feature-branch
```

#### 7.4 PR 被要求修改

维护者要求修改后：

```bash
# 切换到 PR 所用分支
git checkout your-pr-branch

# 在编辑器中按要求修改，然后：
git add .
git commit -m "根据评审意见修改"
git push origin your-pr-branch
```

PR 会自动更新——无需另开新 PR。