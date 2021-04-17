# selenium进阶

> 时间：<span style="color:#42B983;font-weight:bold;">2020年12月19日 19:48:17</span>

## 一、元素定位

### 1、定位方法

<span style="color:#FF531A;font-weight:bold;">id:</span><span style="color:#42B983;font-weight:bold;">    findElement(By.id())</span></br>
<span style="color:#FF531A;font-weight:bold;">name:</span><span style="color:#42B983;font-weight:bold;">    findElement(By.name())</span></br>
<span style="color:#FF531A;font-weight:bold;">class name:</span> <span style="color:#42B983;font-weight:bold;">   findElement(By.className())</span></br>
<span style="color:#FF531A;font-weight:bold;">tag name: </span> <span style="color:#42B983;font-weight:bold;">  findElement(By.tagName())</span></br>
<span style="color:#FF531A;font-weight:bold;">link text:</span><span style="color:#42B983;font-weight:bold;">    findElement(By.linkText())</span></br>
<span style="color:#FF531A;font-weight:bold;">partial link text:</span><span style="color:#42B983;font-weight:bold;">    findElement(By.partialLinkText())</span></br>
<span style="color:#FF531A;font-weight:bold;">xpath:</span><span style="color:#42B983;font-weight:bold;">    findElement(By.xpath())</span></br>
<span style="color:#FF531A;font-weight:bold;">css selector:</span> <span style="color:#42B983;font-weight:bold;">   findElement(By.cssSelector())</span></br>

### 2、定位方法用法

~~~html
<html>
  <head>
  <body link="#0000cc">
    <a id="result_logo" href="/" onmousedown="return c({'fm':'tab','tab':'logo'})">
    <form id="form" class="fm" name="f" action="/s">
      <span class="soutu-btn"></span>
        <input id="kw" class="s_ipt" name="wd" value="" maxlength="255" autocomplete="off">
~~~

#### <span style="color:red;">Q1:我们的目的是要定位input标签的输入框。</span>

- 通过id定位：

  ```java
  driver.findElement(By.id("kw"))
  ```

- 通过name定位：

  ```java
  driver.findElement(By.name("wd"))
  ```

- 通过class name定位：

  ```java
  driver.findElement(By.className("s_ipt"))
  ```

- 通过tag name定位：

  ```java
  driver.findElement(By.tagName("input"))
  ```

- 通过xpath定位，xpath定位有N种写法，这里列几个常用写法：

  ```java
  driver.findElement(By.xpath("//*[@id='kw']"))
  driver.findElement(By.xpath("//*[@name='wd']"))
  driver.findElement(By.xpath("//input[@class='s_ipt']"))
  driver.findElement(By.xpath("/html/body/form/span/input"))
  driver.findElement(By.xpath("//span[@class='soutu-btn']/input"))
  driver.findElement(By.xpath("//form[@id='form']/span/input"))
  driver.findElement(By.xpath("//input[@id='kw' and @name='wd']"))
  ```

- 通过css定位，css定位有N种写法，这里列几个常用写法：

  ```java
  driver.findElement(By.cssSelector("#kw")
  driver.findElement(By.cssSelector("[name=wd]")
  driver.findElement(By.cssSelector(".s_ipt")
  driver.findElement(By.cssSelector("html > body > form > span > input")
  driver.findElement(By.cssSelector("span.soutu-btn> input#kw")
  driver.findElement(By.cssSelector("form#form > span > input")
  ```

#### <span style="color:red;">Q2:我们的页面上有一组文本链接。</span>

```java
<a class="mnav" href="http://news.baidu.com" name="tj_trnews">新闻</a>
<a class="mnav" href="http://www.hao123.com" name="tj_trhao123">hao123</a>
```

- 通过link text定位：

  ```java
  driver.findElement(By.linkText("新闻")
  driver.findElement(By.linkText("hao123")
  ```

- 通过partialLink text定位：

  ```java
  driver.findElement(By.partialLinkText("新")
  driver.findElement(By.partialLinkText("hao")
  driver.findElement(By.partialLinkText("123")
  ```

