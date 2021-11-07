# Hexo 部署博客至 Github
[Hexo](https://hexo.io/zh-cn/docs/#%E5%AE%89%E8%A3%85) 是一个容易上手的**博客框架**。我们使用 Markdown 写博文，Hexo 负责将文档渲染成网页(Hexo 使用 Markdown 渲染引擎来完成这项工作)，于是我们便有了一个静态博客网站，接着我们可以将这个静态网站部署到 Github 仓库，并利用 Github Page 非常方便的将这个网站在 Github 的服务器上部署上线，至此，我们便可以在浏览器中访问我们写的博客了。 

#### 1.安装 Hexo

```cmd
npm install -g hexo-cli
```

安装前提：`Node.js` 和 `Git`

版本说明：Hexo 5.0  、  Node.js  14

#### 2.创建网站项目

```cmd
hexo init blog # blog 为网站项目目录
cd blog
npm install
```

执行：

```cmd
hexo s # hexo server 的缩写
```

访问 `localhost:4000` 即可看到初始页面，默认主题为 landscrape：

![hexo init page](https://image.xiuwujinda.cn//2021/11/05/hexo-init.png)

在 `/_config.yml` 配置站点名称、作者等信息：

```yml
# Site
title: 站点名称
subtitle: ''
description: ''
keywords:
author: 
language: zh-CN
timezone: ''
```

以及，设置跳过渲染 `/source` 目录下的 `README.md` 文件：

```yml
skip_render: README.md
```

#### 3.更换主题为 Next theme

##### 3.1 安装

Hexo 5.0 之后的版本，可以通过 `npm` 来安装 `Next` 主题：

```cmd
npm install hexo-theme-next
```

这种方式，`theme-Next` 是作为一个依赖保存至 `node_modules` 目录下，而通过 `git` 安装是将 `theme-Next` 复制到了 `theme` 目录下。

##### 3.2 启用Next主题

在项目根目录的 `_config.yml`中设置：

```yml
theme: next
```

再次运行：

```cmd
hexo s
```

可以看到 Next 主题已经生效：

![hexo-theme-next](https://image.xiuwujinda.cn//2021/11/05/hexo-theme-next.png)

##### 3.3 配置 Next 主题

1. 在根目录创建 `_config.theme.yml` 文件

2. 复制默认配置到上述文件中：

   ```cmd
   cp node_modules/hexo-theme-next/_config.yml _config.next.yml
   ```

3. 参考 [theme settings](https://theme-next.js.org/docs/theme-settings/) ，在 `./_config.next.yml` 修改 next 主题默认设置， 如：

   ```yml
   # 自定义站点图标，图片应位于 /source/images 目录下：
   favicon:
     small: /images/favicon-16x16-next-leaf.png
     medium: /images/favicon-32x32-next-leaf.png
     apple_touch_icon: /images/apple-touch-icon-next-leaf.png
     safari_pinned_tab: /images/logo-leaf.svg
     
   # 在目录中开启分类、标签、归档等选项
   menu:
     home: / || fa fa-home
     archives: /archives/ || fa fa-archive
     categories: /categories/ || fa fa-th
     tags: /tags/ || fa fa-tags
     
   # 自定义侧边栏的头像
   avatar:
     # Replace the default image and set the url here.
     url: /images/avatar.jpg
     # If true, the avatar will be displayed in circle.
     rounded: true
     # If true, the avatar will be rotated with the cursor.
     rotated: false
     
   # 使侧边栏显示 Posts / Categories / Tags 
   site_state: true
   
   # 代码块样式
   codeblock:
     # Code Highlight theme
     # All available themes: https://theme-next.js.org/highlight/
     theme:
       light: tomorrow-night-eighties
       dark: tomorrow-night
     prism:
       light: prism
       dark: prism-dark
     # Add copy button on codeblock
     copy_button:
       enable: true
       # Available values: default | flat | mac
       style: mac
   ```

#### 4.部署至 Github

##### 4.1 在 `./_config.yml` 配置 `git` 仓库

```yml
# Deployment
deploy:
  type: 'git'
  repository: https://github.com/User/user.github.io
  branch: master
```

**注意：**仓库名须为 `<username>.github.io` 才可以通过 `http(s)://<username>.github.io` 访问

##### 4.2 下载 deploy 命令

```cmd
npm install hexo-deployer-git --save
```

##### 4.3 使用自定义域名

在 `/source` 目录下创建文件 `CNAME`，内容为域名：

```tex
example.com
```

参考 [Manage A Custom Domain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain) 设置域名解析到 Github 的服务器 ip。

##### 4.4 部署

```cmd
hexo clean && hexo d
```

该命令将 `/public` 文件夹下的内容推送至 `<user>.github.io` 仓库，

同时，可以选择将博客项目推送至另一个仓库。

#### 5. 常用命令

##### 5.1 生成一篇新的文章

```cmd
hexo new post 'post name'
```

##### 5.2 渲染生成静态网站

```cmd
hexo g # hexo generate 缩写
```

##### 5.3 本地访问

```cmd
hexo s # hexo server 缩写
```

##### 5.4 更新文章到 Github

```cmd
hexo clean && hexo d
```

##### 5.5 生成新的页面

```cmd
hexo new page categories # 生成分类页面
hexo new page tags # 生成标签页面

hexo new page custom # 生成任意页面，在 source 目录下编辑对应的 custom.md 的内容
```



