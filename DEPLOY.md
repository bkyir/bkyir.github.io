# 部署指南：把博客发布到 GitHub Pages

本文档教你如何把 Hugo 博客托管到 GitHub，实现 **「写完 push，自动上线」**。

---

## 🎯 整体流程

```
本地写文章 (Markdown)
      ↓  git push
GitHub 仓库（保存你的所有内容）
      ↓  自动触发
GitHub Actions（云端自动构建）
      ↓
GitHub Pages（免费托管）
      ↓
全世界访问  https://用户名.github.io
```

**核心**：配置好之后，你**只需要 `git push`**，剩下全自动。

---

## 📋 一次性配置（只做一次）

### 第 1 步：在 GitHub 创建仓库

1. 打开 https://github.com/new
2. 仓库名（二选一，影响最终网址）：

   | 仓库名 | 最终访问地址 | 推荐 |
   |--------|-------------|------|
   | `用户名.github.io` | `https://用户名.github.io/` | ⭐ 简洁 |
   | `my-blog`（任意名）| `https://用户名.github.io/my-blog/` | 一般 |

   > 比如你的 GitHub 用户名是 `zhangsan`，建议仓库名就叫 `zhangsan.github.io`

3. 选 **Public**（公开，免费版 Pages 必须 public）
4. **不要**勾选 "Add a README"（我们本地已经有了）
5. 点 **Create repository**

### 第 2 步：修改 baseURL

打开 `hugo.toml`，根据你选的仓库名修改：

```toml
# 如果仓库叫「用户名.github.io」
baseURL = "https://zhangsan.github.io/"

# 如果仓库叫别的名字（比如 my-blog）
baseURL = "https://zhangsan.github.io/my-blog/"
```

### 第 3 步：把本地代码推送到 GitHub

在 `blog` 目录下依次运行（把地址换成你自己的）：

```bash
# 1. 关联远程仓库（地址换成你的）
git remote add origin https://github.com/用户名/仓库名.git

# 2. 把所有文件加入暂存区
git add .

# 3. 创建第一个提交
git commit -m "初始化 Hugo 博客"

# 4. 推送到 GitHub
git branch -M main
git push -u origin main
```

> 第一次推送会让你登录 GitHub。建议用 [GitHub CLI](https://cli.github.com/) 认证：
> ```bash
> brew install gh
> gh auth login
> ```
> 按提示操作，之后 push 就不用再输密码了。

### 第 4 步：开启 GitHub Pages（关键！）

1. 打开你的仓库网页 → 点 **Settings**（设置）
2. 左侧菜单找 **Pages**
3. **Source** 那一栏选 **GitHub Actions**（不是 Deploy from a branch！）
4. 保存

> 这一步必须做，否则部署会失败。选 "GitHub Actions" 表示「用我们的工作流文件来部署」。

### 第 5 步：等待首次部署

1. 仓库上方点 **Actions** 标签页
2. 会看到一个 "Deploy Hugo to GitHub Pages" 的任务在跑
3. 等 1-2 分钟，绿色 ✅ 就表示成功
4. 回到 **Settings → Pages**，顶部会显示你的网址：`https://用户名.github.io/`

🎉 **恭喜！博客上线了！**

---

## ✏️ 日常写作流程（每次写文章）

以后每次写文章，固定三步：

```bash
# 1. 写文章（用 Hugo 命令创建 + 编辑器修改）
hugo new content posts/我的新文章.md
# 编辑文件，把 draft 改成 false

# 2. 本地预览确认效果
hugo server -D
# 浏览器打开 http://localhost:1313 检查

# 3. 满意了就推送（这三行背下来！）
git add .
git commit -m "发布新文章：我的新文章"
git push
```

**push 之后 1-2 分钟**，GitHub Actions 自动构建，新文章就上线了。

> 💡 这就是你想要的「高效管理 + 方便部署」——本地不用构建，服务器不用管，push 一下全搞定。

---

## 🔧 常用 Git 命令速查

| 命令 | 作用 |
|------|------|
| `git add .` | 把所有改动加入暂存区 |
| `git commit -m "说明"` | 提交（写一句话说明改了啥）|
| `git push` | 推送到 GitHub（触发自动部署）|
| `git status` | 查看当前哪些文件改了 |
| `git log --oneline` | 查看提交历史 |

---

## ❓ 常见问题

### Q: 推送后网站没更新？

1. 检查 **Actions** 标签页，任务是否报错（红色 ❌）
2. 常见错误：`baseURL` 写错、文章 `draft` 没改成 `false`
3. 点进失败的任务看日志，通常能看出原因

### Q: 网站样式错乱 / 图片不显示？

99% 是 `baseURL` 配置错了。仔细对照第 2 步：
- 仓库名带 `.github.io` → URL 结尾**不要**加仓库名
- 仓库名是别的 → URL 结尾**必须**加 `/仓库名/`

### Q: 怎么知道构建成功了？

仓库 → **Actions** 标签 → 看最新一次任务是不是绿色 ✅。

### Q: 可以先在本地测试部署效果吗？

```bash
hugo          # 生成到 public/
# 然后用任意 HTTP 服务器打开 public 文件夹测试
```

### Q: 以后想换域名怎么办？

1. 买域名（阿里云/腾讯云/Namecheap）
2. GitHub 仓库 → Settings → Pages → Custom domain → 填你的域名
3. 把 `hugo.toml` 的 `baseURL` 改成 `https://你的域名/`
4. `git push` 即可

---

## 📚 参考资源

- [Hugo 官方部署文档（GitHub Pages）](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
- [GitHub Pages 官方文档](https://docs.github.com/en/pages)
- [Git 简明教程](https://www.runoob.com/git/git-tutorial.html)
