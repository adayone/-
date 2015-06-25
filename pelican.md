title:使用pelican搭建博客
date:2014-08-18
category:技术

之前的博客用的是octopress， 挺好的， 也没啥毛病， 就是我不会ruby，出个小问题纠结大半天。

因为最熟悉的语言是python，找了下有没有python的frame，发现还真有： pelican。

这下子就简单了：

1. pip install pelican
2. pip install Markdown
3. pelican-quickstart

这些整完后就可以写blog了。

很多人一上手就用了octopress， 反而弄混了，这里解释下：

* <s>github pages, 在gh-pages下都是html文件。</s>
* octopress、pelican、jeklly， 都是用来生成这些html的。
* 理论上， 鉴于国内github的速度， 最好的办法是自己弄一台服务器，开一个最简单的http server。


为了方便起见， 改写了MakeFile文件， 增加了对百度云的支持， 同时因为一部分文档是不适合泄露的，额外建立一个private的content和output的文件夹， 在MakeFile的html和serve那里增加了对这些的支持。

申请了一台阿里云的服务器， 准备部署。

## update
申请到阿里云的服务器了， 在服务器上用了个最简单的httpserver:
```shell
nohup python -m SimpleHTTPServer 80 &
```
在makefile里面配置rsync同步。
大家可以访问这个地址[橘子杉树](http://www.juzishanshu.com)

额， 最终有个坑， gh-pages分支提交的是md文件， 对于我们的case提交的直接是最终的html， 所以应该提交到master这个分支。

### update
今天让多个人访问的时候服务器就顶不住了， 主要原因是木器啊那图片是直接放web server上的。

pelican原生对图片支持不太好， 它的插件库里面有一个better_figures_and_images，能够展示更优美的图片， 但是只支持本地图片， 我修改了这个插件[github地址](https://github.com/getpelican/pelican-plugins/tree/master/better_figures_and_images), 需要的直接clone我的插件库就好， 相关修改已经提交pelican。

申请了阿里云的oss， 简直是业界良心， 便宜好用耐操。 

### update 2014 08 24
服务器换了nginx， 配置很简单， 修改下user， 修改下root之类的就好了。
