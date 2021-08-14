---
title: hexo+github+Next搭建博客
date: 2021-08-14 11:04:52
tags: hexo github Next
---

# 环境安装

根据官网hexo框架安装指引，依赖：Node.js和git。笔者安装了V14.17.4版本。

安装hexo:

```bash
npm install hexo-cli -g
```

# 初始化项目

取项目名称为localhexo进行初始化，命令如下：

```bash
hexo init localhexo
```

初始化创建完成之后，在localhexo目录下面会有source、themes等目录。

进入到localhexo目录中生成静态HMTL页面，命令如下：

```bash
hexo generate
```

在public目录下面会生成js、css等目录。

启动服务：

```bash
hexo server
```

![hexoserver](/img/build-blog-jpg/hexoserver.jpg)

根据提示，在本地4000端口提供了访问。浏览器输入:

```bash
http://localhost:4000
```

可以看到页面信息了：

![hexmainpage](/img/build-blog-jpg/hexmainpage.jpg)

安装、部署、启动过程非常简洁。^_^

# Github页面创建和部署

首先在github上面创建仓库，名称为：username.github.io。

然后修改localhexo目录下_config.yml配置文件：

```bash
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: 'git'
  repo: https://github.com/kuangtu/kuangtu.github.io
  branch: master
```

## 部署

 本地配置完成之后，执行命令部署：

```bash
hexo deploy
```

显示信息：

![hexdeploy_err](/img/build-blog-jpg/hexdeploy_err.jpg)



需要安装git部署插件，命令如下：

```bash
npm install hexo-deployer-git --save
```

安装完成之后，执行部署，会弹出登录框录入github账户和密码：

![githublogin](/img/build-blog-jpg/githublogin.jpg)



<u>然后访问kuangtu.github.io出现了404，因为之前临时创建过kuangtu.github.io仓库，并绑定过自己的域名。</u>

<u>因此增加CNAME文件配置了域名。可以访问了。</u>

![hexo_zt1](/img/build-blog-jpg/hexo_zt1.jpg)



<u>404的情况比较常见，可以先在本地```hexo server```运行是否正常。</u> 



## 博客源代码上传

通过```hexo deploy```部署之后，实际是将本地public目录下面的内容push到github。

如果需要将本地源代码等都上传到github，可以创建分支然后push上去。

虽然```hexo deploy``` 可以部署到github上面，但本地目录不是git仓库，执行命令：

![notgitrepo](/img/build-blog-jpg/notgitrepo.jpg)

可以初始化git仓库，创建分支，推送到github。

```bash
git init
git checkout -b source
git add -A
git commit -m "create blog"
```

然后推送到github站点：

```bash
git remote add origin git@github.com:kuangtu/kuangtu.github.io.git
git push origin source
```

通过github网站可以看到多了source分支，包含了源代码：

![source分支](/img/build-blog-jpg/source分支.jpg)



# 主题修改

hexo有非常多的主题，可以下载配置不同的主题类型。比较流行的有Next主题，可以通过github下载之后进行替换。

```bash
git clone https://github.com/theme-next/hexo-theme-next.git themes/next
```

本地themes目录下多了next主题。修改本地的_config.yml：

```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

然后重新生成页面并启动服务，访问本地连接：

![nexttheme](/img/build-blog-jpg/nexttheme.jpg)



主题已经替换了。

# 站点配置

本地目录下的_config.yml为站点配置目录，各主题下面的_config.yml为主题配置。

先对于站点进行修改。

```yaml
# Site
title: 莫大的博客
subtitle: ''
description: '证券、金融'
keywords:
author: 莫大
language: zh
timezone: ''
```

此时访问页面看到已经做了更新：

![site_conf](/img/build-blog-jpg/site_conf.jpg)





# 写博客

hexo使用了markdown格式编写博客。md文件放置在本地的```source/_posts```目录中。



# 参考

[hexo官网](https://hexo.io/docs/)

[利用 GitHub + Hexo + Next 从零搭建一个博客](https://cuiqingcai.com/7625.html)

[Hexo + GitHub Pages 搭建个人博客及 NexT 主题配置](https://zhuanlan.zhihu.com/p/96641789)



