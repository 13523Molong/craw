# selenium 自动化爬虫
## 引言
Selenium 是一款强大的基于浏览器的开源自动化测试工具，最初由 Jason Huggins 于 2004 年在 ThoughtWorks 发起，它提供了一套简单易用的 API，模拟浏览器的各种操作，方便各种 Web 应用的自动化测试。它的取名很有意思，因为当时最流行的一款自动化测试工具叫做 QTP，是由 Mercury 公司开发的商业应用。Mercury 是化学元素汞，而 Selenium 是化学元素硒，汞有剧毒，而硒可以解汞毒，它对汞有拮抗作用。

Selenium 的核心组件叫做 Selenium-RC（Remote Control），简单来说它是一个代理服务器，浏览器启动时通过将它设置为代理，它可以修改请求响应报文并向其中注入 Javascript，通过注入的 JS 可以模拟浏览器操作，从而实现自动化测试。但是注入 JS 的方法存在很多限制，譬如无法模拟键盘和鼠标事件，处理不了对话框，不能绕过 JavaScript 沙箱等等。就在这个时候，于 2006 年左右，Google 的工程师 Simon Stewart 发起了 WebDriver 项目，WebDriver 通过调用浏览器提供的原生自动化 API 来驱动浏览器，解决了 Selenium 的很多疑难杂症。不过 WebDriver 也有它不足的地方，它不能支持所有的浏览器，需要针对不同的浏览器来开发不同的 WebDriver，因为不同的浏览器提供的 API 也不尽相同，好在经过不断的发展，各种主流浏览器都已经有相应的 WebDriver 了。最终 Selenium 和 WebDriver 合并在一起，这就是 Selenium 2.0，有的地方也直接把它称作 WebDriver。Selenium 目前最新的版本已经是 3.9 了，WebDriver 仍然是 Selenium 的核心。

## 1.准备工作
### 1.1下载pyton
    教程千千万，总有你喜欢的

### 1.2安装Selenium
```basn
pip install selenium

```
### 1.3下载浏览器驱动
    selenim 支持诸多浏览器，各有好处，建议用chrome，先去设置里面找到chrome对应的版本浩，然后去搜索chrome driver 对应版本下载，例如我是'136.0.7.013.xx'

## 2.爬虫入门
### 2.1输入输出
#### 2.11输入
我们打开b站进行搜索，如果是人工操作，一般有两种方式：第一种，在输入框中输入搜索文字，然后回车；第二种，在输入框中输入搜索文字，然后点击搜索按钮。Selenium 和人工操作完全一样，可以模拟这两种方式：
##### 方式一 send keys with return
```python
from selenium import webdriver
import time

# 打开浏览器，并访问b站首页
browser = webdriver.Chrome() //这里第一次运行时需要导入你自己浏览器驱动路径
#这里补充一句 在selenuim4.1.2版本之后 chromedriver 驱动的execute_path参数已经启用；1，推荐使用service进行驱动
browser.get('https://www.bilibili.com')
time.sleep(1)

# 在搜索框中输入关键字，然后回车
search_input = browser.find_element_by_id('keyword')
search_input.send_keys('selenium')
search_input.submit()   # 等价于按下回车键
time.sleep(1)

# 关闭浏览器
browser.close()


#最新版本驱动示例

from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from webdriver_manager.chrome import ChromeDriverManager

# 设置 Chrome 选项
options = Options()
options.binary_location = r'D:\chrome_driver\chrome-win64\chrome-win64\chrome.exe'

# 初始化 WebDriver，自动管理 ChromeDriver
browser = webdriver.Chrome(service=Service(ChromeDriverManager().install()), options=options)
browser.get('https://bilibili.com')

# 等待元素加载
WebDriverWait(browser, 10).until(
    EC.presence_of_element_located((By.CLASS_NAME, 'nav-search-input'))
)

print("元素加载成功！")



```
其中 **find_element_by_id** 方法经常用到，它根据元素的 ID 来查找页面某个元素。类似的方法还有 **find_element_by_name、find_element_by_class_name、find_element_by_css_selector、find_element_by_xpath** 等，都是用于定位页面元素的。另外，也可以同时定位多个元素，例如 **find_elements_by_name、find_elements_by_class_name** 等，就是把 **find_element** 换成 **find_elements，**具体的 API 可以参考 Selenium 中文翻译文档中的 **查找元素** 一节。

##### 方式二 send keys with click
```python
from selenium import webdriver
import time

# 打开浏览器，并访问b站首页
browser = webdriver.Chrome()
browser.get('https://www.bilibili.com')
time.sleep(1)

# 在搜索框中输入关键字，然后点击搜索按钮
search_input = browser.find_element_by_id('keyword')
search_input.send_keys('selenium')
button = browser.find_element_by_id('search_button')
button.click()
time.sleep(1)

```

##### 方式三  execute javascript
```python
from selenium import webdriver
import time

# 打开浏览器，并访问b站首页
browser = webdriver.Chrome()
browser.get('https://www.bilibili.com')
time.sleep(1)

# 在搜索框中输入关键字，然后点击搜索按钮
search_input = browser.find_element_by_id('keyword')
button = browser.find_element_by_id('search_button')
browser.execute_script("arguments[0].click();", button)   # 等价于点击了搜索按钮
time.sleep(1)

```
这个方式完全是同时javascript 方式，不需要找到按钮元素。非常的灵活

#### 2.12 输出
有输入就有输出，当点击搜索按钮之后，如果我们要爬取页面上的搜索结果，我们有几种不同的方法。
##### 方式一  parse page_source
```python
html = browser.page_source
res = parse_html(html)
```
跟传统爬虫十分类似，缺陷就是无法解析Ajax动态加载的内容

##### 方式二 find & parse elements
```python
results = browser.find_elements_by_css_selector("#content_left .c-container")
for result in results:
    link = result.find_element_by_xpath(".//h3/a")
    print(link.text)
```
这种方式需要充分利用上面介绍的 查找元素 技巧，譬如这里如果要解析百度的搜索页面，我们可以根据 #content_left .c-container 这个 CSS 选择器定位出每一条搜索结果的元素节点。然后在每个元素下，通过 XPath .//h3/a 来取到搜索结果的标题的文本。XPath 在定位一些没有特殊标志的元素时特别有用。

##### 方式三  intercept & parse ajax

#### 2.2 等待
上面也提到过，如果页面上有 Ajax 请求，使用 **browser.page_source** 得到的是页面最原始的源码，无法爬到百度搜索的结果。事实上，不仅如此，如果你试过上面** 方式二 find & parse elements** 的例子，你会发现用这个方式程序也爬不到搜索结果。这是因为 **browser.get()** 方法并不会等待页面完全加载完毕，而是等到浏览器的 **onload** 方法执行完就返回了，这个时候页面上的 Ajax 可能还没加载完。如果你想确保页面完全加载完毕，当然可以用 **time.sleep()** 来强制程序等待一段时间再处理页面元素，但是这种方法显然不够优雅。或者自己写一个 while 循环定时检测某个元素是否已加载完，这个做法也没什么问题，但是我们最推荐的还是使用 **Selenium** 提供的 **WebDriverWait** 类。

**WebDriverWait** 类经常和 **expected_conditions** 搭配使用，注意 **expected_conditions** 并不是一个类，而是一个文件，它下面有很多类，都是小写字母，看起来可能有点奇怪，但是这些类代表了各种各样的等待条件。譬如下面这个例子：
```pythonw
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions
 
WebDriverWait(browser, 10).until(
    expected_conditions.presence_of_element_located((By.ID, "kw"))
```
