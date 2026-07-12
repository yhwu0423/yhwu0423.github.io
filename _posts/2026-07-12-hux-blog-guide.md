---
layout: post
title: "Hux Blog 使用指南"
subtitle: "从零开始搭建和使用 Hux 博客模板"
date: 2026-07-12 12:00:00
author: "wyh"
header-style: text
catalog: true
tags:
  - Meta
  - Web
---

> 本文是关于 [Hux Blog](https://github.com/Huxpro/huxpro.github.io) 模板的使用说明，帮助你快速上手搭建和管理自己的博客。

## 前言

Hux Blog 是一个基于 Jekyll 的博客模板，托管在 GitHub Pages 上，免费、简洁、支持 Markdown，非常适合技术人员写博客。本文将从项目结构、配置、写文章、部署等方面做一个全面的介绍。

---

## 一、快速开始

### 1.1 Fork 仓库

1. 前往 [Hux Blog GitHub 仓库](https://github.com/Huxpro/huxpro.github.io)
2. 点击右上角 **Fork**
3. 将 Fork 后的仓库重命名为 `<你的GitHub用户名>.github.io`
4. 等待几分钟后，访问 `https://<你的GitHub用户名>.github.io` 即可看到博客

### 1.2 本地环境搭建

如果你需要在本地预览博客，需要安装 Jekyll 环境：

```bash
# 安装 Ruby（推荐使用 rbenv 或 RubyInstaller）
# 安装 Jekyll 和 Bundler
gem install jekyll bundler

# 进入项目目录
cd <你的仓库名>.github.io

# 安装依赖
bundle install

# 本地运行
bundle exec jekyll serve
```

启动后访问 `http://localhost:4000` 即可在本地预览。

---

## 二、项目结构

```
.
├── _config.yml          # 站点配置文件（最重要）
├── _posts/              # 博客文章目录
│   ├── 2026-07-12-xxx.md
│   └── ...
├── _layouts/            # 页面布局模板
├── _includes/           # 可复用的页面组件
├── img/                 # 图片资源
│   ├── post-bg-*.jpg    # 文章头图
│   ├── in-post/         # 文章内嵌图片
│   └── avatar-*.jpg     # 头像
├── css/                 # 样式文件
├── js/                  # JavaScript 文件
├── about.html           # 关于页面
├── tags.html            # 标签页
├── 404.html             # 404 页面
└── index.html           # 首页
```

---

## 三、站点配置（_config.yml）

这是整个博客最核心的配置文件，主要修改以下字段：

```yaml
# 站点基本信息
title: "你的博客名"
SEOTitle: "你的博客名 | Your Blog"
description: "博客描述"
keyword: "关键词1, 关键词2"
url: "https://<用户名>.github.io"

# 作者信息
email: your@email.com

# 社交账号
github_username: your_github
zhihu_username: your_zhihu
weibo_username: your_weibo
twitter_username: your_twitter

# 分页设置
paginate: 10
```

### 3.1 侧边栏配置

```yaml
# Sidebar settings
sidebar: true
sidebar-about-description: "你的个人简介"
sidebar-avatar: /img/avatar-your.jpg
```

### 3.2 评论系统

Hux Blog 支持多种评论系统：

```yaml
# Disqus
disqus_username: your_disqus_shortname

# Gitalk（推荐国内使用）
gitalk:
  enable: true
  clientID: 'your_client_id'
  clientSecret: 'your_client_secret'
  repo: 'your_repo'
  owner: 'your_github_username'
  admin: ['your_github_username']
```

### 3.3 网站统计

```yaml
# Google Analytics
ga_track_id: 'UA-XXXXXXXX-X'

# 百度统计
ba_track_id: 'your_baidu_track_id'
```

---

## 四、写博客文章

### 4.1 文件命名规范

所有文章放在 `_posts/` 目录下，文件名格式为：

```
YYYY-MM-DD-文章标题.md
```

例如：`2026-07-12-hux-blog-guide.md`

### 4.2 Front Matter（文章头部配置）

每篇文章开头都需要 YAML 格式的 Front Matter：

```yaml
---
layout: post
title: "文章标题"
subtitle: "副标题（可选）"
date: 2026-07-12 12:00:00
author: "你的名字"
header-img: "img/post-bg-xxx.jpg"   # 头图方式一：使用图片
header-style: text                   # 头图方式二：纯文字（二选一）
header-mask: 0.3                     # 头图遮罩透明度（可选）
catalog: true                        # 是否显示目录（可选）
tags:
  - 标签1
  - 标签2
---
```

**注意**：`header-img` 和 `header-style` 二选一。

### 4.3 常用 Front Matter 字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `layout` | 是 | 固定为 `post` |
| `title` | 是 | 文章标题 |
| `subtitle` | 否 | 副标题 |
| `date` | 否 | 发布日期（不填则从文件名推断） |
| `author` | 是 | 作者名 |
| `header-img` | 否 | 文章头图路径 |
| `header-style` | 否 | 设为 `text` 则使用纯文字头部 |
| `header-mask` | 否 | 头图遮罩透明度（0-1） |
| `catalog` | 否 | 是否显示文章目录 |
| `tags` | 是 | 标签列表 |
| `published` | 否 | 设为 `false` 可隐藏文章 |
| `multilingual` | 否 | 设为 `true` 支持双语切换 |

### 4.4 文章内容编写

文章正文使用 Markdown 语法，支持：

- **标题**：`## 二级标题`、`### 三级标题`
- **代码块**：用 ``` 包裹，支持语法高亮
- **图片**：`![描述](img/in-post/xxx.jpg)`
- **引用**：`> 引用内容`
- **数学公式**：支持 MathJax
- **表格**、**列表** 等标准 Markdown 语法

### 4.5 图片管理

- 文章头图放在 `img/` 目录下，命名为 `post-bg-xxx.jpg`
- 文章内图片放在 `img/in-post/` 目录下
- 建议为每篇文章创建子目录：`img/in-post/post-slug/`

---

## 五、高级功能

### 5.1 文章分类目录

在 Front Matter 中设置 `catalog: true` 即可自动生成文章目录（基于 h2、h3 标题）。

### 5.2 标签系统

所有在文章 Front Matter 中使用的 `tags` 会自动出现在标签页（`/tags/`）中。

### 5.3 自定义页面

可以在根目录创建新的 HTML 或 Markdown 文件作为独立页面（如 `about.html`），在导航栏配置中添加链接：

```yaml
# _config.yml
# Nav Bar
nav:
  Home: "/"
  About: "/about/"
  Tags: "/tags/"
  Archive: "/archive/"
```

### 5.4 草稿管理

- 将文章 `published` 设为 `false` 可以隐藏文章
- 也可以将草稿放在 `_drafts/` 目录中
- 本地预览草稿：`bundle exec jekyll serve --drafts`

---

## 六、部署与发布

### 6.1 GitHub Pages 自动部署

只需将代码推送到 GitHub 仓库的 `master`（或 `main`）分支，GitHub Pages 会自动构建和部署：

```bash
git add .
git commit -m "新增文章：xxx"
git push origin master
```

通常几分钟内就能在线上看到更新。

### 6.2 自定义域名

1. 在仓库根目录创建 `CNAME` 文件，写入你的域名
2. 在域名 DNS 中添加 CNAME 记录指向 `<用户名>.github.io`
3. 在 GitHub 仓库 Settings → Pages 中配置

---

## 七、常见问题

### Q1: 本地运行报错？

确保 Ruby 版本 >= 2.5，Jekyll 版本 >= 3.0。可以尝试：
```bash
bundle update
bundle exec jekyll serve
```

### Q2: 文章不显示？

- 检查文件名日期格式是否正确
- 检查 Front Matter 格式是否有误（YAML 语法）
- 如果文章日期是未来的，需要在 `_config.yml` 中设置 `future: true`

### Q3: 图片不显示？

- 确保图片路径正确（相对于站点根目录）
- 图片文件名不要包含中文或空格

### Q4: 如何修改博客主题颜色？

编辑 `less/` 目录下的样式文件，或直接修改 `css/` 中的编译后文件。

---

## 总结

Hux Blog 是一个非常优秀的 Jekyll 博客模板，具备以下优点：

- **简洁美观**：响应式设计，移动端友好
- **功能完善**：支持目录、标签、评论、统计
- **易于使用**：Markdown 写作，Git 管理
- **免费托管**：GitHub Pages 免费部署

希望这篇指南能帮助你快速上手，开始你的博客之旅！
