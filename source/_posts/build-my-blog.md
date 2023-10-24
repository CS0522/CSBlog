---
title: 【文档小记】搭建本博客
toc: true
tags:
  - Hexo
  - Volantis
  - Blog
languages:
  - zh-CN
categories:
  - 文档小记
comments: false
cover: false
date: 2023-10-22 00:00:00
---

记录搭建本博客的过程，以及遇到的一些问题及解决方案

<!-- more -->

> Based on Hexo & Volantis  

> 详细说明参考文档 [Hexo](https://hexo.io/zh-cn/docs/) & [Volantis](https://volantis.js.org/v6/getting-started/)


## 搭建环境

### 安装 Hexo
准备环境：
- nvm（参考 Node & Vue 文档）
- node.js（参考 Node & Vue 文档，版本 > 16.15.0）
- git

npm 安装 Hexo：
```bash
npm install -g hexo-cli
```

### 建立项目
在指定文件夹中新建项目
```bash
hexo init MyBlog
cd MyBlog
npm install 
```

#### _config.yml
主要配置文件，配置网站参数

#### scaffolds
模板文件，填写头部 front-matter 内容

#### source
源文件，包括 about、category 页面、images/ 等资源文件

图片资源引用：`/images/XXX`

### 部署到远程仓库
1. 安装模块
    ```bash
    npm install hexo-deployer-git --save
    ```

2. 修改 _config.yml
   ```yml
   deploy:
     type: git
     repository: git@github.com:CS0522/CS0522.github.io.git
     branch: master
    ```

3. 推送
   ```bash
   hexo clean && hexo g && hexo d
   ```

### 配置网站
仅列出我修改的地方
```yml
# 修改图标
favicon: /images/cicon.ico
# 网站标题
title: 诚实同学的博客
subtitle: ''
description: 'Blog, Computer Science, Learning, Life'
keywords: 
# 作者
author: Chen Shi
language: 
  - zh-CN
  - en
  - ja-JP
# 时区
timezone: 'Asia/Shanghai'

# 文章链接格式
permalink: :category/:title/
permalink_defaults:
  lang: zh-CN
  category: notes

# 文章内图片
# 直接用 ./XXX 相对路径引用
post_asset_folder: true
marked:
  enable: true
  prependRoot: true
  postAsset: true

# Category & Tag
default_category: notes
# 学习笔记 - notes，论文随记 - learnings，文档小记 - docs
category_map: 
  学习笔记: notes
  论文随记: learnings
  文档小记: docs
tag_map:

# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repository: git@github.com:CS0522/CS0522.github.io.git
  branch: master
```


### 常用命令
1. 建立
   ```bash
   hexo init <pro_name>
   cd <pro_name>
   npm install
   ```

2. 新建文章
   ```bash
   hexo new [page/post/docs] [-p about/me] "<file_name>"
   # hexo new page about/me "About Me"
   ```

3. 生成、预览
   ```bash
   hexo clean
   hexo g
   hexo s
   ```

4. 部署推送
   ```bash
   hexo d
   ```

## 网站主题

### 安装 Volantis
1. 修改主题 _config.yml
   ```yml
   theme: volantis
   ```

2. 安装 Volantis
   ```bash
   npm install hexo-theme-volantis --save
   ```

### 配置主题
1. 原主题配置文件位于 `./node_modules/hexo-theme-volantis/_config.yml` 下

2. 项目根目录创建 `_config.volantis.yml` 以自定义主题。其中的内容可以直接复制 1 中的文件内容，再按需修改

以下仅列出我修改的地方

#### 导航栏
<details>
<summary>点击查看</summary>

```yml
########## 导航栏 ########## > start
navbar:
  visiable: auto # always, auto
  logo: # choose [img] or [icon + title]
    img: '' # volantis-static/media/org.volantis/blog/Logo-NavBar@3x.png
    icon:
    title: '诚实同学<sup style="color:#ff9800">Blog</sup>'
  menu:
    - name: 首页
      icon: fa-solid fa-house
      url: /
    - name: 分类
      icon: fa-solid fa-folder-open
      url: categories/
    - name: 标签
      icon: fa-solid fa-tags
      url: tags/
    - name: 归档
      icon: fa-solid fa-archive
      url: archives/
    # - name: 友链
    #   icon: fa-solid fa-link
    #   url: friends/
    - name: 关于我
      icon: fa-solid fa-info-circle
      url: about/
    - name: 暗黑模式
      icon: fa-solid fa-moon
      toggle: darkmode
    # - name: 背景音乐
    #   icon: fa-solid fa-compact-disc
  search: 随便逛逛...   # Search bar placeholder
########## 导航栏 ########## > end
```
</details>


#### 封面
<details>
<summary>点击查看</summary>

```yml
########## 封面 ########## > start
cover:
  height_scheme: full # full, half
  layout_scheme: search # blank (留白), search (搜索), dock (坞), featured (精选), focus (焦点)
  title: '欢迎来到诚实同学的博客'
  subtitle: '"Patience is key in life."'
  # 修改背景（静态，无视差滚动）
  background: /images/91.jpg
  features:
    - name: 首页
      # img: volantis-static/media/twemoji/assets/svg/1f4f0.svg
      icon: fa-solid fa-home
      url: /
    - name: 分类
      # img: volantis-static/media/twemoji/assets/svg/1f516.svg
      icon: fa-solid fa-folder-open
      url: categories/
    - name: 标签
      # img: volantis-static/media/twemoji/assets/svg/1f4af.svg
      icon: fa-solid fa-tags
      url: tags/
    - name: 归档
      # img: volantis-static/media/twemoji/assets/svg/1f5c3.svg
      icon: fa-solid fa-archive
      url: archives/
    - name: 关于我
      # img: volantis-static/media/twemoji/assets/svg/1f9ec.svg
      icon: fa-solid fa-info-circle
      url: about/
########## 封面 ########## > end
```
</details>


#### 文章
<details>
<summary>点击查看</summary>

```yml
########## 文章 ########## > start
article:
  preview:
    scheme: landscape # landscape
    # pin icon for post
    pin_icon: # volantis-static/media/twemoji/assets/svg/1f4cc.svg
    # auto generate title if not exist
    auto_title: true # false, true
    auto_excerpt: true # false, true
    # hide excerpt
    hide_excerpt: false
    # show split line or not
    line_style: solid # hidden, solid, dashed, dotted
    # show author
    author: false # true, false
    # show readmore button
    readmore: auto # auto, always
  body:
    top_meta: [author, category, date, wordcount]
    bottom_meta: [updated, tags, counter, share]
    meta_library:
      author:
        # 个人头像
        avatar: /images/userphoto02.png  # volantis-static/media/org.volantis/blog/favicon/apple-touch-icon.png
        name: Chen Shi
        url: https://github.com/CS0522/
      date:
        icon: fa-solid fa-calendar-alt
        title: '发布于：'
        format: 'll' # 日期格式 http://momentjs.com/docs/
      updated:
        icon: fa-solid fa-edit
        title: '最后更新于：'
        format: 'll' # 日期格式 http://momentjs.com/docs/
      # 文章分类
      category:
        icon: fa-solid fa-folder-open
      # TODO 文章浏览计数
      counter:
        icon: fa-solid fa-eye
        unit: '次浏览'
      # 文章字数和阅读时长
      wordcount:
        icon_wordcount: fa-solid fa-keyboard
        icon_duration: fa-solid fa-hourglass-half
      # 文章标签
      tags:
        icon: fa-solid fa-hashtag
      # 分享
      share:
        - id: qq
          img: volantis-static/media/org.volantis/logo/128/qq.png #  https://cdn.jsdelivr.net/gh/volantis-x/cdn-org/logo/128/qq.png
        - id: qzone
          img: volantis-static/media/org.volantis/logo/128/qzone.png #  https://cdn.jsdelivr.net/gh/volantis-x/cdn-org/logo/128/qzone.png
        - id: weibo
          img: volantis-static/media/org.volantis/logo/128/weibo.png #  https://cdn.jsdelivr.net/gh/volantis-x/cdn-org/logo/128/weibo.png
        - id: # qrcode # 当id为qrcode时需要安装插件  npm i hexo-helper-qrcode
          img: # volantis-static/media/org.volantis/logo/128/wechat.png #  https://cdn.jsdelivr.net/gh/volantis-x/cdn-org/logo/128/wechat.png
        - id: # telegram
          img: # volantis-static/media/org.volantis/logo/128/telegram.png #  https://cdn.jsdelivr.net/gh/volantis-x/cdn-org/logo/128/telegram.png

########## 文章 ########## > end
```
</details>


#### 侧边栏
<details>
<summary>点击查看</summary>

```yml
########## 侧边栏 ########## > start
sidebar:
  for_page: [blogger, category, guide, tagcloud, webinfo]
  for_post: [toc, guide, tagcloud, webinfo]
  widget_library:
    toc:
      class: toc
      display: [desktop, mobile] # [desktop, mobile]
      header:
        icon: fa-solid fa-list
        title: 本文目录
      list_number: true
      min_depth: 2
      max_depth: 5

    # 功能导航
    guide:
      class: grid
      display: [desktop, mobile] # [desktop, mobile]
      header:
        icon: fa-solid fa-map-signs
        title: 功能导航
      fixed: true # 固定宽度
      rows:
        - name: 标签
          icon: fa-solid fa-tags
          url: tags/
        - name: 归档
          icon: fa-solid fa-archive
          url: archives/
        - name: 关于我
          icon: fa-solid fa-info-circle
          url: about/

    blogger:
      class: blogger
      display: [desktop, mobile] # [desktop, mobile]
      # 个人头像
      avatar: /images/userphoto02_scaled.png  # https://gcore.jsdelivr.net/gh/xaoxuu/cdn-assets/avatar/avatar.png
      title: Chen Shi
      subtitle: <p>DLUT DRISE</p><p>2024届本科生</p>
      jinrishici: false # Poetry Today. You can set a string, and it will be displayed when loading fails.
      social:
        - icon: fa-solid fa-envelope
          url: mailto:chenshi020522@outlook.com
        - icon: fa-brands fa-github
          url: https://github.com/CS0522/
        - icon: fa-brands fa-weixin
          url: /images/weixin.png
        - icon: fa-brands fa-qq
          url: /images/qq.jpg
        - icon: fa-brands fa-bilibili
          url: https://space.bilibili.com/37047123/

    # tagcloud widget
    tagcloud:
      class: tagcloud
      display: [desktop, mobile] # [desktop, mobile]
      header:
        icon: fa-solid fa-tags
        title: 标签
        url: tags/
      min_font: 14
      max_font: 24
      color: true
      start_color: '#999'
      end_color: '#555'

    # webinfo widget
    webinfo:
      class: webinfo
      display: [desktop, mobile]
      header:
        icon: fa-solid fa-award
        title: 站点信息
      type:
        article:
          enable: true
          text: '文章数目：'
          unit: '篇'
        runtime:
          enable: true
          data: '2023/11/01'    # 填写建站日期
          text: '已运行时间：'
          unit: '天'
        wordcount:
          enable: true
          text: '本站总字数：'   # 需要启用 wordcount
          unit: '字'
        visitcounter:
          siteuv:
            enable: true
            text: '本站访客数：'
            unit: '人'
          sitepv:
            enable: true
            text: '本站总访问量：'
            unit: '次'
        lastupd:
          enable: true
          friendlyShow: true    # 更友好的时间显示
          text: '最后活动时间：'
          unit: '日'
########## 侧边栏 ########## > end
```
</details>


#### 底部
<details>
<summary>点击查看</summary>

```yml
########### 底部 ########## > start
site_footer:
  layout: [social, source, analytics, copyright]
  copyright: 'Copyright © Since 2023 诚实同学'
  # 个人社交
  social:
    - icon: fa-solid fa-envelope
      url: mailto:chenshi020522@outlook.com
    - icon: fa-brands fa-github
      url: https://github.com/CS0522/
    - icon: fa-brands fa-weixin
      url: /images/weixin.png
    - icon: fa-brands fa-qq
      url: /images/qq.jpg
    - icon: fa-brands fa-bilibili
      url: https://space.bilibili.com/37047123/

########### 底部 ########## > end
```
</details>


#### 插件
我装的插件：
- hexo-deployer-git
- hexo-generator-json-content
- hexo-wordcount
- hexo-math
- hexo-renderer-marked

记得在配置文件中启用插件

<details>
<summary>点击查看</summary>

```yml
########### 插件 ########## > start
plugins:
  # 修改背景
  # 视差滚动效果 Slide Background
  parallax:
    enable: true
    position: cover       # cover: sticky on the cover.   fixed: Fixed as background for the site.
    shuffle: true         # shuffle playlist
    duration: 10000       # Duration (ms)
    fade: 1500            # fade duration (ms) (Not more than 1500)
    images:               # For personal use only. At your own risk if used for commercial purposes !!!
      - /images/91.jpg
      # - volantis-static/media/wallpaper/minimalist/2020/001.webp
      # - volantis-static/media/wallpaper/minimalist/2020/002.webp
      # - volantis-static/media/wallpaper/minimalist/2020/003.webp
      # - volantis-static/media/wallpaper/minimalist/2020/004.webp
      # - volantis-static/media/wallpaper/minimalist/2020/005.webp

  # 计数功能
  wordcount:
    enable: true
  # math expressions render
  mathjax:
    enable: true
    per_page: false
    cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
  # 搜索功能
  search:
    enable: true
    service: hexo
  
  darkmode:
    enable: true
########### 插件 ########## > end
```
</details>


## 网站页面
### 归档页面
自动生成

### 关于页面
<details>
<summary>source/about/index.md</summary>

```md 
---
layout: docs
seo_title: 关于
bottom_meta: false
sidebar: []
twikoo:
  placeholder: 有什么想对我说的呢？
---

下面写关于自己的内容
```
</details>


### 分类页面
<details>
<summary>source/categories/index.md</summary>

```md
---
layout: category
index: true
title: 所有分类
---
```
</details>


### 标签页面
<details>
<summary>source/tags/index.md</summary>

```md
---
layout: tag
index: true
title: 所有标签
---
```
</details>


### 列表页面
<details>
<summary>source/mylist/index.md</summary>

```md
---
layout: list
group: mylist
index: true
---
```
</details>


### 404页面
<details>
<summary>source/404.md</summary>

```md
---
cover: true
robots: noindex,nofollow
sitemap: false
seo_title: 404 Not Found
bottom_meta: false
sidebar: []
twikoo:
  path: /404.html
  placeholder: 请留言告诉我您要访问哪个页面找不到了
---

{% p logo center huge, 404 %}
{% p center bold, 很抱歉，您访问的页面不存在 %}
{% p center small, 可能是输入地址有误或该地址已被删除 %}
```
</details>


## 问题与解答

### 1. 站内文章跳转？
> 参考[官方文档](https://hexo.io/zh-cn/docs/tag-plugins#%E5%BC%95%E7%94%A8%E6%96%87%E7%AB%A0)

```md
{% post_path filename %}
{% post_link filename [title] [escape] %}
```
* `escape` 用于防止特殊字符转义

例子：

* 链接使用文章的标题
  ```md
  {% post_link hexo-3-8-released %}
  ```
  Hexo 3.8.0 Released

* 链接使用自定义文字
  ```md
  {% post_link hexo-3-8-released '通往文章的链接' %}
  ```
  通往文章的链接

* 对标题的特殊字符进行转义
  ```md
  {% post_link hexo-4-released 'How to use <b> tag in title' %}
  ```
  How to use \<b> tag in title

* 禁止对标题的特殊字符进行转义
  ```md
  {% post_link hexo-4-released '<b>bold</b> custom title' false %}
  ```
  <b>bold</b> custom title


### 2. 数学公式渲染？
> 更换渲染引擎

```bash
npm uninstall hexo-renderer-marked
npm uninstall hexo-math

# pandoc
npm install hexo-renderer-pandoc
# 这个插件必须要
npm install hexo-filter-mathjax

sudo apt install pandoc
```