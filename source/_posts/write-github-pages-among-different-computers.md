---
title: 在多台电脑上写GitHub Pages博客
reward: true
tag:
  - GitHub Pages
  - hexo
abbrlink: 3974204864
date: 2018-09-03 00:00:00
---

之前在公司的mac上写过GitHub Pages，最近离职了，于是想用自己的windows本继续写。本以为安装好nodejs，npm等环境再git pull一下就可以在windows本上写博客，实践了才发现一些问题。git pull下来的内容根本没办法直接在本地显示博客内容，因为少了很多配置等文件。下图分别是mac上(已经拷贝到windows本)可以运行的博客和在windows本上pull下来的文件：
<!-- more -->
- mac上可以运行的博客

<img src="/img/blog/write-github-pages-among-different-computers/mac_hexo.jpg" width="70%" height="50%">
- git上pull下来的文件

<img src="/img/blog/write-github-pages-among-different-computers/git_hexo.jpg" width="70%" height="50%">

可以看到在git上的内容少了几个目录，如node_modules(node.js执行需要的库)、scaffolds(生成md文件时用到的模板，可自定义: hexo new [post/page/draft] <title>)、source(自己写的md文件)、themes(主题)，以及package.json、package-lock.json等文件。

之前对node.js，hexo这套东西也不太了解，通过这次折腾算是多了些理解。GitHub Pages保存的内容顾名思义，就是一些html页面，以及支持这些页面显示的字体(fonts)、图片(img), 标签(tags)、js、css等，在执行hexo d命令时，以上的内容会提交到git仓库的master分支上；
而支撑生成这些静态页面的md文件(scaffolds、source)，主题(themes)，以及在本地调试用到的node.js环境(node_modules、package.json、package-lock.json)则不会被提交。

知道了这些就可以利用git的分支解决这个问题了：
1. 支撑生成这些静态页面的md文件(scaffolds、source)，主题(themes)等是需要保存的，可以提交到新的分支上；
2. 本地调试用到的node.js环境(node_modules)就不必提交了，可能各个电脑安装的版本也不一样，需要时 npm install 一条命令就生成了。
> 由于我们是从已有博客恢复环境，而不是从头开始创建环境，所以 <strong>hexo init</strong> 命令不需要执行，此命令会初始化hexo所需环境及基本配置，下图即是运行后生成的目录结构。但以下目录及文件都是已有的(在你的另一台电脑的硬盘上，而不是git仓库)，可以直接拷贝过来。但是我建议node_modulds就别拷贝了，可能不同电脑上版本不同。但是package.json、package-lock.json是需要提交的，其中保存了hexo及package的版本等信息，执行<strong>npm install</strong> 需要这两个文件。  
这里要注意执行<strong>npm install</strong> 后，由于两台电脑上hexo及package版本不一致，这两个文件也可能会被改写，所以如果需要频繁地在多台电脑上写博客，这两个文件可以不再提交。
<img src="/img/blog/write-github-pages-among-different-computers/hexo_init.jpg" width="90%" height="50%">

```
# 创建新分支保存md文件及主题等
git checkout -b hexo
# 编辑.gitignore文件，在新分支上忽略node_modules等
vi .gitignore
# 提交需要保存的文件
git add ... git commit ... git push origin hexo
```

以后换了新电脑，执行下面的命令：
```
# 克隆代码库
git clone your/github/repo dir/for/blog
# 切换分支
cd dir/for/blog
git checkout hexo
# 安装node.js环境
npm install
# 测试
hexo g
hexo s
```

以后写md、调主题就在hexo分支上进行，记得最后都<strong>git add ... git commit ... git push origin hexo</strong>，以便保存到git仓库。在hexo分支上运行<strong>hexo d</strong>就可以，实际上是将<strong>hexo g</strong>生成的静态pages推送到主分支上(这些操作都由 hexo-deployer-git工具完成，对我们来讲是透明的)。
