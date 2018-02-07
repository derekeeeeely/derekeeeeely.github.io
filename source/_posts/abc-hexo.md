---
title: Hexo快速搭建博客
abbrlink: d76cea4a
date: 2017-10-25 19:01:33
tags: hexo
categories: code
top: 100
---

&emsp;利用一天时间搭一个简单漂亮的静态博客，平时记记笔记、写写文章，日积月累，收获也许会让你意想不到。
<!-- more -->

## 前言

&emsp;对于酷爱尝新的程序员们来说，博客可能是为数不多的仍被广泛使用的旧东西之一了。不论是为了技术积累，还是为了提高逼格，多数程序员心中都曾有过搭建博客站的想法。

&emsp;在搭建博客的时候，大佬或者不麻烦会死星人都喜欢撸起袖子自己干，然而对于像我一样的一批渣渣小白来说要脱离现成框架自己搞出个名堂来绝非易事。换工作的空档期，忙得只剩下时间，也手撸了个[博客站](http://blog.derekz.cn/)，除去编码时间，大部分的时间花在服务器部署上，对于一名前端切图狗来说，还是费了不少心思的，有兴趣的这里送上[传送门](https://github.com/derekeeeeely/nuxt-derekzhou)。

&emsp;就效果来说自己动手肯定是可以学到更多东西的，愿意投入时间是值得去尝试的，而且自己开发是可以根据需要个性化定制的，这是第三方服务不能很好地提供的。随之而来的坏处其一是开发成本和学习成本，独立博客站需要一定的技术积累，有一定的门槛，并不适合所有人。其二是即便你有能力有时间做到，后期是否会花心思去维护打理也是一个问题。想起我自己一开始满腔热血，下定决心要搞出个完美博客，结果七天草草收工，预先想加上的功能一个个都无疾而终，甚至到昨天域名都还没备案。

&emsp;我以为独立博客站适合于自由职业者或者能够在短时间内把自己的需要实现的大佬们，因为这两者后期要么有时间维护要么不需要过多时间去维护。在博客站搭建完成以后他们就可以把重心放回到自己的工作和生活当中，在有技术积累或者装逼需要的时候只需要记录、上传这些简单的操作。

&emsp;更加省心的选择是成熟的第三方框架，比如本站所用的[Hexo](https://hexo.io/zh-cn/docs/)，不得不说Hexo提供的功能和样式都是很有吸引力的，本身也不需要多少学习成本，甚至于非程序员也可以跟着教程实现。本文还是针对对于git有了解的想要快速搭建博客的程序员们，如果这方面有问题的话，也可以先去了解下git相关的东西。

## Hexo 走你

### 大胆假设

- 你的SSH key配置无误
- 你安装好了node/npm
- 你在你的Github上建了一个 your_user_name.github.io 的 repo

### 安装

```bash
# 全局安装hexo
cnpm install hexo -g

# 安装hexo deploy包
cnpm install hexo-deployer-git --save

# 初始化博客目录
cd your_blog_dir
hexo init

# 安装依赖包
npm install

# 启动本地服务，访问127.0.0.1:4000，bingo！你将看到hexo默认首页
hexo server
```

### Hexo设置

#### 基本设置

```bash
# 编辑根目录下的配置文件
vi _config.yml

# 部署配置
deploy:
  type: git
  repository: git@github.com:derekeeeeely/derekeeeeely.github.io.git
  branch: master

# 站点设置
title: DZ's Gensokyo
subtitle: Best Time, Best You and Me
description: life are flashes and here is my life
author: Derek Zhou
language: zh-Hans
timezone:

# 你的域名
url: http://derekz.cn
```

> 完成基本设置后可以使用系一部分提到的命令进行创建文章、生成文件、部署到Git等操作

#### 常用命令

```bash
# 新建文章
hexo new XXX

# 在source目录下生成一个XXX文件夹，可以配置到head的导航处（后面有说明）
hexo new page XXX

# local server, 简化hexo s
hexo server

# 生成静态页面，简化hexo g
hexo generate

# 部署到Github，简化hexo d
hexo deploy

# 清除public，修改未生效时（有待研究hexo的热更新）
hexo clean

# 生成+部署
hexo d -g
```

### Hexo主题 --- Next

&emsp;Hexo 提供了很多主题的选项，本站使用的是Next，也是比较流行，个人认为比较好看的主题。笔者用的版本是5.1.3，应该是比较高的版本，本身已经集成了很多有用的插件，配置起来更方便一些。其他版本和其他主题应该也多有相似之处，可供参考。

#### Next 安装

```bash
# 项目根目录clone Next主题代码
git clone https://github.com/iissnan/hexo-theme-next.git themes/next

# 修改项目配置，设置主题为Next，后续的很多设置都在Next的_config.yml 文件中进行，可以看成一个extend的关系
vi _config.yml
theme: next
```

#### Next 插件篇

> 敲黑板，划重点啦，好不好用，逼格高不高就看这里了。

##### 导航条

```bash
# menu设置
menu:
  home: / || home
  categories: /categories/ || th
  tags: /tags/ || tags
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  # sitemap: /sitemap.xml || sitemap
  commonweal: /404.html || heartbeat
  about: /about/ || user

# Enable/Disable menu icons.
menu_icons:
  enable: true

# Next的四种主题，Mist表现中规中矩
scheme: Mist
```

&emsp;menu对应Next的Mist主题下头部导航，需要new出相应的页面，其中归档(archives)是默认有的，分类(categories)、标签(tags)、404页面和关于(about)是需要创建或编辑的。

##### 站内搜索

```bash
# 安装依赖包
cnpm install hexo-generator-search --save
cnpm install hexo-generator-searchdb --save

# Local search 打开
local_search:
  enable: true
```

##### 阅读量统计

```bash
# leanCloud配置 （Next config已存在）
leancloud_visitors:
  enable: true
  app_id: ssssssssssssssssssssssss
  app_key: sssssssssssssssssssssss
```

&emsp;使用leancloud做统计，Next的插件目录底下的`lean-analytics.swig`文件已经存在leanCloud的引入和统计代码，我们只需要注册leanCloud，使用`app_id`和`app_key`做身份校验，按如上填写好就可以了。

##### 字数统计和阅读时间估计

```bash
# 安装依赖包
cnpm install hexo-wordcount --save

# wordcount 设置（Next config已存在）
post_wordcount:
  item_text: true
  wordcount: true
  min2read: true
  totalcount: false
  separated_meta: true
```

##### siderbar设置

```bash
# 社交链接/icon设置
social:
  GitHub: https://github.com/derekeeeeely
  微博: http://weibo.com/u/3248682277
social_icons:
  enable: true
  GitHub: github
  微博: weibo

# links友情链接
links:
  我的小站: http://119.23.217.75
```

&emsp;Next的icon默认使用的是Font Awesome库，需要注意的是key应该保持一致。友情链接可以放些自己的或者自己收藏的社区啊论坛啊大佬博客之类的，方便快速跳转。本站这里是因为自己另一个博客的域名没备案，所以放了IP，不要在意这些细节。有关于RSS和头像这部分没有累列出来，RSS是因为最后得到的是一个XML，老实说我不知道这个XML拿来有啥用，头像的话其实就改样式，不想自己写的话[随便搜搜](http://shenzekun.cn/hexo%E7%9A%84next%E4%B8%BB%E9%A2%98%E4%B8%AA%E6%80%A7%E5%8C%96%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B.html)都有的，也就不贴出来了2333。

##### 文件压缩

```bash
# 安装依赖包
cnpm install hexo-all-minifier --save

# 在根目录配置文件中添加
html_minifier:
  enable: true
  ignore_error: false
  exclude:
css_minifier:
  enable: true
  exclude:
    - '*.min.css'
js_minifier:
  enable: true
  mangle: true
  output:
  compress:
  exclude:
    - '*.min.js'
image_minifier:
  enable: true
  interlaced: false
  multipass: false
  optimizationLevel: 2
  pngquant: false
  progressive: false
```

##### 文章链接唯一化

```bash
# 安装依赖包
npm install hexo-abbrlink --save

# 在根目录配置文件中修改
permalink: posts/:abbrlink/  # “posts/” 可自行更换

# abbrlink config
abbrlink:
  alg: crc32  # 算法：crc16(default) and crc32
  rep: hex    # 进制：dec(default) and hex
```

### 进阶黑科技

#### SEO (观察效果ing)

```bash
# 安装依赖包
cnpm install hexo-generator-sitemap --save
cnpm install hexo-generator-baidu-sitemap --save

# 根目录配置中添加
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml

# source目录添加robots.txt
User-agent: *
Allow: /
Allow: /archives/
Allow: /categories/
Allow: /tags/

Disallow: /vendors/
Disallow: /js/
Disallow: /css/
Disallow: /fonts/
Disallow: /vendors/
Disallow: /fancybox/

Sitemap: http://www.derekz.cn/sitemap.xml
Sitemap: http://www.derekz.cn/baidusitemap.xml
```

&emsp;上述配置完后需要自备梯子翻墙登录google search console，按照提示一步一步走，验证域名所有权->抓取对应路由页面数据->加入资源索引，由于次数有限制，需要确定好哦。

#### 七牛云及图床

&emsp;七牛云注册以后会提供10G的免费存储空间，一般来说存些图片是够用的了，问题在于我们在写markdown的时候图片链接复制是在是太麻烦了。这里提供几个选项：

- ipic 免费版图片上传至微博，每月付费可上传至七牛云（RMB玩家可以考虑）
- picU 只支持七牛云，可以一键截图上传并复制好图片链接（比较好用，但是笔者的这个不知道为啥用得好好的某一天突然就验证不通过了）
- [qiniu-image-tool](https://github.com/jiwenxing/qiniu-image-tool) 需要安装qshell（七牛云的一个命令行工具）以及 Alfred（非RMB玩家能接受破解版的话可以去[这里](http://xclient.info)下载哦）

#### 域名指向

&emsp;推荐[阿里云](https://wanwang.aliyun.com/domain/)，顺便一提，阿里云的ECS最低配首年只要300块，有服务器需求的可以尝试。
&emsp;注册好域名后需要添加域名解析，讲道理的话配置CNAME为www和@、记录值为XXX.github.io就可以的，但是实际操作上我在阿里云域名解析这里尝试配置第二条的时候一直报错，没办法，ping了下XXX.github.io拿到IP后加了一条记录类型为A、记录值为IP的记录，如下图所示：![域名解析](http://opo02jcsr.bkt.clouddn.com/2ca6bdd66256d44f70c4e6217c16f99f.jpg)

&emsp;还需要做的一件事是在source目录底下创建一个CNAME文件，内容的话就填写你的域名，然后push到远端，重新部署。

#### 保存文件

&emsp;为了保留本地文件，参考前辈做法，在自己的这个repo上checkout一个新的分支derekzhou，将代码提交。以后每次更新文章时先push到derekzhou分支，然后执行hexo的命令去生成public文件deploy到master。这样就算本机坏了也没影响，源代码还是可以从git上clone下来。

```bash
# .gitignore
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
/.deploy_git
/public
# /_config.yml
```





&emsp;有任何疑问，欢迎[git](https://github.com/derekeeeeely/derekeeeeely.github.io/issues)、[weibo](http://weibo.com/u/3248682277)各种方式@我，博客的评论功能这两天会加上，因为好像现在梯子都被砍，第三方的一些评论服务也不好用了，有时间再看看吧。感谢各位看官老爷们~
