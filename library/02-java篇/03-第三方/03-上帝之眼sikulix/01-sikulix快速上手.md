# sikulix快速上手

> 时间：<span style="color:#42B983;font-weight:bold;">2020年12月19日15:47:31</span>

## 一、第三方依赖

* maven依赖坐标

  ~~~xml
  <!-- sikulix maven坐标 版本按需选择 -->
  <dependency>
     <groupId>com.sikulix</groupId>
     <artifactId>sikulixapi</artifactId>
     <version>1.1.2</version>
  </dependency>
  ~~~

  

## 二、快速上手

### 1、开启浏览器

~~~java
// 浏览器图标自行截图 存放至制定目录
try {
   Pattern pattern = new Pattern("浏览器图标图片路径");
   Screen screen = new Screen();
   screen.click(pattern);
} catch (Exception e) {
	e.printStackTrace();
}
~~~



### 2、访问百度

~~~java
Pattern pattern = new Pattern("浏览器地址栏图片路径");
Screen screen = new Screen();
screen.click(pattern);
screen.paste("浏览器地址栏图片路径","www.baidu.com");
~~~



## 三、进阶使用

>sikulix参考文档：http://doc.sikuli.org/region.html#Region.text

### 1、文本框输入中文

~~~java
Pattern pattern = new Pattern("输入框图片路径");
Screen screen = new Screen();
screen.click(pattern);
screen.paste("输入框图片路径",要输入的文本); // 方法一
screen.type("输入框图片路径"，要输入的文本)； // 方法二
~~~

<span style="color:red;">注：使用screen类输入文本的两种方法，type存在文本长度的问题，而paste方法没有此类问题</span>