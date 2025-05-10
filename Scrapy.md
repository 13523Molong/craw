# Scrapy 使用教程
前言:
其实爬虫所涉及的技术十分广泛，而当你接到一个系统爬虫的项目时，里面所涉及到很多模块，比如有些负责封装HTTP，有些负责解析网页内容，有些负责数据处理，有些负责调度等等。而Scrapy框架就是将这些模块封装起来，形成一个完整的爬虫系统。因此学习系统的框架十分有必要

而Scrapy爬虫的基本步骤大致可以分为:
1.创建一个Scrapy项目
2.创建一个爬虫脚本进行数据抓取
3.将抓取到数据写入你定义的Item数据模型中
4.利用Item Pipeline存储抓取Item,即数据实体对象

[中文文档](https://docs.scrapy.net.cn/en/latest/index.html)
[官方文档](https://docs.scrapy.org  /en/latest/)
## 目录
- [Scrapy 使用教程](#scrapy-使用教程)
  - [目录](#目录)
  - [Scrapy 简介](#scrapy-简介)
  - [安装与配置](#安装与配置)
    - [基础安装](#基础安装)
  - [项目创建与结构](#项目创建与结构)
    - [创建项目](#创建项目)
    - [创建爬虫文件](#创建爬虫文件)
    - [项目大致结构](#项目大致结构)
  - [Spider 开发详解](#spider-开发详解)
    - [基本结构](#基本结构)
  - [选择器与数据提取](#选择器与数据提取)
  - [item-与数据处理](#item-与数据处理)
    - [item-pipeline-储存](#item-pipeline-储存)
  - [Feed 导出](#feed-导出)
    - [基础配置](#基础配置)
    - [自定义导出器](#自定义导出器)
  - [中间件详解](#中间件详解)
    - [Downloader Middleware](#downloader-middleware)
      - [激活Downloader Middleware](#激活downloader-middleware)
      - [随机User-Agent](#随机user-agent)
- [middlewares.py 示例：随机User-Agent中间件](#middlewarespy-示例随机user-agent中间件)

---

## Scrapy 简介
Scrapy 是一个用 Python 编写的开源网络爬虫框架，用于快速、高效地抓取网站数据。

**核心组件**：
- Scrapy Engine(引擎)：负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯，信号、数据传递等。  
- Scheduler(调度器)：它负责接受引擎发送过来的Request请求，并按照一定的方式进行整理排列，入队，当引擎需要时，交还给引擎。
- Downloader(下载器)：负责下载Scrapy Engine(引擎)发送的所有Requests请求，并将其获取到的Responses交还给Scrapy Engine(引擎)，由引擎交给Spider来处理。
- Spiders(爬虫)：它负责处理所有Responses,从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler(调度器)。
- Item Pipeline：他负责处理收集的到的Item数据模型，并进行后期的处理或存储。

- Middlewares：中间件系统，又分为Spider Middleware 和 Downloader Middleware，分别处理Spider的输入与输出数据以及Downloader的输入与输出数据。


**特点**：
✔️ 异步处理架构  
✔️ 内置CSS/XPath选择器  
✔️ 交互式shell调试  
✔️ 完善的中间件系统  
✔️ 支持多种数据导出格式

---

## 安装与配置

### 基础安装
```bash
pip install scrapy

# 可选依赖
pip install scrapy[ssl]  # HTTPS支持
pip install scrapy[csv]  # CSV导出
pip install scrapy[stats]  # 统计扩展

# 炎症安装
scrapy version

```

配置设置 settings.py
```python
# 常用配置项
USER_AGENT = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'
ROBOTSTXT_OBEY = False  # 是否遵守robots协议
CONCURRENT_REQUESTS = 16  # 并发请求数
DOWNLOAD_DELAY = 0.5  # 下载延迟(秒)
COOKIES_ENABLED = False  # 禁用Cookies可提高性能
DEFAULT_REQUEST_HEADERS = {
    'Accept': 'text/html,application/xhtml+xml',
    'Accept-Language': 'en',
}
```



## 项目创建与结构

### 创建项目 
```bash
scrapy startproject myproject
cd myproject
scrapy genspider example example.com
```
### 创建爬虫文件
```bash
cd 项目名/项目名
# 创建爬虫文件
scrapy genspider -t basic 爬虫名称 域名
```
---

### 项目大致结构
    myproject/
    ├── scrapy.cfg            # 部署配置（环境设置/爬虫部署参数）
    └── myproject/            # 项目模块
        ├── __init__.py       # 包初始化文件
        ├── items.py          # 数据模型定义（定义抓取数据结构）
        ├── middlewares.py    # 中间件（请求/响应预处理）
        ├── pipelines.py      # 数据处理管道（数据清洗/存储）
        ├── settings.py       # 项目设置（全局配置参数）
        └── spiders/          # 爬虫目录（具体爬虫实现）
            ├── __init__.py
            └── example.py    # 示例爬虫
---

## Spider 开发详解
### 基本结构
```python
import scrapy
class MySpider(scrapy.Spider):
    name = 'my_spider'   # 唯一标识符 
    allowed_domains = ['example.com']  # 允许抓取的域名列表，除了允许的域名，其他不做处理
    
    # 需要爬取的URL列表
    start_urls = [
        'http://www.example.com/',
        'http://www.example.com/page2',
    ]
    # 解析URL响应
    def parse(self, response):
        # 处理当前页面
        yield {
            'url': response.url,
            'title': response.css('title::text').get(),
            'content': response.css('meta[name="description"]::attr(content)').get()
        }

        # yield 可以不终止函数，继续进行爬虫操作，同时将数据返回回来
        
        # 跟进分页链接（示例）
        next_page = response.css('a.next-page::attr(href)').get()
        if next_page:
            yield response.follow(next_page, self.parse)

        # 这里返回一个Request对象，Scrapy会自动将该请求加入调度器中，等待下一次调度

```
- `name`：爬虫的唯一标识。
- `allowed_domains`：允许访问的域名列表。
- `start_urls`：起始URLs，用于开始爬取。
- `parse()`方法：解析响应并生成新的请求或提取数据项（Item）。

例如爬取最简单百度首页网页文件，需要注意的是，百度有简单的反爬机制，所以你直接运行是爬不到页面内容的，需要去settings.py文件中设置`ROBOTSTXT_OBEY = False`来忽略robots协议。
```python
import scrapy
import time

class BaiduSpider(scrapy.Spider):
    name = "baidu"  # 爬虫名称 所以我们启动爬虫就需要到指定文件目录下输入scrapy crawl baidu

    allowed_domains = ["www.baidu.com"]
    start_urls = [
        'https://www.baidu.com/',
    ]
    
    
    
    def parse(self, response):
        # 使用时间戳生成唯一文件名
        filename = f'baidu_{int(time.time())}.html'
        
        # 确保使用二进制写入模式
        with open(filename, 'wb') as f:
            f.write(response.body)
        
        self.log(f'Saved file {filename}')
```


## 选择器与数据提取
```python
# 定位单个元素
response.css('h1::text').get()
# 这里的::text是Scrapy特有的伪文本提取，其他解析库类似BS4，lxml并不存在

response.xpath('//title/text()').get()

# 定位多个元素
titles = response.css('h1::text').getall()
titles = response.xpath('//title/text()').getall()

# 获取属性值
hrefs = response.css('a::attr(href)').getall()
hrefs = response.xpath('//a/@href').getall()

# 使用正则表达式提取数据
emails = response.re(r'[\w\.-]+@[\w\.-]+')

# 使用CSS选择器或XPath选择器提取特定内容
content = response.css('.article-body p::text').getall()
content = response.xpath('//div[@class="article-body"]/p/text()').getall()

# 使用嵌套选择器获取更深层次的数据
nested_data = response.css('.item .price::text').getall()
nested_data = response.xpath('//li[contains(@class, "item")]/span[contains(@class, "price")]/text()').getall()
```
---

## item-与数据处理
### item-pipeline-储存
Item Pipeline 负责处理爬取到的数据，可以进行清洗、验证以及存储等操作。一般在pipelines.py文件中进行定义

```python
# pipelines.py 示例
from itemadapter import ItemAdapter
import pymysql

class MultiPipeline:
    # JSON存储
    class JsonPipeline:
        def open_spider(self, spider):
            self.file = open('data.json', 'w', encoding='utf-8')
        
        def close_spider(self, spider):
            self.file.close()
        
        def process_item(self, item, spider):
            line = json.dumps(dict(item)) + "\n"
            self.file.write(line)
            return item

    # MySQL存储
    class MySQLPipeline:
        def __init__(self):
            self.conn = pymysql.connect(
                host='localhost',
                user='root',
                password='123456',
                database='scrapy_db',
                charset='utf8mb4'
            )
            
        def process_item(self, item, spider):
            cursor = self.conn.cursor()
            sql = "INSERT INTO articles (title, content) VALUES (%s, %s)"
            cursor.execute(sql, (item['title'], item['content']))
            self.conn.commit()
            return item

# settings.py启用配置
ITEM_PIPELINES = {
    'myproject.pipelines.MultiPipeline.JsonPipeline': 300,
    'myproject.pipelines.MultiPipeline.MySQLPipeline': 800,
}

# 数量级越小，执行优先度越高
```
## Feed 导出
[官方描述](https://docs.scrapy.org/en/latest/topics/feed-exports.html)
在实现爬虫时，最常需要的功能之一就是能够正确地存储抓取到的数据，
并且通常这意味着生成一个包含抓取数据的“导出文件”（通常称为“导出 Feed”）以便其他系统使用。

### 基础配置
```python
# settings.py
FEED_FORMATS = ['json', 'csv']  # 支持的导出格式
FEED_URI = 'output/%(name)s_%(time)s.%(format)s'
# 导出文件的路径和文件名格式，其中%(name)s表示爬虫名称，%(time)s表示当前时间，%(format)s表示导出格式
FEED_EXPORT_ENCODING = 'utf-8' # 导出文件编码
FEED_EXPORT_BATCH_ITEM_COUNT = 1000  # 分批导出
```

### 自定义导出器
```python
# pipelines.py 示例：阿里云OSS存储
import oss2
class AliyunOSSPipeline:
    def __init__(self):
        auth = oss2.Auth('<ACCESS_KEY>', '<ACCESS_SECRET>')
        self.bucket = oss2.Bucket(auth, 'http://oss-cn-hangzhou.aliyuncs.com', 'my-bucket')

    def process_item(self, item, spider):
        filename = f'data/{spider.name}/{time.time()}.json'
        self.bucket.put_object(filename, json.dumps(dict(item)))
        return item

```

## 中间件详解

### Downloader Middleware
下载器中间件是 Scrapy 请求/响应处理中的一组钩子框架。
它是一个轻量级、低级别的系统，用于全局修改 Scrapy 的请求和响应

#### 激活Downloader Middleware
要激活下载器中间件组件，请将其添加到 DOWNLOADER_MIDDLEWARES 设置中，该设置是一个字典，其键是中间件类路径，其值是中间件顺序
```python
DOWNLOADER_MIDDLEWARES = {
    "myproject.middlewares.CustomDownloaderMiddleware": 543,
}
```
#### 随机User-Agent
```python
# middlewares.py 示例：随机User-Agent中间件
from fake_useragent import UserAgent

class RandomUserAgentMiddleware:
    def process_request(self, request, spider):
        request.headers['User-Agent'] = UserAgent().random




