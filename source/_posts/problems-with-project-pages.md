---
title: 利用GitHub项目主页实现多个独立博客
reward: true
tag:
  - GitHub Pages
  - hexo
abbrlink: 2225712445
date: 2019-05-26 00:00:00
---

之前用github page给自己搭建了一个技术博客，年后想给老爸也搞个博客，记录下他的诗歌，作为生日礼物。

现将这个摸索的过程记下。

首先明确下，一个github账户是可以实现管理多个独立博客的。这主要是利用了“项目主页”。
<!--More-->
# 个人/组织主页和项目主页的区别

区别见: https://help.github.com/en/articles/user-organization-and-project-pages

总结一下，github的目的是，个人/组织主页用于展示个人/组织的整体情况(于个人来讲可能是技术博客等)，而项目主页用于展示前者提到的细节、实现等(比如某一篇博客中实验的全部代码、数据集等不适合放在博客中的内容)。

前者的访问链接: <username/orgname>.github.io  
后者的访问链接: <username/orgname>.github.io/projectname  

在项目主页的目录下，同样利用hexo搭建一个博客，就可以实现多个独立的博客了，只不过链接后缀多了个名字，也可以用于区分博客，无伤大雅。

# 创建项目主页repository

> 由于github页面也会更新升级，所以你看到的页面可能和下面的图片不太一样，不过大体上应该是相同的(本篇博客写于2019-05-26)。

## 1. 创建新repository  
1. 点击 **new repository**
![点击 new repository](/img/blog/problems-with-project-pages/create_new_repo.png)

2. 输入repository名，点击 **create reposityory**，这里将repo命名为demo
![点击 create reposityory](/img/blog/problems-with-project-pages/input-repo-name.png)

## 2. 更改repository为github Pages

1. 点击 **setting**
![点击 setting](/img/blog/problems-with-project-pages/setting.png)

2. 下拉找到 **GitHub Pages选项卡**
![点击 GitHub Pages选项卡](/img/blog/problems-with-project-pages/hint-for-add-content.png)
我们找GitHub Pages选项卡，是因为需要将这个普通repo转换成github pages才能实现博客功能。  
红框中提示我们目前github page是禁用的，需要先添加一个内容到repo，才能将此项目作为github pages站点(也就是博客)使用。
> 之前版本的github是没有这步的，黄框中内容都可以直接选择，但是当前版本是置灰无法选择的。

3. 新建文件
![新建文件](/img/blog/problems-with-project-pages/create-new-file.png)

4. 写入内容
![写入内容](/img/blog/problems-with-project-pages/hello.png)
页面拉到最下方输入commit信息提交。

4. 重新找到 **GitHub Pages选项卡**，红框中内容已改变
![选择数据源](/img/blog/problems-with-project-pages/select-source.png)

5. 点击 **master branch**，页面稍后自动刷新，红框中提示站点已经发布
![选择数据源](/img/blog/problems-with-project-pages/refresh.png)

5. 访问 https://jiahe404.github.io/demo/hello.html，显示内容！
![选择数据源](/img/blog/problems-with-project-pages/access-page.png)
> 这一步通常要过一段时间才能生效，如果提示404，建议换个浏览器或稍等一段时间再试。

至此我们已经成功创建了一个项目主页，接下来结合hexo将此项目主页转化为一个博客！

# 搭建hexo博客

利用项目主页搭建博客，与个人主页相比是类似的，跳过环境问题。

## 1. 在本地初始化hexo博客
```shell
cd your/blog/dir
hexo init
hexo g
hexo s -p 8008
```
浏览器中看到熟悉的默认landscape主题首页:
![landscape](/img/blog/problems-with-project-pages/landscape.png)

## 2. 发布到github项目主页

1. 更改站点目录下(非theme目录)_config.yml，重点！  
>注意！**由于使用了项目主页，链接后缀需要加上项目名称，所以url和root的配置中也要加入项目名称，本例中为demo**；而deploy中的配置项，则和个人主页一致

```
# vim _config.yml
...

url: http://yoursite.com/demo
root: /demo

...

deploy:
  type: git
  repository: https://github.com/jiahe404/demo.git
  branch: master

...
```

2. 发布

```shell
hexo clean
hexo g
hexo d
```
稍等几分钟后访问: https://jiahe404.github.io/demo/， 同样熟悉的landscape2：
![landscape](/img/blog/problems-with-project-pages/landscape2.png)

注意到目前为止**我并没有创建其他分支，如gh-pages等**，一些博客资料上写个人主页只能发布master分支内容(应该是正确的)，而项目主页只能发布gh-pages分支内容(我在[官方文档](https://help.github.com/en/articles/configuring-a-publishing-source-for-github-pages)上并未找到依据)。

下面是官方文档的说明，只支持前半句：
>If your site is a User or Organization Page that has a repository named <username>.github.ioor <orgname>.github.io, you cannot publish your site's source files from different locations. User and Organization Pages that have this type of repository name are only published from the master branch.


接下来就可以开始搞第3个博客了！

接下来就可以开始搞第4个博客了！

接下来就可以开始搞第N个博客了！

没错，项目主页可以创建多个，而个人主页只能有一个，这也是二者的区别之一。

下一篇文章打算延续这个demo，写下更改主题的细节。
