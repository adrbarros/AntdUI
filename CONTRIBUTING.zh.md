# 🦄 参与 AntdUI 贡献

本文档将介绍参与 AntdUI 贡献的标准和工作流程，帮助你顺利协作、保持代码一致性，并高效熟悉项目。

## 目录

1. [目录结构](#1-目录结构)
2. [代码规范](#2-代码规范)
3. [贡献流程](#3-贡献流程)
4. [PR 合并后同步自己的 Fork 仓库](#4-pr-合并后同步自己的-fork-仓库)
5. [异常情况处理](#5-异常情况处理)
   - 5.1 [Fork 与官方仓库不一致](#51-fork-与官方仓库不一致)
   - 5.2 [提交错乱——删除 Fork 重新克隆](#52-提交错乱删除-fork-重新克隆)
   - 5.3 [提交错分支了](#53-提交错分支了)
   - 5.4 [PR 被要求修改](#54-pr-被要求修改)

---

## 1. 目录结构
请先了解项目的目录布局，确保你的贡献内容放在正确位置：

```
AntdUI/
├─ src/                      # 所有核心库源代码
│  ├─ AntdUI/                # 主UI库代码
│  │  ├─ Controls/           # 自定义UI控件
│  │  │  ├─ Chat/            # 聊天相关控件（因逻辑独立或代码量较大而单独存放）
│  │  │  └─ Core/            # 核心绘制类和实现类
│  │  ├─ Design/             # 设计器支持（如Visual Studio设计器集成）
│  │  ├─ Enum/               # 库中通用的枚举（如主题类型、控件状态）
│  │  ├─ Events/             # 事件定义
│  │  ├─ Forms/              # 自定义窗口/表单
│  │  │  └─ LayeredWindow/   # 弹窗/分层窗口（如下拉框、日期选择器、模态框、抽屉等）
│  │  ├─ Lib/                # 工具文件（Win32 API封装、SVG处理、辅助类）
│  │  ├─ Localization/       # 多语言支持（不同地区的资源文件）
│  │  └─ Style/              # 主题定义（如明暗主题、调色板）
│  └─ AntdUI.EmojiFluentFlat/ # Emoji Fluent Flat 资源库
├─ example/                  # 用于展示控件用法的示例项目
│  └─ Demo/                  # 主示例项目（包含所有控件的测试窗体）
└─ doc/                      # 文档（贡献指南、API参考等）
```

- **新增控件**：请添加到 `src/AntdUI/Controls/`（独立或大型控件可像 `Chat/` 一样使用子文件夹）。
- **工具代码**：辅助函数、Win32封装或SVG工具请放在 `src/AntdUI/Lib/`。
- **示例代码**：请更新 `example/Demo/` 以包含新功能的测试用例（有助于验证功能并帮助其他贡献者）。


## 2. 代码规范
为保持一致性并避免常见问题（如内存泄漏、UI异常），请遵循以下核心规则：


### 2.1 绘制逻辑要求
AntdUI 依赖自定义绘制系统实现高质量UI渲染。所有可视化控件需遵循：

- **必须实现 `AntdUI.IControl`**：
  所有可绘制控件**必须继承 `AntdUI.IControl`**（自定义渲染的基础接口）。通过重写 `OnDraw` 方法实现控件特定绘制：
  ```csharp
  public class MyCustomControl : IControl
  {
	  // 重写OnDraw处理渲染逻辑
	  protected override void OnDraw(DrawEventArgs e)
	  {
		  base.OnDraw(e);
		  // 使用e.Canvas进行绘制（见下方说明）
		  e.Canvas.DrawText("Hello AntdUI", _textFont, _textColor, ClientRectangle);
	  }
  }
  ```

- **使用 `Canvas` 进行渲染**：
  不建议直接使用 `Graphics` 绘制，而是通过 `Canvas` 实现渲染，以获得更好的性能和一致性。
  - 通过 `OnDraw` + `OnDrawBg` 方法实现完整渲染
  - 若无法继承 `IControl`（如系统控件封装），可通过 `Graphics.High()` 获取 `Canvas` 实例
  - 若需要进行 `Bitmap` 位图绘制，应使用 `HighLay` 方法
  ```csharp
  // Winform OnPaint 中获取 Canvas
  var canvas = e.Graphics.High(); // 获取Canvas实例
  
  // Bitmap 位图绘制
  using (var bitmap = new Bitmap(width, height))
  using (var graphics = Graphics.FromImage(bitmap))
  using (var canvas = graphics.HighLay()) // 位图绘制使用 HighLay
  {
	  // 绘制逻辑
  }
  ```

- **文本渲染规范**：
  - 不要使用 `StringFormat` 作为布局排版，而是使用 `AntdUI.FormatFlags`
  - 如果包含 Emoji，使用 `MeasureText` + `DrawText`（内部已适配 Emoji），而不是 `MeasureString` + `String`
  ```csharp
  // 推荐：使用 FormatFlags 和 DrawText
  var flags = FormatFlags.Left | FormatFlags.VerticalCenter;
  var size = canvas.MeasureText("Hello 👍", Font, flags);
  canvas.DrawText("Hello 👍", Font, Style.Db.Text, ClientRectangle, flags);
  ```

- **资源释放**：
  GDI资源（如 `Bitmap`、`Brush`、`Pen`）使用后必须立即释放，避免内存泄漏。推荐使用 `using` 语句自动释放：
  ```csharp
  // 推荐：使用using自动释放Brush
  using (var fillBrush = new SolidBrush(_backgroundColor))
  {
	  e.Canvas.Fill(fillBrush, ClientRectangle);
  }

  // 避免：未释放非托管资源
  var badBrush = new SolidBrush(_backgroundColor); 
  e.Canvas.Fill(badBrush, ClientRectangle); // ❌ 导致内存泄漏
  ```


### 2.2 列表类控件规范
对于带滚动内容的控件（如列表框、数据网格），请遵循：

- **使用 `AntdUI.ScrollBar`**：
  禁止使用系统滚动条，必须集成库内置的 `AntdUI.ScrollBar` 组件，以保证样式和行为统一。

- **公共属性**：
  需将滚动条相关属性开放给外部代码（方便用户自定义）。 


### 2.3 `IControl.RenderRegion` 用法
`AntdUI.IControl` 中的 `GraphicsPath RenderRegion` 属性对**遮罩正确渲染**至关重要（如圆角控件上的Spin控件）。

- **圆角控件需设置 `RenderRegion`**：
  若控件为圆角（或非矩形），需定义 `RenderRegion` 以确保Spin控件（或其他覆盖元素）适配控件形状：
  ```csharp
  protected override GraphicsPath RenderRegion
  {
	  get
	  {
		  return ClientRectangle.RoundPath(8 * Config.Dpi);
	  }
  }
  ```  
  - 若不设置 `RenderRegion`，Spin等覆盖元素可能显示为矩形，破坏UI一致性。


## 3. 贡献流程

> 以下演示如何通过 [**合并请求（Pull Request，简称PR）**](https://gitee.com/AntdUI/AntdUI/compare/main...main) 贡献代码


### 3.1 首先在 [AntdUI](https://gitee.com/AntdUI/AntdUI) 官方仓库 Fork 到自己账号

> 点击右上角 **Fork** 按钮

![1](doc/wiki/zh/Img/PR_1.png)

### 3.2 选择 Fork 目标空间

> 默认个人，也就是自己，点击 **确认** 继续

![2](doc/wiki/zh/Img/PR_2.png)

> 等待 Fork 完成，完成后会自动跳转
> ![3](doc/wiki/zh/Img/PR_3.png)

### 3.3 在自己空间地址克隆项目

> 点击 **克隆/下载**，⚠ 注意是自己的**空间地址**

![4](doc/wiki/zh/Img/PR_4.png)

点击第一个 `.git` 地址复制

![5](doc/wiki/zh/Img/PR_5.png)

### 3.4 提交贡献代码到自己仓库

> 打开 [Visual Studio](https://visualstudio.microsoft.com) 克隆刚刚复制的地址，选择目录，点击 **克隆** 继续

![6](doc/wiki/zh/Img/PR_6.png)

克隆成功后，提交需要贡献的代码

### 3.5 发起 [PR](https://gitee.com/AntdUI/AntdUI/compare/main...main)

> 点击 **Pull requests** 选项卡，或者[点击**Pull Request**跳转提交PR页面](https://gitee.com/AntdUI/AntdUI/compare/main...main)

![7](doc/wiki/zh/Img/PR_7.png)

点击 **新建 Pull Request**

![8](doc/wiki/zh/Img/PR_8.png)

### 3.6 大功告成

提交内容调整核实后，点击 **创建 Pull Request**

![9](doc/wiki/zh/Img/PR_9.png)

> 收到后，我会尽快审核，🧙 生活愉快

---

## 4. PR 合并后同步自己的 Fork 仓库

> 维护者合并你的 PR 后，官方仓库已向前推进，但**你的 Fork 不会自动更新**。下次贡献前，请务必将自己的 Fork 与上游 `main` 分支保持一致。

打开你 Fork 后的仓库，点击 **同步** / **Sync fork** → **更新分支** / **Update branch**。若按钮显示 **已是最新** / **No changes**，说明 Fork 已同步。

![4](doc/wiki/zh/Img/PR_4.png)

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

---

## 5. 异常情况处理

本节介绍贡献流程中常见的异常场景与处理办法。

### 5.1 Fork 与官方仓库不一致

**症状**：提交新 PR 时 Gitee 提示“此分支已过时”/“存在冲突，必须解决”。

**处理**：先按 [第 4 节：同步 Fork 仓库](#4-pr-合并后同步自己的-fork-仓库) 同步。如果分支上已经存在冲突：

```bash
git fetch upstream
git merge upstream/main
# 在编辑器中解决冲突后：
git add .
git commit
git push origin your-branch
```

### 5.2 提交错乱——删除 Fork 重新克隆

如果本地提交历史严重错乱、rebase 失败或冲突无法解决，最干净的做法是**删除 Fork 重新开始**：

1. **手动备份本地修改**。将你仍需保留的修改文件复制到仓库外的目录（例如 `D:\AntdUI_backup\`）。重新 Fork 会丢失所有提交和分支。
2. **删除 Fork**：Gitee 进入你的 Fork 仓库 → **管理** → 滚动到底部 → **删除仓库**，按提示输入完整仓库名确认。
3. **重新 Fork 官方仓库**：按 [3.1](#31-首先在-antdui-官方仓库-fork-到自己账号) 重新操作。
4. **重新克隆**：按 [3.3](#33-在自己空间地址克隆项目) 重新克隆新 Fork 到本地。
5. **手动恢复你保留的改动**：从备份目录将代码覆盖回新仓库，提交并发起新的 PR。

> ⚠ 警告：此操作具有破坏性。**删除 Fork 前**务必确认备份完整。任何未提交到远程且未备份的本地改动将永久丢失。

### 5.3 提交错分支了

如果不小心直接提交到 `main` 而非功能分支，可将提交迁移到新分支：

```bash
# 创建并切换到新分支，保留刚才的提交
git checkout -b my-feature-branch
# 将 main 重置回上游
git checkout main
git reset --hard upstream/main
git checkout my-feature-branch
```

### 5.4 PR 被要求修改

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

---

感谢您为AntdUI的改进提供帮助——您的贡献让这个项目变得更好！🚀