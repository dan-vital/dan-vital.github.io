---
layout: article
title: "入门python爬虫"
tags: python
key: p2022-02-28-spider-img
aside:
  toc: true
category: [Study, _python] 
---
用python爬虫抓取网络图片

## 零、前言
今天学了学爬虫，想要记录一下

## 一、所需工具
1. python
2. 相应的库：resquest、ltml、vthread、mkdir
--------------------------------
## 二、步骤
1. **导入所需要的库**
```python

from os import mkdir  #为了新建目录
import requests  
from lxml import etree  # 导入lxml的etree库
import vthread  #github上一个大佬写的多线程库，安装后导入即可

```

2. **获取目标网址（你得先确定从哪开始爬虫）**  

```python 
    resp = requests.get(
        'https://www.58pic.com/piccate/53-525-0-ty10-p%d.html' % (n))  # 爬虫的目标网址，为了抓取不同页面，而将页码定义为参数
        #如果是只爬一个网页，直接放网址即可
    resp.encoding = resp.apparent_encoding  # 设置编码格式

```
3. **分析所获取的html**   

```python

    html = etree.HTML(resp.text)  # 读取需要进行解析的网页：这是从字符串读取
    # '//' 表示获取当前节点的子孙节点，'*'表示所有节点， '//*'表示获取当前节点下所有节点， '@'表示获取相应属性
    # 使用xpath查找html文件中的元素，最好去学习一下xpath怎么用
    result = html.xpath(
        "//a/div[1]/img/@data-original|//a/div[2]/div[1]/p[@class='text']/text()") 
        #第一路径是图片的网址，第二个路径是图片的名字
```

到这一步就已经获取到图片网址和图片名称了，下一步就是要把图片资源下载到本地，要组织一下，全放在在一个文件就太乱了

4. **处理*
```python 
    cnt = 0
    total = len(result)/2 #得到的rusult是个列表list，存放顺序是网址1，图名1，网址2，图名2……
    mkdir("img/%d" % n)  #为每一页新建一个文件夹以放置每页所下载的图片
    #
    for img_uri, img_name in zip(result[0::2], result[1::2]):
        img_uri = "http:" + img_uri
        img_name = img_name + ".jpg" 
        img_resp = requests.get(img_uri)
        with open("./img/%d/" % n+img_name, "w+b") as f:
            f.write(img_resp.content)
        cnt += 1
    print("[%d] finished %d/%d" % (n, cnt, total)) #方便的打印时候看看完成进度

```
5. **完成简单爬虫项目啦**   
为了获取多个网页的图片，可把以上步骤涉及的代码写成一个函数，然后相当于在主函数中再调用这个函数，全部代码如下所示：

```python 

from os import mkdir
import requests
from lxml import etree  # 导入lxml的etree库
import vthread


@vthread.pool(10) # 只用加这一行就能实现10条线程池的包装
# 定义了单个网页的抓取函数
def img_spider(n):   
    resp = requests.get(
        'https://www.58pic.com/piccate/53-525-0-ty10-p%d.html' % (n))  # 爬虫的目标网址，为了抓取不同页面，而将页码定义为参数
    resp.encoding = resp.apparent_encoding  # 设置编码格式
    html = etree.HTML(resp.text)  # 读取需要进行解析的网页：这是从字符串读取

    # '//' 表示获取当前节点的子孙节点，'*'表示所有节点， '//*'表示获取当前节点下所有节点， '@'表示获取相应属性
    # 使用xpath查找html文件中的元素
    result = html.xpath(
        "//a/div[1]/img/@data-original|//a/div[2]/div[1]/p[@class='text']/text()") #第一路径是图片的网址，第二个路径是图片的名字
    cnt = 0
    total = len(result)/2 #得到的rusult是个列表list，存放顺序是网址1，图名1，网址2，图名2……
    mkdir("img/%d" % n)  #为每一页新建一个文件夹以放置每页所下载的图片
    #
    for img_uri, img_name in zip(result[0::2], result[1::2]):
        img_uri = "http:" + img_uri
        img_name = img_name + ".jpg" 
        img_resp = requests.get(img_uri)
        with open("./img/%d/" % n+img_name, "w+b") as f:
            f.write(img_resp.content)
        cnt += 1
    print("[%d] finished %d/%d" % (n, cnt, total)) #方便的打印时候看看完成进度


if __name__ == "__main__":
    mkdir("img")
    for i in range(1, 21):  #爬取第一页到20页
        img_spider(i)

```


