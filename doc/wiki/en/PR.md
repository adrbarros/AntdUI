[Home](Home.md)・[UpdateLog](UpdateLog.md)・[Config](Config.md)・[Theme](Theme.md)

## Contributing to [AntdUI](https://github.com/AntdUI/AntdUI) on GitHub

This guide demonstrates how to contribute code via [**Pull Request**](https://github.com/AntdUI/AntdUI/compare/main...main) (referred to as "PR" below). We will use the **your-account** placeholder to represent your personal fork (replace with your own GitHub username).

***

### 1. Fork the official [AntdUI](https://github.com/AntdUI/AntdUI) repository to your own account

> Click the **Fork** button in the upper right corner

![1](Img/PR_1.png)

### 2. Confirm the fork target

> By default, it will fork to your personal account. Click **Create fork** to continue

![2](Img/PR_2.png)

> Wait for the forking process to complete. The page will refresh automatically once finished.
> ![3](Img/PR_3.png)

### 3. Clone the repository from your personal account

> Click **Code**, then copy the `.git` URL from the pop-up window. ⚠ Note: Make sure it is the URL from **your own repository**

![5](Img/PR_5.png)

Open [Visual Studio](https://visualstudio.microsoft.com), clone the URL you just copied, and after successful cloning, **commit the code you want to contribute**.

### 4. Initiate [PR](https://github.com/AntdUI/AntdUI/compare/main...main)

> Click the **Pull requests** tab, or [click **Pull Request** to jump to the PR submission page](https://github.com/AntdUI/AntdUI/compare/main...main), then click **New pull request** to start the PR

![6](Img/PR_6.png)

### 5. Final step

> After verifying and adjusting your submission, click **Create pull request**

![7](Img/PR_7.png)

> Fill in the PR title and description. Clearly state what changes you made and why you made them. Click **Create pull request** again to confirm submission

![8](Img/PR_8.png)

> I will review your PR as soon as I receive it. 🧙 Have a nice day!

---

### 6. Sync your fork after the PR is merged

> Once the maintainer merges your PR, the official repository moves forward but **your fork is not updated automatically**. Before starting any new contribution, always bring your fork up to date with the upstream `main` branch.

Open your forked repository on GitHub, click **Sync fork** → **Update branch**. If the button reads **No changes** your fork is already in sync.

![4](Img/PR_4.png)

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

### 7. Troubleshooting

This section covers the most common abnormal situations you may encounter during the contribution workflow.

#### 7.1 Your fork is out of sync with the official repository

**Symptom**: GitHub shows "This branch is out-of-date" / "This branch has conflicts that must be resolved" when you submit a new PR.

**Solution**: Follow [step 6: Sync your fork](#6-sync-your-fork-after-the-pr-is-merged) first. If conflicts already exist on your branch:

```bash
git fetch upstream
git merge upstream/main
# Resolve conflicts in your editor, then:
git add .
git commit
git push origin your-branch
```

#### 7.2 Delete and re-fork when commits are messed up

If your local commits have diverged badly, you have rebased incorrectly, or merge conflicts are unresolvable, the cleanest approach is to **delete the fork and start fresh**:

1. **Manually back up your local changes**. Copy any modified files you still need into a folder outside the repository (e.g., `D:\AntdUI_backup\`). The re-fork process will lose all commits and branches.
2. **Delete the fork** on GitHub: open your forked repo → **Settings** → scroll to bottom → **Delete this repository**. Confirm by typing the full repo name.
3. **Re-fork the official repository** by following [step 1](#1-fork-the-official-antdui-repository-to-your-own-account) again.
4. **Re-clone** the new fork locally by following [step 3](#3-clone-the-repository-from-your-personal-account).
5. **Re-apply your saved changes** manually from the backup folder, then commit and submit a fresh PR.

> ⚠ Warning: This is a destructive operation. Make sure your backup is complete **before** deleting the fork. Any unsubmitted local-only work that you did not back up will be lost forever.

#### 7.3 You committed to the wrong branch

If you committed to `main` directly instead of a feature branch, you can move the commit to a new branch:

```bash
# Create and switch to a new branch, keeping the commit
git checkout -b my-feature-branch
# Reset main back to upstream
git checkout main
git reset --hard upstream/main
git checkout my-feature-branch
```

#### 7.4 Your PR receives review feedback requiring changes

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
