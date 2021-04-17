# hexo基于github_pages搭建个人博客

## 一、前言

> 时间：2020年12月05日13:48:16



### 1、blog  建议

| 新手           | Hexo                                      |
| -------------- | ----------------------------------------- |
| Vue开发者      | VuePress                                  |
| React开发者    | Gatsby                                    |
| 喜欢自己造轮子 | 不是用框架、用自己熟悉的语言从0开始搭建》 |

<span style="color:red">注：本文作者是新手，博客初探所以使用Hexo + github_pages的方式\(^o^)/~</span>



### 2、前置知识

> 为了更好、更快和更高效的搭建自己的博客，各位博主需要掌握或者有一定了解以下的知识

**HTML & JS & CSS **

**Node & NPM    Git & GitHub**

<span style="color:red">注：如果对以上知识从未了解过，可以跟着本文一步一步操作完成  0.o</span>



### 3、官方文档

* 官方网站 	https://hexo.io/
* 官方Wiki     https://hexo.io/zh-cn/docs/



## 二、安装教程

> 时间：2020年12月05日  安装过程中如有出入详询[官方wiki](https://hexo.io/zh-cn/docs/)



### 1、安装前提

> 安装hexo相当简单，但须安装以下前置依赖

#### Node.js

~~~text
Node.js (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)
window/macOS：下载安装即可 http://nodejs.cn/download/
linux（archlinux）：sudo pacman -S nodejs  注：我用的是archlinux发行版（不同的发新版包管理不同）
~~~

#### Git

~~~text
window/macOS:搜所/包管理 下载安装即可
linux（archlinux）：sudo pacman -S git
~~~



### 2、正式安装

> 所有准备工作做完后，我们开始安装hexo



#### 安装 hexo 脚手架

~~~shell
# npm是国外的，如果嫌慢可以用cnpm （使用的是淘宝的源）
$ npm install -g hexo-cli  # -g 全局安装（hexo设置为全局变量）
~~~



#### 博客初始化

~~~shell
# 第一种方式：先创建blog目录  在初始化
$ mkdir blog_bin  		#创建博客目录
$ cd blog_bin	  		#进入博客目录
$ hexo init		  		#初始化博客

# 第二种方式：blog目录交由初始化一并完成
$ hexo init blog_bin	#在当前目录下创建blog_bin目录并在该目录下完成blog初始化

# 初始化博客会比较慢  安装完成后会在你的博客目录（blog_bin）中有以下目录
 _config.yml			# hexo应用配置
 node_modules			# node的一些依赖
 package.json			
 scaffolds				# md模板
 source					# 文章和页面md
 themes					# 主题目录
 yarn.lock

~~~

#### 博客本地启动

~~~shell
# 博客初始化完成后，就已经完成了hexo的本地安装 接下来我们本地启动看一下

$ hexo g 				# 同 hexo generate  相当于编译将source中的文章转成html ->/public
$ hexo s				# 同 hexo server 启动服务展现“编译”好的文章
~~~



### 3、站点发布

#### github仓库初始化

<span style="color:red">注：仓库名尽量使用    <用户名>.github.io   如下图</span>

![image-20201205144629836](amWiki/images/hexo/image-20201205144629836.png)

#### 安装发布插件

~~~shell
# hexo-deployer-git 发布插件：可以将你编译后的项目发布到你的git仓库

# 第一种方式 npm安装
$ npm install hexo-deployer-git --save

# 第二种方式 yarn安装   yarn就是重新设计的npm客户端
$ yarn add hexo-deployer-git
~~~

#### 配置发布仓库

~~~text
1 修改 hexo应用配置文件 _config.yml deploy节点如下：
	···
	deploy：
		type： ‘git’
		repo： ‘https://github.com/ryanc-site/ryanc-site.github.io.git’  # 你的github仓库
		branch： main  # 这里需要注意：2020.06之前是master 之后改成了main
	···
~~~

#### 发布

~~~shell
# 开始发布你的博客到github.io站点

# 清除缓存
$ hexo clean				# 清除你/public下的文件

# 编译
$ hexo g					# 编译生成html文件 ->public

# 发布
$ hexo d					# 同 hexo deploy  发布你博客（或改动）到你的站点

# 命令合体
$ hexo clean && hexo g --d	# 这个就是上边三个命令的合体
~~~

#### 配置站点

> 在仓库的setting -> GitHub Pages 节点配置如下

![image-20201205153908766](amWiki/images/hexo/image-20201205153908766.png)

#### 访问

部署后的站点开始愉快的玩耍吧
