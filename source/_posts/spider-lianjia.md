---
title: 链家小区均价数据爬取
brief: 链家小区均价数据爬取
tag:
  - 爬虫
categories:
  - 爬虫
abbrlink: 2739431502
date: 2018-09-24 00:00:00
---

最近有个需求，需要了解市场上小区的均价，于是试着写了个爬虫把链家上小区的信息爬取了一下。这次的爬虫任务比较简单，数据量不大，爬取链接中也没加密字段等反爬取策略，感觉还挺适合作为爬虫入门的例子。下载数据主要利用python requests库来完成，解析html页面用到xpath，下面分享些技巧和经验。
<!--More-->
>1. 开发环境:  
python3.6  
pip install requests  
pip install lxml
2. 浏览器:  
Chrome

## 1. 网站分析
链家网上有小区专门的页面，域名格式为: 城市代码.lianjia.com/xiaoqu/城区代码/页码，期中页码格式为pg1、pg2这样，如果不加页码，则默认显示第1页数据。比如下面显示北京西城区的小区情况，可以看到有1716个小区，小区的名字、均价等信息都有。页面下方有页码。通过更改城市代码、城区名以及页码，就可以获取对应的小区信息。  
<img src="/img/blog/spider-lianjia/zone1.jpg" width="100%" height="50%">
如果想要下载全国城市小区价格信息，大致分为3个步骤:
1. 获取城市代码
2. 获取每个城市的城区代码
3. 获取某城区所有页码数，拼接url, 下载具体某个城市的某城区信息

下面按照3, 2, 1的顺序讲解，先把一个城区的数据搞下来，再想办法扩展到一个城市和全国范围。

## 2. 下载城区数据
### 1. 定位数据位置
浏览器选择Chrome，打开Chrome开发者工具(在页面上单击右键选择**检查**菜单)，依次点击**Network**栏，**Doc**栏，会显示本次网络请求到的页面，点击左下**Name**窗口中的条目，即可在右侧窗口中看到此页面的相关内容，如Headers(请求头)、Preview(预览)、Response(返回内容)、Cookies、Timing。点击**Preview**可以看到所需要的数据就包含在一个静态页面中，所以只要把这个页面下载下来，就可以提取其中的数据了。
<img src="/img/blog/spider-lianjia/zone2.jpg" width="100%" height="50%">
> 爬虫任务的第一步就是要明确自己需要的数据在哪里。通常在页面上看到的早已渲染好的数据在后端会以两种方式传递给前端:
1. 一种即是本例中，数据是早已嵌在页面中。通常这种情况数据在**Doc**中可以看到；
2. 另一种情况中，页面结构和数据是分离的，数据通常以json格式通过ajax异步请求返回给客户端，再通过js程序加载到页面结构中。这种情况下数据可以在**XHR**栏看到。

