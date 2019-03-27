---
title: python数据分析之dataframe VS sql
tag:
  - python
  - pandas
  - dataframe
  - sql
abbrlink: 3393309510
date: 2018-11-22 00:00:00
---

在分析数据时，dataframe的很多方法和sql是类似的，本文总结一些二者中的相通问题，方便互相转移，下面以mysql语法为例。
<!--More-->
# 数据准备
为了同时使用sql和dataframe进行分析，分别准备数据库和文本文件(下面的sql和python代码都可以直接运行)。

## 1. mysql表
假如有一个table，包含一些影视音乐作品，数据分4列，分别为一、二级分类，作品名，评分:

```sql
create table multi_category_data (
    cat1 varchar(8),
    cat2 varchar(8),
    name varchar(8),
    score int
)engine=myisam default charset=utf8;

insert into multi_category_data values('影视', '电影', 'a', 5);
insert into multi_category_data values('影视', '连续剧', 'b', 6);
insert into multi_category_data values('影视', '连续剧', 'c', 7);
insert into multi_category_data values('音乐', '流行', 'd', 10);
insert into multi_category_data values('音乐', '流行', 'e', 8);
insert into multi_category_data values('音乐', '民谣', 'f', 9);
insert into multi_category_data values('音乐', '摇滚', 'g', 4);
insert into multi_category_data values('音乐', '摇滚', 'h', 4);
```

## 2. 文本文件
为了用dataframe分析，将同样的数据保存在csv_file文件中，并以dataframe格式保存到变量data:
```python
multi_category_data = """
cat1,cat2,name,score
影视,电影,a,5
影视,连续剧,b,6
影视,连续剧,c,7
音乐,流行,d,10
音乐,流行,e,8
音乐,民谣,f,9
音乐,摇滚,g,4
"""
csv_file = io.StringIO(multi_category_data)
data = pd.read_csv(csv_file, sep=',', names=None)
```

# 分析实战
## 1. 去重问题 —— drop_duplicates() VS distinct
drop_duplicates()方法可以对数据按列名去重，类似于sql中的distinct。  

现在我想了解作品都有哪些一、二级分类，可以如下实现:

- linux sort(一句话的事，但不是本文的重点)
```shell
sort -t , -k1,1 multi_category_data
```

- sql

```sql
select
    distinct cat1, cat2
from
    multi_category_data;
# 或
select
    cat1, cat2
from
    multi_category_data
group by
    cat1, cat2;
```

- dataframe
```python
data[['cat1', 'cat2']].drop_duplicates()
# data[['cat1', 'cat2']].drop_duplicates(keep='first')
# data[['cat1', 'cat2']].drop_duplicates(keep='last')
```
输出:  
![](/img/blog/df/distinct1.png)

drop_duplicates的参数keep指定在数据重复时保留首行或末行，可以通过第1列行号来区别保留的哪一行，如默认保留首行，所以我们看到有重复行的[影视,连续剧]、[音乐,流行]前的行号分别为1和3，对应原始数据其首次出现的行号。

## 2. 分组问题
### 1. 对单列或多列执行相同的聚合操作
比如想看看各个二级类目下都有多少作品:
- sql
```sql
select
    cat1, cat2, count(name)
from
    multi_category_data
group by
    cat1, cat2
```
- dataframe
```python
data.groupby(['cat1', 'cat2'])[['name']].count()
```
输出:  
![](/img/blog/df/group1.png)

更多例子:
```python
data.groupby(['cat1', 'cat2'])[['score']].max()
data.groupby(['cat1', 'cat2'])[['score']].min()
data.groupby(['cat1', 'cat2'])[['score']].mean()
data.groupby(['cat1', 'cat2'])[['score']].sum()
data.groupby(['cat1', 'cat2'])[['name', 'score']].max()
```

### 2. 对多列分别执行不同的聚合操作
比如想看看所有二级分类下作品分值情况，如平均值、极值等，这里主要借助numpy的内置方法:
- sql
```sql
select
    cat1, cat2,
    count(name), avg(score), sum(score), min(score), max(score)
from
    multi_category_data
group by
    cat1, cat2
```
- dataframe
```python
data.groupby(['cat1', 'cat2']).agg({'name': [np.size], 'score': [np.mean, np.sum, np.min, np.max]})
```
输出:
![](/img/blog/df/group2.png)

### 3. 自定义聚合方法

如何实现count(distinct column)操作呢？可以仿照numpy中内置的聚合方法，自定义一个:
```python
def my_distinct(rows):
    return len(set(e for e in rows))

data.groupby(['cat1', 'cat2']).agg({'name': [np.size], 'score': [np.mean, np.sum, np.min, np.max, my_distinct]})
```
也可以采用lamba表达式实现匿名函数
```python
data.groupby(['cat1', 'cat2']).agg({'name': [np.size], 'score': [np.mean, np.sum, np.min, np.max, lambda rows: len(set(e for e in rows))]})
```
输出:
![](/img/blog/df/group3.png)

### 4. 行转列: group_concat
sql中的group_concat可以实现将同组的多行字段拼接成一列，也就是行转列，numpy.unique可以轻松实现，比如下面我们合并每个二级分类下的所有分数(例子并不很恰当，学操作就行):
- sql
```sql
select
    cat1, cat2,
    count(name), avg(score), sum(score), min(score), max(score),
    count(distinct score), group_concat(score separator  ',')
from
    multi_category_data
group by
    cat1, cat2
```
- dataframe
```python
data.groupby(['cat1', 'cat2']).agg({'name': [np.size], 'score': [np.mean, np.sum, np.min, np.max, my_distinct, np.unique]})
```
输出:  
![](/img/blog/df/group4.png)

先到这里，后续再补充~
