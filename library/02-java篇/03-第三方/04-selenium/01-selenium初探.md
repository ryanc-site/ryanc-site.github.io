# selenium初探

> 时间：<span style="color:#42B983;font-weight:bold;">2020年12月19日18:47:54</span>

## 一、第三方依赖

### 1、maven依赖坐标

~~~xml
<!-- selenium 的java api ：版本按需选择 -->
<dependency>
    <groupId>org.seleniumhq.selenium</groupId>
    <artifactId>selenium-java</artifactId>
    <version>3.4.0</version>
</dependency>
~~~



### 2、浏览器驱动

> 当selenium升级到3.0之后，对不同的浏览器驱动进行了规范。如果想使用selenium驱动不同的浏览器，必须单独下载并设置不同的浏览器驱动。

* 各浏览器驱动

  Firefox浏览器驱动：[geckodriver](https://github.com/mozilla/geckodriver/releases) </br>
  Chrome浏览器驱动：[chromedriver](https://sites.google.com/a/chromium.org/chromedriver/home)[taobao备用地址 </br>](https://npm.taobao.org/mirrors/chromedriver)
  IE浏览器驱动：[IEDriverServer](http://selenium-release.storage.googleapis.com/index.html) </br>
  Edge浏览器驱动：[MicrosoftWebDriver](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/) </br>
  Opera浏览器驱动：[operadriver](https://github.com/operasoftware/operachromiumdriver/releases) </br>
  PhantomJS浏览器驱动：[phantomjs](http://phantomjs.org/) </br>

  <span style="color:red;">注：部分浏览器驱动地址需要科学上网。</span>



## 二、快速上手

### 1、开启浏览器

> 这里我们以谷歌浏览器为例（访问百度）

~~~java
// 设置浏览器驱动程序位置
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 

// 开启一个谷歌浏览器
WebDriver webDriver = new ChromeDriver();

// 窗口最大化
webDriver.manage().window().maximize();

// 访问百度
webDriver.get("https://www.baidu.com")
~~~

