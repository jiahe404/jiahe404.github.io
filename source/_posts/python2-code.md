---
title: Python2 编码问题
date: 2018-04-29
reward: true
tag:
  - python
  - 编码
---

## 一些定义
* **字符(character)**
<br>**字符**是文字的最小的组成单位，其为一种抽象定义(不要与 java 或 c 中的 char 类型混淆，后者为特定计算机语言的数据类型)，取决于语言或是上下文环境，比如'A', 'a'为英文中的字符，'纺','织'是汉语中的字符(注意绞丝旁并不能称为是一个字符，因为在汉语中，它无法单独成为一个字) 。
<!-- more -->
* **图像字符(glyph)**
<br>人们在交谈时通过独特的发音来表达一个字符，而当在屏幕或是纸面上时，则是通过一些特定图形来表达一个字符。比如在汉字中用一条横线来表示'一'这个字符。这个图像化的形象即被称为**图像字符**，在计算机中可以通过矢量图表示。

* **字符串(characters/string)**
<br>**字符串**由若干字符组成的串。

* **码点(code point)**
<br>码点也有译为码位(code position)，是一个整数，常用16进制表示。Unicode标准就是维护了一张**字符与码点的映射表**，说白了，就是将一个字符用某个唯一的整数表示，这样全世界的计算机上都保存这样一张表，数据传输时只需要传输一堆数字就好，再用这张表去解析，找到对应的字符即可，而无需去传输字符所对应的矢量图。这就是标准的作用。<br>
下图是'一'的 Unicode 编码示例(图片源自[Charbase](https://www.charbase.com/4e00-unicode-cjk-unified-ideograph)):<br>
![one](/img/blog/one.png)

## 既然有了 Unicode，为什么还有一众编码方式？
现在我们知道了，世界上有这么一个统一的标准，那为什么还有 latin1, utf8, gbk 这些编码方式呢？为何不只用这一种编码方式，也省去了 decode, encode 的麻烦了。其实这是概念上的混淆。如果你还不知道自己错在哪了，请思考一个问题，作为程序员，我们都或多或少地了解 MVC 模式，那我想问，既然 MVC 这么好，为什么还要用 MFC(vc++框架), struts(java web 框架), ci(php 框架)呢？相信你可以理解这个荒唐的问题。
>MVC 是一种设计模式，一种思想，将 model，view, controller 解耦。而上面提到的几种框架，其实是对这种设计模式的针对不同的场景进行的实现。可以将 MVC当作一个类，而具体干活的，其实是需要 new 一个对象出来，也就是MFC等一众对 MVC 思想的具体实现。

与 MVC 这个问题类似，Unicode 其实是多种编码体系(如ISO/IEC 10646)中的一种，而具体实现这种体系的，则为 UTF(Unicode Transformation Format)，包括 utf8, utf16, utf32等编码方式。
>Unicode标准**的确**维护了一张**字符与码点的映射表**，所以你可以查看某个字符的 Unicode 编码，正如在上面'一'的 Unicode 编码示例图中可见其 Unicode 编码为'\u4e00'。然而对于码点在网络传输或物理存储具体要如何操作，如码点前置0怎么处理，用多少字节表示一个字符等问题，Unicode标准并未规定，具体是由UTF来实现的。

写到这里，上面提到的所有概念和内容与python没有半毛钱关系，对于任何编程语言上面的概念都是放诸四海皆准的。上文所提到的 Unicode 全部指代 Unicode 标准，千万不要和 python 中的 unicode 类型混淆(注意两个概念在本文中用首字母的大小写来区别)！

每种编码方式都有各自的特点，如汉字'一'用 gbk 及 utf16 编码占2个字节，用 utf8编码则占3个字节，'a'用 utf8编码则只占一个字节，用 utf16仍然占用2个字节，而 Unicode 统一使用4个字节来表示所有字符，但是对于定长的东西，在内存中寻址就很快很方便，这也是**在计算机内存中统一使用Unicode，而当要保存或传输数据时，转换成 UTF 编码**(当然也可以采用其他编码)的原因。
> gbk是针对汉字的编码方式，虽然可以用较少的字节来表示一个汉语字符，但对于很多其他国家的字符就无能为力了，而 utf 编码则基本囊括了世界上所有语言的字符集；
utf16，utf32分别用2、4个字节表示一个字符，然而对于英文等语言环境来讲，一个字节完全可以表示一个字符，就会浪费存储空间。

不同的编码方式都有其诞生及存在的合理性，不能只凭字符占用字节数，能表达的语言种类或是大小端处理等来衡量优劣，适合的就是最好的。所以，我们无法用一种编码方式走遍天下，那就得好好了解下如何在多种编码方式中处理数据，下面就来看下 python 中对于字符编码的处理。

## Python2中编码的转换
在python2中，字符串可以用 unicode 或 str 这两种类型来表示。
unicode类型的字符串由若干Unicode字符组成，本文中称之为**unicode串**，注意 unicode 与 unicode串是类与实例的关系，如下面代码中，a，b，c，d都是unicode串，a，b，c，d 的类型都是unicode：
```
# -*- coding: utf8 -*-
a = u'a'
b = u'一'
c = u'一a'
d = u'一a\u4e00' # 汉字'一'的 unicode 编码为\u4e00
e = u'一a\u4e00\x61\141'
print a, b, c, d, e
print type(a), type(b), type(c), type(d), type(e)
```
输出：
```
a 一 一a 一a一 一a一aa
<type 'unicode'> <type 'unicode'> <type 'unicode'> <type 'unicode'> <type 'unicode'>
```
>如果对变量 e 的内容有疑问，可以参考[这篇文章](http://python-reference.readthedocs.io/en/latest/docs/str/escapes.html)

str类型的字符串其实为字节序列，本文中称之为**str串**，一个str串的编码方式是需要指定的，这点和 unicode 串很不一样。因为 unicode 串一定是由 Unicode字符组成，这个规则是确定的。但是 python 不知道一个 str 串是采用了何种编码方式，只能使用默认的编码方式来处理，这个过程就有可能出错，事实上关于字符串转码的问题大多就出现在这里。

在 python2 中，将 unicode串转换成 str串称为编码(encode), 而将str串转换成 unicode串称之为解码(decode)。
### 编码(encode):
将内存中的unicode串保存到外存时，需要将之编码(encode)为 str串(即字节序列)，看下面的例子：
```
# # -*- coding: utf8 -*-
one = u'一'
with open('one.txt', 'w') as f:
    f.write(one)
```
运行报错：
```
UnicodeEncodeError: 'ascii' codec can't encode character u'\u4e00' in position 0: ordinal not in range(128)
```
意思是说采用 ascii 编解码标准无法对'一'进行编码，这是因为f.write(one)这行代码在执行时被解释成f.write(one.encode('ascii'))。python 默认会采用 ascci 编解码标准对unicode串进行编码，而汉字'一'的Unicode 编码为4e00(16进制)，超出了 ascii 编解码标准的上限128，所以报错。

正确做法是在将变量保存至外存时，显式地指定恰当的编码方式：
```
# -*- coding: utf8 -*-
one = u'一'
with open('one.txt', 'w') as f:
    f.write(one.encode('utf8'))
```
### 解码(decode):
当需要将不同编码类型的字符串放在一起处理时(比如从数据库中读入一个字段，在内存中其为一个unicode串，将之与一个 utf8编码的文件中的第一行数据拼接成一个字符串)，最好将 str串解码成unicode串，看下面的代码示例：
```
# -*- coding: utf8 -*-
database_field = u'一'
file_first_line = '二'
print database_field, type(database_field)
print file_first_line, type(file_first_line)
print database_field \
      + file_first_line  # 分行是为方便查看是哪行转换出错
```
运行报错：
```
一 <type 'unicode'>
二 <type 'str'>
Traceback (most recent call last):
  File "/Users/baidu/PhpstormProjects/scripts_dev/python/grammer/coding.py", line 7, in <module>
    + file_first_line  # 分行是为方便查看是哪行转换出错
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)
```
可以看到单独处理（输出）两个变量是没问题的，不过放在一起就完蛋了。从报错信息可以看到是对file_first_line按照ascii decode 时出错，原因不难理解，unicode串与 str串类型不一致，无法直接拼接，python 在尝试将 str串解码为 unicode 串时，采用了错误的 ascii 编解码标准。

可以通过将二者统一转换成一种类型来解决：
```
# -*- coding: utf8 -*-
database_field = u'一'
file_first_line = '二'
print database_field, type(database_field)
print file_first_line, type(file_first_line)
print database_field + file_first_line.decode('utf8'), \
    type(database_field + file_first_line.decode('utf8'))
print database_field.encode('utf8') + file_first_line, \
    type(database_field.encode('utf8') + file_first_line)
```
运行如下：
```
一 <type 'unicode'>
二 <type 'str'>
一二 <type 'unicode'>
一二 <type 'str'>
```
