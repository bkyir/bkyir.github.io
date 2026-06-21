+++
date = 2026-06-21T11:00:00+08:00
draft = false
title = 'Hugo 新手入门指南'
tags = ['Hugo', '教程', '前端']
categories = ['学习笔记']
summary = '介绍 Hugo 的基本用法：如何新建文章、本地预览、生成静态文件，以及常用命令速查。'
+++

这篇笔记记录一下 Hugo 的日常用法，方便以后查阅。

## 常用命令速查

| 命令 | 作用 |
|------|------|
| `hugo new content posts/xxx.md` | 新建一篇文章 |
| `hugo server` | 启动本地预览（默认 http://localhost:1313）|
| `hugo server -D` | 预览时也显示草稿（draft = true 的文章）|
| `hugo` | 生成静态网站到 `public/` 文件夹 |
| `hugo -D` | 生成时也包含草稿 |

## 目录结构说明

```
blog/
├── archetypes/      文章模板（新建文章时的默认格式）
├── content/         你的文章（Markdown 文件）
│   └── posts/       博客文章放在这里
├── layouts/         自定义页面布局（覆盖主题）
├── static/          静态资源（图片、文件等，原样复制到网站）
├── themes/          主题文件夹
├── hugo.toml        站点配置文件（最重要！）
└── public/          生成的网站（hugo 命令后出现，可忽略）
```

## 如何写一篇新文章

1. 运行命令：

   ```bash
   hugo new content posts/my-post.md
   ```

2. 打开 `content/posts/my-post.md`，编辑头部信息和正文。
3. 把 `draft = true` 改成 `draft = false`（否则文章不显示）。
4. 运行 `hugo server -D` 在浏览器预览效果。

## Front Matter（文章头部）说明

每篇文章最上方 `+++` 之间的内容叫 Front Matter：

```toml
+++
date = 2026-06-21T10:00:00+08:00   # 发布时间
draft = false                       # false = 正式发布，true = 草稿
title = '文章标题'
tags = ['标签1', '标签2']           # 标签
categories = ['分类']               # 分类
summary = '一句话摘要'              # 列表里显示的摘要
cover = 'cover.jpg'                 # 封面图（放在 static/ 里）
+++
```

## 小技巧

- 📁 图片放到 `static/` 文件夹，引用时直接写 `/图片名.jpg`
- 🏷️ 标签和分类会自动生成聚合页面
- 🔍 配置好 `[outputs]` 的 `JSON` 后，主题内置搜索功能可用
- 📤 部署时只需把 `public/` 文件夹内容上传到服务器或 GitHub Pages

---

祝写作愉快！遇到问题多查 [Hugo 官方文档](https://gohugo.io/documentation/)。