关于xpaht和css的定位比较复杂，请参考：[ xpath语法](http://www.w3school.com.cn/xpath/xpath_syntax.asp)、[css选择器](http://www.w3school.com.cn/cssref/css_selectors.asp)

#### <span style="color:red;">Q3:定位一组元素</span>

~~~java
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com/");

WebElement search_text = driver.findElement(By.id("kw"));
search_text.sendKeys("selenium");
search_text.submit();
Thread.sleep(2000);

//匹配第一页搜索结果的标题， 循环打印
List<WebElement> search_result = driver.findElements(By.xpath("//div/div/h3"));

//打印元素的个数
System.out.println(search_result.size());

// 循环打印搜索结果的标题
for(WebElement result : search_result){
    System.out.println(result.getText());
}

System.out.println("-------我是分割线---------");

//打印第n结果的标题
WebElement text = search_result.get(search_result.size() - 10);
System.out.println(text.getText());

driver.quit();
~~~



## 二、控制浏览器窗口

### 1、浏览器窗口大小

>- maximize() 设置浏览器最大化
>- setSize() 设置浏览器宽高

~~~java
// 设置浏览器驱动程序位置
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 

// 开启浏览器 访问pc版百度
WebDriver driver= new ChromeDriver();
driver.get("https://www.baidu.cn");
 
// 设置窗口最大话
driver.manage().window().maximize();
Thread.sleep(2000);
 
// 访问移动版百度，自定义窗口大小
driver.get("https://m.baidu.cn");
driver.manage().window().setSize(new Dimension(480, 800));
Thread.sleep(2000);
 
// 关闭浏览器（注：这种有时候会关不掉）
driver.quit();
~~~



### 2、浏览器前进后退

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
 
//get 到百度首页
driver.get("https://www.baidu.com/");
System.out.printf("now accesss %s \n", driver.getCurrentUrl());
Thread.sleep(2000);

//点击“新闻” 链接
driver.findElement(By.linkText("新闻")).click();
System.out.printf("now accesss %s \n", driver.getCurrentUrl());
Thread.sleep(2000);

//执行浏览器后退
driver.navigate().back();
System.out.printf("back to %s \n", driver.getCurrentUrl());
Thread.sleep(2000);

//执行浏览器前面
driver.navigate().forward();
System.out.printf("forward to %s \n", driver.getCurrentUrl());
Thread.sleep(2000);

driver.quit();
~~~

### 3、刷新页面

> refresh() 刷新页面（F5）

~~~java
……
// 刷新页面
driver.navigate().refresh();
……
~~~

## 三、webDriver常用方法

### 1、常用方法

> - clear() 清除文本。
> - sendKeys(*value) 模拟按键输入。
> - click() 单击元素

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com/");

// 获取目标元素
WebElement search_text = driver.findElement(By.id("kw"));
WebElement search_button = driver.findElement(By.id("su"));

// clear & sendKeys & click
search_text.sendKeys("Java");
search_text.clear();
search_text.sendKeys("Selenium");
search_button.click();

driver.quit();
~~~

### 2、其他方法

> submit()

~~~java
……
WebElement search_text = driver.findElement(By.id("kw"));
search_text.sendKeys("Selenium");
search_text.submit();// 模拟执行提交
……
~~~

> - getSize() 返回元素的尺寸。
> - getText() 获取元素的文本。
> - getAttribute(name) 获得属性值。
> - isDisplayed() 设置该元素是否用户可见。

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com/");

//获得百度输入框的尺寸
WebElement size = driver.findElement(By.id("kw"));
System.out.println(size.getSize());

//返回百度页面底部备案信息
WebElement text = driver.findElement(By.id("cp"));
System.out.println(text.getText());

//返回元素的属性值， 可以是 id、 name、 type 或元素拥有的其它任意属性
WebElement ty = driver.findElement(By.id("kw"));
System.out.println(ty.getAttribute("type"));

//返回元素的结果是否可见， 返回结果为 True 或 False
WebElement display = driver.findElement(By.id("kw"));
System.out.println(display.isDisplayed());

driver.quit();
~~~

## 四、鼠标操作

> - contextClick() 右击
> - clickAndHold() 鼠标点击并控制
> - doubleClick() 双击
> - dragAndDrop() 拖动
> - release() 释放鼠标
> - perform() 执行所有Actions中存储的行为

### 1、悬停显示下拉菜单

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com/");

WebElement search_setting = driver.findElement(By.linkText("设置"));
Actions action = new Actions(driver);
action.clickAndHold(search_setting).perform();

driver.quit();
~~~



### 2、鼠标其他操作

~~~java
......
Actions action = new Actions(driver);
 
// 鼠标右键点击指定的元素
action.contextClick(driver.findElement(By.id("element"))).perform();
 
// 鼠标右键点击指定的元素
action.doubleClick(driver.findElement(By.id("element"))).perform();
 
// 鼠标拖拽动作， 将 source 元素拖放到 target 元素的位置。
WebElement source = driver.findElement(By.name("element"));
WebElement target = driver.findElement(By.name("element"));
action.dragAndDrop(source,target).perform();
 
// 释放鼠标
action.release().perform();
......
~~~

## 五、键盘操作

> sendKeys(Keys.BACK_SPACE) 回格键（BackSpace）
> sendKeys(Keys.SPACE) 空格键(Space)
> sendKeys(Keys.TAB) 制表键(Tab)
> sendKeys(Keys.ESCAPE) 回退键（Esc）
> sendKeys(Keys.ENTER) 回车键（Enter）
> sendKeys(Keys.CONTROL,‘a’) 全选（Ctrl+A）
> sendKeys(Keys.CONTROL,‘c’) 复制（Ctrl+C）
> sendKeys(Keys.CONTROL,‘x’) 剪切（Ctrl+X）
> sendKeys(Keys.CONTROL,‘v’) 粘贴（Ctrl+V）
> sendKeys(Keys.F1) 键盘 F1
> ……

### 1、模拟键盘操作

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com");

WebElement input = driver.findElement(By.id("kw"));

//输入框输入内容
input.sendKeys("seleniumm");
Thread.sleep(2000);

//删除多输入的一个 m
input.sendKeys(Keys.BACK_SPACE);
Thread.sleep(2000);

//输入空格键+“教程”
input.sendKeys(Keys.SPACE);
input.sendKeys("教程");
Thread.sleep(2000);

//ctrl+a 全选输入框内容
input.sendKeys(Keys.CONTROL,"a");
Thread.sleep(2000);

//ctrl+x 剪切输入框内容
input.sendKeys(Keys.CONTROL,"x");
Thread.sleep(2000);

//ctrl+v 粘贴内容到输入框
input.sendKeys(Keys.CONTROL,"v");
Thread.sleep(2000);

//通过回车键盘来代替点击操作
input.sendKeys(Keys.ENTER);
Thread.sleep(2000);

driver.quit();
~~~

## 六、元素等待

> WebDriver提供了两种类型的等待：<span style="color:#42B983;font-weight:bold;">显式等待</span> 和 <span style="color:#42B983;font-weight:bold;">隐式等待</span>。

### 1、显示等待

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com");

//显式等待， 针对某个元素等待
WebDriverWait wait = new WebDriverWait(driver,10,1);

wait.until(new ExpectedCondition<WebElement>(){
    @Override
    public WebElement apply(WebDriver text) {
        return text.findElement(By.id("kw"));
    }
}).sendKeys("selenium");

driver.findElement(By.id("su")).click();
Thread.sleep(2000);

driver.quit();
~~~



### 2、隐式等待

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();

//页面加载超时时间设置为 5s
driver.manage().timeouts().pageLoadTimeout(5, TimeUnit.SECONDS);
driver.get("https://www.baidu.com/");

//定位对象时给 10s 的时间, 如果 10s 内还定位不到则抛出异常
driver.manage().timeouts().implicitlyWait(10, TimeUnit.SECONDS);
driver.findElement(By.id("kw")).sendKeys("selenium");

//异步脚本的超时时间设置成 3s
driver.manage().timeouts().setScriptTimeout(3, TimeUnit.SECONDS);

driver.quit();

~~~

## 七、多表单切换

> 在 Web 应用中经常会遇到 frame/iframe 表单嵌套页面的应用， WebDriver 只能在一个页面上对元素识别与 定位， 对于 frame/iframe 表单内嵌页面上的元素无法直接定位。 这时就需要通过 switchTo().frame()方法将当前定 位的主体切换为 frame/iframe 表单的内嵌页面中。

~~~html
<html>
  <body>
    ...
    <iframe id="x-URS-iframe" ...>
      <html>
         <body>
           ...
           <input name="email" >
~~~

> 126邮箱登录框的结构大概是这样子的，想要操作登录框必须要先切换到iframe表单。

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("http://www.126.com");

WebElement xf = driver.findElement(By.xpath("//*[@id='loginDiv']/iframe"));
driver.switchTo().frame(xf);
driver.findElement(By.name("email")).clear();
driver.findElement(By.name("email")).sendKeys("username");
driver.findElement(By.name("password")).clear();
driver.findElement(By.name("password")).sendKeys("password");
driver.findElement(By.id("dologin")).click();
driver.switchTo().defaultContent();
//……
~~~

<span style="color:red;">注:如果完成了在当前表单上的操作，则可以通过switchTo().defaultContent()方法跳出表单。</span>

## 八、多窗口切换

> - getWindowHandle()： 获得当前窗口句柄。
> - getWindowHandles()： 返回的所有窗口的句柄到当前会话。
> - switchTo().window()： 用于切换到相应的窗口，与上一节的switchTo().frame()类似，前者用于不同窗口的切换， 后者用于不同表单之间的切换。

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com");

//获得当前窗口句柄
String search_handle = driver.getWindowHandle();

//打开百度注册窗口
driver.findElement(By.linkText("登录")).click();
Thread.sleep(3000);
driver.findElement(By.linkText("立即注册")).click();

//获得所有窗口句柄
Set<String> handles = driver.getWindowHandles();

//判断是否为注册窗口， 并操作注册窗口上的元素
for(String handle : handles){
    if (handle.equals(search_handle)==false){
        //切换到注册页面
        driver.switchTo().window(handle);
        System.out.println("now register window!");
        Thread.sleep(2000);
        driver.findElement(By.name("userName")).clear();
        driver.findElement(By.name("userName")).sendKeys("user name");
        driver.findElement(By.name("phone")).clear();
        driver.findElement(By.name("phone")).sendKeys("phone number");
        //......
        Thread.sleep(2000);
        //关闭当前窗口
        driver.close();
    }
}
Thread.sleep(2000);

driver.quit();
~~~

## 九、下拉框选择

> 有时我们会碰到下拉框，WebDriver提供了Select类来处理下接框。

* html 代码

  ~~~java
  <select id="nr" name="NR">
    <option value="10" selected>每页显示 10 条</option>
    <option value="20">每页显示 20 条</option>
    <option value="50">每页显示 50 条</option>
  <select>
  ~~~

* java代码

  ~~~java
  // 设置浏览器驱动程序位置 & 开启浏览器
  System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
  WebDriver driver = new ChromeDriver();
  driver.get("https://www.baidu.com");
  
  driver.findElement(By.linkText("设置")).click();
  driver.findElement(By.linkText("搜索设置")).click();
  Thread.sleep(2000);
  
  //<select>标签的下拉框选择
  WebElement el = driver.findElement(By.xpath("//select"));
  Select sel = new Select(el);
  sel.selectByValue("20");
  Thread.sleep(2000);
  
  driver.quit();
  ~~~

  

## 十、警告框处理

> - getText()： 返回 alert/confirm/prompt 中的文字信息。
> - accept()： 接受现有警告框。
> - dismiss()： 解散现有警告框。
> - sendKeys(keysToSend)： 发送文本至警告框。
> - keysToSend： 将文本发送至警告框。

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com");

driver.findElement(By.linkText("设置")).click();
driver.findElement(By.linkText("搜索设置")).click();
Thread.sleep(2000);

//保存设置
driver.findElement(By.className("prefpanelgo")).click();

//接收弹窗
driver.switchTo().alert().accept();
Thread.sleep(2000);

driver.quit();
~~~





## 十一、Cookies操作

> - getCookies() 获得所有 cookie 信息。
> - getCookieNamed(String name) 返回字典的key为“name”的Cookie信息。
> - addCookie(cookie dict) 添加Cookie。“cookie_dict”指字典对象，必须有 name和value值。
> - deleteCookieNamed(String name) 删除Cookie 信息。 “name”是要删除的 cookie的名称； “optionsString” 是该Cookie的选项，目前支持的选项包括“路径” ， “域” 。
> - deleteAllCookies() 删除所有 cookie 信息

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();
driver.get("https://www.baidu.com");

Cookie c1 = new Cookie("name", "key-aaaaaaa");
Cookie c2 = new Cookie("value", "value-bbbbbb");
driver.manage().addCookie(c1);
driver.manage().addCookie(c2);

//获得 cookie
Set<Cookie> coo = driver.manage().getCookies();
System.out.println(coo);

//删除所有 cookie
//driver.manage().deleteAllCookies();

driver.quit();

~~~



## 十二、执行js

>window.scrollTo()方法用于设置浏览器窗口滚动条的水平和垂直位置。方法的第一个参数表示水平的左间距，第二个参数表示垂直的上边距

~~~java
// 设置浏览器驱动程序位置 & 开启浏览器
System.setProperty("webdriver.chrome.driver", "浏览器驱动路径"); 
WebDriver driver = new ChromeDriver();

//设置浏览器窗口大小
driver.manage().window().setSize(new Dimension(700, 600));
driver.get("https://www.baidu.com");

//进行百度搜索
driver.findElement(By.id("kw")).sendKeys("webdriver api");
driver.findElement(By.id("su")).click();
Thread.sleep(2000);

//将页面滚动条拖到底部
((JavascriptExecutor)driver).executeScript("window.scrollTo(100,450);");
Thread.sleep(3000);

driver.quit();
~~~



## 小进阶

### 1、页面缩放

~~~java
JavascriptExecutor js = (JavascriptExecutor) driver;
js.executeScript("document.body.style.zoom = '30%'");
// 或
((JavascriptExecutor)driver).executeScript("document.body.style.zoom ='70%'");
~~~

### 2、强制杀死浏览器及驱动程序

~~~java
WindowsUtils.killByName("chrome.exe");
WindowsUtils.killByName("firefox.exe");
WindowsUtils.killByName("chromedriver.exe");
WindowsUtils.killByName("geckodriver.exe");
// 注：由于selenium的版本问题 可能会存在方法名不一样情况
~~~

### 3、获取新窗口关闭原窗口

~~~java
String currentHand = driver.getWindowHandle();
for(String hand : driver.getWindowHandles()){
    if(!currentHand.equals(hand)){
        driver.close();
        driver.switchTo().window(hand);
        break;
    }
}
~~~

### 4、selenium获取验证码图片

~~~java
System.setProperty("webdriver.chrome.driver","C:/chromedriver.exe");
WebDriver driver = new ChromeDriver();
driver.manage().window().maximize();
driver.get("https://agent.lionairthai.com/B2BAdmin/Login.aspx?CU=207&culture=en-GB");

WebElement ele = driver.findElement(By.id("ucAgentLogin_rdCapImage_CaptchaImageUP"));
WrapsDriver wrapsDriver = (WrapsDriver) ele;
File screenshot = ((TakesScreenshot)wrapsDriver.getWrappedDriver()).getScreenshotAs(OutputType.FILE);
BufferedImage fullImg=ImageIO.read(screenshot);
int screenshotWidth = fullImg.getWidth();
Dimension dimension = driver.manage().window().getSize();
double scale = (double) dimension.getWidth() / screenshotWidth;
Point point= ele.getLocation();
int eleWidth= ele.getSize().getWidth()+10;
int eleHeight= ele.getSize().getHeight()+10;
BufferedImage eleScreenshot= fullImg.getSubimage((int)(point.getX() / scale),(int)(point.getY() / scale), (int)(eleWidth / scale), (int)(eleHeight / scale));
ImageIO.write(eleScreenshot, "jpg", screenshot);
File screenshotLocation= new File("D:\\tmp\\test.jpg");
FileUtils.copyFile(screenshot, screenshotLocation);
driver.quit();
~~~

