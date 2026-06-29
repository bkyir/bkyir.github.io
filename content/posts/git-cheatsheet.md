+++
date = 2026-06-22T09:30:00+08:00
draft = false
title = 'Git 常用操作速查'
tags = ['Git', '工具', '效率']
categories = ['工具']
summary = '整理日常工作中常用的 Git 操作：Patch 补丁、Tag 标签、Worktree 工作区。'
+++

整理日常工作中常用的 Git 操作，方便查阅。

涵盖三类：**Patch 补丁**、**Tag 标签**、**Worktree 工作区**。

> GitLab API 相关内容见 [GitLab API 常用接口]({{< relref "gitlab-api.md" >}})。

---

## 1. Patch 补丁

用于把改动导出成 `.patch` 文件，再应用到另一个仓库或分支（比如把修复合并到 Release 分支）。

Git 提供两套补丁机制，按改动来源选择：

| 方式 | 格式 | 是否含提交信息 | 应用结果 |
|------|------|------|------|
| `git diff` + `git apply` | 纯差异 | 否 | 只改工作区，不产生提交 |
| `git format-patch` + `git am` | 邮箱（mailbox） | 是（保留作者/说明） | 直接生成新提交 |

### 1.1 基于工作区改动：git diff + git apply

改动**尚未提交**（还在工作区 / 暂存区）时用这套，导出纯差异补丁。

**生成 patch**

```bash
# 把已跟踪文件的改动导出成 patch
# --output 直接写文件，避免 PowerShell 管道把内容编码成 UTF-16
git diff --output=my.patch
```

**包含未跟踪文件**

`git diff` 默认不包含未跟踪（untracked）的文件。需要先 `git add` 进暂存区，再用 `--cached` 导出：

```bash
git add .
git diff --cached --binary --output=my.patch
```

> `--binary`：如果改动里有二进制文件（图片、so 库等），必须加这个参数，否则二进制内容会丢失。

**应用 patch**

```bash
# 自动修复空格问题（行尾多余空格、Tab/空格混用等）
git apply --whitespace=fix my.patch
```

其他常用选项：

| 选项 | 作用 |
|------|------|
| `--check` | 只检查能否干净应用，不实际改动（建议先用） |
| `--3way` | 冲突时走三方合并，留下冲突标记而不是直接失败 |
| `-R` | 反向应用（撤销 patch） |

```bash
# 推荐流程：先 check 再 apply
git apply --check my.patch && git apply my.patch
```

### 1.2 基于已提交 commit：git format-patch + git am

改动**已经提交**时用这套。`git format-patch` 基于 commit 生成邮箱（mailbox）格式补丁，包含作者、提交信息等元数据；配套用 `git am` 应用，会保留这些信息并直接生成新提交。

**生成补丁**

```bash
# 导出最近 1 个 commit（自动命名 0001-提交标题.patch）
git format-patch -1 HEAD

# 导出区间 [base, head) 内的提交（不含 base）
git format-patch <base-commit>..<head-commit>

# 导出 base 之后的所有提交（等价于 base..HEAD）
git format-patch <base-commit>

# 指定输出目录，避免 .patch 散落在仓库里
git format-patch -o patches <base-commit>..<head-commit>
```

**应用补丁**

```bash
# 应用 patch（支持多个文件 / 通配符）
git am patches/0001-*.patch

# 冲突时：手动解决后继续
git add .
git am --continue

# 完全放弃本次应用，回到应用前状态
git am --abort
```

---

## 2. Tag 标签

给某个 commit 打版本标签，常用于发版。

### 2.1 创建 tag

```bash
# 给指定 commit 打轻量标签
git tag V2.0.12 <commit-hash>

# 给当前 HEAD 打附注标签（推荐，带说明信息）
git tag -a V2.0.12 -m "发布 V2.0.12"
```

### 2.2 推送 tag

```bash
# 推送单个 tag
git push origin V2.0.12

# 一次性推送所有本地 tag
git push origin --tags
```

### 2.3 其他常用

```bash
# 查看所有 tag
git tag

# 删除本地 tag
git tag -d V2.0.12

# 删除远程 tag
git push origin :refs/tags/V2.0.12
```

---

## 3. Worktree 工作区

**核心场景**：在不切换当前分支、不打扰现有工作区的前提下，在另一个目录里检出别的分支。

比如正在 `feature` 上写代码，突然要修个紧急 bug——用 worktree 新开一个目录检出 `hotfix` 分支，修完直接回去，互不干扰。

### 3.1 创建 worktree

```bash
# 把已有的分支检出到新目录
git worktree add ../feature-branch feature/awesome

# 新建分支并检出到新目录（-b 创建分支）
git worktree add -b hotfix ../hotfix-branch
```

> 路径建议放在仓库**外面**（如 `../xxx`），不要放在仓库内部，避免被 git 跟踪。

### 3.2 查看所有 worktree

```bash
git worktree list
```

### 3.3 删除 worktree

```bash
# 删除（会自动清理关联目录）
git worktree remove ../feature-branch

# 强制删除（即使该目录有未提交的修改）
git worktree remove -f ../feature-branch
```

> 删除后若残留元数据，可执行 `git worktree prune` 清理。
