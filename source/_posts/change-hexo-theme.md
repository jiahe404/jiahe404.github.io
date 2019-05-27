---
title: 优雅地更改hexo主题
tag:
  - GitHub Pages
  - hexo
  - git submodule
abbrlink: 1054489955
date: 2019-05-26 14:49:05
categories:
  - 博客搭建
---

本文延续《利用GitHub项目主页实现多个独立博客》，讲下更新默认主题的一些问题。

我自己的技术博客采用nexT主题，不过给老爸放诗歌这个主题就不太适合了，看了下[官网主题列表](https://hexo.io/themes/)，最终选择了[hueman](https://blog.zhangruipeng.me/hexo-theme-hueman/)这款。主要考虑到这个主题的布局、文章缩略图、画廊功能，很适合写日记。
<!--More-->
[hueman github链接](https://github.com/ppoffice/hexo-theme-hueman)

# 更换主题

## 1. 下载主题并配置:

```shell
cd your/blog/dir
# 错误的做法，先不要这么干！
git clone https://github.com/ppoffice/hexo-theme-hueman.git themes/hueman
mv themes/hueman/_config.yml.example themes/hueman/_config.yml
vi _config.yml
# 将站点根目录下的_config.yml(非themes/hueman/_config.yml)中theme更改为hueman
hexo clean
hexo g
hexo s -p 8008
```

## 2. 打开浏览器，不一样的hueman:

![hueman主题](/img/blog/change-hexo-theme/hueman.png)

# 保存markdown文档到hexo分支

做到这里，我想按照[《在多台电脑上写GitHub Pages博客》](https://jiahe404.github.io/3974204864.html)方法，将markdown文档、图片等保存到repo的hexo分支上，但却出现了一些问题。

## 1. 初始化仓库
```shell
cd your/blog/dir
# 将当前目录初始为git仓库
git init
# 添加远端仓库
git remote add origin git@github.com:jiahe404/demo.git
# 建立本地分支hexo
git checkout -b hexo
```

## 2. 添加文件时报错
```shell
git add ./

output:
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)
  (commit or discard the untracked or modified content in submodules)

        modified:   themes/hueman (modified content)
```

提示说themes/hueman是一个子git仓库模块，并且里面有更改。

原因是我在上面运行过:
```
mv themes/hueman/_config.yml.example themes/hueman/_config.yml
```

我的博客是一个git仓库，而里面用到的theme也是一个独立的git仓库，这就会引发一些冲突。如何优雅地处理这个问题呢，自然想到用git submodule。
> 这里面最粗暴的解决方法，就是将themes/hueman/.git等目录文件删掉，将其作为普通文件由博客的git仓库统一管理，不过后续再想更新这个主题就麻烦了。我之前应该就是这样做的，这次不想用这种ugly方式了。

如果现在用 git submodule 命令会提示错误(所以如果不处理这个问题，将无法使用git submodule):
```shell
git submodule
# output:
# fatal: no submodule mapping found in .gitmodules for path 'themes/hueman'
```

## 3. 正确的做法

1. 彻底删掉git clone过来的主题仓库，见[delete submodule](https://gist.github.com/myusuf3/7f645819ded92bda6677)
```shell
git rm --cached themes/hueman -f
vi .gitmodules
# 删掉对应代码块
vi .git/config
# 删掉对应代码块
```

2. 利用git submodule添加主题
```shell
git submodule add https://github.com/ppoffice/hexo-theme-hueman.git themes/hueman
git submodule
# output显示正确子模块信息:
# 5bf00fd62e5c79c430553bde4142dbde314ab3db themes/hueman (v0.4.0-20-g5bf00fd)
mv themes/hueman/_config.yml.example themes/hueman/_config.yml
```

3. 将theme目录下_config.yml加入代码库(换了台电脑还是需要这个文件的)
```shell
cd themes/hueman/
vi _config.yml
# 删掉_config.yml
git add ./
git commit -m "Save _config.yml for hueman theme."
cd -
# 在submodule目录中commit后，回到外层仓库仍需要add、commit
git add themes/hueman
git commit -m "Modify hueman config."
git push origin hexo
```

之前急于搭建一个博客，没有关注恰当的方法，现在补上，同时复习下git submodule相关命令，感觉一切都回到了正轨。
