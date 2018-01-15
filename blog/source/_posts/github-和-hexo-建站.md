---
toc: true
title: github 和 hexo 建站
date: 2018-01-15 14:58:12
tags: hexo
categories: 建站
---


## **Outline**

昨天闲着无聊，决定像大牛学习，看到牛X的大牛都有各种blog 来分享心得。所以本着开源分享的原则，决定自己也建一个blog，像大牛学习。

这个blog 是通过hexo + github page +  aliyun 的域名服务 建造的。这里写简单介绍一下如何最基本的搭建hexo + github page.  

## **Github page**

##### 什么是github page

[Github page](https://pages.github.com/) 是一个静态的网站，免费哦！

##### **建立Github Page**

1. 建立一个repo

   ​	去Github 建立一个repository 叫做 <u>***username.github.io***</u> 然后username就是你github 的username。

   ​

2. 复制你的repo到你的terminal

```bash
$ git clone https://github.com/username/username.github.io
```

3.  Hello world
```bash
$ cd username.github.io
$ echo "Hello World" > index.html
```

4.  Push it
```bash
$ git add --all
$ git commit -m "Initial commit"
$ git push -u origin master
```

5.  访问 **https://username.github.io** 你的网站建好了！



## **Hexo**

##### 什么是hexo？

[Hexo](https://hexo.io/zh-cn/index.html) 是一个Blog framework，还是免费哦！它是一个简单的,轻量级,基于Nodejs的一个静态的博客框架. 我们可以很方便的使用Hexo搭建个人博客.  Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

##### 安装前提

安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：

- [Node.js](http://nodejs.org/)
- [Git](http://git-scm.com/)
  如果您的电脑中已经安装上述必备程序，那么恭喜您！接下来只需要使用 npm 即可完成 Hexo 的安装。
```bash
$ sudo npm install -g hexo-cli
```
没安装git 和 node.js 的话 下好了 再装一下就好了。

##### 安装hexo
```bash
$ sudo hexo init blog
$ cd blog
$ sudo npm install
$ hexo g # 或者hexo generate
$ hexo s # 或者hexo server，可以在http://localhost:4000/ 查看
```

#####发表新文章

用hexo发表新文章

```bash
$ hexo n "article title" #写文章 
```

##### Hexo主题设置

这里以主题hexo-theme-yilia为例进行说明。

安装主题，在blog 目录下

```bash
$ hexo clean
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

启用主题

修改Hexo博客目录下的_config.yml配置文件中的theme属性，将其设置为yilia

```latex
theme: yilia 
```



更新主题

```bash
$ cd themes/yilia
$ git pull
$ cd ..
$ hexo clean
$ hexo g # 生成
$ hexo s # 启动本地web服务器， 我就把本地服务器当成一个beta了
```

现在打开http://localhost:4000/ ，会看到我们已经应用了一个新的主题。

现在有很多很多有趣的主题， 大家可以凭借自己的喜好选一个哇、

[有哪些好看的 Hexo 主题?](https://www.zhihu.com/question/24422335/answer/46357100)  不同主题会有不同的configuration，所以基本都大同小异了。

##### 使用hexo deploy部署到Github page

hexo deploy可以部署到很多平台，具体可以参考这个[链接](https://hexo.io/zh-cn/docs/deployment.html). 如果部署到github，需要在配置文件_config.xml中作如下修改：

```latex
deploy:
   type: git
   repo: https://github.com/username/username.github.io.git
   branch: master
```

然后在命令行中执行

```bash
$ hexo deploy # 发送到你的blog 站点上
```

**注意需要提前安装一个扩展：**
```bash
$ sudo npm install hexo-deployer-git --save
```

使用git命令行部署（optional）
不幸的是，上述命令虽然简单方便，但是偶尔会有莫名其妙的问题出现，因此，我们也可以追本溯源，使用git命令来完成部署的工作。



## Aliyun 阿里云:

马云爸爸就是牛X。我就这样随随便便的买了一个域名了。chinoliu.com 还不贵哈哈哈哈。

##### 购买域名

去[阿里云](https://wanwang.aliyun.com/) 注册一个账号，然后在国外的同学可以选择International 区域，购买域名哇。找一个又臭屁，又便宜的吧。 哈哈哈哈哈。

比如我就买了一个*chinoliu.com*

然后呢就是解析域名了：

![域名解析](http://chuantu.biz/t6/206/1516055658x-1566671321.jpg)

然后就是点击这个解析了、 我们要创造2个解析！

![](http://chuantu.biz/t6/206/1516055838x-1566671297.jpg)

记录类型选择CNAME.

主机记录填www。 然后再解析一次再建一个CNAME， 主机记录填@

记录值就是你github page 的url 了。

##### 配置hexo CNAME 绑定你的域名

很重要的一点哇，你要记得给你的hexo 也加CNAME

在source文件夹中新建一个CNAME文件（无后缀名），然后用文本编辑器打开，在首行添加你的网站域名，如[chinoliu.com](chinoliu.com)，注意前面没有http://，也没有www，然后使用hexo g && hexo d上传部署。

## Other

 因为github page是一个静态网站，所以要想解决图片，评论啊等其他问题还需要第三方支持。

比如说评论系统, 国内推荐: [畅言](https://changyan.kuaizhan.com/) 国外就是: [disqus](https://disqus.com/)

然后图片储存系统，国内推荐：[七牛](https://www.qiniu.com/) 国外还没找到，等我再找找。

Hexo 还有一些好玩的插件，感兴趣的同学可以自己上github 或者官网档上再找一找。

哦哦，再推荐一个比较好的markdown 编辑器 [Typora](https://typora.io/) 用起来很舒服。

 