> 想要定位自己所需的数据在哪，需要借助浏览器的开发者工具，分析一次请求的过程都发生了什么，先看看**Doc**、**XHR**栏里的内容，依次点点每个请求内容(**Name**栏中的条目)，其中**Doc**栏中的内容在预览情况下比较容易看，就是一个页面，而**XHR**栏下的通常是json格式的数据，就需要好好和页面上渲染好的数据作比较了。
### 2. 下载数据
接下来我们就可以开始写代码爬数据了。
首先定义一个Spider类:
```
class Spider(object):

    def __init__(self, city):
        self.city = city
        self.domain = 'https://%s.lianjia.com' % city
```
构造方法接收一个城市代码，在本例中即是**bj**，构造方法中为成员变量domain赋值。  
接下来定义一个download_page方法，用于下载一个页面。
```
def download_page(self, url, file_out):
    res = requests.get(url)
    content = res.text
    with open(file_out, 'w') as fo:
        fo.write(content)
```
注意参数url格式为self.domain/xiaoqu/城区代码/页码。  
编写一个main方法运行试验下，这里先将城区和页码固定:
```
def main():
    city = 'bj'
    spider = Spider(city)
    page_file = 'test.html'
    residence = 'xicheng'
    page = 'pg2'
    url = '%s/xiaoqu/%s/%s' % (spider.domain, residence, page)
    spider.download_page(url, page_file)
```
运行后可以看到当前目录下多了一个test.html
> 将每次网络请求的数据保存下来是个好习惯，有以下好处:
1. 后续开发解析模块可以利用这些文件，不必重新请求数据，减少时延，节省时间，同时也能减少请求次数，防止被ban;
2. 方便排查问题，如果程序上线了结果解析出错，可以查看文件内容，看看是网站改版了还是请求的页面错误等。
## 3. 解析数据
### 1. 找到节点xpath  
解析用到xpath，开发者工具对xpath支持很好。先看下面的例子:
<img src="/img/blog/spider-lianjia/zone3.jpg" width="100%" height="50%">
按照步骤操作后可以获得一个节点的xpath。在上例中，可以获得价格的xpath:
```
'/html/body/div[4]/div[1]/ul/li[1]/div[2]/div[1]/div[1]/span'
```
通过开发者工具的**Console**栏可以方便地验证xpath:
<img src="/img/blog/spider-lianjia/zone4.jpg" width="100%" height="50%">
在Console栏中的命令行输入$x()方法，将上面复制的xpath粘贴进去，$x()方法接收xpath路径作为参数，返回对应的节点列表，通过[0]选择第1个节点，节点的innterText变量保存了对应的内容。通过更改xpath，就可以获取各个小区的名字和价格等信息了。
### 2. 优化xpath
上面通过开发者工具获取的xpath是一条绝对路径，可以看到路径上有很多节点，如果以后网站改版了，路径上的某个节点改变了，那这条xpath就不能指向正确的位置了。所以通常可以利用节点的相对位置和属性等来写出更友好的xpath，比如我将上面4条xpath改写一下:
<img src="/img/blog/spider-lianjia/zone5.jpg" width="100%" height="50%">
同样获取到了需要的数据，而且此时看xpath也容易理解，比如第1条取小区名，是取class="listContent"的ul下，第1个li下，class="title"的div下，a标签的内容；如果想要取价格，就把div的class改为totalPrice，a标签换成span。这比绝对路径上一堆节点好理解多了。
```
$x('//ul[@class="listContent"]/li[1]//div[@class="title"]/a')[0].innerText
$x('//ul[@class="listContent"]/li[1]//div[@class="totalPrice"]/span')[0].innerText
$x('//ul[@class="listContent"]/li[2]//div[@class="title"]/a')[0].innerText
$x('//ul[@class="listContent"]/li[2]//div[@class="totalPrice"]/span')[0].innerText
```
>Chrome的命令行提供了很多方便的工具，$x()即是其一，利用好了事半功倍，更多知识请参见[Chrome命令行](https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference?hl=zh-cn)。
### 3. python代码
在Spider类中添加parse方法:
```
def parse(self, page_file):
    with open(page_file) as f:
        content = f.read()
        selector = etree.HTML(content)
        # 获取li数量
        path = '//ul[@class="listContent"]/li'
        size = len(selector.xpath(path))
        # 遍历li节点
        for i in range(1, size + 1):
            li_path = '//ul[@class="listContent"]/li[%s]' % i
            # 获取小区名
            path = li_path + '//div[@class="title"]/a'
            name = selector.xpath(path)[0].text
            # 获取价格
            path = li_path + '//div[@class="totalPrice"]/span'
            price = selector.xpath(path)[0].text
            print(name, price)
```
运行试验下:
```
def main():
    city = 'bj'
    spider = Spider(city)
    page_file = 'test.html'
    residence = 'xicheng'
    page = 'pg2'
    url = '%s/xiaoqu/%s/%s' % (spider.domain, residence, page)
    # spider.download_page(url, page_file)
    spider.parse(page_file)
```
输出:
```
锦官苑 139610
马甸南村 140382
玺源台 99726
黄寺大街24号院 134787
新街口西里三区 111055
...
```
需要的数据就都有了！
## 3. 其他工作
至于如何按页码、按城区、按城市下载全国数据，代码都已经都实现了，也不难，有时间下回分解，中秋快乐！
