# 我的博客（Hugo + PaperMod）

一个用 [Hugo](https://gohugo.io/) 静态网站生成器 + [PaperMod](https://github.com/adityatelange/hugo-PaperMod) 主题搭建的中文博客。

## 🚀 快速开始

### 1. 启动本地预览

```bash
cd blog
hugo server -D
```

然后在浏览器打开 **http://localhost:1313**

> `-D` 表示同时显示草稿（`draft = true` 的文章）。如果想只看成稿，去掉 `-D`。

### 2. 修改成你自己的

打开 `hugo.toml`，修改这几处：

```toml
title = "我的博客"              # 网站标题
baseURL = "https://example.com/" # 改成你的正式域名

[params]
  author = "你的名字"
  description = "我的个人博客"

  [params.homeInfoParams]
    Title = "你好，我是 你的名字 👋"   # ← 改成你的名字
    Content = "欢迎来到我的博客！..."   # ← 改成你的介绍

  [[params.socialIcons]]
    url = "https://github.com/你的用户名"  # ← 改成你的 GitHub
  [[params.socialIcons]]
    url = "mailto:you@example.com"        # ← 改成你的邮箱
```

## 📁 目录结构

```
blog/
├── archetypes/
│   └── default.md       文章模板（新建文章时的默认格式）
├── content/             ★ 你的文章都在这里
│   ├── posts/           博客文章
│   │   ├── hello-world.md
│   │   ├── hugo-startup-guide.md
│   │   └── markdown-tutorial.md
│   ├── about.md         「关于」页面
│   ├── archives.md      「归档」页面（自动生成）
│   └── search.md        「搜索」页面（自动生成）
├── static/              静态资源（图片放这里，引用时写 /文件名.jpg）
├── themes/PaperMod/     主题文件（不要手动改，靠 git 更新）
├── hugo.toml            ★ 站点配置文件（最重要！）
└── public/              生成的网站（hugo 命令后出现，可忽略）
```

## ✏️ 日常写作流程

### 新建一篇文章

```bash
hugo new content posts/my-new-post.md
```

### 编辑文章

打开 `content/posts/my-new-post.md`，修改头部和正文：

```toml
+++
date = 2026-06-21T10:00:00+08:00
draft = false                    # ← 改成 false 才会显示
title = '我的新文章'
tags = ['标签1', '标签2']
categories = ['分类']
summary = '一句话摘要'
+++

正文用 Markdown 写...
```

### 实时预览

保持 `hugo server -D` 运行，保存文件后浏览器**自动刷新**。

## 📋 常用命令

| 命令 | 作用 |
|------|------|
| `hugo new content posts/xxx.md` | 新建文章 |
| `hugo server` | 启动预览（不含草稿）|
| `hugo server -D` | 启动预览（含草稿）|
| `hugo` | 生成静态网站到 `public/` |
| `hugo -D` | 生成（含草稿）|

## 🎨 自定义外观

### 改主题色 / 明暗模式

编辑 `hugo.toml`：

```toml
[params]
  defaultTheme = "auto"   # auto(跟随系统) / light(浅色) / dark(深色)
```

### 加文章封面图

1. 把图片放进 `static/` 文件夹（比如 `static/cover.jpg`）
2. 在文章头部加上：

```toml
cover = 'cover.jpg'
```

### 加头像

如果想用头像模式（而不是文字介绍），参考 [PaperMod 文档](https://adityatelange.github.io/hugo-PaperMod/) 的 `profileMode`。

## 🚢 部署上线

生成静态文件：

```bash
hugo
```

然后把 `public/` 文件夹里的**所有内容**上传到：
- [GitHub Pages](https://pages.github.com/)（免费，推荐）
- [Vercel](https://vercel.com/) / [Netlify](https://www.netlify.com/)（免费，自动部署）
- 任何静态托管服务

详细部署教程：https://gohugo.io/hosting-and-deployment/

## 📚 学习资源

- [Hugo 官方文档](https://gohugo.io/documentation/)
- [PaperMod 主题文档](https://adityatelange.github.io/hugo-PaperMod/)
- [Markdown 中文教程](https://markdown.com.cn/)

## ❓ 常见问题

**Q: 改了 `hugo.toml` 但没生效？**
A: 配置文件的修改通常需要重启服务器（`Ctrl+C` 停止后重新 `hugo server`）。

**Q: 文章不显示？**
A: 检查文章头部 `draft` 是不是 `false`，预览时是否加了 `-D` 参数。

**Q: 中文显示乱码？**
A: 确保文件用 UTF-8 编码保存（大部分编辑器默认就是）。

---

祝你写作愉快！遇到问题可以随时问我。😊
