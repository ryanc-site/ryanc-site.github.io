# hexo的初始化配置

> 时间：2020年12月08日10:46:54

## 一、初始化配置

> 一些简单的配置：头像、title、作者、时区、语言等等

### 1、head信息配置

~~~shell
# head信息配置：title、subtitle、description、keywords、author、language、timezone

# 修改hexo应用配置文件_config.yml Site节点
title:ryan'site
subtitle: '璟芮の小窝'
description: '技术来源于生活，生活运用技术创造美好未来'
keywords: 'java,python,摄影，旅游，滑雪...'
author: Ryan丶璟芮
language: zh-CN
timezone: 'Asia/Shanghai'
~~~



### 2、 主题配置

> hexo的主题来源有两种渠道：hexo官网  和   github主题项目
>
> hexo 官方theme:https://hexo.io/themes   在github中搜索：hexo-theme  （通过排序知道最受欢迎主题）

#### 下载主题

~~~shell
# 不管是用官方链接 还是 github搜索都需要将主题下载（克隆）到本地

# 克隆主题到本地（%hexo_home%/themes）
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia

# 修改hexo应用配置文件 theme节点的值为你中意的主题名
	```
	theme: yilia
	```

# 重新发布即可
~~~



### 3、头像配置

* 准备头像

  > 准备一个自己喜欢的头像，并上传至%hexo_home%/themes/yilia/source/img

* 修改配置

  > 修改yilia主题目录下的_config.yml配置文件 中的头像节点

### 4、主题相关其他配置

> 其他配置如：菜单、友链、微博、qq、邮箱等等按照yilia/_config.yml中注释的提示修改即可

background-image:url("/img/main_left_bg.jpeg");

e6db74



## 二、进阶配置

### 1、左侧背景配置

* 取消上半部背景颜色

  ~~~html
  <!-- 删除以下代码上上边的一行,并修改成以下的样子-->

  <div class="overlay" >
  ~~~

* 添加背景图片

  ~~~ css
  /* 第一步上传背景图片-> %hexo_home%/themes/yilia/source/img/xxxx.png */
  /* 第二步修改css文件(%hexo_home%/themes/yilia/source/main.xxxx.css)添加背景 */
  .left-col {
      /* background:#fff; */
      background: linear-gradient(rgba(255, 255, 255, 0.5), rgba(255, 255, 255, 0.5)),
      url('http://bucket836.oss-cn-shenzhen.aliyuncs.com/wallpaper/381535373.jpeg') no-repeat 0% 20%/ cover;
      width:300px;
      position:fixed;
      opacity:1;
      transition:all .3s ease-in;
      -ms-transition: all .3s ease-in;
      height:100%;
      z-index:999
  }
  ~~~

* 修改字体颜色

  ~~~css
  /* 注意下边的 “+” 和 “-”  +：表示新增 -：表示删除 */

  .left-col #header a {
  -    color:#696969
  +    color:#673ab7^M
   }
   .left-col #header a:hover {
  -    color:#b0a0aa
  +    color: #03A9F4^M
   }
   .left-col #header .header-subtitle {
       text-align:center;
  -    color:#999;
  +    color:#673ab7;
       font-size:18px;
  ~~~
