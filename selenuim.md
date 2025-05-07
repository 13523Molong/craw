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
# 通常我们习惯将其简写成EC，即:
# from selenium.webdriver.support import expected_conditions as EC
 
WebDriverWait(browser, 10).until(
    expected_conditions.presence_of_element_located((By.ID, "kw"))
```

这段代码的意思是，等待 10 秒直到页面上出现 ID 为 kw 的元素。**WebDriverWait** 类还有很多其他的用法，譬如 **until_not、visibility_of_element_located、text_to_be_present_in_element** 等

#### 2.3窗口句柄
当我们点击页面的某个元素时，有可能里面嵌套了新的页面，譬如点击一个链接打开了新窗口。这个时候我们如果想操作这个新打开的页面，就需要切换到对应的窗口句柄上。
```python
main_window = browser.current_window_handle   # 保存当前窗口句柄

all_windows = browser.window_handles   # 获取所有窗口句柄

for window in all_windows:   # 遍历所有窗口句柄
    if window != main_window:   # 如果不是当前窗口句柄，则切换到该窗口
        browser.switch_to_window(window)
        break

#点击视频跳转时的等待条件判断

WebDriverWait(browser, 10).until(
    EC.presence_of_element_located(lamba:d:len(d.window_handles)>1)
)
# 即有新窗口出现时完成显示等待
new_window = [window for window in browser.window_handles if window != main_window][0]

browser.switch_to_window(new_window)
```
## 3.iframe处理
### 3.1 iframe定位
有时候一个网页里面会利用iframe来嵌入另外一个页面，譬如b站的视频播放页。这个时候如果要爬取这个页面的内容，就需要先切换到 iframe 里面去切换到iframe的方式有三种

#### 方式一 :通过iframe的name属性进行定位
```python

iframe = browser.find_element_by_name('bilibili_player')
brower.switch_to_frame(iframe)
```
#### 方式二：通过element元素进行切换
```python
ele_iframe = browser.find_element(By.XAPTH, '//iframe[@id="bilibili_player"]')
browser.switch_to_frame(ele_iframe)
```
#### 方式三：通过索引进行切换
```python
browser.switch_to_frame(0)  # 假设这里的索引是从0开始的
browser.switch_to_frame(1)  
```

当然，切换到iframe里面，也会有需求再回到原来的HTML页面中，此时就需要用到 **switch_to_default_content()** 方法。
```python
browser.switch_to_default_content()

#如果是切换到父级的iframe，则可以这样切换
browser/switch_to_parent_frame()
```
## 4.窗口滚动
当我们使用selenium进行页面爬取时，经常会需要滚动页面，以便爬取到更多的内容。譬如，在b站的视频播放页中，如果想获取下面的评论区的内容，就需要将页面向下滚动，必须使得元素可见之后，才能进行下一步的爬取，为此我就需要模拟窗口滚动的技术
而我们实现滚动窗口分为两种方式:

### 方式一：通过js实现滚动
```python
# 滚动到底部
browser.execute_script("window.scrollTo(0, document.body.scrollHeight);")  
time.sleep(1)

# 滚动到顶部
browser.execute_script("window.scrollTo(0, 0);")   

 #滚动到垂直位置500像素的地方
browser.execute_script("window.scrollBy(0, 500);")  

#将窗口滚动到可视区域
#这个方法适用于绝大多数的场景，尤其是在需要精细化定位的地方
element = driver.find_element(By.ID, "element_id")
driver.execute_script("arguments[0].scrollIntoView();", element)  
```
### 方式二：使用 ActionChains 实现滚动

```python
from selenium.webdriver.common.action_chains import ActionChains

actions = ActionChains(browser)
actions.scroll_to_element(element).perform()  
```
这种方式适用于需要模拟用户实际操作的场景如触发某些依赖鼠标悬停或滚动的动态效果。
### 方式三 通过键盘操作实现滚动
例如通过发送 **PASS_DOWN** , **PASS_UP** 事件来实现滚动

```python
from selenium.webdriver.common.keys import Keys

body = driver.find_element(By.TAG_NAME, 'body')
body.send_keys(Keys.PAGE_DOWN)
```
 这个方法十分快速简单，适合用在需要快速滚动的场景，但是不适合用在需要精细定位的场景。
### 方式四 通过定位特殊元素实现滚动
例如个别网页种存在下拉条的特别元素，当然利用元素定位符定位到之后在操作也可以实现滚动效果
```python
scrollable_div = driver.find_element(By.CLASS_NAME, "scrollable")
driver.execute_script("arguments[0].scrollTop = arguments[0].scrollHeight", scrollable_div)
```

## 5.Cookies处理
### 5.1获取cookies
```python
cookies = driver.get_cookies()
print(cookies)
```
### 5.2设置cookies
```python
cookie_dict = {
    'name': 'xxx',
    'value': 'yyy'
}
driver.add_cookie(cookie_dict)
```
### 5.3












