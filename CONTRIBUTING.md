# 🦄 Contributing to AntdUI

This guide explains the standards and workflows to help you collaborate smoothly, maintain code consistency, and navigate the project effectively.

## Table of Contents

1. [Directory Structure](#1-directory-structure)
2. [Code Standards](#2-code-standards)
3. [Contribution Workflow](#3-contribution-workflow)
4. [Sync Your Fork After the PR Is Merged](#4-sync-your-fork-after-the-pr-is-merged)
5. [Troubleshooting](#5-troubleshooting)
   - 5.1 [Your fork is out of sync with the official repository](#51-your-fork-is-out-of-sync-with-the-official-repository)
   - 5.2 [Delete and re-fork when commits are messed up](#52-delete-and-re-fork-when-commits-are-messed-up)
   - 5.3 [You committed to the wrong branch](#53-you-committed-to-the-wrong-branch)
   - 5.4 [Your PR receives review feedback requiring changes](#54-your-pr-receives-review-feedback-requiring-changes)

---

## 1. Directory Structure
Familiarize yourself with the directory layout to ensure your contributions are placed in the correct location:

```
AntdUI/
├─ src/                      # All core library source code
│  ├─ AntdUI/                # Main UI library code
│  │  ├─ Controls/           # Custom UI controls
│  │  │  ├─ Chat/            # Chat-related controls (separated due to independent logic or large code volume)
│  │  │  └─ Core/            # Core drawing classes and implementation classes
│  │  ├─ Design/             # Designer support (e.g., Visual Studio designer integration)
│  │  ├─ Enum/               # Enumerations used across the library (e.g., theme types, control states)
│  │  ├─ Events/             # Event definitions
│  │  ├─ Forms/              # Custom windows/forms
│  │  │  └─ LayeredWindow/   # Popup/layered windows (e.g., dropdowns, date pickers, modals, drawers)
│  │  ├─ Lib/                # Utility files (Win32 API wrappers, SVG handlers, helper classes)
│  │  ├─ Localization/       # Multi-language support (resource files for different locales)
│  │  └─ Style/              # Theme definitions (e.g., light/dark themes, color palettes)
│  └─ AntdUI.EmojiFluentFlat/ # Emoji Fluent Flat resource library
├─ example/                  # Demo projects to showcase control usage
│  └─ Demo/                  # Main demo project (includes test forms for all controls)
└─ doc/                      # Documentation (contribution guides, API references, etc.)
```

- **New Controls**: Add custom controls to `src/AntdUI/Controls/` (use subfolders like `Chat/` for independent or large controls).
- **Utility Code**: Place helper functions, Win32 wrappers, or SVG tools in `src/AntdUI/Lib/`.
- **Demo Code**: Update `example/Demo/` to include test cases for new features (helps verify functionality and assist other contributors).

## 2. Code Standards
To maintain consistency and avoid common issues (e.g., memory leaks, UI glitches), follow these core rules:

### 2.1 Drawing Logic Requirements
AntdUI relies on a custom drawing system for high-quality UI rendering. All visual controls must follow:

- **Must Implement `AntdUI.IControl`**:
  All drawable controls **must inherit from `AntdUI.IControl`** (the base interface for custom rendering). Override the `OnDraw` method to implement control-specific drawing:
  ```csharp
  public class MyCustomControl : IControl
  {
	  // Override OnDraw to handle rendering logic
	  protected override void OnDraw(DrawEventArgs e)
	  {
		  base.OnDraw(e);
		  // Use e.Canvas for drawing (see explanation below)
		  e.Canvas.DrawText("Hello AntdUI", _textFont, _textColor, ClientRectangle);
	  }
  }
  ```

- **Use `Canvas` for Rendering**:
  Direct use of `Graphics` is not recommended. Instead, use `Canvas` for rendering to achieve better performance and consistency.
  - Implement complete rendering through `OnDraw` + `OnDrawBg` methods
  - If you cannot inherit `IControl` (e.g., for system control wrappers), obtain `Canvas` instance via `Graphics.High()`
  - For `Bitmap` drawing, use `HighLay` method
  ```csharp
  // Get Canvas in Winform OnPaint
  var canvas = e.Graphics.High(); // Get Canvas instance
  
  // Bitmap drawing
  using (var bitmap = new Bitmap(width, height))
  using (var graphics = Graphics.FromImage(bitmap))
  using (var canvas = graphics.HighLay()) // Use HighLay for bitmap drawing
  {
	  // Drawing logic
  }
  ```

- **Text Rendering Specification**:
  - Do not use `StringFormat` for layout; instead, use `AntdUI.FormatFlags`
  - For text containing Emoji, use `MeasureText` + `DrawText` (Emoji-adapted internally) instead of `MeasureString` + `String`
  ```csharp
  // Recommended: Use FormatFlags and DrawText
  var flags = FormatFlags.Left | FormatFlags.VerticalCenter;
  var size = canvas.MeasureText("Hello 👍", Font, flags);
  canvas.DrawText("Hello 👍", Font, Style.Db.Text, ClientRectangle, flags);
  ```

- **Resource Disposal**:
  Always release GDI resources (e.g., `Bitmap`, `Brush`, `Pen`) immediately after use to avoid memory leaks. Use `using` statements for automatic disposal:
  ```csharp
  // Recommended: Auto-dispose Brush with 'using'
  using (var fillBrush = new SolidBrush(_backgroundColor))
  {
	  e.Canvas.Fill(fillBrush, ClientRectangle);
  }

  // Avoid: Unmanaged resource leaks
  var badBrush = new SolidBrush(_backgroundColor); 
  e.Canvas.Fill(badBrush, ClientRectangle); // ❌ Causes memory leak
  ```

### 2.2 List Control Standards
For controls with scrollable content (e.g., list boxes, data grids), follow these rules:

- **Use `AntdUI.ScrollBar`**:
  Do not use system scrollbars. You must integrate the library's built-in `AntdUI.ScrollBar` component to ensure consistent styling and behavior.

- **Public Properties**:
  Expose scrollbar-related properties to external code (for user customization).

### 2.3 `IControl.RenderRegion` Usage
The `GraphicsPath RenderRegion` property in `AntdUI.IControl` is crucial for **correct mask rendering** (e.g., Spin controls on rounded-corner components).

- **Set `RenderRegion` for Rounded Controls**:
  If your control has rounded corners (or non-rectangular shapes), define `RenderRegion` to ensure Spin controls (or other overlay elements) adapt to the control's shape:
  ```csharp
  protected override GraphicsPath RenderRegion
  {
	  get
	  {
		  return ClientRectangle.RoundPath(8 * Config.Dpi);
	  }
  }
  ```  
  - Without `RenderRegion`, overlay elements like Spin may display as rectangular, breaking UI consistency.

## 3. Contribution Workflow

> To demonstrate how to contribute code using [**Pull Request**](https://github.com/AntdUI/AntdUI/compare/main...main) (referred to as "PR" below)

### 3.1 First, Fork the official [AntdUI](https://github.com/AntdUI/AntdUI) repository to your own account

> Click the **Fork** button in the upper right corner

![1](doc/wiki/en/Img/PR_1.png)

### 3.2 Confirm the fork target

> By default, it will fork to your personal account. Click **Create fork** to continue

![2](doc/wiki/en/Img/PR_2.png)

> Wait for the forking process to complete. The page will refresh automatically once finished.
> ![3](doc/wiki/en/Img/PR_3.png)

### 3.3 Clone the repository from your personal account

> Click **Code**, then copy the `.git` URL from the pop-up window. ⚠ Note: Ensure it's the URL from your **own repository**

![5](doc/wiki/en/Img/PR_5.png)

Open [Visual Studio](https://visualstudio.microsoft.com), clone the URL you just copied, and after successful cloning, **commit the code you want to contribute**.

### 3.4 Initiate [PR](https://github.com/AntdUI/AntdUI/compare/main...main)

> Click the **Pull requests** tab, or [click **Pull Request** to jump to the PR submission page](https://github.com/AntdUI/AntdUI/compare/main...main), then click **New pull request** to start the PR

![6](doc/wiki/en/Img/PR_6.png)

### 3.5 Final step

> After verifying and adjusting your submission, click **Create pull request**

![7](doc/wiki/en/Img/PR_7.png)

> Fill in the PR title and description. Clearly state what changes you made and why you made them. Click **Create pull request** again to confirm submission

![8](doc/wiki/en/Img/PR_8.png)

> I will review your PR as soon as I receive it. 🧙 Have a nice day!

---

## 4. Sync Your Fork After the PR Is Merged

> Once the maintainer merges your PR, the official repository moves forward but **your fork is not updated automatically**. Before starting any new contribution, always bring your fork up to date with the upstream `main` branch.

Open your forked repository on GitHub, click **Sync fork** → **Update branch**. If the button reads **No changes** your fork is already in sync.

![4](doc/wiki/en/Img/PR_4.png)

Then pull the latest code locally so your working copy matches the upstream:

```bash
# Make sure you are on the main branch
git checkout main

# Pull the latest changes from your fork (origin)
git pull origin main
```

> Tip: If you also set up an `upstream` remote pointing at the official repository, you can sync directly from upstream:
> ```bash
> git remote add upstream https://github.com/AntdUI/AntdUI.git
> git fetch upstream
> git merge upstream/main
> git push origin main
> ```

---

## 5. Troubleshooting

This section covers the most common abnormal situations you may encounter during the contribution workflow.

### 5.1 Your fork is out of sync with the official repository

**Symptom**: GitHub shows "This branch is out-of-date" / "This branch has conflicts that must be resolved" when you submit a new PR.

**Solution**: Follow [Section 4: Sync your fork](#4-sync-your-fork-after-the-pr-is-merged) first. If conflicts already exist on your branch:

```bash
git fetch upstream
git merge upstream/main
# Resolve conflicts in your editor, then:
git add .
git commit
git push origin your-branch
```

### 5.2 Delete and re-fork when commits are messed up

If your local commits have diverged badly, you have rebased incorrectly, or merge conflicts are unresolvable, the cleanest approach is to **delete the fork and start fresh**:

1. **Manually back up your local changes**. Copy any modified files you still need into a folder outside the repository (e.g., `D:\AntdUI_backup\`). The re-fork process will lose all commits and branches.
2. **Delete the fork** on GitHub: open your forked repo → **Settings** → scroll to bottom → **Delete this repository**. Confirm by typing the full repo name.
3. **Re-fork the official repository** by following [3.1](#31-first-fork-the-official-antdui-repository-to-your-own-account) again.
4. **Re-clone** the new fork locally by following [3.3](#33-clone-the-repository-from-your-personal-account).
5. **Re-apply your saved changes** manually from the backup folder, then commit and submit a fresh PR.

> ⚠ Warning: This is a destructive operation. Make sure your backup is complete **before** deleting the fork. Any unsubmitted local-only work that you did not back up will be lost forever.

### 5.3 You committed to the wrong branch

If you committed to `main` directly instead of a feature branch, you can move the commit to a new branch:

```bash
# Create and switch to a new branch, keeping the commit
git checkout -b my-feature-branch
# Reset main back to upstream
git checkout main
git reset --hard upstream/main
git checkout my-feature-branch
```

### 5.4 Your PR receives review feedback requiring changes

After the maintainer requests changes:

```bash
# Switch to the branch used for the PR
git checkout your-pr-branch

# Apply the requested changes in your editor, then:
git add .
git commit -m "address review feedback"
git push origin your-pr-branch
```

The PR will automatically update — no need to open a new one.

---

Thank you for helping improve AntdUI—your contributions make this project better for everyone! 🚀