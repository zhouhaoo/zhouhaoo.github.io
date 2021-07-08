---
layout:     post
title:      "利用HEXO+GitHub创建博客"
subtitle:   " \"Hello World, Hello Blog\""
date:       2016-02-28 12:46:25
author:     "zhouhaoo"
header-img: "img/post-bg-2015.jpg"
tags:
    - 博客
---

![hexo图片](/img/hexo.png) 
　　在学习Android的过程中，一直有写笔记的习惯，但是并没有上传到网站上，后来发现能利用hexo和github仓库创建一个属于自己的blog，所以在网上找了教程摸索出来，域名用的是github仓库的名字。下面叙述一下我整个搭建的流程。
<!-- more -->
## 开头语：
__[Hexo](https://hexo.io)__ 是一款基于 Node.js 的快速的、简单的博客框架，能够创建一个项目进行编辑后生成一套静态网页，比较适合个人博客搭建。因为 Hexo 生成的网页不依赖数据库和任何 Web 工具，所以可以把它放在 Github仓库上，然后配下仓库名称，就可以访问了。
本博客是基于Mac环境下搭建的，至于win下，原理目测差不多，废话不多说，下面开始。

### 1.准备环境配置：
#### 下载安装Node.js 
因为 Hexo 是基于 Node.js 的第三方模块，所以缺少 Node.js 不可。访问 **[Node.js官网](https://nodejs.org/en/)**下载适合自己系统的 Node.js 安装包。
#### 安装HEXO
Hexo 是基于 Node.js 的第三方模块，所以我们也需要安装。mac打开终端，一行命令:
```shell
npm install -g hexo
```
#### git环境（用于静态网站同步）
配置git环境这个就不多说了。
没有账号的，直接上官网注册就行，新建好一个仓库备用仓库名一定要是：<font color=#ff0000>“你的github用户名.github.io”</font>，这样才能打开网站！例如我的就是:<font color=#ff0000>zhouhaoo.github.io</font>。
不会配置git的请参考：**[git下载配置官网](https://git-scm.com/downloads
)**
到此，请确认所有环境已经安装好，下面可以开始进行新建博客了。

### 2.初始化HEXO：
cd到你想要创建博客的目录下，我是cd blog 然后输入命令：
```shell
hexo init /** 创建一个Hexo的新框架 **/
```
之后在文件夹下出现了很多文件：
![](/img/blog.png) 
#### 后面继续在控制台按以下hexo命令输入：
__1、开始新建文章__
``` bash
hexo n "文章名称" == hexo new "文章名称" #新建文章
 
```
这个命令完成后，会在<font color=#ff0000>/source/_posts</font> 文件夹下生成一个.md文件，打开进去编辑想要书写的博客内容。完成后保存即可。
 __2、生成静态网页文件__

```
hexo g == hexo generate 
```
 __3、部署后开启本地服务器__
```
hexo s == hexo server #启动服务预览
```
成功开启服务器后，就可以根据提示本地预览网页了，在浏览器地址栏输入：<font color=#ff0000>http://localhost:4000/</font>  进入。就能看见网站了。
![](/img/hexoold.jpg) 
到此为止，hexo开启完毕。

 __4、配置HEXO — “_config.yml”__
Hexo 的每一个功能的配置文件都是 _config.yml，所有信息都是这里配置的，这里推荐可以用[HBudilder](http://www.dcloud.io)，将这个hexo目录导入，更加方便修改配置信息，不用每次命令进入了。
```
# Hexo Configuration
## Docs: http://zespia.tw/hexo/docs/configuration.html
## Source: https://github.com/tommy351/hexo/

# Site               ##修改以适应搜索引擎的收录
title: Hexo          ##定义网站的标题
subtitle:            ##定义网站的副标题（不一定会显示）
description:         ##定义网站的描述
author: John Doe     ##定义网站的负责人
email:               ##定义网站负责人的电子邮件
language:            ##定义网站的语言，默认zh-Hans

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://yoursite.com                  ##定义访问的域名
root: /                                   ##定义所在Web文件夹的哪一个目录
permalink: :year/:month/:day/:title/      ##定义时间格式
tag_dir: tags                             
archive_dir: archives
category_dir: categories
code_dir: downloads/code

# Directory
source_dir: source           ##定义从哪个文件夹获取博客资料              
public_dir: public           ##定义生成静态网站到哪个文件夹

# Writing
new_post_name: :title.md # File name of new posts
default_layout: post
auto_spacing: false # Add spaces between asian characters and western characters
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
max_open_file: 100
multi_thread: true
filename_case: 0
render_drafts: false
post_asset_folder: false
highlight:
enable: true
line_number: true
tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:

# Archives
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 2
category: 2
tag: 2

# Server
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000                     ##定义测试访问的端口号
server_ip: 0.0.0.0             
logger: false
logger_format:

# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: MMM D YYYY
time_format: H:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# Disqus
disqus_shortname:

# Extensions
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
theme: landscape                   ##定义使用的主题
exclude_generator:

# Markdown
## https://github.com/chjj/marked
markdown:
gfm: true
pedantic: false
sanitize: false
tables: true
breaks: true
smartLists: true
smartypants: true

# Stylus
stylus:
compress: false

# Deployment
## Docs: http://zespia.tw/hexo/docs/deployment.html
deploy:
type:
```
 __5、完成后部署到git__
配置 _config.yml文件，重点，刚刚第一次我弄得时候，没注意这个，导致不同步。repo按照下面这个格式，填上之前准备好的你自己的github仓库地址，例如我的是：

```
deploy:

     type: git

     repo: https://github.com/zhouaoo/zhouhaoo.github.io.git

     branch: master
```

配置好，保存，下面进行同步到git。

```
hexo d == hexo deploy#部署
```

最后，等待git同步完成后，期间需要输入git的密码。这个完成之后，就可以直接在浏览器输入你的<font color=#ff0000>“你的github用户名.github.io”</font>，比如我的就是:<font color=#ff0000>https://zhouhaoo.github.io</font>。
到此利用hexo+github 完成博客的搭建。


hexo常用命令参考: <br/>[hexo常用命令笔记，by小弟调调](https://segmentfault.com/a/1190000002632530)
### 写在最后
另外还有换主题什么的的，就不累述了，有兴趣的可以进 __[hexo主题官网](https://hexo.io/themes/)__ 或者进github搜索更换自己喜欢的主题。

参考链接：<br/>
1、[HEXO搭建个人博客](http://baixin.io/2015/08/HEXO搭建个人博客/)<br/>
2、[使用Hexo生成一套静态博客网页](http://ninghao.net/blog/1412)