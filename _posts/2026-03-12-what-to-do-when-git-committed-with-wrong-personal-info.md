---
layout: article
title: Git 提交信息填错了怎么办
subtitle: 一文讲透 Author 与 Committer 的区别及修复方案
description: 你是否曾因为配置错误的 user.name 或 user.email，导致 GitHub 上的提交者显示不正常？本文将带你深入理解 Git 中 Author 与 Committer 的核心区别，并提供一套基于官方推荐工具 `git-filter-repo` 的完整修复方案，帮你优雅地重写历史。
tags:
  - explanation
  - git
lang: zh
---
> "推送完才发现，GitHub 历史中的头像怎么是两个灰色的脑袋"
>
> —— 头一次发现原来还有 Author 和 Committer 的不同。
> 
> 等等，我的用户名怎么是另一个项目的？

看着刚刚推送的代码，我发现一件奇怪的事：几次提交的 Author 显示的是另一个项目 repo 中使用的邮箱和用户名。而且只有我一个人提交和推送，怎么会冒出来两个头像？

检查配置才发现，本地 Git 把在全局范围内都配置了那个邮箱和用户名：

```bash
git config user.email "old_email@company.com"
git config user.name "old name"
```

提交已经推送到远程了。要改吗？要，强迫症忍不了。。怎么改？Author 和 Committer 到底有什么区别？

就让这篇文章来记录我理清这两个概念，并修复历史提交的过程。

## 一、Author 和 Committer 到底是什么

在 Git 的设计里，每次提交都记录了两组身份信息。很多人用了几年 Git，却从未注意过它们的区别。

| 角色 | 含义 | 关键问题 |
|------|------|----------|
| **Author** | 代码的原始编写者 | 谁写了这段代码？ |
| **Committer** | 代码的最终入库者 | 谁执行了 `git commit`？ |

它们各自还有独立的时间戳：
- **Author Date**：代码最初被创建的时间
- **Committer Date**：代码被应用到当前仓库的时间

在个人开发时，Author 和 Committer 都是你，两者完全一致。这也是为什么很多人从未意识到它们的存在。

但在协作场景下，它们经常不同：

**场景一：应用补丁**
小明写了一个 Bug 修复，发给你。你应用补丁并提交。Author 是小明，Committer 是你。

**场景二：Cherry-pick**
你在分支 A 上提交了一个功能，管理员用 `git cherry-pick` 把它引入主干。Author 还是你，但 Committer 变成了管理员，时间也更新为拣选的时刻。

**场景三：Rebase**
你对历史提交进行变基。Author 信息保留，但 Committer 更新为执行变基的人。

要看到完整的提交信息，用：

```bash
git log --format=fuller -n 1
```

输出大概是：

```
commit a1b2c3d4...
Author:     Old Name <old@example.com>
AuthorDate: Mon Mar 10 10:00:00 2026 +0800
Commit:     Old Name <old@example.com>
CommitDate: Mon Mar 10 10:05:00 2026 +0800
```

如果配置错了，Author 和 Committer 的邮箱都是错的。**修复时必须同时修改两者**，否则 GitHub 依然无法正确关联你的账号。

## 二、重写历史：用 git-filter-repo 修复

既然提交已经推送到远程，我们需要重写本地历史，然后强制推送覆盖。

**注意**：这会改变提交的哈希值。如果是多人协作的仓库，务必先通知团队成员，因为他们需要重置本地分支。个人仓库则可以直接操作。

*其实如果只是名字和邮箱的问题，不改也就不改了，实在没必要冒着团队成员都得闻风而动的风险来修改个名字——如果你是强迫症，那拜托长点教训吧。*

### 为什么不用 git filter-branch？

很多老教程推荐 `git filter-branch`，但它已被 Git 官方标记为过时。它速度慢、语法复杂，而且容易出错。

现在官方推荐的工具是 [`git-filter-repo`](https://github.com/newren/git-filter-repo)。基于 Python，速度快，也更安全。

### 安装

```bash
pip install git-filter-repo
# macOS 用户也可以用 brew
brew install git-filter-repo
```

### 编写修复脚本

假设：
- 错误邮箱：`wrong@example.com`
- 正确名字：`Your Name`
- 正确邮箱：`correct@example.com`

执行：

```bash
git filter-repo --commit-callback '
if commit.author_email == b"wrong@example.com":
    commit.author_name = b"Your Name"
    commit.author_email = b"correct@example.com"

if commit.committer_email == b"wrong@example.com":
    commit.committer_name = b"Your Name"
    commit.committer_email = b"correct@example.com"
'
```

### 处理报错："Refusing to destructively overwrite..."

第一次运行时，你很可能会遇到：

> `Aborting: Refusing to destructively overwrite repo history since this does not look like a fresh clone.`

这是 `git-filter-repo` 的安全机制：它默认拒绝在非"新鲜克隆"的仓库上运行，防止误操作破坏复杂的本地历史。

**稳妥的做法是重新克隆**：

```bash
cd ..
git clone <仓库URL> repo-fixed
cd repo-fixed
# 再次运行上面的修复命令
```

确认无误后，删除旧的仓库文件夹。

如果你确定当前仓库可以操作，也可以加 `--force` 跳过检查：

```bash
git filter-repo --force --commit-callback '...'
```

### 重新添加远程仓库

`git-filter-repo` 执行后会清空远程仓库信息，需要重新配置：

```bash
git remote add origin git@github.com:username/repo.git
```

### 强制推送

历史重写完成，本地提交的哈希值全变了。强制覆盖远程：

```bash
git push --force --all
git push --force --tags
```

## 三、验证修复结果

本地检查：

```bash
git log --pretty=format:"%h %an <%ae>" -n 5
```

确认邮箱已更新。

刷新 GitHub 页面：
- 头像应该变成你账号绑定的头像
- 贡献图里的灰色格子应该变成绿色
- 点击提交记录，Author 和 Committer 都显示正确

### 团队协作的善后

如果仓库有其他人协作，他们拉取代码时会报错。通知他们执行：

```bash
git fetch --all
git reset --hard origin/main  # 替换为主分支名
```

这会丢弃他们本地未推送的提交，务必提醒他们先备份。

再次提醒，如果仅仅因为名称问题，能不修改就不修改了。。。

## 四、写在最后

这次经历让我意识到：

1. **Author 和 Committer 是两个独立的概念**。个人开发时它们相同，配错时需同时修复。
2. **`git-filter-repo` 已取代 `filter-branch`**，更快、更安全。
3. **GitHub 的贡献统计强依赖邮箱匹配**，这是很多人"格子不绿"的真正原因。

代码历史可以重写，但良好的配置习惯更能避免未来的麻烦。**如果经常在不同代码库里使用不同的~~马甲~~身份，最好在每次初始化仓库时就设置好用户名和邮箱，或者至少在全局设一个最常用的。**

```bash
git config --global user.name "你的名字"
git config --global user.email "你的常用邮箱"
```

毕竟，谁也不想再对着一片灰色的贡献图发呆。
