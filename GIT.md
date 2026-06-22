# Git 常用操作速查

整理日常工作中常用的 Git 操作，方便查阅。

涵盖三类：**Patch 补丁**、**Tag 标签**、**Worktree 工作区**。

> GitLab API 相关内容已拆分到 [GITLAB.md](./GITLAB.md)。

---

## 1. Patch 补丁

用于把改动导出成 `.patch` 文件，再应用到另一个仓库或分支（比如把修复合并到 Release 分支）。

### 1.1 生成 patch

```bash
# 把已跟踪文件的改动导出成 patch
# --output 直接写文件，避免 PowerShell 管道把内容编码成 UTF-16
git diff --output=my.patch
```

### 1.2 包含未跟踪文件

`git diff` 默认不包含未跟踪（untracked）的文件。需要先 `git add` 进暂存区，再用 `--cached` 导出：

```bash
git add .
git diff --cached --binary --output=my.patch
```

> `--binary`：如果改动里有二进制文件（图片、so 库等），必须加这个参数，否则二进制内容会丢失。

### 1.3 应用 patch

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
