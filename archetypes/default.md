+++
# 这是文章的「头部信息」（Front Matter），用 TOML 格式。
# 两行 +++ 之间的内容是元数据，不会显示在文章正文里。
# 下面是新建文章时会自动填入的模板。

date = '{{ .Date }}'
draft = true
title = '{{ replace .File.ContentBaseName "-" " " | title }}'

# 自定义字段（可选，按需填写或删除）
tags = ['未分类']        # 文章标签，多个用逗号分隔
categories = ['随笔']    # 文章分类
summary = ''            # 文章摘要，会显示在文章列表里（留空则自动截取）
# cover = 'cover.jpg'   # 文章封面图（把图片放到 static/ 文件夹，然后写文件名）
+++

<!-- 下面写文章正文，用 Markdown 格式。 -->
